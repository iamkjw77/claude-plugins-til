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

> 아래에서 두 가지를 뜯어본다: **①훅으로 스킬을 자가 주입**하는 방식(§1),
> **②스킬이 모델의 행동을 막는 관문을 설치**하는 방식(§2, `brainstorming` 사례).

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

## 2. brainstorming 뜯어보기 — 스킬이 행동을 막는 관문(HARD-GATE)을 설치한다

> 📂 원문: [`sources/superpowers/skills/brainstorming/SKILL.md`](../sources/superpowers/skills/brainstorming/SKILL.md)

Frontend Design의 `description`이 *"~하면 써도 좋다"*였다면, brainstorming은 톤이 완전히 다르다.
**"코드부터 짜지 마라. 대화로 설계를 확정하고 승인받은 다음에 구현해라"**를 강제한다.
Superpowers의 행동 통제 기법이 가장 잘 드러나는 문서다.

### ① frontmatter부터 강제 톤

```yaml
description: "You MUST use this before any creative work - creating features,
building components, adding functionality, or modifying behavior..."
```

"기능·컴포넌트·동작 수정 등 창의적 작업 **전엔 반드시(MUST)** 이 스킬을 써라."
Frontend Design이 *"Use when..."*(허용)이었다면 여기는 *"MUST ... before"*(강제)다.

### ② HARD-GATE — 차단봉을 세운다

```markdown
<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any project,
or take any implementation action until you have presented a design and the
user has approved it. This applies to EVERY project regardless of perceived
simplicity.
</HARD-GATE>
```

**설계 제시 + 사용자 승인 전까지는** 코드 작성·구현 스킬 호출·스캐폴딩 등 **모든 구현 행동 금지.**
모델은 본능적으로 "빨리 코드부터" 뽑으려 하는데, 그 길목에 **"승인 전엔 통과 금지"라는 차단봉**을
프롬프트로 세운 것이다. 아무리 단순해 보여도 **모든** 프로젝트에 예외 없이 적용된다.

### ③ Anti-Pattern을 이름으로 차단

> **"This Is Too Simple To Need A Design"** — 할 일 목록도, 함수 하나짜리 유틸도, 설정 변경도
> 전부 이 과정을 거친다. **'간단한' 프로젝트야말로 검토 안 한 가정 때문에 헛수고가 가장 많이 난다.**

모델이 게이트를 빠져나가려 흔히 대는 핑계("이건 사소하니까 그냥 하죠")를 **미리 이름 붙여 봉쇄**한다.
Frontend Design이 "Inter 쓰지 마"로 실패를 지목한 것과 **같은 기법**(흔한 실패를 이름으로 금지).

### ④ 번호 매긴 9단계 Checklist (+ task 강제)

> "각 항목을 **task로 만들고 순서대로** 완료해야 한다."

| # | 단계 | 요지 |
|---|---|---|
| 1 | 프로젝트 맥락 파악 | 파일·문서·최근 커밋 확인 |
| 2 | 시각 도구 제안(just-in-time) | 미리 X, "말보다 그림"이 나을 질문에서만 |
| 3 | 명료화 질문 | **한 번에 하나씩** (목적·제약·성공 기준) |
| 4 | 접근법 2~3개 제안 | 트레이드오프 + **추천안 먼저** |
| 5 | 설계 제시 | 복잡도에 맞춰 섹션별, **섹션마다 승인** |
| 6 | 설계 문서 작성 | `docs/superpowers/specs/YYYY-MM-DD-<주제>-design.md` + 커밋 |
| 7 | 스펙 셀프 리뷰 | 플레이스홀더·모순·범위·모호함 점검 후 인라인 수정 |
| 8 | 사용자 스펙 검토 | 스펙 파일을 사용자가 확인 |
| 9 | 구현으로 전환 | `writing-plans` 스킬 호출 |

### ⑤ 흐름도를 프롬프트에 내장 — 관문은 두 개

문서 안에 `dot`(Graphviz) 순서도가 통째로 박혀 있다. 핵심은 **승인 관문이 두 번** 있다는 것:

```
설계 제시 →[승인?]─no(수정)─▶ 다시 설계
              └─yes─▶ 문서 작성 → 셀프리뷰 →[사용자 검토?]─변경─▶ 다시 작성
                                                    └─승인─▶ writing-plans (종료)
```

> **종료 상태는 오직 `writing-plans` 호출.** `frontend-design`·`mcp-builder` 등 다른 구현 스킬은
> 절대 부르지 않는다. → **스킬 간 이동 경로 자체를 프롬프트로 못 박는다.**

