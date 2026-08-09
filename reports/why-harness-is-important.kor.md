<!--
  이 문서는 reports/why-harness-is-important.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Why Harness is Important

Claude Code의 기능이 "그저 프롬프트를 위장한 것"이 아닌 이유, 그리고 하니스(harness)가 장난감 수준의 결과물과 프로덕션급 엔지니어링 작업을 실제로 갈라놓는 요인인 이유.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Executive Summary

숙련된 Claude Code 사용자들 사이에서 흔히 나오는 단순화는 이렇다: *"스킬, 커맨드, 서브에이전트, 훅 — 결국 전부 모델에게 전달되는 프롬프트가 되므로, 잘 쓴 프롬프트 하나면 그와 동등하다."*

모델의 최종 추론 호출이라는 층위에서 보면 이는 기술적으로 맞다. 모델은 언제나 토큰만 볼 뿐이다.

그러나 그 외의 모든 층위 — 즉 실제 소프트웨어 엔지니어링이 벌어지는 층위 — 에서는 **이 단순화가 무너진다.** 하니스는 프롬프트 전달 시스템이 아니다. 그것은 **프롬프트 구성 시스템이자, 결정론적 실행 시스템이며, 컨텍스트 아키텍처 시스템** 이고, 이러한 역량은 더 강한 문구로 대체할 수 없다.

이 리포트는 이 단순화가 어디서 옳고 어디서 실패하는지, 그리고 "모델이 보는 것"과 "시스템이 하는 것"을 혼동하는 것이 어째서 실무자들을 Claude Code의 실질적 레버리지를 만들어 내는 기능들로부터 멀어지게 하는지를 설명한다.

---

## The Reduction That Sounds Right

**단발성 원자적 작업(single-shot atomic task)** — "재귀 피보나치 함수를 작성해 줘" — 에 대해서는, 하니스가 결과물의 품질에 기여하는 바가 없다. 동일한 토큰을 동일한 모델에게 넘기면, 그것이 스킬로 왔든 커맨드로 왔든 순수 프롬프트로 왔든 동일한 결과 분포를 얻는다.

이 좁은 영역에서는 단순화가 성립한다:

> Output quality ≈ Prompt quality

이 영역은 Claude Code가 일반 챗봇에 비해 별다른 가치를 주지 못하는 영역이다. 또한 이 단순화가 암묵적으로 가정하는 영역이기도 하다 — 그리고 정확히, 실제 엔지니어링 작업이 놓여 있지 않은 영역이다.

---

## Where the Reduction Breaks Down

하니스의 열 가지 아키텍처적 역량은 프롬프트가 접근할 수 없는 층위에서 작동한다.

| # | Capability | What it does | Why a prompt can't replicate |
|---|------------|--------------|-------------------------------|
| 1 | **Context isolation** | 서브에이전트가 별도의 컨텍스트 윈도우에서 실행된다 | 프롬프트는 하나의 윈도우를 채운다. N개의 병렬 서브에이전트는 약 N배의 유효 컨텍스트를 제공한다. |
| 2 | **Harness-enforced tool restrictions** | `allowed-tools` / `disallowedTools`가 모델이 도구를 사용하기 전에 이를 차단한다 | 프롬프트 지시는 권고에 불과하며 모델이 무시할 수 있다. 거부 규칙은 무시될 수 없다. |
| 3 | **Lazy-loaded rules & memory** | `paths:` 프론트매터와 하위 `CLAUDE.md` 파일은 Claude가 매칭되는 경로를 건드릴 때만 로드된다 | 프롬프트는 정적이다 — 런타임에 어떤 파일을 읽고 있는지에 따라 조건부로 로드할 수 없다. |
| 4 | **Hooks: deterministic code execution** | 셸 명령이 라이프사이클 이벤트(PreToolUse, PostToolUse, Stop 등)에서 실행되며 도구 호출을 **차단**할 수 있다 | 프롬프트는 자신의 도구 호출을 가로챌 수 없다. 훅은 모델이 "원하지" 않더라도 실행된다. |
| 5 | **Model routing** | `model: haiku` 또는 `model: opus`가 호출을 다른 모델 엔드포인트로 라우팅한다 | 프롬프트 안의 그 어떤 토큰도 어떤 모델이 응답할지를 바꿀 수 없다. |
| 6 | **Parallelism** | 여러 서브에이전트가 동시에 실행된다 | 프롬프트는 순차적이다. 하니스는 병렬 프로세스로부터 결과를 스케줄링하고 수집한다. |
| 7 | **Cross-session persistence** | 메모리 시스템과 설정 계층이 대화를 넘어 지속된다 | 프롬프트는 세션이 끝나면 사라진다. |
| 8 | **Modular system prompt** | CLI는 활성화된 기능에 따라 110개 이상의 시스템 프롬프트 조각을 조건부로 로드한다 | 사용자는 CLI 내부의 프롬프트 조각을 직접 작성하거나 갈아 끼울 수 없다. |
| 9 | **Skill preloading** | `skills:` 필드가 스킬의 전체 내용을 서브에이전트의 시작 컨텍스트에 주입한다 | 사용자는 다른 에이전트의 컨텍스트를 미리 채워 넣을 수 없다 — 오직 하니스 로더만 가능하다. |
| 10 | **Permission classification** | `auto` 권한 모드는 백그라운드 분류기를 사용해 도구 호출을 사전 승인하거나 차단한다 | 프롬프트는 자기 자신에게 실행 이전 안전 계층을 더할 수 없다. |

