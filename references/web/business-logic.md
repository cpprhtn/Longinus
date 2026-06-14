# Business logic flaws (A06:2025 Insecure Design) & race conditions

The bugs scanners can't find. The code works exactly as written — the *design* lets an attacker abuse
the intended workflow to get money, access, or state they shouldn't. Finding these requires
understanding what the app is *for*, then asking "what if I do this out of order / negative / twice /
too fast?" CWE-840, CWE-841, CWE-362 (race), CWE-770 (no limits). Also covers A10:2025 (Mishandling of
Exceptional Conditions).

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
