# ADR 0001 — Adopt Strands + AgentCore stack from `gonsoomoon-ml/developer-briefing-agent`

- **Date**: 2026-05-05
- **Status**: Accepted
- **Cycle**: 1A
- **Related**: `design/research/0002-strands-agentcore-news-pattern.md`
- **Affects**: `design/specs/0001-mvp-spec.md` §2 / §6

---

## Context

본 프로젝트(`briefing-news-agent`)는 일일 뉴스 브리핑을 이메일로 푸시하는 에이전트. BRD가 명시한 기술 스택은 Amazon Bedrock + Strands Agent SDK + Amazon Bedrock AgentCore.

사용자 본인(`gonsoomoon-ml`)이 동일 도메인(에이전트 기반 일일 브리핑)의 검증된 리포 `developer-briefing-agent`를 보유. `design/resource.md`에 "베이스 코드 (직접 차용)"로 명시.

본 ADR은 Cycle 1A에서 *처음으로 발생한* 진짜 architectural decision으로, `CLAUDE.md`의 "통증 기반 형식화" 룰에 따라 ADR 폴더(`design/decisions/`)를 신설하는 트리거가 된다.

---

## Decision

`gonsoomoon-ml/developer-briefing-agent` 리포의 기술 스택과 아키텍처 패턴을 *답습*한다.

**그대로 사용**:

- **에이전트 프레임워크**: Strands `Agent(tools=[...])` 패턴, Bedrock `anthropic.claude-sonnet-4-6` 모델
- **런타임**: AgentCore Runtime + `@app.entrypoint` 데코레이터, local↔runtime 동일 로직
- **Prompt Caching**: 3-layer (tool defs / system prompt / turn boundaries) — ~52% 비용 절감
- **대화 관리**: `SlidingWindowConversationManager(window_size=20)`
- **프로젝트 구조**: `local-agent/`, `managed-agentcore/`, `shared/`, `prompts/`, `setup/`

**본 프로젝트에 맞게 변경**:

- **데이터 소스**: GitHub REST API → 5개 뉴스 출처 (RSS + HTML 스크래핑)
- **도구 리스트**: `[shell, file_read]` → RSS 페치 / HTTP / SES 발송 도구 (정확한 `strands_tools` 모듈명은 구현 시 확정)
- **출력 채널**: 터미널 / SSE → AWS SES 이메일
- **트리거**: on-demand → EventBridge Scheduler cron (KST 07:00)
- **AgentCore Memory**: 사용 → **미사용** (MVP 비목표, 1인 단발성 푸시)

---

## Consequences

**긍정**:

- 학습 비용 최소 — 검증된 패턴이라 "동작하지 않을 위험" 거의 없음.
- 일관성 — 사용자 본인의 다른 에이전트 프로젝트와 동일한 멘탈 모델.
- 시간 절약 — 아키텍처 결정에 추가 사이클 불필요.
- Prompt caching · SlidingWindow 등 운영 최적화 즉시 확보.

**부정 / 제약**:

- 베이스 리포의 의사결정에 종속 — 베이스 리포가 잘못된 선택을 했으면 본 프로젝트도 영향.
- 베이스 리포 업데이트 시 cherry-pick 또는 동기화 작업 필요 (의식적 운영 부담).
- 베이스 리포의 "developer-briefing"이라는 도메인 어휘가 *코드 식별자*에 잔재할 수 있음 (예: `dev_name` 변수 → `topic` 등으로 rename 필요).

---

## Alternatives Considered

| 대안 | 거부 사유 |
|---|---|
| **From-scratch (Strands + AgentCore 직접 조립)** | 학습 비용·시간 대비 가치 낮음 — 베이스 리포가 이미 답을 가짐. |
| **Spec-Kit 기반 코드 자동 생성** | spec → code 변환은 Strands 같은 멀티 도구·LLM 에이전트엔 적합도 떨어짐. (`design/research/0001-spec-driven-development.md` §3 참조) |
| **Kiro IDE로 전환** | 본 프로젝트는 Claude Code 환경. IDE 전환 비용 > 편익. (`design/research/0001-spec-driven-development.md` §1 참조) |
| **LangGraph로 변경** | BRD가 Strands SDK를 명시. 변경 시 BRD 수정 + 학습 비용 양쪽 발생. |

---

## Validation

본 결정의 검증은 **MVP 운영**으로 한다 — `design/specs/0001-mvp-spec.md` §5 성공 기준 4개를 7일 연속 충족 시 본 ADR의 답습 결정이 유효함을 확인.

만약 검증 실패 시 (예: AgentCore가 Strands SDK 기능을 제대로 지원하지 않음, prompt caching이 작동하지 않음), 별도 ADR 0002+로 *재결정*한다.