### 🔑 배운 것 3가지

4. **HARD-GATE = 모델의 "성급한 구현" 본능을 막는 강제 관문.**
   설명이 아니라 통제다. 특정 행동을 "승인 전까지 금지"로 잠가 행동 **순서**를 강제한다.

5. **회피 핑계를 미리 이름 붙여 봉쇄한다.**
   Anti-Pattern 섹션으로 "너무 간단한데요"라는 탈출 경로를 선제 차단. (Frontend Design의 `NEVER`와 동일 발상)

6. **절차를 "번호 + task + 흐름도"로 삼중 고정.**
   자연어 지침만으론 모델이 단계를 건너뛴다. 그래서 체크리스트·task 생성·순서도(dot)로
   **빠져나갈 틈을 없앤다.** 종료 상태(`writing-plans`)까지 하나로 고정.

---

## 3. 스킬 안을 더 깊이 — Visual Companion (실행 코드가 붙은 스킬)

> 📂 원문: [`skills/brainstorming/`](../sources/superpowers/skills/brainstorming/) 전체
> (SKILL.md + `visual-companion.md` + `spec-document-reviewer-prompt.md` + `scripts/`)

Frontend Design 스킬은 `SKILL.md` **딱 하나**였다. 그런데 brainstorming 폴더를 열어보면
파일이 여러 개다. 여기서 **"잘 자란 스킬의 내부 구조"**가 드러난다.

### 스킬 내부 파일은 4가지 유형으로 나뉜다

| 유형 | 예 (brainstorming) | 읽는 주체 / 시점 |
|---|---|---|
| ① 메인 프롬프트 | `SKILL.md` | 항상 로드 |
| ② 보조 지침서 | `visual-companion.md` | **나 자신**이 그 기능 쓸 때만 (점진적 공개) |
| ③ 위임용 프롬프트 | `spec-document-reviewer-prompt.md` | **다른 서브에이전트**에게 넘기는 대본 |
| ④ 실행 코드 | `scripts/*.cjs, *.sh, *.html, *.js` | 프롬프트가 아니라 **실제로 돌아가는 프로그램** |

> 🔑 핵심: `SKILL.md`은 **"언제/왜"만 적고, "어떻게"는 밖으로 뺀다.** 무거운 판단 기준은
> ②로, 위임 작업 대본은 ③으로, 실제 기능 구현은 ④로 분리 → SKILL.md은 가볍게 유지된다.

### Visual Companion이 뭐고, 왜 브라우저 + 웹소켓인가

brainstorming은 설계 중 *"이 레이아웃 중 뭐가 나아요?"*처럼 **말보다 보여주는 게 나은 순간**을
위해 브라우저 기반 시각 도구(`scripts/`)를 갖고 있다. 왜 이런 무거운 구성이 필요할까?

- **왜 브라우저?** Claude는 **글자만 나오는 터미널**에 산다. 진짜 목업·다이어그램·색을
  렌더링하려면 "그림을 그릴 수 있는 창"을 밖에 하나 띄워야 한다 → 브라우저.
- **왜 웹소켓?** 브레인스토밍은 대화가 계속 흐른다. **같은 탭에 새 화면을 실시간으로
  갈아끼우고**(새로고침 없이), 사용자의 **클릭을 즉시 되받아야** 한다. 요청-응답 후 끊기는
  일반 웹페이지로는 안 되고, **살아있는 양방향 연결**이 필요하다.

### 동작 메커니즘 — 파일 감시 + 양방향

```
[ Claude(터미널) ]
   │  ① content/ 에 화면 HTML 한 장 write
   ▼
[ server.cjs ]  content/ 디렉터리를 fs.watch 로 감시
   │  파일 생기면 broadcast({type:"reload"}) → 열린 탭 자동 새로고침
   ▼  ⇅ WebSocket (server.cjs가 RFC6455 직접 구현)
[ 브라우저 탭 ]  frame-template.html(껍데기)에 화면이 채워짐
   │  ② 사용자가 [data-choice] 요소 클릭
   ▼  helper.js가 클릭 감지 → 웹소켓으로 {choice:"a"} 전송
[ server.cjs ]  handleMessage() → state/events 파일에 JSON 한 줄 append
   ▲
[ Claude ]  ③ 다음 턴에 events 파일을 읽어 "A 골랐구나" 파악
```

| 방향 | 통로 | 무엇 |
|---|---|---|
| Claude → 사용자 | `content/*.html` → 서버가 push | **화면 보여주기** |
| 사용자 → Claude | 클릭 → 웹소켓 → `state/events` | **선택 되돌려받기** |