각 행은 "강한 문구"가 범주적으로 대체물이 될 수 없는 차원이다.

---

## The Two Uses of the Word "Prompt"

이 단순화는 말장난(equivocation)에 기대고 있다. *prompt* 라는 단어는 서로 매우 다른 두 가지를 가리키는 데 사용된다:

| Meaning | Who controls it | Size |
|---------|-----------------|------|
| (a) 사용자가 입력한 것 | 사용자 | ~6–60 토큰 |
| (b) 추론 시점에 모델이 보는 것 | 하니스 | ~5,000–50,000+ 토큰 |

챗봇에서는 (a)와 (b)가 같은 것이다.
Claude Code에서는 이 둘이 근본적으로 다르다.

하니스의 역할은 바로 (b)를 (a)보다 훨씬 풍부하게 만드는 것이다:

```
User types: "write a recursive flatten function"   ← (a) ~6 tokens

What the model actually sees at inference:         ← (b) ~15,000 tokens
  ├── CLAUDE.md (project conventions)
  ├── Matching .claude/rules/*.md (loaded via paths: frontmatter)
  ├── Modular system prompt fragments
  ├── Tool definitions
  ├── Environment context (cwd, git status, platform)
  ├── Prior turn history
  ├── Files read by the model via Read/Grep tools
  └── User's 6-token request
```

**결과물의 품질은 (a)가 아니라 (b)의 함수다.** 하니스가 (b)를 구성한다. "강한 프롬프트 하나"로는 (b)를 재현할 수 없는데, 그 대부분이 사용자가 작성한 것이 아니기 때문이다.

---

## Even for Output Quality, the Harness Is Doing Work

동일한 프롬프트 — "write a recursive flatten function" — 를 세 가지 환경에서 살펴보자:

| Environment | What the model sees | Typical result |
|-------------|---------------------|----------------|
| Chatbot, no tools | 문장 | 교과서적 재귀, 일반적인 스타일 |
| Claude Code, no reading | 문장 + CLAUDE.md | 선언된 프로젝트 컨벤션에 맞춤 |
| Claude Code, agentic loop | 문장 + CLAUDE.md + 인접 파일 읽기 + 테스트 실행 | 실제 코드베이스 패턴에 맞고, 테스트를 통과하며, 기존 코드가 처리하는 엣지 케이스를 처리 |

같은 모델. 같은 사용자 프롬프트. **세 가지 다른 결과 품질.** 그 차이는 하니스다 — 구체적으로는, 하니스가 조립하는 유효 컨텍스트와 그것이 가능하게 하는 반복 루프(iteration loop)다.

사소하지 않은(non-trivial) 작업에서 결과물의 품질은 다음의 함수다:

```
Output quality = f(effective context, model capability, iteration loop)
```

사용자는 *유효 컨텍스트* 중 극히 일부(자신이 입력한 프롬프트)만 통제한다. 나머지는 하니스가 통제하며 — 반복 루프는 전적으로 하니스가 통제한다.

---

