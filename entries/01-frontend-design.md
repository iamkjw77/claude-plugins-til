# TIL 01 — Frontend Design 해부: 파일 2개짜리 최소 플러그인

> 대상: `frontend-design` (Anthropic 공식, 로컬에 설치된 실제 소스로 분석)
> 날짜: 2026-07-14

## 0. 한 줄 요약

Claude Code 플러그인은 **결국 마크다운 프롬프트 + 얇은 설정(JSON) + 선택적 훅 스크립트**의 묶음이다.
"코드"보다 **"언제 어떤 프롬프트를 모델의 컨텍스트에 밀어넣느냐"**가 본질이다.

---

## 1. 플러그인의 5대 구성요소

플러그인은 아래 중 일부 또는 전부를 담는다:

| 구성요소 | 위치 | 역할 |
|---|---|---|
| **manifest** | `.claude-plugin/plugin.json` | 이름/설명/버전 등 메타데이터 |
| **skills** | `skills/<name>/SKILL.md` | 모델에게 주입되는 절차적 지식(프롬프트) |
| **agents** | `agents/*.md` | 위임 가능한 서브에이전트 정의 |
| **commands** | `commands/*.md` | `/명령어` 슬래시 커맨드 |
| **hooks** | `hooks/hooks.json` + 스크립트 | 특정 이벤트에 자동 실행되는 훅 |

핵심 통찰: **이 중 하나만 있어도 플러그인이 된다.** Frontend Design은 그 최소 극단을 보여준다.
(반대 극단 — 풀세트 — 은 [TIL 02: Superpowers](02-superpowers.md) 참고.)

---

## 2. Frontend Design — "파일 2개짜리 플러그인"

> 📂 원문: [`sources/frontend-design/`](../sources/frontend-design/)

Anthropic 공식, 설치 수 1위(~829K)인데 **실제 파일은 딱 2개**다.

```
frontend-design/
├── .claude-plugin/plugin.json   # 7줄짜리 메타데이터
└── skills/frontend-design/SKILL.md   # 프롬프트 본체
```

### manifest (`plugin.json`)

```json
{
  "name": "frontend-design",
  "description": "Frontend design skill for UI/UX implementation",
  "author": { "name": "Anthropic", "email": "support@anthropic.com" }
}
```

놀랍도록 단순하다. **로직이 전혀 없다.**

### SKILL.md — 진짜 힘은 여기 있다

```markdown
---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces...
             Use this skill when the user asks to build web components,
             pages, or applications...
---

## Design Thinking
Before coding, understand the context and commit to a BOLD aesthetic direction:
- Tone: Pick an extreme: brutally minimal, maximalist chaos, ...
NEVER use generic AI-generated aesthetics like overused font families
(Inter, Roboto, Arial), cliched color schemes (particularly purple
gradients on white backgrounds)...
```

### 🔑 배운 것 3가지

1. **`description`이 곧 라우팅 로직이다.**
   코드로 "언제 발동할지"를 if문으로 짜지 않는다. 대신 자연어 `description`에
   *"Use this skill when the user asks to build web components..."* 라고 쓰면,
   모델이 그 설명을 보고 스스로 발동 여부를 판단한다. → **트리거링 = 프롬프트 엔지니어링**

2. **프롬프트는 "하지 마라"를 구체적으로 박는다.**
   `NEVER use ... Inter, Roboto ... purple gradients on white` — 추상적 지침이 아니라
   **AI가 흔히 저지르는 실패를 이름으로 지목**한다. 이게 "AI slop 방지"의 실체다.

3. **플러그인 = 잘 설계된 시스템 프롬프트 조각.**
   빌드 스텝도, 의존성도 없다. 프론트 개발자 입장에서 진입장벽이 거의 0이다.

---

## 3. SKILL.md 본문 뜯어보기 — 6조각으로

`SKILL.md`은 코드 한 줄 없이 **전부 Claude에게 주는 명령(프롬프트)**이다.
크게 6조각으로 나눠 읽으면 "이 파일이 무슨 일을 하는지"가 또렷해진다.

### ① frontmatter — "언제 발동할지" 스위치

```yaml
---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces...
             Use this skill when the user asks to build web components,
             pages, or applications...
license: Complete terms in LICENSE.txt
---
```

> "사용자가 웹 컴포넌트·페이지·앱을 만들어 달라고 하면 이 스킬을 써라."

이게 **자동 발동 스위치**다. if문이 아니라 자연어로 조건을 적어두면, 모델이 `description`을 읽고
"지금 상황이 여기 해당하네" 하고 **스스로 켠다.** (→ 트리거링 = 프롬프트 엔지니어링)

