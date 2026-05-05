# 0002 — Strands + AgentCore + Bedrock 뉴스 다이제스트 패턴 리서치

- **작성일**: 2026-05-05
- **작성 단계**: 1단계 (탐색/리서치)
- **사이클**: 1A
- **부모 문서**:
  - `design/specs/0001-mvp-spec.md` (MVP v0.1)
  - `design/research/0001-spec-driven-development.md` (방법론)
- **결정 기록**: `design/decisions/0001-adopt-strands-agentcore-from-base-repo.md` (ADR 0001)
- **결론(요약)**: Spec §6의 *open* 의존성 3개 + 신규 발견된 스케줄링 의존성을 모두 close. Strands+AgentCore stack은 베이스 리포 답습, 출처는 5개 큐레이션, 이메일은 AWS SES sandbox, 스케줄링은 EventBridge Scheduler universal target 직접 호출(2단계 패턴).

---

## 1. 질문 (Research Questions)

본 리서치는 `design/specs/0001-mvp-spec.md` §6의 3개 *open* 의존성을 1차 자료에 기반해 좁힌다:

1. **구체 기술 스택** — Strands SDK 사용 패턴 / AgentCore Runtime 호출 / 도구 구성
2. **출처 풀 확장** — `aitimes.com` + Anthropic 외 추가 후보
3. **이메일 전송 메커니즘** — AWS-native vs 외부

추가로 인벤토리 과정에서 발견된 4번째 의존성:

4. **스케줄링/트리거** — KST 07:00 매일 발화 메커니즘

---

## 2. 베이스 리포 인벤토리 — `gonsoomoon-ml/developer-briefing-agent`

`design/resource.md`가 "베이스 코드 (직접 차용)"로 명시한 리포. 사용자 본인 작성. 동일 도메인(에이전트 기반 일일 브리핑)의 검증된 패턴 보유.

### 2.1 답습 가능 (Reuse) — 95%

| 영역 | 베이스 리포의 답 |
|---|---|
| Strands SDK | `Agent(tools=[...])` + Bedrock `anthropic.claude-sonnet-4-6` |
| AgentCore Runtime | `@app.entrypoint` 데코레이터, local↔runtime 동일 로직 |
| Prompt Caching | 3-layer (tool defs / system prompt / turn boundaries), ~52% 비용 절감 |
| SlidingWindow | `SlidingWindowConversationManager(window_size=20)` |
| 프로젝트 구조 | `local-agent/`, `managed-agentcore/`, `shared/`, `prompts/`, `setup/` |
| SSE 스트리밍 | `agent.stream_async(prompt)` + `yield` 패턴 |

### 2.2 본 프로젝트 갭 (변경 필요)

| 영역 | 베이스 리포 | 본 프로젝트 |
|---|---|---|
| 데이터 소스 | GitHub REST API → JSON | RSS + HTML 스크래핑 (5개 출처) |
| 출력 채널 | 터미널 / SSE | AWS SES 이메일 |
| 트리거 | on-demand | EventBridge Scheduler cron (KST 07:00) |
| AgentCore Memory | 사용 (cross-session) | **미사용** — MVP 비목표 |
| 도구 리스트 | `[shell, file_read]` | RSS / HTTP / SES 발송 도구 (정확한 모듈명은 구현 시 `strands_tools` 확정) |

---

## 3. 출처 풀 (5개 최종)

| # | 출처 | 언어 | Fetch 방법 | 검증 |
|---|---|---|---|---|
| 1 | `aitimes.com` | 한국어 | RSS — `https://www.aitimes.com/rss/allArticle.xml` | ✅ 직접 fetch 검증 (RSS 2.0, 한국어 본문) |
| 2 | Anthropic news | 영어 | **HTML 스크래핑** | ⚠️ 공식 RSS 부재 — `https://www.anthropic.com/news` 직접 파싱 |
| 3 | AWS Machine Learning Blog | 영어 | RSS — `https://aws.amazon.com/blogs/machine-learning/feed/` | ✅ AWS 표준 패턴 |
| 4 | OpenAI blog | 영어 | RSS — `https://openai.com/blog/rss/` | ✅ 공식 RSS |
| 5 | Google DeepMind blog | 영어 | RSS — `https://deepmind.com/blog/feed/basic/` | ✅ 공식 RSS |

**제외된 후보**: ZDNet Korea AI 섹션 (`?lstcode=0050` URL 404). 통증 발생(=한국어 비즈니스 뉴스 부족) 시 다른 한국어 출처(Maeil경제 IT, 디지털타임스 등)로 swap 검토.

**Strands SDK 자체 RSS 도구**: `strands-agents/tools` 리포지토리에 RSS feed 처리 도구가 이미 존재 (path traversal 보안 이슈 fix 이력 있음). 직접 작성 불필요 — 구현 시 `strands_tools` 내 RSS 도구 활용.

---

## 4. 이메일 — AWS SES sandbox

### 4.1 결정

AWS SES sandbox 모드, 발신자·수신자 둘 다 verify.

### 4.2 셋업 단계 (구현 사이클로 이관)

1. SES에서 발신자 이메일 verify (예: `briefing-noreply@<custom-domain>` 또는 `gonsoomoon+briefing@gmail.com`)
2. SES에서 수신자 `gonsoomoon@gmail.com` verify
3. AgentCore Runtime IAM 역할에 `ses:SendEmail` 권한 부여
4. boto3 `boto3.client('ses').send_email(...)` 호출 (또는 Strands 도구로 래핑)

### 4.3 비용

Free tier: AWS 서비스 내부 발신 시 월 62,000건 무료. 1일 1건 발송은 *완전 무료*.

### 4.4 향후 확장 트리거

다중 수신 / 외부 가입자 추가 시 → production 모드 신청 필요 (며칠 소요).

