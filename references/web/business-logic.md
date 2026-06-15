# Business logic flaws (A06:2025 Insecure Design) & race conditions

The bugs scanners can't find. The code works exactly as written — the *design* lets an attacker abuse
the intended workflow to get money, access, or state they shouldn't. Finding these requires
understanding what the app is *for*, then asking "what if I do this out of order / negative / twice /
too fast?" CWE-840, CWE-841, CWE-362 (race), CWE-770 (no limits). Also covers A10:2025 (Mishandling of
Exceptional Conditions).

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — Client-sent values in sensitive logic**
```bash
rg -n "req\.body\.(price|total|amount|discount|quantity|balance|role|isAdmin|userId)" .
```
- **SKIP if:** the value is recomputed/validated server-side from DB records
- **FINDING if not skipped:** Type: Price/Value Tampering | Severity: High | Fix: Recompute all financial/privilege values server-side from trusted DB records

**STEP 2 — Check-then-act without transaction**
```bash
rg -n "if.*balance|if.*stock|if.*quantity|if.*count|if.*available" .
```
- **SKIP if:** the check and modification are inside the same DB transaction with row lock (`FOR UPDATE`)
- **FINDING if not skipped:** Type: Race Condition | Severity: Medium | Fix: Use DB transaction with row lock (SELECT ... FOR UPDATE) or atomic update

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## How to hunt (mindset, not a payload list)

1. **Model the intended flow** for each valuable feature (checkout, signup, transfer, redeem, vote,
   refund, subscription, invite, KYC). List every step, precondition, and assumption.
2. **Attack each assumption.** Logic bugs live where the developer assumed users behave.

Probes that surface logic flaws:

- **Skip / reorder steps:** jump straight to step 3 (confirm-payment) without step 2 (pay). Reach a
  "success" endpoint directly. Complete a multi-stage flow out of order.
- **Tamper with values the client computes:** price, quantity, currency, discount, tax, totals, item
  IDs, `userId`, `accountFrom`. Negative quantities (refund yourself), `quantity=0`, huge numbers
  (integer overflow / free), price = `0.00` or `0.01`, changing currency to a cheaper one.
- **Replay & reuse:** reuse a one-time coupon/gift card/referral, replay a payment confirmation,
  redeem the same token twice, re-trigger a "first purchase" bonus.
- **Boundary & type abuse:** negative, zero, float where int expected, leading zeros, scientific
  notation, very long strings, unicode look-alikes, array where scalar expected.
- **Bypass limits & gates:** trial limits, rate caps, "one per customer", KYC/age gates, approval
  workflows — can you reach the gated state without passing the gate?
- **Abuse refunds/credits/loops:** any flow where value moves — try to make it move the wrong way or
  more than once.

## Race conditions (TOCTOU) — the high-value subclass

When the same resource is read-checked-then-acted-on without atomicity, firing **concurrent** requests
can make the check pass multiple times:

- **Find:** any "check then act" on a limited resource — redeem-once coupon, withdraw/transfer
  (balance check), "claim" a unique item, increment a counter, apply-credit, vote-once,
  approve-once.
- **Test:** send N identical requests *simultaneously* (single-packet attack / Turbo Intruder /
  parallel `curl`). If you redeem a one-time coupon 5×, withdraw more than your balance, or claim a
  unique item twice → race condition.
- **Confirm:** show the post-condition that should be impossible (balance went negative; coupon used
  twice; two users own the unique item).
- **Fix:** make the operation **atomic** — DB transaction with the right isolation level + row locking
  (`SELECT ... FOR UPDATE`), atomic conditional update (`UPDATE ... WHERE balance >= amount`), unique
  constraints, idempotency keys, or distributed locks. Don't rely on read-then-write in app code.

## Static signs

```bash
# value trust + non-atomic check-then-act
rg -n "req\.body\.(price|amount|total|quantity|cost|discount)" .       # client-computed money
rg -n "if .*balance|if .*credit|coupon|redeem|voucher|referral" -i .   # gated value flows
```
Red flags: prices/totals taken from the request instead of recomputed server-side; coupon "used" flag
set *after* granting the benefit; no unique constraint / no transaction around a limited resource;
"isPaid" trusted from the client.

## How to fix (patterns)

- **Recompute on the server.** Never trust client-sent prices, totals, discounts, entitlements, or
  identity. The client sends *intent* (item IDs, quantities), the server computes *value*.
- **Enforce state machines.** Each step verifies the prior step actually happened (server-side state),
  not a client flag.
- **Atomicity + idempotency** for anything involving limited resources or money (see race fix above).
- **Server-side limits** for "one per customer", trials, and rate caps — keyed to a robust identity,
  not a cookie.
- **Handle exceptional conditions** explicitly (A10:2025): fail closed, validate boundaries, don't let
  errors/timeouts leave value half-transferred or grant default access.

## DO NOT report as a business logic flaw if…

- The price/total is **recomputed server-side** and the client value is used only for display/UX
- The "race condition" is inside an **atomic DB operation** (transaction + row lock + unique constraint)
- The "skipped step" is actually **server-enforced** (the endpoint checks prior state before proceeding)
- The "limit bypass" is a **UI-only limit** with server-side enforcement you missed (check middleware)
- You **cannot articulate an unauthorized outcome** the attacker achieves — no impact = no bug

---

## Vulnerable ↔ fixed code examples

### Price tampering (the most common business logic bug)