### ② 도입부 — "무엇을 만들 것인가"

> This skill guides creation of distinctive, production-grade frontend interfaces
> that avoid generic "AI slop" aesthetics.

번역: "흔해 빠진 **'AI가 대충 만든 티 나는(AI slop)'** 디자인을 피하고, **개성 있고 실무에 바로 쓸 수 있는**
프론트엔드를 만들도록 안내한다. 실제로 동작하는 코드를, 미적 디테일에 극도로 신경 써서 구현하라."

### ③ Design Thinking — "코딩 전에 방향부터 정해라"

> Before coding, understand the context and commit to a **BOLD** aesthetic direction.

코드 짜기 전에 네 가지를 먼저 정하라고 한다:

| 항목 | 쉬운 뜻 |
|---|---|
| **Purpose** | 이 화면이 푸는 문제? 누가 쓰나? |
| **Tone** | 분위기를 **극단적으로** 하나 고르기 (미니멀 / 맥시멀 카오스 / 레트로퓨처 / 브루탈리즘 / 파스텔 …) |
| **Constraints** | 프레임워크·성능·접근성 등 기술 제약 |
| **Differentiation** | **딱 하나, 잊을 수 없게 만들 포인트** |

> **CRITICAL:** Bold maximalism and refined minimalism both work — the key is
> **intentionality, not intensity.**

핵심 메시지: **"세든 약하든 상관없다. 애매한 게 문제다."** 명확한 컨셉 하나를 정밀하게 실행하라.

### ④ Frontend Aesthetics Guidelines — 구체적 5대 무기

| 항목 | 핵심 |
|---|---|
| **Typography(폰트)** | 아름답고 독특한 폰트. Arial·Inter 금지. **개성 있는 제목용 + 정제된 본문용** 짝짓기 |
| **Color & Theme(색)** | 하나의 미감에 올인, CSS 변수로 일관성. **지배적 색 + 날카로운 포인트 색** > 골고루 퍼진 색 |
| **Motion(움직임)** | 자잘한 효과 여러 개보다 **잘 짜인 페이지 로드 애니메이션(시차 등장) 하나**가 더 감동적 |
| **Spatial(배치)** | 예상 밖 레이아웃 — 비대칭·겹침·대각선·격자 깨기. 넉넉한 여백 **또는** 통제된 밀도 |
| **Backgrounds(배경/디테일)** | 단색으로 도망가지 말고 **분위기·깊이** — 그라디언트 메시, 노이즈, 그림자, 그레인 오버레이 등 |

### ⑤ NEVER 규칙 — 이 스킬의 진짜 핵심

> NEVER use generic AI-generated aesthetics like overused font families
> (**Inter, Roboto, Arial**, system fonts), cliched color schemes
> (particularly **purple gradients on white backgrounds**) ...
> NEVER converge on common choices (**Space Grotesk**, for example) across generations.

"좋게 만들어라" 같은 추상적 지침이 아니라, **AI가 흔히 저지르는 실패를 이름으로 콕 집어 금지**한다.
(Inter, 흰 배경 위 보라 그라디언트, Space Grotesk…) — 이게 "AI slop 방지"의 실체다.

### ⑥ 마무리 — 복잡도 매칭 + 자신감

> **IMPORTANT:** Match implementation complexity to the aesthetic vision.

"맥시멀리즘엔 애니메이션 잔뜩 든 정교한 코드가, 미니멀엔 절제와 정밀함이 필요하다.
**우아함은 비전을 잘 실행할 때 나온다.**" 그리고 끝은 자신감 주입 — *"Don't hold back."*

### 한 문장 요약

> `SKILL.md`은 **"AI 티 나는 뻔한 UI를 만들지 않게 하는 프롬프트"**다. 방식은 두 가지 —
> ① "과감한 컨셉 하나에 올인하라"고 **방향**을 잡아주고,
> ② "Inter·보라 그라디언트·Space Grotesk 쓰지 마라"고 **흔한 실패를 이름으로 금지**한다.

---

## 4. 내 플러그인에 훔쳐올 것 (Frontend Design편)

- [ ] **description에 트리거 조건을 자연어로 명시** ("Use when the user builds an RN screen...")
- [ ] **AI가 흔히 틀리는 걸 이름으로 금지** (예: 하드코딩된 hex 색상, 인라인 스타일 남발)
- [ ] 최소 시작은 **`plugin.json` + `SKILL.md` 2개**면 충분 — 부담 없이 시작
