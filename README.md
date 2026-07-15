# Claude Code Plugins — 해부 & 제작 TIL

인기 Claude Code 플러그인을 **역설계(reverse-engineering)** 하면서 "왜 잘 동작하는가"를 기록하고,
그 원리로 **직접 플러그인을 만드는** 과정을 담은 학습 저장소.

> 목표: "플러그인을 쓸 줄 안다"가 아니라 **"플러그인의 내부 구조와 발동 원리를 이해하고 직접 만든다"**
> 를 증명하는 것. (프론트/앱 개발자 + AI 활용 방향 이직 준비)

## 왜 요약이 아니라 해부인가

플러그인 소개글 요약은 누구나 한다. 채용에서 신호가 되는 건 **원리 이해 + 제작 경험**이다.
그래서 이 저장소는 소개글이 아니라 **실제 설치된 플러그인의 소스 코드**를 읽고 분석한다.
해부한 원문은 [`sources/`](sources/)에 발췌해 뒀다(출처·라이선스 명시).

## 진행 상황

| # | 제목 | 상태 | 다룬 플러그인 |
|---|------|------|--------------|
| 01 | [Frontend Design 해부 — 파일 2개짜리 최소 플러그인](entries/01-frontend-design.md) | ✅ | frontend-design |
| 02 | [Superpowers 해부 — 스스로 부팅하는 플러그인](entries/02-superpowers.md) | ✅ | superpowers |
| 03 | (예정) 내가 만들 플러그인 설계 결정 | ⬜ | — |
| 04 | (예정) MVP 구현 회고 | ⬜ | — |
| 05 | (예정) 배포 & marketplace 등록 | ⬜ | — |

## 학습 표본으로 고른 플러그인 (2026.6 기준 인기)

| 플러그인 | 유형 | 설치 수 | 왜 골랐나 |
|---|---|---|---|
| **Frontend Design** (Anthropic) | 순수 스킬형 | ~829K | 최소 플러그인의 뼈대. 파일 2개뿐 |
| **Superpowers** | 스킬+훅+스크립트 | ~752K | 스킬 아키텍처의 교과서. 자가 발동 메커니즘 |
| **Context7** | MCP형 | ~348K | (예정) 외부 데이터 주입 패턴 |

## 다음 할 일 (Next)

> 다른 컴퓨터에서 `git clone` 후 여기부터 이어가기.

- [ ] **TIL 01 / 02 내용 보충** — 파일만 분리해 둔 상태. 각 파일 하단 `TODO` 참고.
- [ ] **Week 2 — 만들 플러그인 아이디어 확정** (→ TIL 03)
  - 체크업로그(RN 앱) 만들며 겪은 **반복 작업 1개**를 고른다.
  - 후보: RN 컴포넌트 스캐폴더 / 디자인 토큰 가드(하드코딩 색상·spacing 차단) / 접근성 린터 / 스크린샷→컴포넌트
  - `superpowers:brainstorming` 스킬로 범위 좁히고, 설계 결정을 TIL 03에 기록.
- [ ] (선택) **Context7(MCP형) 해부 추가** → 스킬/훅/MCP 3대 유형 완성
- [ ] Week 3 — MVP 구현 (plugin.json + SKILL.md 최소 구성부터)
- [ ] Week 4 — marketplace.json 만들어 배포 & ClaudePluginHub 등록

### TIL 01·02에서 "훔쳐올 것" 체크리스트 (제작 시 참고)
- description에 트리거 조건을 자연어로 명시
- AI가 흔히 틀리는 걸 `NEVER ...`로 이름 지목
- 강제 순서엔 `<HARD-GATE>` 패턴
- 스킬 커지면 부트스트랩 + 점진 로딩으로 토큰 절약

## 참고

- [ClaudePluginHub](https://www.claudepluginhub.com/) — 플러그인 디렉토리
- [awesome-claude-plugins](https://github.com/ComposioHQ/awesome-claude-plugins) — 큐레이션 리스트
- [Superpowers (obra/superpowers)](https://github.com/obra/superpowers) — 해부 대상 소스
