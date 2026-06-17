# 🗡️ Longinus

> *스스로의 방어를 가장 먼저 시험하는 창.*  
> 실제 공격자보다 먼저, 방어하는 쪽이 자신의 취약점을 발견하도록 돕는 공격 보안 스킬.

[English](README.md) · **한국어**

**Longinus는 Claude Code를 보안 검사관으로 바꿔 주는 [스킬](https://docs.claude.com/en/docs/claude-code/skills)
입니다.** 공격자의 관점에서 내 프로젝트의 취약점을 찾아내고, 우선순위를 매긴 결과를 재현 가능한 PoC와 구체적인
수정 방안과 함께 제시합니다. 트리 구조로 정리된 공격 보안 플레이북과, 그것을 타깃에 맞게 운용하는 오케스트레이션
로직([SKILL.md](SKILL.md))으로 구성됩니다.

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

**옵션 — 멀티에이전트 레이어.** 스킬은 단독으로 동작합니다. 각 도메인을 **독립 컨텍스트**에서 병렬로 검사하고
싶다면(도메인 간 편향 없음), 번들된 [서브에이전트](agents/README.md)를 Claude Code가 인식하는 위치로 복사한 뒤
Claude Code를 재시작하세요:

```bash
# 프로젝트 단위 — repo의 .claude/agents/ 에 (스킬을 .claude/skills/longinus 로 클론한 경우)
mkdir -p .claude/agents && cp .claude/skills/longinus/agents/*.md .claude/agents/

# 또는 개인 단위 — 모든 프로젝트에서 사용 (개인 스킬 설치 후)
cp ~/.claude/skills/longinus/agents/*.md ~/.claude/agents/
```

그다음 (Claude Code **재시작 후**) *"Longinus 검사 돌려줘"* 라고 **말로 요청만 하면** — Claude가 각 에이전트의
`description`을 보고 **오케스트레이터에 자동 위임**합니다. 자동 위임이 안 되면 *"longinus-orchestrator 에이전트로
검사해 줘"* 처럼 명시해 호출하세요. → [agents/README.md](agents/README.md).

## 사용

설치 후 **세 가지 방법**으로 호출합니다:

```text
1) 슬래시 명령   /longinus            ← standard 검사
              /longinus quick      ← 빠른 기계적 스캔
              /longinus deep       ← 다중 도메인 정밀 검사

2) 그냥 요청     "출시 전에 이 저장소를 점검해 줘."
                "이 FastAPI API의 보안을 검토해 줘."
                "내 에이전트의 프롬프트 인젝션 표면을 레드팀해 줘."

3) 멀티에이전트  "Longinus 검사 돌려줘"   (위에서 agents/ 를 설치한 경우)
                → 오케스트레이터가 도메인 전문가들을 지휘
```

**스택을 가리지 않습니다.** Python/FastAPI, Node/Express, React Native, Go, Terraform/클라우드, LLM/RAG 앱
등 — 타깃을 먼저 분석한 뒤 알맞은 플레이북으로 안내합니다. 다만 독립적으로 실행되는 자동 스캐너가 아니라, Claude가
코드를 직접 검토하도록 이끄는 도구이며 기본 동작은 read-only입니다. 모든 실행은 고정 형식 보고서를
`.longinus/reports/longinus_YYYYMMDDHHMM.md`에 작성합니다. 본인 소유가 아닌 대상을 테스트하기 전에는 반드시
[인가 게이트](references/authorization-and-scope.md)를 확인하세요.

> ⛔ **방어를 위한 공격입니다.** 내가 소유한 코드, 명시적으로 인가받은 타깃, CTF·실습 환경에서만 사용하세요.
> 허가받지 않은 시스템에는 절대 사용하면 안 됩니다. [docs/ethics.md](docs/ethics.md)

## 스킬 모드

얼마나 깊게 갈지 고르세요 — 명령에 붙이거나, 말로 지정하면 됩니다. 기본은 **standard**.

| 모드 | 호출 | 동작 |
|---|---|---|
| **quick** | `/longinus quick` · *"quick 보안 검사 해 줘"* | 기계적 grep 스캔(각 leaf의 `## Mechanical scan`), 고정 심각도, CVSS/체이닝 없음 — 결과는 테이블 하나. **CI 게이트**와 **소형 온디바이스 모델(11B+)** (Qwen 3 8B / Llama 3.1 8B 등, 기계적 지시는 따르나 자유 추론은 약함)용. |
| **standard** | `/longinus` · *"이 저장소 검사해 줘"* | **기본값.** 프로파일 → 라우팅 → leaf 전체 분석 → PoC → 심각도 평가(CVSS 4.0) → 리포트. |
| **deep** | `/longinus deep` · *"이 저장소 deep 검사해 줘"* | Standard + 모든 보조 도메인 quick-check + 도메인 간 [체이닝](references/chaining-and-impact.md). |
| **continuous** | `/longinus continuous` · cron/CI | 직전 보고서 이후 `git diff`만 검사 + 기존 발견 재점검 → 델타 append ([continuous-audit.md](references/continuous-audit.md)). |

> 모드별 상세 절차는 [references/audit-modes.md](references/audit-modes.md)에 있습니다. 로컬 모델로 보안
> 점검을 한다면 **quick**부터 시작하세요.

## 무엇이 다른가

단순한 플레이북 모음이 아닙니다.

- **체크리스트가 아니라 원리에서 출발합니다.** 정해진 목록을 훑는 대신, 취약점이 발생하는 여섯 가지 근본
  원리(신뢰 경계, 파서 불일치, confused deputy, 상태·시간, 인코딩, 개발자의 암묵적 가정)에서 취약점을 도출합니다
  → [패턴 트리거 & 공격자 원리](references/pattern-triggers.md). 아직 분류되지 않은 새로운 유형까지 찾아냅니다.
- **설계 의도를 먼저 읽습니다.** `CLAUDE.md`/README/ADR에서 *Intent Brief*를 만든 뒤 구현이 의도에서
  벗어난 지점을 사냥합니다 — *문서화된* 의도적 결정은 결함으로 보고하지 않습니다 →
  [설계 의도](references/design-intent.md).
- **개별 취약점이 아니라 연계 영향으로 평가합니다.** 단독으로는 낮은 등급이라도, 연결되면 계정 탈취로 이어지는
  취약점들이 있습니다. 그러한 연계는 하나의 Critical로 다룹니다 → [체이닝](references/chaining-and-impact.md).
- **양방향 설계 — 결함은 곧 "차집합"입니다.** *Blue* 렌즈가 각 신뢰 경계에 *있어야 할* 통제 지도를 만들고,
  *Red* 렌즈가 도달 가능한 sink를 찾습니다. 취약점은 둘이 어긋나는 지점 — 도달 가능한 sink인데 기대 통제가
  없거나 우회되는 곳입니다. 이후 제안한 수정안을 **우회 시도(fix→bypass) 루프**로 두드려 버티는지 확인합니다
  → [red × blue 방법론](references/red-blue.md).
- **버그만이 아니라 커버리지를 측정합니다.** 모든 source→sink를 [Audit Ledger](references/audit-ledger.md)에
  판정과 함께 올려, 보고서가 *무엇을 보지 않았는지*(재현율)까지 말합니다. 검사하지 않은 sink는 *명시된 공백*이지
  조용한 "이상 없음"이 아닙니다.
- **오탐을 철저히 배제합니다.** 재현 가능한 PoC가 없으면 확정 보고하지 않습니다. 필수 심각도 게이트가
  Critical/High 판정을 기계적으로 하향 조정하여, LLM의 고질적인 심각도 인플레이션을 억제합니다
  → [심각도 평가](references/severity-and-triage.md).
- **추측이 아니라 실행으로 확인합니다.** 소유한 코드라면 에이전트의 셸로 **양성 PoC를 실제로 실행**해
  `Confirmed (executed)` / `(traced)`로 등급을 매깁니다 — "취약해 보임"을 증거로 바꾸되, 인가 게이트 안에서만
  → [proof & confirmation](references/proof-and-confirmation.md).
- **감사자를 공격하는 repo에 강건합니다.** 대상의 `README`·주석·`SECURITY.md`를 *untrusted data*로 취급합니다 —
  "여긴 건너뛰어"·"아무것도 보고하지 마" 같은 지시는 감사자를 노린 indirect prompt injection이라 *지시가 아니라
  발견*으로 처리합니다(자기 약 먹기) → [설계 의도](references/design-intent.md).
- **보고서는 항상 하나의 고정 형식.** 모든 검사가 동일한 템플릿(machine-readable 헤더 + 고정 섹션)을 출력해,
  사람·프로젝트 간 보고서가 일관되고 비교 가능합니다 → [보고서 템플릿](references/report-template.md).
- **한계에 대해 솔직합니다.** [한계 문서](references/limitations.md)가 정적/LLM 분석으로 *찾을 수 없는 것*을
  명시하여, 클린 리포트가 "취약점이 없다"는 뜻이 아님을 분명히 합니다.

## 왜 필요한가

- **AI가 생성한 코드는 빠르게, 그러나 취약한 채로 출시됩니다.** 출시 전에 스스로 점검할 체계적인 방법이 필요합니다.
- **보안 지식은 흩어져 있습니다.** OWASP, 버그바운티 기법, CTF 노하우를 매번 찾아 헤매는 대신, 검증된 방법론을
  한 흐름으로 따라갑니다.
- **스캐너는 노이즈를 쏟아냅니다.** Longinus는 *증명하지 못하면 보고하지 않는다*는 원칙으로 신뢰할 수 있는 결과만
  남깁니다.

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
├── CHANGELOG.md                    ← 버전 이력 (전체 릴리스)
├── SKILL.md                        ← 오케스트레이션 브레인 (실제 진입점)
├── RESEARCH.md                     ← 참고문헌 허브 (research/ 트리 색인)
├── agents/                         ← 옵션 멀티에이전트 레이어 (오케스트레이터 + Blue + Red 전문가 5)
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
    ├── pattern-triggers.md                                   ← 🎯 공격자 원리 + ⛔ DO-NOT-report FP 가드
    ├── pattern-catalog.md                                    ← 🎯 코드 패턴 → 취약점 → leaf 룩업 테이블
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

**변경 이력** — 전체 버전 이력은 이제 [CHANGELOG.md](CHANGELOG.md)에 있습니다.