설계상 클릭은 **보조 데이터**이고, 최종 확정은 여전히 **터미널 대화가 주(主)**다.
(스킬도 *"터미널에 답해주세요, 원하면 클릭도"*라고 안내)

### 직접 띄워서 확인함 (실증)

이 스킬의 `scripts/`를 실제로 실행해 봤다. `start-server.sh`로 로컬 서버를 띄우고
`content/`에 목업 HTML을 떨어뜨리니 **브라우저 탭이 자동으로 열리며** 화면이 떴고,
새 파일을 넣을 때마다 **같은 탭이 실시간으로 교체**됐다. A 카드를 클릭하자
`state/events`에 아래 한 줄이 기록됐다:

```json
{"type":"click","text":"...A · 점수 강조형...","choice":"a","timestamp":1784096691762}
```

- **화면에 뜨는 건 브레인스토밍 "전체"가 아니다.** *질문 단위*로, "보는 게 나은" 것만
  한 장씩 띄웠다 지운다. 나머지 대화는 터미널, 최종 결과는 스펙 `.md`에 남는다.
- **목업뿐 아니라 아키텍처 다이어그램·플로우차트도** 같은 방식으로 보여준다(인라인 SVG로
  그려 push). 단 CSP 때문에 외부 라이브러리(mermaid 등)는 못 쓰고 순수 SVG/CSS로 그려야 한다.

### 🔑 배운 것

7. **"잘 만든 스킬"은 SKILL.md 하나가 아니다.** 무거운 건 밖으로(보조 지침·위임 대본·실행 코드)
   빼고 SKILL.md은 가볍게 — Frontend Design(1파일)과 brainstorming(다파일)의 차이가 이것.
8. **스킬은 프롬프트만이 아니라 실제 프로그램도 품을 수 있다.** Visual Companion은 웹소켓 서버까지
   포함한 미니 앱이다. "플러그인 = 마크다운"이라는 인상보다 실제는 더 넓다.

---

## 4. 테스트 — "프롬프트도 코드처럼 테스트한다"

> 📂 원문: [`tests/`](../sources/superpowers/tests/) (52파일, 스킬 폴더 다음으로 큼)

"플러그인 = 마크다운 프롬프트"인데 왜 테스트가 있을까? superpowers는 ①실행 코드가 있고
②프롬프트 라우팅의 신뢰성이 곧 품질이며 ③여러 하네스를 지원하기 때문이다. 테스트는 3층위다.

### 층위 1 — 실행 코드 유닛 테스트
`tests/brainstorm-server/*.test.js` — §3의 Visual Companion 서버 검증.
- `ws-protocol.test.js` — 웹소켓 프레임 인코딩/디코딩 (RFC 6455 직접 구현했으니)
- `auth.test.js`(토큰 인증), `lifecycle.test.js`(켜기/끄기·idle), `helper.test.js`(재연결 백오프)
- **브라우저는 안 띄운다.** `helper.js`를 `window` 없는 샌드박스에서 돌려 **순수 함수만** 검사.

### 층위 2 — 🔑 프롬프트 발동 테스트 (이 저장소 주제의 증거)
`tests/explicit-skill-requests/` — **"이렇게 말하면 Claude가 그 스킬을 실제로 발동하나?"**를 검증.

```
# 입력: prompts/please-use-brainstorming.txt (딱 한 줄)
please use the brainstorming skill to help me think through this feature

# 판정: run-test.sh 가 claude -p 로 헤드리스 실행 후, 로그를 grep
if grep '"name":"Skill"' && grep '"skill":"brainstorming"' ; then PASS
```

> TIL 01에서 *"description이 곧 라우팅 로직"*이라 했는데, 그 **자연어 라우팅이 회귀로 깨지지
> 않는지를 자동 검증**하는 것. `please-*`, `skip-formalities` 등 여러 말투로도 시험한다.

### 층위 3 — 크로스플랫폼/이식 테스트
`tests/{kimi,opencode,codex,pi,antigravity}/` — 같은 플러그인을 다른 하네스에 이식해도
매니페스트·로딩·훅이 동작하는지. `tests/hooks/test-session-start.sh`는 §1의 자가 주입 훅 검증.

### 언제 도나 — 런타임이 아니라 "개발 시점"

```
[ 사용자가 플러그인 쓸 때 ]        [ 저자가 플러그인 고칠 때 ]
프롬프트 입력 → 스킬 발동           코드/프롬프트 수정 → 테스트 수동 실행 → 확인 후 커밋
❌ 테스트 안 돎 (런타임에 없음)     ✅ 회귀(깨짐) 검사용
```

