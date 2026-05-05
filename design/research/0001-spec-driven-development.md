# 0001 — Spec-Driven Development: 산업 표준 + 오픈소스 풀 리서치

- **작성일**: 2026-05-05
- **작성 단계**: 1단계 (탐색/리서치)
- **목적**: 본 프로젝트(`briefing-news-agent`)의 SDD 방법론을 *근거 기반*으로 결정하기 위해, 산업 표준 표기법과 오픈소스 프레임워크 풀을 매핑한다.
- **결론(요약)**: **Superpowers를 척추(spine)로 두고, EARS 표기법(요구사항)과 ADR(아키텍처 결정)을 산출물 표준으로 추가 도입한다.** 외부 SDD 프레임워크(Spec-Kit/Kiro/cc-sdd/BMAD/OpenSpec)는 *고유 가치만 부분 차용*하고 도구 자체는 도입하지 않는다.

---

## 1. 질문 (Research Questions)

1. SDD에 *진짜* 산업 표준이 있는가? 있다면 무엇인가?
2. 2026년 현재 가장 많이 쓰이는 오픈소스 SDD 프레임워크는 무엇이며, 어떻게 다른가?
3. Claude Code 네이티브 `/plan` 모드는 외부 도구와 어떤 관계인가?
4. 이미 사용 중인 superpowers는 이 풀에서 어디에 위치하는가?
5. 본 프로젝트(점진적 확장, 1인, AWS Bedrock+AgentCore+Strands 기반)에 가장 합리적인 조합은?

---

## 2. 핵심 발견 — SDD는 *두 개의 레이어*

```
┌─────────────────────────────────────────────────────────────┐
│ Layer A — 표기법 표준 (File-format standards, 도구 독립)        │
│   EARS (2009)  ADR (2011)  Gherkin/BDD (2003)  C4/arc42      │
│   → 어떤 프레임워크를 쓰든 산출물 안에 박을 수 있음              │
└─────────────────────────────────────────────────────────────┘
            ▲ (위에 얹음)
┌─────────────────────────────────────────────────────────────┐
│ Layer B — 프레임워크 표준 (AI-era SDD toolchains, 2025~2026)  │
│   superpowers  Spec-Kit  Kiro  OpenSpec  BMAD-METHOD  cc-sdd │
│   → 워크플로우·슬래시 명령·산출물 강제                            │
└─────────────────────────────────────────────────────────────┘
```

**시사**: 표기법은 *수십 년* 검증된 산업 표준이고, 프레임워크는 *AI 시대에 갓 등장*한 신생 도구 모음이다. 둘은 직교(orthogonal)하므로 *한 가지를 고르고 다른 한 가지를 포기*할 필요가 없다.

---

## 3. Layer A — 표기법 산업 표준

### 3.1 EARS (Easy Approach to Requirements Syntax)

