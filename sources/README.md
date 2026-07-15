# sources/ — 해부 대상 원문 (제3자 소스)

이 디렉토리는 TIL에서 역설계·분석하는 플러그인의 **전체 레포 스냅샷**이다.
로컬 설치본을 통째로 복사해 두었고, VCS·런타임 파일(`.git`, `.in_use`)만 제외했다.

> ⚠️ 여기 있는 코드는 **내가 작성한 것이 아니다.** 각 플러그인 저작자의 것이며,
> 학습·인용 목적으로 라이선스 조건을 지켜 원문 그대로 포함했다. **편집·개선 금지.**

## 출처 및 라이선스

| 폴더 | 플러그인 | 저작자 | 라이선스 | 원본 |
|---|---|---|---|---|
| `frontend-design/` | Frontend Design | Anthropic | Apache-2.0 | `anthropics/claude-plugins-official` |
| `superpowers/` | Superpowers | Jesse Vincent | MIT | https://github.com/obra/superpowers |

두 라이선스(Apache-2.0, MIT) 모두 라이선스 고지·저작자 표시를 유지하면 재배포를 허용한다.
각 폴더의 `LICENSE` 파일을 원문 그대로 함께 보관했다.

## TIL 연결 & 눈여겨볼 파일

전체 레포가 다 들어있지만, 각 TIL이 실제로 인용·분석한 핵심 파일은 아래와 같다.

### `frontend-design/` — [TIL 01](../entries/01-frontend-design.md)
파일 4개짜리 최소 플러그인. 사실상 전부가 핵심이다.
- `.claude-plugin/plugin.json` — 메타데이터
- `skills/frontend-design/SKILL.md` — 프롬프트 본체(트리거 `description` + `NEVER ...` 규칙)

### `superpowers/` — [TIL 02](../entries/02-superpowers.md)
스킬 14개 + 훅 + 스크립트 + 테스트 + 크로스플랫폼 설정이 모두 포함된 풀세트.
- `.claude-plugin/{plugin.json, marketplace.json}` — 메타데이터
- `hooks/hooks.json` — `SessionStart` 훅 등록
- `hooks/session-start` — **자가 발동 스크립트** (using-superpowers를 컨텍스트에 주입)
- `hooks/run-hook.cmd` — 크로스플랫폼 실행 래퍼
- `skills/using-superpowers/SKILL.md` — 훅이 주입하는 부트스트랩 스킬
- `skills/brainstorming/SKILL.md` — `<HARD-GATE>` 패턴 예시
- `skills/` 나머지 12개 — TDD, systematic-debugging, writing-plans 등 (아직 미분석)

## 버전 스냅샷

- frontend-design: 로컬 설치본 (version `unknown`)
- superpowers: `6.1.1`

로컬 설치본에서 복사했으므로 최신 원본과 다를 수 있다. 최신은 위 "원본" 링크 참고.
