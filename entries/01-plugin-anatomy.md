# TIL 01 — 플러그인 해부: 최소 뼈대부터 자가 발동까지

> 대상: `frontend-design`, `superpowers` (둘 다 로컬에 설치된 실제 소스로 분석)
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

핵심 통찰: **이 중 하나만 있어도 플러그인이 된다.** 아래 두 사례가 양 극단을 보여준다.

---

## 2. 사례 A — Frontend Design: "파일 2개짜리 플러그인"

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

## 3. 사례 B — Superpowers: "스스로 부팅하는 플러그인"

> 📂 원문: [`sources/superpowers/`](../sources/superpowers/) — 특히 [`hooks/session-start`](../sources/superpowers/hooks/session-start)

설치 수 2위(~752K). 이쪽은 풀세트다.

```
superpowers/
├── .claude-plugin/{plugin.json, marketplace.json}
├── skills/           # 14개 스킬 (brainstorming, TDD, systematic-debugging ...)
├── hooks/            # ← 자가 발동의 핵심
├── scripts/
└── tests/
```

### 스킬은 "게이트"를 건다

`brainstorming/SKILL.md`에서:

```markdown
<HARD-GATE>
Do NOT invoke any implementation skill, write any code, ... until you have
presented a design and the user has approved it. This applies to EVERY
project regardless of perceived simplicity.
</HARD-GATE>
```

단순 설명이 아니라 **강제 게이트**. 모델의 행동 순서를 프롬프트로 통제한다.

### 🔑 결정적 발견 — SessionStart 훅으로 자가 주입

`hooks/hooks.json`:

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup|clear|compact",
      "hooks": [{
        "type": "command",
        "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
        "async": false
      }]
    }]
  }
}
```

이 훅이 실행하는 `session-start` 스크립트는:

```bash
# using-superpowers 스킬 전문을 읽어서
using_superpowers_content=$(cat "${PLUGIN_ROOT}/skills/using-superpowers/SKILL.md")
# JSON escape 후
# additionalContext 로 컨텍스트에 주입
printf '{ "hookSpecificOutput": { "hookEventName": "SessionStart",
         "additionalContext": "%s" } }' "$session_context"
```

즉 **매 세션 시작마다 `using-superpowers` 스킬을 통째로 모델 컨텍스트에 밀어넣는다.**

> 💡 이 문서를 쓰는 지금 이 세션의 시스템 프롬프트 최상단에도
> `<EXTREMELY_IMPORTANT> You have superpowers... 'using-superpowers' skill...`
> 이 그대로 떠 있었다. **훅이 실제로 나를 부팅시킨 증거를 직접 목격함.**

### 🔑 배운 것 3가지

4. **"자동 발동"의 정체는 훅이다.**
   플러그인이 알아서 켜지는 게 아니라, `SessionStart` 같은 이벤트에 스크립트를 걸어
   **필요한 프롬프트를 그 타이밍에 주입**한다. 부트스트랩(using-superpowers) 하나만
   주입하고, 나머지 13개 스킬은 그 안내서를 본 모델이 `Skill` 툴로 필요할 때 꺼내 쓴다.

5. **Progressive disclosure(점진적 공개) 패턴.**
   14개 스킬 전문을 매번 다 주입하면 컨텍스트 낭비 + 산만. 그래서 "목차+사용법"만
   먼저 주입하고, 본문은 호출 시 로드. 토큰 예산 설계의 정석.

6. **크로스 플랫폼을 훅에서 분기한다.**
   스크립트가 `CLAUDE_PLUGIN_ROOT` / `CURSOR_PLUGIN_ROOT` / `COPILOT_CLI` 환경변수로
   Claude Code·Cursor·Copilot에 각각 다른 JSON 필드명으로 출력한다. 같은 플러그인이
   여러 하네스에서 동작하게 만드는 실무 디테일.

---

## 4. 두 플러그인 비교

| | Frontend Design | Superpowers |
|---|---|---|
| 유형 | 순수 스킬 | 스킬+훅+스크립트 |
| 파일 수 | 2 | 수십 개 |
| 발동 방식 | 모델이 `description` 보고 판단 | 훅이 세션 시작 시 강제 주입 |
| 핵심 기법 | 실패 사례를 이름으로 금지 | HARD-GATE + 점진적 공개 |
| 진입 난이도 | ★☆☆☆☆ | ★★★☆☆ |

---

## 5. 내 플러그인에 훔쳐올 것 (→ TIL 02로 이어짐)

- [ ] **description에 트리거 조건을 자연어로 명시** ("Use when the user builds an RN screen...")
- [ ] **AI가 흔히 틀리는 걸 이름으로 금지** (예: 하드코딩된 hex 색상, 인라인 스타일 남발)
- [ ] 강제 순서가 필요하면 **`<HARD-GATE>` 패턴** 차용
- [ ] 스킬이 커지면 **부트스트랩 + 점진 로딩**으로 토큰 절약
- [ ] 최소 시작은 **`plugin.json` + `SKILL.md` 2개**면 충분 — 부담 없이 시작

## 한 줄 회고

> 플러그인 제작의 90%는 소프트웨어 엔지니어링이 아니라 **프롬프트 설계 + 주입 타이밍 설계**다.
> 프론트 개발자에게 유리한 건, "AI가 만드는 UI의 흔한 실패"를 내가 이미 눈으로 알기 때문에
> `NEVER ...` 규칙을 남들보다 구체적으로 쓸 수 있다는 점.
