# TIL 02 — Superpowers 해부: 스스로 부팅하는 플러그인

> 대상: `superpowers` v6.1.1 (로컬에 설치된 실제 소스로 분석)
> 날짜: 2026-07-14

> 플러그인의 5대 구성요소 개념은 [TIL 01 §1](01-frontend-design.md)을 먼저 참고.
> Frontend Design이 "최소 뼈대"라면, Superpowers는 그 반대 극단인 **풀세트**다.

---

## 1. Superpowers — "풀세트 구조"

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

1. **"자동 발동"의 정체는 훅이다.**
   플러그인이 알아서 켜지는 게 아니라, `SessionStart` 같은 이벤트에 스크립트를 걸어
   **필요한 프롬프트를 그 타이밍에 주입**한다. 부트스트랩(using-superpowers) 하나만
   주입하고, 나머지 13개 스킬은 그 안내서를 본 모델이 `Skill` 툴로 필요할 때 꺼내 쓴다.

2. **Progressive disclosure(점진적 공개) 패턴.**
   14개 스킬 전문을 매번 다 주입하면 컨텍스트 낭비 + 산만. 그래서 "목차+사용법"만
   먼저 주입하고, 본문은 호출 시 로드. 토큰 예산 설계의 정석.

3. **크로스 플랫폼을 훅에서 분기한다.**
   스크립트가 `CLAUDE_PLUGIN_ROOT` / `CURSOR_PLUGIN_ROOT` / `COPILOT_CLI` 환경변수로
   Claude Code·Cursor·Copilot에 각각 다른 JSON 필드명으로 출력한다. 같은 플러그인이
   여러 하네스에서 동작하게 만드는 실무 디테일.

---

## 2. 두 플러그인 비교 (Frontend Design vs Superpowers)

| | Frontend Design | Superpowers |
|---|---|---|
| 유형 | 순수 스킬 | 스킬+훅+스크립트 |
| 파일 수 | 2 | 수십 개 |
| 발동 방식 | 모델이 `description` 보고 판단 | 훅이 세션 시작 시 강제 주입 |
| 핵심 기법 | 실패 사례를 이름으로 금지 | HARD-GATE + 점진적 공개 |
| 진입 난이도 | ★☆☆☆☆ | ★★★☆☆ |

---

## 3. 내 플러그인에 훔쳐올 것 (Superpowers편)

- [ ] 강제 순서가 필요하면 **`<HARD-GATE>` 패턴** 차용
- [ ] 스킬이 커지면 **부트스트랩 + 점진 로딩**으로 토큰 절약
- [ ] 자동 발동이 필요하면 **`SessionStart` 훅**으로 부트스트랩 스킬만 주입

## 한 줄 회고

> 플러그인 제작의 90%는 소프트웨어 엔지니어링이 아니라 **프롬프트 설계 + 주입 타이밍 설계**다.
> 프론트 개발자에게 유리한 건, "AI가 만드는 UI의 흔한 실패"를 내가 이미 눈으로 알기 때문에
> `NEVER ...` 규칙을 남들보다 구체적으로 쓸 수 있다는 점.

<!-- TODO: 내용 보충 예정 — 14개 스킬 목록/분류, run-hook.cmd 크로스플랫폼 분기 상세, using-superpowers 본문 분석 -->