---

## 5. 스케줄링 — EventBridge Scheduler Universal Target

### 5.1 결정

EventBridge Scheduler → AgentCore Runtime **직접** 호출 (2단계, Lambda 제거).

### 5.2 핵심 메커니즘

EventBridge Scheduler의 *universal target* 기능으로 AWS SDK API 직접 호출 가능.

ARN 패턴:

```
arn:aws:scheduler:::aws-sdk:bedrock-agentcore:invokeAgentRuntime
```

스케줄 표현 (KST 07:00 = UTC 22:00, 시간대 명시 가능):

```
cron(0 22 * * ? *)
```

또는 시간대 명시:

```
cron(0 7 * * ? *) — Asia/Seoul
```

### 5.3 호출 페이로드 (예시)

```json
{
  "AgentRuntimeArn": "arn:aws:bedrock-agentcore:us-west-2:<acct>:runtime/<id>",
  "RuntimeSessionId": "<unique-uuid-per-day>",
  "Payload": "<base64-encoded JSON: {\"prompt\": \"daily briefing\"}>"
}
```

### 5.4 IAM

EventBridge Scheduler의 실행 역할에 `bedrock-agentcore:InvokeAgentRuntime` 권한 부여 필요.

### 5.5 에러 핸들링

- **Dead-letter queue (DLQ)** — EventBridge Scheduler에 SQS DLQ 설정. 실패 이벤트 보존, 운영자가 검사.
- **Retry policy** — Scheduler는 기본 retry 정책 보유 (지수 백오프).
- **Streaming response** — AgentCore Runtime은 streaming response 반환. EventBridge Scheduler universal target은 *fire-and-forget* 패턴이므로 응답 본문은 무시되지만 invocation 성공/실패는 추적됨.

### 5.6 boto3 직접 호출 패턴 (참고용 — 로컬 테스트 시)

```python
import boto3, json, uuid

client = boto3.client('bedrock-agentcore')
response = client.invoke_agent_runtime(
    agentRuntimeArn=agent_arn,
    runtimeSessionId=str(uuid.uuid4()),
    payload=json.dumps({"prompt": "daily briefing"}).encode()
)
```

---

## 6. Spec §6 의존성 종결

| 항목 | 이전 | 현재 | 근거 |
|---|---|---|---|
| 구체 기술 스택 (Strands 사용 패턴) | *open* | **resolved** | §2.1 (베이스 리포 답습) + ADR 0001 |
| 출처 풀 확장 | *open* | **resolved** | §3 (5개 최종) |
| 이메일 전송 메커니즘 | *open* | **resolved** | §4 (AWS SES sandbox) |
| 스케줄링/트리거 (신규 발견) | (미명시) | **resolved** | §5 (EventBridge Scheduler universal target) |
| AgentCore Memory | (사용 가정) | **excluded from MVP** | §2.2 (1인 일일 푸시 불필요) |
| AWS Bedrock + AgentCore | *fixed* | unchanged | BRD `기술 스택` |

---

## 7. 출처 (Sources)

### Strands Agents SDK

- [Introducing Strands Agents — AWS Open Source Blog](https://aws.amazon.com/blogs/opensource/introducing-strands-agents-an-open-source-ai-agents-sdk/)
- [Strands Agents 공식 사이트](https://strandsagents.com/)
- [strands-agents/sdk-python (GitHub)](https://github.com/strands-agents/sdk-python)
- [strands-agents/tools releases (RSS 도구 포함)](https://github.com/strands-agents/tools/releases)
- [Amazon Bedrock model provider docs](https://strandsagents.com/docs/user-guide/concepts/model-providers/amazon-bedrock/)
- [Strands SDK 기술 deep dive — AWS ML Blog](https://aws.amazon.com/blogs/machine-learning/strands-agents-sdk-a-technical-deep-dive-into-agent-architectures-and-observability/)

### Bedrock AgentCore Runtime

- [Invoke an AgentCore Runtime agent — AWS docs](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-invoke-agent.html)
- [InvokeAgentRuntime API reference](https://docs.aws.amazon.com/bedrock-agentcore/latest/APIReference/API_InvokeAgentRuntime.html)
- [boto3 invoke_agent_runtime](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/bedrock-agentcore/client/invoke_agent_runtime.html)
- [AgentCore Runtime Quickstart (starter toolkit)](https://aws.github.io/bedrock-agentcore-starter-toolkit/user-guide/runtime/quickstart.html)

### EventBridge Scheduler

- [Universal targets in EventBridge Scheduler](https://docs.aws.amazon.com/scheduler/latest/UserGuide/managing-targets-universal.html)
- [EventBridge Scheduler new universal targets including Bedrock](https://aws-news.com/article/0190557f-c98d-620a-a22e-863702e75e78)

### 출처 RSS Feeds

- [aitimes.com RSS](https://www.aitimes.com/rss/allArticle.xml)
- [AWS Machine Learning Blog feed](https://aws.amazon.com/blogs/machine-learning/feed/)
- [OpenAI blog RSS](https://openai.com/blog/rss/)
- [DeepMind blog RSS](https://deepmind.com/blog/feed/basic/)
- [Anthropic news (RSS 부재 — 스크래핑)](https://www.anthropic.com/news)

### 베이스 리포

- [gonsoomoon-ml/developer-briefing-agent](https://github.com/gonsoomoon-ml/developer-briefing-agent)
- [awslabs/agentcore-samples](https://github.com/awslabs/agentcore-samples)

---

## 8. 변경 이력

| 일자 | 변경 |
|---|---|
| 2026-05-05 | 최초 작성 — Cycle 1A 1단계 산출물. 의존성 4개 모두 close. ADR 0001 동반 발행. |
