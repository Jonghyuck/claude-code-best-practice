<!--
  이 문서는 tips/claude-thariq-tips-16-apr-26.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Using Claude Code: Session Management & 1M Context — Thariq

Claude Code에서 세션, 컨텍스트 윈도우, 컴팩션을 관리하는 방법에 대한 가이드로, Thariq ([@trq212](https://x.com/trq212))가 2026년 4월 16일에 공유했습니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Context

1M 토큰 컨텍스트 윈도우 덕분에 Claude Code는 더 긴 작업을 더 안정적으로 처리할 수 있습니다. 하지만 세션 관리를 의식적으로 하지 않으면 컨텍스트 오염의 여지도 함께 열립니다. 언제 새로 시작할지, 언제 컴팩션할지, 언제 되감을지, 언제 서브에이전트에 위임할지 — 세션 관리는 그 어느 때보다 중요해졌습니다.

<img src="assets/thariq-26-4-16/1.png" alt="Thariq intro tweet" width="50%" />

<img src="assets/thariq-26-4-16/2.png" alt="Session management intro" width="50%" />

---

## A Quick Primer on Context, Compaction & Context Rot

컨텍스트 윈도우는 모델이 다음 응답을 생성할 때 한 번에 "볼" 수 있는 모든 것입니다. 여기에는 시스템 프롬프트, 지금까지의 대화, 모든 도구 호출과 그 출력, 그리고 읽어들인 모든 파일이 포함됩니다. Claude Code의 컨텍스트 윈도우는 **100만 토큰**입니다.

아쉽게도 컨텍스트를 사용하는 데는 약간의 비용이 따릅니다 — 바로 **context rot(컨텍스트 부패)**입니다. 컨텍스트가 커질수록 어텐션이 더 많은 토큰에 분산되고, 오래되고 무관한 내용이 현재 작업을 방해하기 시작하면서 모델 성능이 저하됩니다. 1M 컨텍스트 모델의 경우 어느 정도의 context rot는 **약 30만~40만 토큰** 부근에서 발생하지만, 이는 작업에 따라 크게 달라지며 절대적인 규칙은 아닙니다.

컨텍스트 윈도우는 명확한 한계선입니다. 끝에 다다르면 작업을 요약하고 새 컨텍스트 윈도우에서 이어가야 하는데, 이것이 **컴팩션(compaction)**입니다. 컴팩션은 직접 트리거할 수도 있습니다.

<img src="assets/thariq-26-4-16/3.png" alt="Context window diagram" width="50%" />

<img src="assets/thariq-26-4-16/4.png" alt="Context rot explanation" width="50%" />

---

## Every Turn Is a Branching Point

Claude가 한 턴을 마치면, 다음에 무엇을 할지에 대해 놀라울 만큼 많은 선택지가 있습니다.

- **Continue** — 같은 세션에서 메시지를 하나 더 보냅니다
- **/rewind (esc esc)** — 이전 메시지로 되돌아가 그 지점부터 다시 시도합니다
- **/clear** — 새 세션을 시작합니다. 대개 방금 배운 내용에서 추려낸 브리프와 함께 시작합니다
- **Compact** — 지금까지의 세션을 요약하고 그 요약 위에서 계속 진행합니다
- **Subagents** — 다음 작업 덩어리를 자체적인 깨끗한 컨텍스트를 가진 에이전트에 위임하고, 그 결과만 다시 가져옵니다

가장 자연스러운 선택은 그냥 continue하는 것이지만, 나머지 네 가지 선택지는 컨텍스트를 관리하는 데 도움을 주기 위해 존재합니다.

<img src="assets/thariq-26-4-16/5.png" alt="Compaction and branching diagram" width="50%" />

<img src="assets/thariq-26-4-16/6.png" alt="Five options after a turn" width="50%" />

각 선택지는 기존 컨텍스트를 서로 다른 양만큼 앞으로 가져갑니다.

| Fresh session | Compact | Subagent | Rewind | Continue |
|:---:|:---:|:---:|:---:|:---:|
| 브리프만 | 손실 있는 요약 | 전체 + 결과 | 앞부분 유지, 뒷부분 잘림 | 전부 유지 |
| *아무것도 안 남김* | | | | *전부 남김* |

<img src="assets/thariq-26-4-16/7.png" alt="Context carry-forward spectrum" width="50%" />

---

## When to Start a New Session

새로운 1M 컨텍스트 윈도우는 이제 더 긴 작업을 더 안정적으로 할 수 있다는 뜻입니다 — 예를 들어 풀스택 앱을 처음부터 만드는 일이 그렇습니다. 하지만 모델이 컨텍스트를 다 쓰지 않았다고 해서 새 세션을 시작하지 말아야 한다는 뜻은 아닙니다.

**일반적인 경험칙: 새 작업을 시작할 때는 새 세션도 함께 시작해야 합니다.**

애매한 영역은 일부 컨텍스트는 여전히 필요하지만 전부는 아닌 관련 작업을 하고 싶을 때입니다. 예를 들어 방금 구현한 기능의 문서를 작성하는 경우가 그렇습니다. 새 세션을 시작할 수도 있지만, 그러면 Claude가 파일들을 다시 읽어야 해서 더 느리고 비싸집니다. 문서 작성은 지능에 크게 민감한 작업이 아닐 수 있으므로, 추가 컨텍스트를 유지하는 편이 효율성 면에서 이득일 가능성이 큽니다.

<img src="assets/thariq-26-4-16/8.png" alt="When to start a new session" width="50%" />

---

## Rewinding Instead of Correcting

Thariq가 좋은 컨텍스트 관리를 보여주는 습관을 딱 하나 꼽아야 한다면, 그것은 **rewind(되감기)**입니다.

Claude Code에서 Esc를 두 번 누르면(또는 `/rewind`를 실행하면) 이전 메시지 아무 곳으로나 되돌아가 그 지점부터 다시 프롬프트를 넣을 수 있습니다. 그 지점 이후의 메시지들은 컨텍스트에서 제거됩니다.

**Correcting(수정)** — 실패한 시도 A 뒤에 "아니, B를 해봐"라고 말하는 것 — 은 실패한 시도를 컨텍스트에 남깁니다.
> context = reads + 2 failed attempts + 2 corrections + the fix

**Rewinding(되감기)** — 실패한 시도 이전으로 돌아가 배운 것을 반영해 다시 프롬프트를 넣는 것 — 은 더 깔끔합니다.
> context = reads + one informed prompt + the fix

Rewind가 더 나은 접근일 때가 많습니다. 예를 들어 Claude가 파일 다섯 개를 읽고 어떤 접근을 시도했는데 잘 안 됐다고 합시다. 여러분의 본능은 "그건 안 됐어, 대신 X를 해봐"라고 입력하는 것일 수 있습니다. 하지만 더 나은 수는 파일 읽기 직후 지점으로 되감고, 배운 것을 반영해 다시 프롬프트를 넣는 것입니다. "접근 A는 쓰지 마, foo 모듈이 그걸 노출하지 않아 — 바로 B로 가."

또한 **"summarize from here"**를 사용하면 Claude가 배운 내용을 요약하고 인계 메시지를 만들게 할 수 있습니다. 무언가를 시도했다가 실패한 미래의 Claude가 이전 세대의 Claude에게 보내는 메시지 같은 것입니다.

<img src="assets/thariq-26-4-16/9.png" alt="Correcting vs rewinding diagram" width="50%" />

<img src="assets/thariq-26-4-16/10.png" alt="Rewind with summarize from here" width="50%" />

---

## Compacting vs. Fresh Sessions

세션이 길어지면 무게를 덜어낼 방법이 두 가지 있습니다. `/compact` 또는 `/clear`(그리고 새로 시작)입니다. 둘은 비슷해 보이지만 동작은 매우 다릅니다.

**Compact**는 모델에게 지금까지의 대화를 요약하게 한 다음, 히스토리를 그 요약으로 교체합니다. 손실이 있습니다 — 무엇이 중요했는지 Claude가 판단하도록 맡기는 것이지만, 대신 여러분이 직접 무언가를 쓸 필요는 없습니다. Claude가 중요한 배움이나 파일을 포함하는 데 더 철저할 수도 있습니다. 지시를 함께 전달해 방향을 잡아줄 수도 있습니다 (`/compact focus on the auth refactor, drop the test debugging`).

- **작업 중간**이라 흐름을 유지하고 싶을 때 — 세부는 흐릿해도 됨
- 저렴하니 계속 진행

**Fresh + brief** (`/clear`)는 *여러분이* 무엇이 중요한지 적어두고("우리는 인증 미들웨어를 리팩터링 중이고, 제약은 X이며, 중요한 파일은 A와 B이고, 접근 Y는 배제했다") 깨끗하게 시작하는 것입니다. 더 수고롭지만, 그 결과 남는 컨텍스트는 *여러분이* 관련 있다고 판단한 것입니다.

- **위험 부담이 큰** 다음 단계 — 100K 규모의 탐색 끝에 사실 하나를 찾았을 때
- 더 수고롭고, 더 정확함

<img src="assets/thariq-26-4-16/11.png" alt="Compacting vs fresh sessions" width="50%" />

<img src="assets/thariq-26-4-16/12.png" alt="Compact vs fresh diagram" width="50%" />

---

## What Causes a Bad Compact?

장시간 세션을 많이 돌려봤다면, 컴팩션이 유독 안 좋게 되는 경우를 눈치챈 적이 있을 것입니다. 나쁜 컴팩션은 모델이 여러분의 작업이 나아갈 방향을 예측하지 못할 때 발생할 수 있습니다.

예를 들어 긴 디버깅 세션 후에 오토컴팩트가 발동해 그 조사 과정을 요약합니다. 여러분의 다음 메시지는 "이제 bar.ts에서 봤던 그 다른 경고를 고쳐줘"입니다. 하지만 세션이 디버깅에 집중되어 있었기 때문에, 그 다른 경고는 요약에서 빠졌을 수 있습니다.

이것이 특히 까다로운 이유는, context rot 때문에 모델이 컴팩션할 때 가장 덜 똑똑한 상태에 있기 때문입니다. 100만 컨텍스트가 있으면, 하려는 일에 대한 설명과 함께 미리 능동적으로 `/compact`할 시간이 더 많습니다.

<img src="assets/thariq-26-4-16/13.png" alt="Bad compact diagram" width="50%" />

<img src="assets/thariq-26-4-16/14.png" alt="Bad compact explanation" width="50%" />

---

## Subagents & Fresh Context Windows

서브에이전트는 컨텍스트 관리의 한 형태로, 어떤 작업 덩어리가 다시는 필요 없을 중간 출력물을 많이 만들어낼 것을 미리 아는 경우에 유용합니다.

Claude가 Agent 도구를 통해 서브에이전트를 띄우면, 그 서브에이전트는 자체적인 깨끗한 컨텍스트 윈도우를 갖습니다. 필요한 만큼 작업을 수행한 뒤 결과를 종합하므로, 최종 보고서만 부모 컨텍스트로 돌아옵니다.

판단 기준: **이 도구 출력이 다시 필요할까, 아니면 결론만 필요할까?**

탐색 과정에서 생긴 잡음은 서브에이전트가 종료될 때 가비지 컬렉션됩니다 — 파일 읽기 20번, grep 12번, 막다른 길 3번 — 오직 최종 보고서만 부모로 돌아옵니다.

Claude Code가 서브에이전트를 자동으로 호출하기도 하지만, 명시적으로 그렇게 하라고 지시하고 싶을 수 있습니다. 예를 들면 다음과 같습니다.

- "다음 스펙 파일을 기준으로 이 작업의 결과를 검증할 서브에이전트를 띄워줘"
- "이 다른 코드베이스를 읽고 인증 플로우를 어떻게 구현했는지 요약할 서브에이전트를 띄운 다음, 같은 방식으로 네가 직접 구현해"
- "내 git 변경사항을 기준으로 이 기능의 문서를 작성할 서브에이전트를 띄워줘"

<img src="assets/thariq-26-4-16/15.png" alt="Subagent context diagram" width="50%" />

<img src="assets/thariq-26-4-16/16.png" alt="Subagent explanation" width="50%" />

<img src="assets/thariq-26-4-16/17.png" alt="When to use subagents" width="50%" />

---

## Summary

Claude가 한 턴을 끝내고 여러분이 새 메시지를 보내려는 순간, 하나의 의사결정 지점이 있습니다. 시간이 지나면 Claude가 이를 스스로 처리하겠지만, 지금으로서는 이것이 Claude의 출력을 이끄는 방법 중 하나입니다.

| Situation | Reach for | Why |
|-----------|-----------|-----|
| 같은 작업이고, 컨텍스트가 여전히 유효함 | **Continue** | 윈도우 안의 모든 것이 여전히 핵심적임 — 그것을 다시 쌓는 데 비용을 치르지 말 것 |
| Claude가 잘못된 길로 감 | **Rewind** (double-Esc) | 유용한 파일 읽기는 유지하고, 실패한 시도는 버리고, 배운 것을 반영해 다시 프롬프트 |
| 작업 중간이지만 세션이 오래된 디버깅/탐색으로 부풀어 있음 | **/compact \<hint\>** | 노력이 적게 듦; 무엇이 중요했는지 Claude가 판단. 필요하면 힌트로 방향을 잡아줄 것 |
| 진짜로 새로운 작업을 시작함 | **/clear** | rot 제로; 무엇을 앞으로 가져갈지 여러분이 정확히 통제 |
| 다음 단계가 많은 출력을 만들지만 결론만 필요함 | **Subagent** | 중간 도구 잡음은 자식의 컨텍스트에 남고, 결과만 돌아옴 |

<img src="assets/thariq-26-4-16/18.png" alt="Summary" width="50%" />

<img src="assets/thariq-26-4-16/19.png" alt="Decision table" width="50%" />

---

## Sources

- [Thariq (@trq212) on X — April 16, 2026](https://x.com/trq212)
