# CLAUDE.md

## Workflow
- 모든 작업은 4단계 사이클로 진행: 탐색/리서치 → 계획 → 구현 → 검증
- 단계 전환 시 사용자 승인 필수 (1→2, 2→3, 4 완료)
- 메서돌로지 척추: Superpowers 14 스킬 (외부 SDD 프레임워크 미채택).
  - 단계별 일차 스킬: 1=brainstorming / 2=writing-plans / 3=executing-plans + test-driven-development / 4=verification-before-completion
  - 그 외(subagent-driven-development, code-reviewer, systematic-debugging, using-git-worktrees 등)도 필요 시 호출.
- 한 사이클 = 한 결정/기능 단위. 여러 결정을 묶지 않는다.
- 점진적 확장 원칙: MVP 후 작은 사이클 반복. YAGNI 준수.
- 메서돌로지 형식화도 통증 기반: 추가 표기법·템플릿·외부 SDD 도구는 *실제 통증 발생 시* 한 가지씩 도입한다. 사전 박제 금지. 후보 풀은 `design/research/0001-spec-driven-development.md` 참조.

## Review Cadence
- 검토할 항목이 여러 개일 때, 터미널에 한 번에 하나씩 제시한다.
- 각 항목에 대해 사용자의 승인을 받으면 다음 항목으로 넘어간다.
- 승인 없이 다음 항목으로 진행하지 않는다.

## Language
- 사용자는 영어로 입력하지만, 답변은 한국어 친화적으로 작성한다 (자연스러운 한국어, 필요한 기술 용어는 영어 병기)
- 코드 주석/식별자는 영어

## Memory
- 중요한 마일스톤이 완료되면 memory에 기록한다 (project type 메모리).
- 마일스톤 예시: 사이클 완료(1~4단계 한 바퀴), MVP 출시, 주요 기능 추가, 아키텍처 결정.
- 기록 내용: 무엇이 / 언제(절대 날짜) / 왜 중요한지 / 다음에 어떤 영향을 주는지.

## Research Discipline
- 새로운 라이브러리·API·패턴 사용 전, 1차 자료(공식 문서, 블로그, 논문)를 먼저 확인한다.
- 활용 도구: WebSearch, WebFetch, context7 MCP (`mcp__plugin_context7_context7__query-docs`).
- 리서치 결과는 `design/research/NNNN-topic.md` 형식으로 보존한다 (출처 URL 포함).
- 출처 없이 추측으로 코드 작성 금지. 모르면 "모른다"고 말하고 리서치한다.

## Project Structure
- `design/` — BRD, 리서치 노트, 스펙, 계획 문서
  - `design/research/` — 리서치 노트
  - `design/specs/NNNN-title.md` — 릴리즈/MVP 스펙 (보존)
  - `design/plans/NNNN-title.md` — 사이클별 구현 계획서 (보존)
- `src/` — 애플리케이션 코드
- `tests/` — 테스트 코드
- `infra/` — AWS 인프라 정의 (도구는 첫 인프라 사이클에서 결정)
- 새 폴더 추가 시 이 섹션을 갱신한다.

## Secrets & AWS Credentials
- 자격증명·API 키를 코드에 하드코딩 금지. 환경변수 또는 AWS Secrets Manager만 사용.
- `.env`, `*.pem`, `credentials.json` 등은 커밋 금지. `.gitignore`에 포함되어 있는지 매번 확인.
- 로그·예제·커밋 메시지·문서에 자격증명이 노출되지 않는지 점검 후 커밋.
- AWS 리전·계정 ID 등 민감하지 않은 설정도 가능하면 환경변수로 분리한다.