| 항목 | 내용 |
|---|---|
| 출처 | Mavin, A., Wilkinson, P., Harwood, A., Novak, M. — Rolls-Royce PLC |
| 발표 | IEEE Requirements Engineering Conference 2009 (RE'09) |
| 채택 기업 | **Airbus, Bosch, Dyson, Honeywell, Intel, NASA, Rolls-Royce, Siemens** |
| 라이선스 | 공개 표준 (논문 기반) |

**5형식**:
- **Ubiquitous**: "The system **shall** \<system response\>"
- **Event-driven**: "**When** \<trigger\>, the system shall \<response\>"
- **State-driven**: "**While** \<state\>, the system shall \<response\>"
- **Optional**: "**Where** \<feature included\>, the system shall \<response\>"
- **Unwanted**: "**If** \<undesired condition\>, then the system shall \<response\>"

**해결하는 8가지 모호성**: ambiguity, complexity, vagueness, redundancy, untestability, omission, scope, level mixing.

**본 프로젝트 적용**: BRD(`design/biz_requirement.md`)의 빈 섹션과 향후 research 노트의 요구사항 절은 EARS 5형식으로 작성. 학습 비용 1쪽 이내. 모호성 제거 ROI 매우 높음.

### 3.2 ADR (Architecture Decision Record)

| 항목 | 내용 |
|---|---|
| 제창 | Michael Nygard, 2011 |
| 표준 | `adr.github.io` |
| 핵심 양식 | Title / Status / Context / Decision / Consequences |
| 2026 트렌드 | **ADR + SDD 결합** — "ADR이 *why*, Spec이 *what*". 결정은 spec과 분리 보존하여 미래 변경 제안이 *현재 아키텍처를 읽고 설계* 가능 (intent-driven.dev, 2026-04) |

**본 프로젝트 적용**: `design/decisions/NNNN-title.md` 폴더 신설(차기 사이클). 첫 ADR 후보:
- `0001` — Strands SDK over LangGraph (이유 / 트레이드오프 / 결과)
- `0002` — AgentCore vs ECS for runtime
- `0003` — Bedrock 모델 선택 (Claude Sonnet/Haiku, Amazon Nova)
- `0004` — 상태 저장소 선택 (DynamoDB / Aurora / 없음)

**plan 파일과의 분리 이유**: plan은 *일시적*(사이클 종료 시 의미 소멸), ADR은 *영구*(아키텍처 변경 시까지 유효). 둘을 섞으면 plan이 비대해지고 ADR이 매몰된다.

### 3.3 Gherkin / BDD (Behavior-Driven Development)

| 항목 | 내용 |
|---|---|
| 제창 | Dan North, 2003 |
| 도구화 | Cucumber (Aslak Hellesøy) |
| 양식 | Given / When / Then |
| 시장 규모 | $120M (2024) → $300M (2033) 전망 — *living* |

**적합한 사용처**: 에이전트 행동 계약(예: "Given 토픽 X / When 새 뉴스 도착 / Then 요약 포맷 Y").

**본 프로젝트 보류 이유**: MVP 단계에 과도. 에이전트 행동이 *복잡해진 후* (acceptance test가 필요한 시점) 재검토.

### 3.4 (참고) C4 Model, arc42

아키텍처 다이어그램·문서 표준. 본 프로젝트 1인 규모엔 과함. 미래 (다인 협업 시점) 참조.

---

## 4. Layer B — 오픈소스 프레임워크 풀 (6강)

### 4.1 비교 테이블

| 프레임워크 | 저장소 | Stars (2026) | 형태 | 핵심 차별점 | 본 프로젝트 적합도 |
|---|---|---|---|---|---|
| **superpowers** | `obra/superpowers` | **121k+** (4월) | Skills 기반 플러그인 | 14 스킬 — TDD/검증/리뷰/디버깅/스킬작성 디시플린 강제. **Anthropic 공식 마켓플레이스 등록** (2026-01-15) | ★★★★★ **이미 사용 중** |
| **BMAD-METHOD** | `bmad-code-org/BMAD-METHOD` | 19~37k | 멀티 에이전트 프레임워크 | **12+ 에이전트 페르소나** (PM/Architect/Dev/UX/QA/Tech Writer). Party Mode. Scale-adaptive | ★★★ 페르소나 인사이트만 차용 |
| **GitHub Spec-Kit** | `github/spec-kit` | (공식) | Python CLI | **Constitution-first** + 8 슬래시 명령 (`constitution`/`specify`/`clarify`/`plan`/`tasks`/`analyze`/`checklist`/`implement`) + 14+ 에이전트 호환 | ★★★ 산출물 분리 패턴 차용 |
| **OpenSpec** | `Fission-AI/OpenSpec` | (성장 중) | 경량 마크다운 | API key/MCP 불필요. **Propose → Apply → Archive**. 30+ 도구 지원 | ★★★ 가장 가벼움, 차선 후보 |
| **cc-sdd** v3.0.2 | `gotalab/cc-sdd` | (신생, 2026-04-13) | Skills 번들 | **Discovery 라우팅** + 자율 구현 + per-task review. 8 에이전트 호환 | ★★★ Discovery 패턴만 인사이트 |
| **Kiro** | `kirodotdev/Kiro` (Amazon) | (proprietary) | VS Code 포크 IDE | **EARS** + Agent Hooks + Multi-model routing (Claude+Nova via Bedrock) | ★★ IDE 전환 비용 큼 |
| (참고) PromptX | — | — | — | (BMAD/Spec-Kit/OpenSpec과 함께 4강) | 미조사 |

### 4.2 주요 프레임워크 산출물 매핑

| 프레임워크 | 산출물 파일 |
|---|---|
| **superpowers** | (writing-plans 호출 시) `design/plans/NNNN-*.md` (포맷 자유) |
| **Spec-Kit** | `.specify/memory/constitution.md`, `specs/[FEATURE]/spec.md`, `plan.md`, `data-model.md`, `contracts/`, `tasks.md` |
| **Kiro** | `requirements.md` (EARS), `design.md`, `tasks.md` |
| **OpenSpec** | (Propose 단계 spec 문서, Archive 시 보존) |
| **cc-sdd** | `brief.md`, `roadmap.md`, `requirements.md`, `design.md`, `tasks.md` |
| **BMAD-METHOD** | (페르소나별 워크플로우 산출물) |

**관찰**: 4개 도구가 `requirements.md` / `design.md` / `tasks.md` 3-파일 구조에 *수렴*. 이는 Kiro에서 시작했지만 *de facto* 표준이 됨.

### 4.3 Claude Code 네이티브 `/plan` 모드의 위치

- **invocation**: Shift+Tab 두 번 (또는 `/plan` 슬래시)
- **차단**: 파일 편집, 상태 변경 셸 명령, 커밋 — 승인 전까지 모두
- **세션 미보존**: Claude Code 재시작 시 OFF
- **2026 best practice (anyonebuilds, claudedirectory)**: "plan을 *contract*로 다룰 것" / "Plan Mode + worktrees + subagents 트리오가 2026 고품질 작업의 토대"

**관계**: `/plan`은 SDD 프레임워크와 **경쟁이 아니라 primitive**. 모든 프레임워크가 plan mode 위에 *얹혀서* 더 강력해진다.

---

## 5. 본 프로젝트와의 정합성 분석

### 5.1 현재 상태 (2026-05-05 기준)

- 베이스라인 커밋 `ccba71d` (2026-05-04) — `CLAUDE.md`, `design/biz_requirement.md`, `design/resource.md`
- CLAUDE.md가 4단계(탐색/리서치 → 계획 → 구현 → 검증)를 명시 — 이는 사실상 superpowers 스킬 매핑
- `src/`, `tests/`, `infra/` 미생성 — 첫 사이클 진입 전
- 사용자 프로필: Korean, AWS Bedrock+AgentCore+Strands 실무자 (`gonsoomoon-ml/developer-briefing-agent` 등 보유)

### 5.2 superpowers ↔ 4단계 매핑 (이미 정합)

| 단계 | 일차 스킬 | 보조 스킬 |
|---|---|---|
| 1. 탐색/리서치 | `brainstorming` | `using-git-worktrees`, `systematic-debugging` |
| 2. 계획 | `writing-plans` | (네이티브 `/plan` 게이트 병용) |
| 3. 구현 | `executing-plans`, `test-driven-development` | `subagent-driven-development`, `dispatching-parallel-agents` |
| 4. 검증 | `verification-before-completion` | `requesting-code-review`, `receiving-code-review`, `finishing-a-development-branch` |
| 메타 | `writing-skills` | `using-superpowers` |

### 5.3 현재 갭 (=다른 프레임워크가 채워줄 수 있는 부분)

| 갭 | 채울 수단 | 도입 비용 |
|---|---|---|
| 요구사항 모호성 | **EARS 표기법** (Layer A 차용) | 30분 |
| "왜 이 결정인가" 보존 | **ADR 폴더** (Layer A 차용) | 30분 |
| Plan 파일 구조 자유도 → 일관성 부족 | **Kiro의 R/D/T 3섹션** 차용 | 15분 |
| 편집 차단 강제력 | **네이티브 `/plan` 모드** 병용 | 0 (사용자 습관) |
| 멀티 에이전트 협업 | Claude Code 서브에이전트 (이미 보유, BMAD 페르소나 사고법만 차용) | 0 |
| 산출물 자동 동기화 (living spec) | (보류 — 통증 본 후) | — |

---

## 6. 권장 (Recommendation)

### 6.1 채택 (Adopt) — 즉시

1. **Superpowers를 *명시적* 척추로 못 박는다.** CLAUDE.md에 "이 프로젝트의 4단계는 superpowers 스킬을 호출한다"를 한 줄 명시.
2. **EARS 표기법을 BRD와 research 노트의 *요구사항* 절에 적용한다.**
3. **ADR 폴더(`design/decisions/NNNN-*.md`)를 신설한다.** Michael Nygard 표준 형식 사용.
4. **Plan 파일에 R/D/T 3섹션(Requirements/Design/Tasks)을 의무화한다.**
5. **네이티브 `/plan` 모드를 writing-plans 호출 시 병용한다.**

### 6.2 보류 (Defer) — 통증 데이터 수집 후 재검토

- Spec-Kit 채택 — CLAUDE.md가 이미 constitution 역할
- BMAD-METHOD 채택 — 1인 프로젝트엔 페르소나 분리 과함
- OpenSpec 채택 — superpowers와 기능 중복
- cc-sdd 채택 — 신생, 첫 사이클 통증 본 후
- Kiro 전환 — IDE 전환 비용 > 편익
- BDD/Gherkin — 에이전트 행동 복잡해진 후
- Living-spec 자동 동기화 — 통증 미발생

### 6.3 도입하지 않음 (Reject)

- API-first 도구 (OpenAPI/Smithy/Fern/Stainless/Speakeasy) — 본 프로젝트는 API 소비자, 공급자 아님

---

## 7. 다음 단계 (Open Questions for Cycle 1)

다음 사이클 진입 시 결정해야 할 사항:

- **Q1.** 첫 실 사이클 주제는 (a) Strands+AgentCore 뉴스 다이제스트 패턴 리서치인가, (b) BRD 갭 보강(이해관계자/범위/성공기준)인가?
- **Q2.** EARS 도입을 (β′) 작은 한 발로 시범할 것인가, (α′) 메타 사이클로 박제할 것인가, (γ) 첫 실 사이클 진행 중 자연스럽게 흡수할 것인가?
- **Q3.** ADR 폴더 도입 시점 — 첫 ADR(Strands SDK 선택 등)이 자연스럽게 발생할 때인가, 사전 박제인가?

---

## 8. 출처 (Sources)

### EARS
- [Alistair Mavin EARS 공식 가이드](https://alistairmavin.com/ears/)
- [EARS PDF (Mavin & Wilkinson, 2009)](https://ccy05327.github.io/SDD/08-PDF/Easy%20Approach%20to%20Requirements%20Syntax%20(EARS).pdf)
- [EARS — IEEE Xplore (RE'09)](https://ieeexplore.ieee.org/document/5328509)
- [FAQ about EARS Notation — Jama Software](https://www.jamasoftware.com/requirements-management-guide/writing-requirements/frequently-asked-questions-about-the-ears-notation-and-jama-connect-requirements-advisor/)

### ADR
- [adr.github.io 공식](https://adr.github.io/)
- [Martin Fowler — Architecture Decision Record](https://martinfowler.com/bliki/ArchitectureDecisionRecord.html)
- [joelparkerhenderson/architecture-decision-record — templates](https://github.com/joelparkerhenderson/architecture-decision-record)
- [ADR + Spec-Driven 결합 (intent-driven.dev, 2026-04)](https://intent-driven.dev/blog/2026/04/29/spec-driven-development-with-adr/)
- [Microsoft Azure Well-Architected — Maintain an ADR](https://learn.microsoft.com/en-us/azure/well-architected/architect-role/architecture-decision-record)

### BDD / Gherkin
- [Cucumber BDD 공식](https://cucumber.io/docs/bdd/)
- [BDD essential guide 2026 — monday.com](https://monday.com/blog/rnd/behavior-driven-development/)
- [Wikipedia — Behavior-driven development](https://en.wikipedia.org/wiki/Behavior-driven_development)

### 오픈소스 SDD 프레임워크
- [obra/superpowers GitHub](https://github.com/obra/superpowers)
- [Superpowers 공식 — Anthropic](https://claude.com/plugins/superpowers)
- [Superpowers Plugin for Claude Code (Builder.io)](https://www.builder.io/blog/claude-code-superpowers-plugin)
- [BMAD-METHOD GitHub](https://github.com/bmad-code-org/BMAD-METHOD)
- [BMAD vs spec-kit vs OpenSpec vs PromptX 비교](https://redreamality.com/blog/-sddbmad-vs-spec-kit-vs-openspec-vs-promptx/)
- [GitHub Spec-Kit](https://github.com/github/spec-kit)
- [Spec-driven development with AI (GitHub Blog)](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)
- [OpenSpec (Fission-AI)](https://github.com/Fission-AI/OpenSpec)
- [OpenSpec 공식](https://openspec.dev/)
- [cc-sdd v3.0.2 (gotalab/cc-sdd)](https://github.com/gotalab/cc-sdd)
- [Kiro 공식](https://kiro.dev/)
- [Kiro Specs Documentation](https://kiro.dev/docs/specs/)
- [Introducing Kiro](https://kiro.dev/blog/introducing-kiro/)

### Claude Code 네이티브 `/plan`
- [Claude Code Plan Mode 2026 가이드 (anyonebuilds)](https://www.anyonebuilds.com/guides/claude-code-plan-mode)
- [Claude Code Plan Mode (claudedirectory)](https://www.claudedirectory.org/blog/claude-code-plan-mode-guide)
- [Claude Code Best Practices — DataCamp](https://www.datacamp.com/tutorial/claude-code-best-practices)
- [Spec-Driven Development with Claude Code (Heeki Park, 2026-03)](https://heeki.medium.com/using-spec-driven-development-with-claude-code-4a1ebe5d9f29)

### 비교·랜드스케이프
- [Best Open-Source AI Coding Tools 2026 (Frontman)](https://frontman.sh/blog/best-open-source-ai-coding-tools-2026/)
- [Best Spec-Driven Development Tools 2026 (Augment Code)](https://www.augmentcode.com/tools/best-spec-driven-development-tools)
- [Best Kiro Alternatives 2026 (Augment Code)](https://www.augmentcode.com/tools/best-kiro-alternatives)
- [API-First Development Platforms 2026 (Fern)](https://beta.buildwithfern.com/post/api-first-development-platforms)

---

## 9. 변경 이력

| 날짜 | 변경 |
|---|---|
| 2026-05-05 | 최초 작성 — 1단계(탐색/리서치) 산출물 |