- 이 레포엔 **자동 CI(`.github/workflows`) 없음** — 저자가 `run-skill-tests.sh`를 손으로 돌린다.
- `.pre-commit-config.yaml`은 `evals/` 파이썬 린트만 자동. 스킬/서버 테스트는 수동.
- `ws-protocol.test.js` 주석에 *"Module doesn't exist yet (**TDD** — tests written before
  implementation)"* — 테스트 먼저 쓰고 구현하는 **TDD 흔적**. (저자가 자기 `test-driven-development`
  스킬을 자기 개발에 씀 → self-hosting)

### 헷갈리기 쉬운 점 — "로그/grep은 언제?"

처음엔 *"프롬프트 쓸 때마다 로그 찍고 grep하나?"* 싶은데, 그렇지 않다. **로그와 grep은
테스트를 실행한 순간에만, 테스트가 별도로 띄운 Claude에 대해서만** 일어난다.

- **당신이 쓰는 Claude** ≠ **테스트가 띄우는 Claude**. 층위 2 테스트는 `run-test.sh`가
  `claude -p "<샘플 프롬프트>" --output-format stream-json` 으로 **새 Claude 인스턴스**를
  띄우고, 그 실행이 남긴 JSON 로그를 grep한다. 입력은 당신 프롬프트가 아니라 `.txt` **샘플**이다.
- 발동 방식은 **채팅이 아니라 터미널에서 셸 스크립트 실행**(`./run-skill-tests.sh`), 돌리는
  주체는 보통 **플러그인을 고치는 개발자**. 그냥 쓰는 사용자는 평생 안 돌릴 수도 있다.
- 참고로 층위 1(`brainstorm-server/*.test.js`)은 미니앱(Visual Companion 서버)을 위한 테스트가
  **맞다.** 다만 그게 전부가 아니라 프롬프트(층위2)·플랫폼(층위3) 테스트도 함께 있는 것.

> 비유: 공장이 **출고 전 시험용 차 한 대를 따로 몰며 계기판 로그를 기록**하는 것. 당신이 산 차
> (평소 주행)는 그런 로그를 안 남긴다. 시험 주행(테스트)은 **별도 차량으로 명령 시에만** 한다.

### 🔑 배운 것

9. **프롬프트는 회귀 테스트의 대상이 될 수 있다.** "말투 A로도 스킬이 발동하나"를 `claude -p`
   로그 grep으로 자동 판정. 프롬프트 엔지니어링을 **엔지니어링답게** 다루는 실물 사례.

---

## 5. 두 플러그인 비교 (Frontend Design vs Superpowers)

| | Frontend Design | Superpowers |
|---|---|---|
| 유형 | 순수 스킬 | 스킬+훅+스크립트 |
| 파일 수 | 2 | 수십 개 |
| 발동 방식 | 모델이 `description` 보고 판단 | 훅이 세션 시작 시 강제 주입 |
| 핵심 기법 | 실패 사례를 이름으로 금지 | HARD-GATE + 점진적 공개 |
| 진입 난이도 | ★☆☆☆☆ | ★★★☆☆ |

---

## 6. 내 플러그인에 훔쳐올 것 (Superpowers편)

- [ ] 강제 순서가 필요하면 **`<HARD-GATE>` 패턴** 차용
- [ ] 모델이 규칙을 빠져나가는 **회피 핑계를 Anti-Pattern 섹션으로 선제 차단**
- [ ] 절차는 **번호 + task 생성 + 흐름도(dot)**로 삼중 고정
- [ ] 스킬이 커지면 **부트스트랩 + 점진 로딩**으로 토큰 절약
- [ ] 자동 발동이 필요하면 **`SessionStart` 훅**으로 부트스트랩 스킬만 주입
- [ ] SKILL.md은 가볍게 — **무거운 지침·위임 대본·실행 코드는 별도 파일로 분리**
- [ ] 정말 필요하면 스킬에 **실행 코드(스크립트)**도 붙일 수 있다 (단 꼭 필요할 때만)

## 한 줄 회고

> 플러그인 제작의 90%는 소프트웨어 엔지니어링이 아니라 **프롬프트 설계 + 주입 타이밍 설계**다.
> 프론트 개발자에게 유리한 건, "AI가 만드는 UI의 흔한 실패"를 내가 이미 눈으로 알기 때문에
> `NEVER ...` 규칙을 남들보다 구체적으로 쓸 수 있다는 점.

<!-- TODO: 추가 보충 후보 — 14개 스킬 목록/분류, run-hook.cmd 크로스플랫폼 분기 상세, using-superpowers 본문 분석 -->