```python
# ❌ VULNERABLE — trusts client-sent price
@app.route('/checkout', methods=['POST'])
def checkout():
    item_id = request.json['item_id']
    price = request.json['price']       # attacker sends price=0.01
    quantity = request.json['quantity']
    total = price * quantity
    charge_card(user, total)            # charges $0.01 instead of $99.99
    fulfill_order(user, item_id, quantity)

# ✅ FIXED — server recomputes price from trusted source
@app.route('/checkout', methods=['POST'])
def checkout():
    item_id = request.json['item_id']
    quantity = request.json['quantity']
    item = Item.query.get_or_404(item_id)
    total = item.price * quantity       # price from DB, not from request
    charge_card(user, total)
    fulfill_order(user, item_id, quantity)
```

### Race condition — coupon double-spend

```javascript
// ❌ VULNERABLE — check-then-act without atomicity
app.post('/redeem', async (req, res) => {
  const coupon = await Coupon.findById(req.body.couponId);
  if (coupon.used) return res.status(400).send('Already used');
  // RACE WINDOW: 20 parallel requests all pass the check above
  await applyDiscount(req.user, coupon.value);
  coupon.used = true;
  await coupon.save();   // too late — discount applied N times
});

// ✅ FIXED — atomic conditional update (one winner, others no-op)
app.post('/redeem', async (req, res) => {
  const result = await Coupon.updateOne(
    { _id: req.body.couponId, used: false },   // condition IS the lock
    { $set: { used: true, usedBy: req.user.id, usedAt: new Date() } }
  );
  if (result.modifiedCount === 0) return res.status(400).send('Already used');
  const coupon = await Coupon.findById(req.body.couponId);
  await applyDiscount(req.user, coupon.value);
});
```

### Step-skipping (workflow bypass)

```python
# ❌ VULNERABLE — no server-side state machine; client can jump to confirmation
@app.route('/order/confirm', methods=['POST'])
def confirm_order():
    order = Order.query.get(request.json['order_id'])
    order.status = 'confirmed'   # no check that payment actually succeeded
    db.session.commit()
    ship_order(order)

# ✅ FIXED — enforce state transition server-side
@app.route('/order/confirm', methods=['POST'])
def confirm_order():
    order = Order.query.get(request.json['order_id'])
    if order.status != 'paid':                         # requires prior state
        return abort(400, 'Order must be paid first')
    if order.user_id != current_user.id:               # ownership check
        return abort(403)
    order.status = 'confirmed'
    db.session.commit()
    ship_order(order)
```

### Negative quantity (refund abuse)

```javascript
// ❌ VULNERABLE — negative quantity creates a credit
app.post('/cart/add', (req, res) => {
  const { itemId, quantity } = req.body;  // quantity = -5 → refunds $50
  const item = catalog.get(itemId);
  cart.total += item.price * quantity;     // negative × positive = negative total
});

// ✅ FIXED — validate range server-side
app.post('/cart/add', (req, res) => {
  const { itemId, quantity } = req.body;
  if (!Number.isInteger(quantity) || quantity < 1 || quantity > 100) {
    return res.status(400).send('Invalid quantity');
  }
  const item = catalog.get(itemId);
  cart.total += item.price * quantity;
});
```

---

## Advanced: single-packet race conditions

Modern race exploitation isn't about raw speed — it's **simultaneity**. The **single-packet attack**
(Kettle, 2023) sends 20–30 requests in one TCP packet (HTTP/2 multiplexing, or HTTP/1.1 last-byte
synchronization) so they reach the server within ~1 ms, erasing network jitter. Use **Turbo Intruder**
(`engine=Engine.HTTP2`, `gate`/`openRequests`) or Burp Repeater's **"Send group in parallel."**

- **Limit-overrun:** redeem a one-time coupon/gift-card, withdraw balance, accept an invite, or use a
  vote/like N times before the check commits.
- **Time-of-check/time-of-use & multi-endpoint races:** confirm-email→change-email, disable-2FA races,
  "apply discount" ∥ "checkout", register-then-verify. Any **check-then-act on a shared resource without
  a DB lock / atomic update** is a candidate.
- Reference: [Smashing the state machine](https://portswigger.net/research/smashing-the-state-machine).

## CTF angle

Logic challenges: buy with a negative quantity to gain credit, integer-overflow a price to 0, race a
one-time purchase to buy the flag item twice with one balance, or skip a verification step to reach an
admin/"win" endpoint. Read the app's economy, then break its arithmetic or its ordering.

## Real-world cases

Disclosed HackerOne reports with PoCs:
[business-logic errors](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPBUSINESSLOGIC.md) ·
[race conditions](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPRACECONDITION.md)
(the bugs scanners miss — read these for the creativity).

## References

[OWASP A06:2025](https://owasp.org/Top10/2025/) (Insecure Design) · A10:2025 · WSTG-BUSL ·
CWE-[840](https://cwe.mitre.org/data/definitions/840.html)/[841](https://cwe.mitre.org/data/definitions/841.html)/[362](https://cwe.mitre.org/data/definitions/362.html)/[770](https://cwe.mitre.org/data/definitions/770.html) ·
[PortSwigger: Logic flaws](https://portswigger.net/web-security/logic-flaws) / [Race conditions](https://portswigger.net/web-security/race-conditions).
Full bibliography: [research/web.md](../../research/web.md). Back to [web/](README.md).
