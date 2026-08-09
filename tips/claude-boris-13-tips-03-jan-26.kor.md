<!--
  이 문서는 tips/claude-boris-13-tips-03-jan-26.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# How I Use Claude Code — 13 Tips from Boris Cherny

Claude Code의 제작자 Boris Cherny([@bcherny](https://x.com/bcherny))가 2026년 1월 3일에 공유한 설정 팁 요약입니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Context

Boris는 자신의 개인 Claude Code 설정을 공유하면서, 그것이 "놀라울 정도로 기본 상태(vanilla)"라고 언급했습니다. Claude Code는 별다른 설정 없이도 훌륭하게 동작하기 때문에 그는 커스터마이징을 많이 하지 않습니다. 사용하는 데 정답은 하나만 있는 것이 아닙니다. 팀은 여러분이 원하는 대로 사용하고, 커스터마이징하고, 해킹할 수 있도록 의도적으로 만들고 있습니다. Claude Code 팀의 각 구성원은 각자 매우 다른 방식으로 사용합니다.

<a href="https://x.com/bcherny/status/2007179832300581177"><img src="assets/boris-26-1-3/0.png" alt="Boris Cherny intro tweet" width="50%" /></a>

---

## 1/ Run 5 Claudes in Parallel

터미널에서 Claude 5개를 병렬로 실행하세요. 탭에 1–5로 번호를 매기고, 어떤 Claude가 입력을 필요로 하는지 알기 위해 시스템 알림을 활용하세요.

참고: [Terminal Setup Docs](https://code.claude.com/docs/en/terminal)

<a href="https://x.com/bcherny/status/2007179833990885678"><img src="assets/boris-26-1-3/1.png" alt="Run 5 Claudes in parallel" width="50%" /></a>

---

## 2/ Use claude.ai/code for Even More Parallelism

로컬 Claude들과 함께 claude.ai/code에서 Claude 5–10개를 병렬로 실행하세요. `claude.ai/code`를 사용해 로컬 세션을 웹 세션으로 넘기고, Chrome에서 수동으로 세션을 시작하며, 양쪽을 자유롭게 오갈 수 있습니다.

<a href="https://x.com/bcherny/status/2007179836704600237"><img src="assets/boris-26-1-3/2.png" alt="claude.ai/code parallelism" width="50%" /></a>

---

## 3/ Use Opus with Thinking for Everything

모든 작업에 thinking을 켠 Opus 4.5를 사용하세요. Boris가 지금까지 써본 것 중 최고의 코딩 모델입니다. Sonnet보다 크고 느리긴 하지만, 덜 조정해도 되고 도구 사용에 더 능숙하기 때문에 결국에는 더 작은 모델을 쓰는 것보다 거의 항상 더 빠릅니다.

<a href="https://x.com/bcherny/status/2007179838864666847"><img src="assets/boris-26-1-3/3.png" alt="Opus with thinking" width="50%" /></a>

---

## 4/ Share a Single CLAUDE.md with Your Team

저장소마다 하나의 `CLAUDE.md`를 공유하세요. git에 체크인하고, 팀 전체가 일주일에 여러 번 기여하도록 하세요. Claude가 무언가를 잘못할 때마다 그 내용을 `CLAUDE.md`에 추가해, 다음번에는 Claude가 그렇게 하지 않도록 알려주세요.

<a href="https://x.com/bcherny/status/2007179840848597422"><img src="assets/boris-26-1-3/4.png" alt="Shared CLAUDE.md" width="50%" /></a>

---

## 5/ Tag @claude on PRs to Update CLAUDE.md

코드 리뷰 중에 동료의 PR에서 `@claude`를 태그해 `CLAUDE.md`에 무언가를 추가하는 것을 PR의 일부로 만드세요. 이를 위해 Claude Code GitHub action([install-@hub-action](https://github.com/apps/claude))을 사용하세요. Boris 버전의 Compounding Engineering입니다.

<a href="https://x.com/bcherny/status/2007179842928947333"><img src="assets/boris-26-1-3/5.png" alt="Tag @claude on PRs" width="50%" /></a>

---

## 6/ Start Most Sessions in Plan Mode

대부분의 세션을 Plan 모드(shift+tab 두 번)로 시작하세요. 목표가 Pull Request를 작성하는 것이라면, Plan 모드를 사용해 계획이 마음에 들 때까지 Claude와 주고받으세요. 그다음 auto-accept edits 모드로 전환하면 Claude가 대개 한 번에 해냅니다(1-shot). 좋은 계획이 정말 중요합니다.

<a href="https://x.com/bcherny/status/2007179845336527000"><img src="assets/boris-26-1-3/6.png" alt="Plan mode" width="50%" /></a>

---

## 7/ Use Slash Commands for Inner Loop Workflows

하루에 여러 번 반복하는 모든 "inner loop" 워크플로에 슬래시 명령을 사용하세요. 이렇게 하면 반복적인 프롬프트 입력을 줄일 수 있고, Claude도 이 워크플로들을 사용할 수 있게 됩니다. 명령은 git에 체크인되며 `.claude/commands/`에 위치합니다.

예시: `/commit-push-pr` — 커밋, 푸시, 그리고 PR 열기.

<a href="https://x.com/bcherny/status/2007179847949500714"><img src="assets/boris-26-1-3/7.png" alt="Slash commands" width="50%" /></a>

---

## 8/ Use Subagents to Automate Common Workflows

몇 가지 서브에이전트를 정기적으로 사용하세요. `code-simplifier`는 Claude가 작업을 마친 뒤 코드를 단순화하고, `verify-app`은 Claude Code를 엔드투엔드로 테스트하기 위한 상세한 지침을 담고 있습니다. 서브에이전트는 가장 흔한 워크플로를 자동화하는 것이라고 생각하면 됩니다. 슬래시 명령과 비슷합니다.

서브에이전트는 `.claude/agents/`에 위치합니다.

<a href="https://x.com/bcherny/status/2007179850139000872"><img src="assets/boris-26-1-3/8.png" alt="Subagents" width="50%" /></a>

---

## 9/ Use a PostToolUse Hook to Auto-Format Code

Claude의 코드를 포맷하기 위해 `PostToolUse` 훅을 사용하세요. Claude는 대개 잘 정리된 코드를 기본으로 생성하지만, 이 훅은 나중에 CI에서 포맷 오류가 나지 않도록 마지막 10%를 처리해 줍니다.

```json
"PostToolUse": [
  {
    "matcher": "Write|Edit",
    "hooks": [
      {
        "type": "command",
        "command": "bun run format || true"
      }
    ]
  }
]
```

<a href="https://x.com/bcherny/status/2007179852047335529"><img src="assets/boris-26-1-3/9.png" alt="PostToolUse hook for formatting" width="50%" /></a>

---

## 10/ Pre-allow Permissions Instead of --dangerously-skip-permissions

`--dangerously-skip-permissions`를 사용하지 마세요. 대신 `/permissions`를 사용해, 여러분의 환경에서 안전하다고 알고 있는 일반적인 bash 명령을 미리 허용하여 불필요한 권한 프롬프트를 피하세요. 이들 대부분은 `.claude/settings.json`에 체크인되어 팀과 공유됩니다.

<a href="https://x.com/bcherny/status/2007179854077407667"><img src="assets/boris-26-1-3/10.png" alt="Pre-allow permissions" width="50%" /></a>

---

## 11/ Let Claude Use All Your Tools via MCP

Claude Code는 여러분의 모든 도구를 사용합니다. 자주 Slack에서 검색하고 글을 올리며(MCP 서버를 통해), 분석 질문에 답하기 위해 BigQuery 쿼리를 실행하고(`bq` CLI 사용), Sentry에서 에러 로그를 가져오는 등의 일을 합니다. Slack MCP 설정은 `.mcp.json`에 체크인되어 팀과 공유됩니다.

<a href="https://x.com/bcherny/status/2007179856266789204"><img src="assets/boris-26-1-3/11.png" alt="MCP tools" width="50%" /></a>

---

## 12/ Verify Long-Running Tasks with Background Agents

아주 오래 걸리는 작업의 경우, (a) 작업이 끝났을 때 백그라운드 에이전트로 결과를 검증하도록 Claude에 프롬프트를 주거나, (b) 이를 더 확실하게 처리하도록 에이전트 Stop 훅을 사용하거나, (c) ralph-wiggum 플러그인(원래 @GeoffreyHuntley가 구상)을 사용하세요.

<a href="https://x.com/bcherny/status/2007179858435281082"><img src="assets/boris-26-1-3/12.png" alt="Long-running tasks verification" width="50%" /></a>

---

## 13/ Give Claude a Way to Verify Its Work

Claude Code에서 훌륭한 결과를 얻기 위해 아마도 가장 중요한 것 — Claude에게 자신의 작업을 검증할 방법을 주세요. Claude가 그런 피드백 루프를 가지면, 최종 결과물의 품질이 2–3배로 좋아집니다.

Claude는 Boris가 반영하는 모든 변경 사항을 하나하나 테스트합니다.

<a href="https://x.com/bcherny/status/2007179861115511237"><img src="assets/boris-26-1-3/13.png" alt="Give Claude a way to verify" width="50%" /></a>

---

## Sources

- [Boris Cherny (@bcherny) on X — January 3, 2026](https://x.com/bcherny/status/2007179832300581177)
