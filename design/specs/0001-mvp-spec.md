# 0001 — MVP Spec: News Briefing Agent (v0.1)

- **사이클**: 1B (BRD 갭 보강)
- **작성일**: 2026-05-05
- **상태**: Draft
- **부모 문서**: `design/biz_requirement.md` (BRD)
- **관련 리서치**: `design/research/0001-spec-driven-development.md`

---

## 1. 이해관계자

| 역할 | 주체 |
|---|---|
| 소유자 / 유일 사용자 (MVP) | Gonsoo Moon (`gonsoomoon@gmail.com`) |
| 운영자 | 동일 |
| AWS 계정 보유자 | 동일 |

**비목표 (Out-of-scope)**: 외부 구독 폼, 구독 해지 흐름, 다중 수신자 관리, 인증 시스템.

---

## 2. MVP 범위

### 2.1 주제 (In-scope)

- LLM / Foundation Model 동향 — 모델 릴리즈, 논문, 벤치마크
- AI 인프라·프레임워크 — Bedrock, NVIDIA, LangChain/LangGraph, Strands, AgentCore 등
- 산업·비즈니스 — AI 관련 자금조달·M&A
- AWS Bedrock + AgentCore + Strands + Agent 프레임워크 동향 — 해당 영역 *main news*

### 2.2 언어

- Korean-friendly 우선. 한국어 출처 가산점 / 영어 출처는 한국어로 요약.

### 2.3 출처

MVP 5개 출처 (Cycle 1A 결정 — 자세한 fetch 방법은 `design/research/0002-strands-agentcore-news-pattern.md` §3):

| # | 출처 | 언어 | Fetch |
|---|---|---|---|
| 1 | `aitimes.com` | 한국어 | RSS |
| 2 | Anthropic news (`anthropic.com/news`) | 영어 | HTML 스크래핑 (공식 RSS 부재) |
| 3 | AWS Machine Learning Blog | 영어 | RSS |
| 4 | OpenAI blog | 영어 | RSS |
| 5 | Google DeepMind blog | 영어 | RSS |

확장 트리거: 5개 운영 시 분량 부족 통증 발생 시 후속 사이클로 추가.

### 2.4 비범위 (Out-of-scope)

- Non-AI 기술 일반 뉴스
- AI 정책·규제 (통증 발생 시 후속 사이클로 이관)
- 영상·팟캐스트 transcript (텍스트 출처만)
- Real-time push 채널 (Slack/Discord 등 — 이메일만)
- AgentCore Memory cross-session — 1인 단발성 푸시라 불필요 (Cycle 1A 결정)

---

## 3. 출력 포맷

### 3.1 분량

- 기사 수: **5~10개** (range)
- 기사당 1문단(3~5문장) 요약
- 약 10~15분 읽기

### 3.2 이메일 본문 구조

```
[헤드라인 1]
카테고리: <LLM | Infra | Biz | Bedrock-AgentCore-Strands>
요약: <1문단, 3~5문장>
원문: <URL>

[헤드라인 2]
...
```

---

## 4. 스케줄

| 항목 | 값 |
|---|---|
| 발송 시각 | 매일 **KST 07:00 ±15분** |
| 입력 윈도우 | 직전 24시간 (전날 KST 07:00 ~ 당일 KST 07:00) |
| 트리거 | EventBridge Scheduler universal target → AgentCore Runtime 직접 invoke (2단계, Cycle 1A 결정) |

---

## 5. 성공 기준 (MVP 합격 조건)

다음 4가지를 *모두* 충족 시 MVP 합격:

1. **정시 발송** — KST 07:00 ±15분 자동 발송, **7일 연속 무중단**
2. **분량 준수** — 매 발송 **5~10 기사** 포함 (§3 포맷 준수)
3. **사실 부합** — 요약이 원문 사실과 일치 (hallucination 없음 / 사용자 spot check)
4. **출처 다양성** — `aitimes.com` + Anthropic 두 출처 **모두 매일 ≥1개씩 포함**

---

## 6. 의존성

| 항목 | 상태 | 근거 |
|---|---|---|
| 구체 기술 스택 (Strands 사용 패턴) | **resolved** | ADR 0001 + research 0002 |
| 출처 풀 (5개) | **resolved** | research 0002 §3 |
| 이메일 전송 메커니즘 | **resolved** | research 0002 §4 (AWS SES sandbox) |
| 스케줄링 / 트리거 | **resolved** | research 0002 §5 (EventBridge Scheduler universal target) |
| AWS Bedrock + AgentCore | *fixed* | BRD `기술 스택` |

---

## 7. 변경 이력

| 일자 | 변경 |
|---|---|
| 2026-05-05 | v0.1 초안 — Cycle 1B brainstorming 결과 |
| 2026-05-05 | v0.2 — Cycle 1A 반영: §2.3 5개 출처, §2.4 AgentCore Memory 비목표, §4 트리거 명시, §6 4개 의존성 resolved |