## What the Reduction Gets Right (And What It Gets Wrong)

| Claim | Verdict |
|-------|---------|
| "추론 시점에 모델은 오직 토큰만 본다." | ✅ True |
| "스킬, 커맨드, 서브에이전트 프롬프트는 모두 어떤 컨텍스트에 토큰을 기여한다." | ✅ True |
| "진공 속의 원자적 작업이라면 프롬프트 품질이 결과물 품질을 좌우한다." | ✅ True |
| "따라서 강한 프롬프트는 기능을 사용하는 것과 동등하다." | ❌ False |
| "따라서 하니스는 결과물 품질에 중요하지 않다." | ❌ False on real engineering tasks |

앞의 세 진술은 정확한 관찰이다. 네 번째로의 도약에서 추론이 실패한다: 그것은 모델을 그것을 감싸는 시스템과 혼동하고, 원자적 작업을 실제 엔지니어링 작업과 혼동한다.

---

## The Correct Mental Model

> **프롬프트는 모델에게 무엇을 하라고 요청하는지를 통제한다.**
> **하니스는 모델이 닿을 수 없는 층위에서 시스템이 무엇을 하는지를 통제한다** — 토큰이 도착하기 전, 토큰이 생성된 후, 세션을 가로질러, 컨텍스트를 가로질러, 그리고 프로세스를 가로질러.

기능은 단계가 몇 개 더 붙은 프롬프트가 아니다. 그것은 **하니스 수준의 프리미티브(primitive)** — 결정론적 실행, 컨텍스트 아키텍처, 인프라 라우팅 — 이며, 모델이 목소리를 낼 수 없는 층위에서 작동한다.

유용한 비유 하나:

| Layer | Chatbot | Claude Code |
|-------|---------|-------------|
| Recipe | 사용자의 메시지 | 사용자의 메시지 + 하니스가 조립한 컨텍스트 |
| Kitchen | 없음 — 그저 학생 한 명 | 도구, 훅, 메모리, 병렬 워커, 라이프사이클 이벤트 |

세상에서 가장 훌륭한 레시피를 쓸 수는 있다. 그러나 주방이 없다면, 대규모로 요리할 수 없다.

---

## Takeaways for Practitioners

1. **원자적 질문에 대해서는 프롬프트 품질이 거의 전부다.** 하니스는 무관하다. 그게 필요한 전부라면 챗봇을 쓰라.
2. **실제 코드베이스 작업에서는 하니스가 조용히 무거운 짐을 지고 있다.** 추론 시점의 유효 프롬프트는 사용자가 입력한 것이 아니라 대부분 하니스가 구성한 것이다.
3. **프롬프트가 범주적으로 할 수 없는 것에 기능을 사용하라:** 결정론(훅), 격리(서브에이전트), 지연 로딩(`paths:`를 사용한 규칙), 지속성(메모리), 라우팅(에이전트별 `model:`), 그리고 병렬성.
4. **강한 프롬프트는 필요조건이지만 충분조건은 아니다.** 기능은 프롬프트가 줄 수 없는 결정론, 격리, 그리고 조합(composition)을 제공한다. 이 둘은 상호 보완적이지 대체재가 아니다.

---

## Sources

- [Agents vs Commands vs Skills](claude-agent-command-skill.md) — 기능별 컨텍스트 격리, 모델 오버라이드, 도구 제한을 보여줌
- [Claude Agent SDK vs CLI System Prompts](claude-agent-sdk-vs-cli-system-prompts.md) — 110개 이상의 모듈형 시스템 프롬프트 조각을 문서화
- [Claude Agent Memory](claude-agent-memory.md) — `memory:` 스코프를 통한 세션 간 지속성
- [Claude Memory Best Practice](../best-practice/claude-memory.md) — 지연 로딩되는 하위 `CLAUDE.md` 파일
- [Claude Subagents Best Practice](../best-practice/claude-subagents.md) — 하니스가 강제하는 역량에 대한 프론트매터 레퍼런스
- [Claude Settings Best Practice](../best-practice/claude-settings.md) — 권한 규칙 평가 및 `auto` 모드 분류기
- [Orchestration Workflow](../orchestration-workflow/orchestration-workflow.md) — 단순화가 실패함을 보여주는 구체적 시연
