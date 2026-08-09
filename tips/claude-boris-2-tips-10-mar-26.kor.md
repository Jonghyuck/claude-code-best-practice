<!--
  이 문서는 tips/claude-boris-2-tips-10-mar-26.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Code Review & Test Time Compute — Tips from Boris Cherny

Claude Code의 창시자 Boris Cherny([@bcherny](https://x.com/bcherny))가 2026년 3월 10일에 공유한 인사이트 요약입니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 1/ Introducing Code Review

Claude Code의 새로운 기능: **Code Review**. 에이전트 팀이 모든 PR에 대해 심층 리뷰를 수행합니다.

- Anthropic 자체 팀을 위해 가장 먼저 만들어졌습니다 — 올해 엔지니어 1인당 코드 산출량이 **200%** 증가하면서 리뷰가 병목이 되었기 때문입니다
- Boris는 몇 주간 이 기능을 사용해 왔으며, 그렇지 않았다면 놓쳤을 실제 버그들을 많이 잡아낸다는 것을 발견했습니다
- PR이 열리면 Claude가 버그를 찾아내기 위해 에이전트 팀을 파견합니다

<a href="https://x.com/bcherny/status/2031089411820228645"><img src="assets/boris-26-3-10/0.png" alt="Boris Cherny announcing Code Review" width="50%" /></a>

---

## 2/ Test Time Compute & Multiple Context Windows

대략적으로, 코딩 문제에 더 많은 토큰을 투입할수록 결과가 더 좋아집니다. Boris는 이를 **test time compute**라고 부릅니다.

- **별도의 컨텍스트 윈도우**를 사용하면 결과가 더욱 좋아집니다 — 이것이 서브에이전트가 동작하는 원리이며, 하나의 에이전트가 버그를 일으키고 (동일한 모델을 사용하는) 다른 에이전트가 그것을 찾아낼 수 있는 이유입니다
- 엔지니어링 팀과 유사합니다: Boris가 버그를 만들면, 코드를 리뷰하는 동료가 그 자신보다 더 안정적으로 버그를 찾아낼 수 있습니다
- 궁극적으로는 에이전트가 버그 없는 완벽한 코드를 작성하게 될 것입니다 — 그때까지는 **서로 상관관계가 없는 여러 개의 컨텍스트 윈도우**를 사용하는 것이 대체로 좋은 접근 방식입니다

<a href="https://x.com/bcherny/status/2031151689219321886"><img src="assets/boris-26-3-10/1.png" alt="Boris Cherny on test time compute" width="50%" /></a>

---

## Sources

- [Boris Cherny (@bcherny) on X — March 10, 2026](https://x.com/bcherny)
