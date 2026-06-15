# 🗡️ Longinus

> *스스로의 방어를 가장 먼저 시험하는 창.*
> 실제 공격자보다 먼저, 방어하는 쪽이 자신의 취약점을 발견하도록 돕는 공격 보안 스킬.

[English](README.md) · **한국어**

**Longinus는 Claude Code를 보안 감사관으로 바꿔 주는 [스킬](https://docs.claude.com/en/docs/claude-code/skills)
입니다.** 공격자의 관점에서 내 프로젝트의 취약점을 찾아내고, 우선순위를 매긴 결과를 재현 가능한 PoC와 구체적인
수정 방안과 함께 제시합니다. 트리 구조로 정리된 공격 보안 플레이북과, 그것을 타깃에 맞게 운용하는 오케스트레이션
로직([SKILL.md](SKILL.md))으로 구성됩니다.

## 왜 필요한가

- **AI가 생성한 코드는 빠르게, 그러나 취약한 채로 출시됩니다.** 출시 전에 스스로 점검할 체계적인 방법이 필요합니다.
- **보안 지식은 흩어져 있습니다.** OWASP, 버그바운티 기법, CTF 노하우를 매번 찾아 헤매는 대신, 검증된 방법론을
  한 흐름으로 따라갑니다.
- **스캐너는 노이즈를 쏟아냅니다.** Longinus는 *증명하지 못하면 보고하지 않는다*는 원칙으로 신뢰할 수 있는 결과만
  남깁니다.

## 누구를 위한 것인가

- 출시를 앞두고 (대개 AI로 작성한) 코드를 직접 점검하려는 **개발자·창업자**
- 인가된 범위에서 반복 가능한 방법론이 필요한 **버그바운티 헌터·펜테스터**

## 무엇이 다른가

단순한 플레이북 모음이 아닙니다.

- **세 가지 감사 모드 (quick / standard / deep).** Quick 모드는 grep 기반의 기계적 스캔으로 CI 게이트와
  소형 온디바이스 모델(11B+)에 적합합니다. Standard는 전체 단일 타깃 감사, Deep은 다중 도메인 교차 분석입니다.
- **체크리스트가 아니라 원리에서 출발합니다.** 정해진 목록을 훑는 대신, 취약점이 발생하는 여섯 가지 근본
  원리(신뢰 경계, 파서 불일치, confused deputy, 상태·시간, 인코딩, 개발자의 암묵적 가정)에서 취약점을 도출합니다
  → [패턴 트리거 & 공격자 원리](references/pattern-triggers.md). 아직 분류되지 않은 새로운 유형까지 찾아냅니다.
- **개별 취약점이 아니라 연계 영향으로 평가합니다.** 단독으로는 낮은 등급이라도, 연결되면 계정 탈취로 이어지는
  취약점들이 있습니다. 그러한 연계는 하나의 Critical로 다룹니다 → [체이닝](references/chaining-and-impact.md).
- **오탐을 철저히 배제합니다.** 재현 가능한 PoC가 없으면 확정 보고하지 않습니다. 필수 심각도 게이트가
  Critical/High 판정을 기계적으로 하향 조정하여, LLM의 고질적인 심각도 인플레이션을 억제합니다
  → [심각도 평가](references/severity-and-triage.md).
- **의존성 건강 점검이 통합되어 있습니다.** SCA 도구가 감사 흐름의 일부로 실행되며, 도달 가능성이 확인되지 않은
  CVE는 Info로만 표기하고 High/Critical로 부풀리지 않습니다.
- **코드 패턴을 기계적으로 매칭합니다.** [패턴 트리거 테이블](references/pattern-triggers.md)이 코드 패턴을
  취약점 점검으로 직접 연결해 주어, 원리에서 합성하는 과정 없이도 바로 확인이 가능합니다.
- **한계에 대해 솔직합니다.** [한계 문서](references/limitations.md)가 정적/LLM 분석으로 *찾을 수 없는 것*을
  명시하여, 클린 리포트가 "취약점이 없다"는 뜻이 아님을 분명히 합니다.

## 감사 모드

| 모드 | 용도 | 동작 |
|---|---|---|
| **quick** | 빠른 1차 스캔, CI 게이트, **소형 온디바이스 모델 (11B+)** | 각 leaf의 `## Mechanical scan` 섹션에서 grep 기반 기계적 스캔 실행. 고정 심각도, CVSS 미산출, 체이닝 분석 없음. 결과는 테이블 하나. |
| **standard** | 기본값. 단일 타깃 전체 감사. | 프로파일 → 라우팅 → leaf 전체 분석 → PoC → 심각도 평가 (CVSS 4.0) → 리포트. |
| **deep** | 정식 다중 도메인 정밀 감사. | Standard + 모든 보조 도메인 quick-check + 도메인 간 체이닝 ([chaining-and-impact.md](references/chaining-and-impact.md)). |

**Quick 모드는 소형 온디바이스 모델에 최적화되어 있습니다** (예: Qwen 3 8B, Llama 3.1 8B). 이들 모델은
기계적 지시를 따를 수 있지만 자유 추론은 어렵습니다. Quick 모드는 전적으로 패턴 매칭으로만 동작하여 원리
합성, CVSS 계산, 체이닝 분석이 필요 없습니다. 로컬 모델로 보안 점검을 시작한다면 여기서 출발하세요.

호출 시 모드를 지정합니다: *"quick 보안 감사 실행해 줘"*, *"이 저장소 deep 감사해 줘"*, 또는 보안 질문을 하면
**standard**가 기본입니다.

## 의존성 CVE 점검

Longinus는 자체 CVE 데이터베이스를 갖고 있지 않습니다. 각 생태계의 SCA 도구에 위임하고, 그 결과에 자체
심각도 규칙을 적용합니다:

```
Step 1   스택 식별 (package.json → Node, requirements.txt → Python, …)
Step 1.5 해당 SCA 도구 실행:
           Node → npm audit    Python → pip-audit / osv-scanner
           Go   → govulncheck  Rust   → cargo audit
           Ruby → bundler-audit  Java → trivy fs .
                ↓
         SCA 도구가 자체 CVE DB 조회
         (GitHub Advisory DB, OSV, NVD 등)
                ↓
         Longinus가 결과를 필터링:
           도달 가능 경로 확인됨     → 정상 심각도 평가
           도달 가능성 미확인        → Info / "patch anyway"로만 표기
           환각 CVE (검증 불가)      → 보고 금지
                ↓
         리포트의 "Dependency Health" 섹션에 별도 기재
```

SCA 도구는 사용자 환경에 설치되어 있어야 합니다. `npm audit`은 Node 프로젝트에 기본 포함되지만,
`pip-audit`, `trivy`, `govulncheck` 등은 별도 설치가 필요합니다. 도구가 없으면 해당 점검은
건너뛰게 됩니다 — 이는 스킬이 아닌 환경의 제약입니다.

## 설치

Longinus는 Claude Code 스킬이며, `skills/` 디렉터리에 두면 Claude Code가 자동으로 인식합니다.

```bash
# 프로젝트 단위 — repo에 함께 커밋하여 팀과 공유
git clone https://github.com/cpprhtn/Longinus.git .claude/skills/longinus

# 개인 단위 — 내 모든 프로젝트에서 사용
git clone https://github.com/cpprhtn/Longinus.git ~/.claude/skills/longinus
```

`SKILL.md`가 폴더 최상위에 있고 `references/`, `research/`가 그 옆에 있어야 하므로 **전체 트리를 그대로
클론**하세요. 내부 링크가 상대경로라, 일부 파일만 복사하면 동작하지 않습니다.

## 사용

설치 후에는 보안 관련 요청을 하거나 `/longinus`를 호출하면 됩니다.

> "출시 전에 이 저장소를 점검해 줘." · "이 FastAPI API의 보안을 검토해 줘." · "이 React Native 앱도 확인해 줘."

**스택을 가리지 않습니다.** Python/FastAPI, Node/Express, React Native, Go, Terraform/클라우드, LLM/RAG 앱
등 — 타깃을 먼저 분석한 뒤 알맞은 플레이북으로 안내합니다. 다만 독립적으로 실행되는 자동 스캐너가 아니라, Claude가
코드를 직접 검토하도록 이끄는 도구이며 기본 동작은 read-only입니다. 본인 소유가 아닌 대상을 테스트하기 전에는
반드시 [인가 게이트](references/authorization-and-scope.md)를 확인하세요.

> ⛔ **방어를 위한 공격입니다.** 내가 소유한 코드, 명시적으로 인가받은 타깃, CTF·실습 환경에서만 사용하세요.
> 허가받지 않은 시스템에는 절대 사용하면 안 됩니다. [docs/ethics.md](docs/ethics.md)

## 문서

처음부터 끝까지 읽도록 만든 문서가 아닙니다. 필요한 항목만 따라 들어가세요.

| 목적 | 위치 |
|---|---|
| **바로 사용하기** | [docs/usage.md](docs/usage.md) → [SKILL.md](SKILL.md) |
| **왜 만들었는지** | [docs/why.md](docs/why.md) |
| **대상과 결과물** | [docs/who-its-for.md](docs/who-its-for.md) |
| **윤리·인가 규칙** | [docs/ethics.md](docs/ethics.md) ⛔ 능동 테스트 전 필독 |
| **플레이북 둘러보기** | [references/00-map.md](references/00-map.md) (증상→파일 점프 테이블) |
| **프레임워크·도구 링크** | [RESEARCH.md](RESEARCH.md) (참고문헌 허브) |

## 저장소 구조

```
Longinus/
├── README.md                       ← 영문 메인 (랜딩 + 문서 지도)
├── README.ko.md                    ← 한국어 안내 (이 문서)
├── SKILL.md                        ← 오케스트레이션 브레인 (실제 진입점)
├── RESEARCH.md                     ← 참고문헌 허브 (research/ 트리 색인)
├── docs/                           ← 개념 설명 (문서별로 분리)
│   ├── why.md                      ← 왜 만들었나 (3가지 흐름)
│   ├── who-its-for.md              ← 대상 + 결과물 (스캐너 덤프가 아닌 리포트)
│   ├── ethics.md                   ← 윤리·인가 요약
│   └── usage.md                    ← 사용법 / 지식 베이스로 읽기
├── research/                       ← 도메인별 참고문헌 (프레임워크 + 도구 링크)
│   ├── rationale.md  recon.md  web.md  api.md  identity.md
│   ├── secrets-supply-chain.md  ai-llm.md  cloud-infra.md  mobile.md
│   └── ctf.md  process.md  meta-resources.md
└── references/                     ← 실제 플레이북 트리
    ├── 00-map.md                   ← 전체 트리 지도 + 증상→파일 점프 테이블
    ├── authorization-and-scope.md                            ← ⛔ 인가 게이트
    ├── pattern-triggers.md                                   ← 🎯 공격자 원리 + 코드 패턴 → 취약점 매핑
    ├── limitations.md                                        ← ⚠️ 이 스킬이 찾을 수 없는 것 (정직한 한계)
    ├── methodology.md  chaining-and-impact.md                ← 진행 절차 + 🔗 발견 연계
    ├── severity-and-triage.md  reporting-and-disclosure.md   ← 거버넌스 spine
    ├── recon/  web/  api/  identity/  secrets-and-supply-chain/
    ├── ai-llm/  cloud-and-infra/  mobile/
    └── binary-exploitation/  reverse-engineering/  cryptography/  forensics/  tooling/
```

트리는 전문 공격이 조직되는 방식을 그대로 따릅니다. 공통으로 적용되는 **방법론·거버넌스** spine이 있고, 타깃에
따라 선택해 내려가는 **전문 분야 브랜치**가 이어집니다. 각 플레이북 리프는 정전(canonical) 프레임워크·도구 링크를
모은 `research/<도메인>.md`로 연결됩니다.

## 상태

도메인 리프와 `research/` 참고문헌은 지속적으로 발전시키는 살아있는 문서이며, 기법의 변화에 맞춰 함께 갱신합니다
(정책: [research/meta-resources.md](research/meta-resources.md)).

### v0.2.2

- **체인 카탈로그** — `chaining-and-impact.md`를 허브로 승격, `references/chaining/` 아래 단계별 실전
  공격 체인 playbook(account-takeover, cloud-takeover, rce-chains, data-exfiltration, ai-agent-chains)을
  색인. 각 playbook은 **공격 주도(offense-driven)** 이며 🛡️ *defensive seal*(체인을 끊는 단 하나의 게이트)로 끝남.
- **양방향 방법론** — 신규 운영원리 *"두 렌즈, 하나의 결함"*: 공격 렌즈(어디서 터지나)와 방어 렌즈(어떤
  통제가 있어야 하나)로 동시에 사냥, **공격이 driver·방어가 seal**. `enforce-forward.md`를 이 방법론의 방어 렌즈로 재배치.

### v0.2.1

- **Enforce-forward** — 발견이 단순 패치가 아니라 *구조적 통제 + CI/lint 게이트*로 끝나도록. 신규
  `references/enforce-forward.md`에 카테고리별 **표준 → 통제 → 게이트** 매트릭스와 실행 가능한
  `references/templates/`(validation-layer, ci-gates, policy-as-code, pre-commit-and-secrets) 추가;
  *Fix-forward* 원리를 *Enforce-forward*로 격상. 각 게이트는 *그것이 막는 공격*으로 프레이밍.
- **C/C++·임베디드 소스용 시큐어 코딩 표준** — 신규 `references/secure-coding-standards.md`
  (보안=CERT-C/CWE · 안전=MISRA/ISO 26262), binary-exploitation/reverse-engineering의 소스 컴플라이언스 짝.
- 도메인 README들에 카테고리별 Enforce-forward 포인터 추가; identity에 NIST 800-63/FAPI 보강.
- SKILL.md를 `audit-modes.md` 분리로 lean dispatcher화; quick-check를 `## Mechanical scan`으로 통일;
  `severity-and-triage.md`를 오탐 규율의 정전(canonical) 출처로 지정.

### v0.2.0

- 세 가지 감사 모드: quick (grep 기반 기계적 스캔, 소형 온디바이스 모델 11B+) / standard / deep
- 필수 심각도 게이트 — Critical/High 판정이 기계적 체크리스트를 통과하지 못하면 강제 하향, LLM 심각도
  인플레이션 억제
- 통합 의존성 건강 점검(SCA)을 감사 흐름의 Step 1.5로 추가; 도달 가능성 미확인 CVE는 Info로만 표기
- 세션 오염 경고 및 감사 간 `/clear` 권장 사항 추가
- 전 22개 도메인 leaf에 `## Mechanical scan` 섹션 추가 (소형 모델 호환)
- `attacker-mindset.md`를 `pattern-triggers.md`로 통합 (토큰 최적화)
- 도메인별 quick-check를 SKILL.md에서 각 도메인 README로 이동
- SKILL.md와 00-map.md 간 라우팅 중복 제거

### v0.1.0

- 초기 버전: 도메인 플레이북, 거버넌스 spine, 참고문헌, 최초 접촉 프로파일링
