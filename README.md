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
| 01 | [플러그인 해부 — 최소 뼈대부터 자가 발동까지](entries/01-plugin-anatomy.md) | ✅ | frontend-design, superpowers |
| 02 | (예정) 내가 만들 플러그인 설계 결정 | ⬜ | — |
| 03 | (예정) MVP 구현 회고 | ⬜ | — |
| 04 | (예정) 배포 & marketplace 등록 | ⬜ | — |

## 학습 표본으로 고른 플러그인 (2026.6 기준 인기)

| 플러그인 | 유형 | 설치 수 | 왜 골랐나 |
|---|---|---|---|
| **Frontend Design** (Anthropic) | 순수 스킬형 | ~829K | 최소 플러그인의 뼈대. 파일 2개뿐 |
| **Superpowers** | 스킬+훅+스크립트 | ~752K | 스킬 아키텍처의 교과서. 자가 발동 메커니즘 |
| **Context7** | MCP형 | ~348K | (예정) 외부 데이터 주입 패턴 |

## 참고

- [ClaudePluginHub](https://www.claudepluginhub.com/) — 플러그인 디렉토리
- [awesome-claude-plugins](https://github.com/ComposioHQ/awesome-claude-plugins) — 큐레이션 리스트
- [Superpowers (obra/superpowers)](https://github.com/obra/superpowers) — 해부 대상 소스
