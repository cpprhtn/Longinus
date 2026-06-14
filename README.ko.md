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
- 특정 분야를 깊이 파고드는 **CTF 참가자·학습자**

## 무엇이 다른가

단순한 플레이북 모음이 아닙니다.

- **체크리스트가 아니라 원리에서 출발합니다.** 정해진 목록을 훑는 대신, 취약점이 발생하는 여섯 가지 근본
  원리(신뢰 경계, 파서 불일치, confused deputy, 상태·시간, 인코딩, 개발자의 암묵적 가정)에서 취약점을 도출합니다
  → [공격자의 사고법](references/attacker-mindset.md). 아직 분류되지 않은 새로운 유형까지 찾아냅니다.
- **개별 취약점이 아니라 연계 영향으로 평가합니다.** 단독으로는 낮은 등급이라도, 연결되면 계정 탈취로 이어지는
  취약점들이 있습니다. 그러한 연계는 하나의 Critical로 다룹니다 → [체이닝](references/chaining-and-impact.md).
- **오탐을 철저히 배제합니다.** 재현 가능한 PoC가 없으면 확정 보고하지 않습니다. LLM이 가장 취약한 지점이
  '그럴듯한 허위 취약점'이므로, 낮은 오탐률이 곧 신뢰성입니다 → [심각도 평가](references/severity-and-triage.md).

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
    ├── authorization-and-scope.md  attacker-mindset.md       ← ⛔ 게이트 + 🧠 공격자 사고법
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

`v0.1.0` — 초기 버전입니다. 도메인 리프와 `research/` 참고문헌은 지속적으로 발전시키는 살아있는 문서이며, 기법의
변화에 맞춰 함께 갱신합니다 (정책: [research/meta-resources.md](research/meta-resources.md)).
