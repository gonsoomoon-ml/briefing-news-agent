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

- **확정 출처 (최소 보장)**:
  - `https://www.aitimes.com/`
  - Anthropic 공식 블로그·뉴스
- **출처 풀 확장**: 추가 후보는 **Cycle 1A** (Strands + AgentCore 리서치) 결과로 결정.

### 2.4 비범위 (Out-of-scope)

- Non-AI 기술 일반 뉴스
- AI 정책·규제 (통증 발생 시 후속 사이클로 이관)
- 영상·팟캐스트 transcript (텍스트 출처만)
- Real-time push 채널 (Slack/Discord 등 — 이메일만)

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

---

## 5. 성공 기준 (MVP 합격 조건)

다음 4가지를 *모두* 충족 시 MVP 합격:

1. **정시 발송** — KST 07:00 ±15분 자동 발송, **7일 연속 무중단**
2. **분량 준수** — 매 발송 **5~10 기사** 포함 (§3 포맷 준수)
3. **사실 부합** — 요약이 원문 사실과 일치 (hallucination 없음 / 사용자 spot check)
4. **출처 다양성** — `aitimes.com` + Anthropic 두 출처 **모두 매일 ≥1개씩 포함**

---

## 6. 의존성

| 항목 | 상태 |
|---|---|
| 구체 기술 스택 (Strands 사용 패턴) | *open* — Cycle 1A 결과 |
| 출처 풀 확장 | *open* — Cycle 1A 결과 |
| AWS Bedrock + AgentCore | *fixed* — BRD `기술 스택` |
| 이메일 전송 메커니즘 | *open* — 별도 사이클 |

---

## 7. 변경 이력

| 일자 | 변경 |
|---|---|
| 2026-05-05 | v0.1 초안 — Cycle 1B brainstorming 결과 |
