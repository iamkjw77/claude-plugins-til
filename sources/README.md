# sources/ — 해부 대상 원문 (제3자 소스)

이 디렉토리는 TIL에서 **역설계·인용한 플러그인 원문 파일**을 보기 편하게 모아둔 것이다.
전체 레포가 아니라, 각 TIL에서 실제로 인용·분석한 **핵심 파일만** 발췌했다.

> ⚠️ 여기 있는 코드는 **내가 작성한 것이 아니다.** 각 플러그인 저작자의 것이며,
> 학습·인용 목적으로 라이선스 조건을 지켜 포함했다.

## 출처 및 라이선스

| 폴더 | 플러그인 | 저작자 | 라이선스 | 원본 |
|---|---|---|---|---|
| `frontend-design/` | Frontend Design | Anthropic | Apache-2.0 | `anthropics/claude-plugins-official` |
| `superpowers/` | Superpowers | Jesse Vincent | MIT | https://github.com/obra/superpowers |

두 라이선스(Apache-2.0, MIT) 모두 라이선스 고지·저작자 표시를 유지하면 재배포를 허용한다.
각 폴더의 `LICENSE` 파일을 원문 그대로 함께 보관했다.

## 포함한 파일과 TIL 연결

### `frontend-design/` — [TIL 01](../entries/01-frontend-design.md)
- `plugin.json` — 7줄짜리 메타데이터
- `skills/frontend-design/SKILL.md` — 프롬프트 본체(트리거 `description` + `NEVER ...` 규칙)

### `superpowers/` — [TIL 02](../entries/02-superpowers.md)
- `plugin.json`, `marketplace.json` — 메타데이터
- `hooks/hooks.json` — `SessionStart` 훅 등록
- `hooks/session-start` — **자가 발동 스크립트** (using-superpowers를 컨텍스트에 주입)
- `skills/using-superpowers/SKILL.md` — 훅이 주입하는 부트스트랩 스킬
- `skills/brainstorming/SKILL.md` — `<HARD-GATE>` 패턴 예시

## 버전 스냅샷

- frontend-design: 로컬 설치본(version `unknown`)
- superpowers: `6.1.1`

로컬 설치본에서 발췌했으므로, 최신 원본과 다를 수 있다. 최신은 위 "원본" 링크 참고.
