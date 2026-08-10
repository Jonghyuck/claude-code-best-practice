<!--
  이 문서는 README.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·앵커는 원본과 동일하게 보존하고 산문·표 설명만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md — English → README.md, 한글 → README.kor.md
-->

# claude-code-best-practice
바이브 코딩에서 에이전틱 엔지니어링으로 — 연습이 Claude를 완벽하게 만든다 (practice makes claude perfect)

![updated with Claude Code](https://img.shields.io/badge/updated_with_Claude_Code-Aug%2009%2C%202026%209%3A45%20AM%20PKT-white?style=flat&labelColor=555) <a href="https://github.com/shanraisshan/claude-code-best-practice/stargazers"><img src="https://img.shields.io/github/stars/shanraisshan/claude-code-best-practice?style=flat&label=%E2%98%85&labelColor=555&color=white" alt="GitHub Stars"></a><br>

[![Best Practice](!/tags/best-practice.svg)](best-practice/) [![Implemented](!/tags/implemented.svg)](implementation/) [![Orchestration Workflow](!/tags/orchestration-workflow.svg)](orchestration-workflow/orchestration-workflow.md) [![Claude](!/tags/claude.svg)](https://code.claude.com/docs) [![Boris](!/tags/boris-cherny.svg)](#-tips-and-tricks) [![Community](!/tags/community.svg)](#-subscribe) ![Click on these badges below to see the actual sources](!/tags/click-badges.svg)<br>
<img src="!/tags/a.svg" height="14"> = 에이전트(Agents) · <img src="!/tags/c.svg" height="14"> = 커맨드(Commands) · <img src="!/tags/s.svg" height="14"> = 스킬(Skills)

<p align="center">
  <img src="!/claude-jumping.svg" alt="Claude Code mascot jumping" width="120" height="100"><br>
  <a href="https://github.com/trending"><img src="!/root/github-trending-day.svg" alt="GitHub Trending #1 Repository Of The Day"></a>
</p>

<p align="center">
  <img src="!/root/supported-label.svg" alt="Supported by:" height="34">&nbsp;&nbsp;<a href="https://disrupt.com/?utm_source=github&utm_campaign=shayan_claude_code_best_practice"><img src="!/root/supported-disrupt.svg" alt="Disrupt.com — Ventures Reimagined" height="34"></a>&nbsp;&nbsp;<a href="https://claudekit.cc/?utm_source=github&utm_medium=sponsorship&utm_campaign=shayan_claude_code_best_practice"><img src="!/root/supported-claudekit.svg" alt="ClaudeKit — Production-ready skills and workflows" height="34"></a>
</p>

<p align="center">
  <img src="!/root/boris-slider.gif" alt="Boris Cherny on Claude Code" width="600"><br>
  Boris Cherny on X (<a href="https://x.com/bcherny/status/2007179832300581177">tweet 1</a> · <a href="https://x.com/bcherny/status/2017742741636321619">tweet 2</a> · <a href="https://x.com/bcherny/status/2021699851499798911">tweet 3</a>)
</p>

> [!TIP]
> 이 저장소를 최대한 활용하려면 [**How to Use**](#how-to-use) 섹션을 방문하세요.

## 🧠 CONCEPTS

| 기능 | 위치 | 설명 |
|---------|----------|-------------|
| <img src="!/tags/a.svg" height="14"> [**Subagents**](https://code.claude.com/docs/en/sub-agents) | `.claude/agents/<name>.md` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-subagents.md) [![Implemented](!/tags/implemented.svg)](implementation/claude-subagents-implementation.md) |
| <img src="!/tags/c.svg" height="14"> [**Commands**](https://code.claude.com/docs/en/commands) | `.claude/commands/<name>.md` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-commands.md) [![Implemented](!/tags/implemented.svg)](implementation/claude-commands-implementation.md) |
| <img src="!/tags/s.svg" height="14"> [**Skills**](https://code.claude.com/docs/en/skills) | `.claude/skills/<name>/SKILL.md` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-skills.md) [![Implemented](!/tags/implemented.svg)](implementation/claude-skills-implementation.md) [Official Skills](https://github.com/anthropics/skills/tree/main/skills) · [Skills for Mono-repos](reports/claude-skills-for-larger-mono-repos.md) |
| [**Workflows**](https://code.claude.com/docs/en/common-workflows) | [`.claude/commands/weather-orchestrator.md`](.claude/commands/weather-orchestrator.md) | [![Orchestration Workflow](!/tags/orchestration-workflow.svg)](orchestration-workflow/orchestration-workflow.md) |
| [**Hooks**](https://code.claude.com/docs/en/hooks) | `.claude/hooks/` | [![Best Practice](!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-hooks) [![Implemented](!/tags/implemented.svg)](https://github.com/shanraisshan/claude-code-hooks) [Guide](https://code.claude.com/docs/en/hooks-guide) |
| [**MCP Servers**](https://code.claude.com/docs/en/mcp) | `.claude/settings.json`, `.mcp.json` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-mcp.md) [![Implemented](!/tags/implemented.svg)](.mcp.json) |
| [**Plugins**](https://code.claude.com/docs/en/plugins) | distributable packages | [Marketplaces](https://code.claude.com/docs/en/discover-plugins) · [Create Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) |
| [**Settings**](https://code.claude.com/docs/en/settings) | `.claude/settings.json` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-settings.md) [![Implemented](!/tags/implemented.svg)](.claude/settings.json) [Permissions](https://code.claude.com/docs/en/permissions) · [Model Config](https://code.claude.com/docs/en/model-config) · [Output Styles](https://code.claude.com/docs/en/output-styles) · [Sandboxing](https://code.claude.com/docs/en/sandboxing) · [Keybindings](https://code.claude.com/docs/en/keybindings) · [Auto Mode Config](https://code.claude.com/docs/en/auto-mode-config) |
| [**Status Line**](https://code.claude.com/docs/en/statusline) | `.claude/settings.json` | [![Best Practice](!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-status-line) [![Implemented](!/tags/implemented.svg)](.claude/settings.json) |
| [**Memory**](https://code.claude.com/docs/en/memory) | `CLAUDE.md`, `.claude/rules/`, `~/.claude/rules/`, `~/.claude/projects/<project>/memory/` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-memory.md) [![Implemented](!/tags/implemented.svg)](CLAUDE.md) [Auto Memory](https://code.claude.com/docs/en/memory) · [Auto Memory Deep-dive](reports/claude-agent-memory.md) · [Rules](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) |
| [**Checkpointing**](https://code.claude.com/docs/en/checkpointing) | automatic (file-edit tracking) |  |
| [**CLI Startup Flags**](https://code.claude.com/docs/en/cli-reference) | `claude [flags]` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-cli-startup-flags.md) [Interactive Mode](https://code.claude.com/docs/en/interactive-mode) · [Env Vars](https://code.claude.com/docs/en/env-vars) |
| **AI Terms** | | [![Best Practice](!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-codex-cursor-gemini/blob/main/reports/ai-terms.md) |
| [**Best Practices**](https://code.claude.com/docs/en/best-practices) | | [Prompt Engineering](https://github.com/anthropics/prompt-eng-interactive-tutorial) · [Extend Claude Code](https://code.claude.com/docs/en/features-overview) |

### 🔥 Hot

| 기능 | 위치 | 설명 |
|---------|----------|-------------|
| [**Ultrareview**](https://code.claude.com/docs/en/ultrareview) ![beta](!/tags/beta.svg) | `/code-review ultra`, `claude ultrareview [target]` | [Tasks tracking](https://code.claude.com/docs/en/ultrareview#track-a-running-review) |
| [**Devcontainers**](https://code.claude.com/docs/en/devcontainer) | `.devcontainer/` |  |
| [**Channels**](https://code.claude.com/docs/en/channels) ![beta](!/tags/beta.svg) | `--channels`, plugin-based | [Reference](https://code.claude.com/docs/en/channels-reference) |
| [**No Flicker Mode**](https://code.claude.com/docs/en/fullscreen) ![beta](!/tags/beta.svg) | `/tui fullscreen`, `CLAUDE_CODE_NO_FLICKER=1` | [![Best Practice](!/tags/best-practice.svg)](https://x.com/bcherny/status/2039421575422980329) |
| [**Auto Mode**](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) | `--permission-mode auto`, `Shift+Tab` | [![Best Practice](!/tags/best-practice.svg)](https://x.com/claudeai/status/2036503582166393240) [Blog](https://claude.com/blog/auto-mode) |
| [**Power-ups**](best-practice/claude-power-ups.md) | `/powerup` | [![Best Practice](!/tags/best-practice.svg)](best-practice/claude-power-ups.md) |
| [**Fast Mode**](https://code.claude.com/docs/en/fast-mode) ![beta](!/tags/beta.svg) | `/fast`, `"fastMode": true` |  |
| [**Advisor**](https://code.claude.com/docs/en/advisor) ![beta](!/tags/beta.svg) | `/advisor`, `advisorModel`, `--advisor` | [Blog](https://claude.com/blog/the-advisor-strategy) |
| [**Computer Use**](https://code.claude.com/docs/en/computer-use) ![beta](!/tags/beta.svg) | `computer-use` MCP server | [Desktop](https://code.claude.com/docs/en/desktop#let-claude-use-your-computer) |
| [**Agent SDK**](https://code.claude.com/docs/en/agent-sdk/overview) | `npm` / `pip` package | [Quickstart](https://code.claude.com/docs/en/agent-sdk/quickstart) · [Examples](https://github.com/anthropics/claude-agent-sdk-demos) |
| [**Ralph Wiggum Loop**](https://github.com/anthropics/claude-code/tree/main/plugins/ralph-wiggum) | plugin | [![Best Practice](!/tags/best-practice.svg)](https://github.com/ghuntley/how-to-ralph-wiggum) [![Implemented](!/tags/implemented.svg)](https://github.com/shanraisshan/ralph-wiggum-self-evolving-loop) |
| [**Chrome**](https://code.claude.com/docs/en/chrome) | `--chrome`, extension | [![Best Practice](!/tags/best-practice.svg)](reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| [**Claude Code Web**](https://code.claude.com/docs/en/claude-code-on-the-web) ![beta](!/tags/beta.svg) | `claude.ai/code` | [Routines](https://code.claude.com/docs/en/routines) |
| [**Artifacts**](https://code.claude.com/docs/en/artifacts) ![beta](!/tags/beta.svg) | `/share`, `Artifact` tool |  |
| [**Slack**](https://code.claude.com/docs/en/slack) | `@Claude` in Slack |  |
| [**Code Review**](https://code.claude.com/docs/en/code-review) ![beta](!/tags/beta.svg) | GitHub App (managed) | [![Best Practice](!/tags/best-practice.svg)](https://x.com/claudeai/status/2031088171262554195) [Blog](https://claude.com/blog/code-review) [Local /code-review](https://code.claude.com/docs/en/commands) |
| [**GitHub Actions**](https://code.claude.com/docs/en/github-actions) | `.github/workflows/` | [GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd) |
| [**Remote Control**](https://code.claude.com/docs/en/remote-control) | `/remote-control`, `/rc` | [![Best Practice](!/tags/best-practice.svg)](https://x.com/noahzweben/status/2032533699116355819) [Headless Mode](https://code.claude.com/docs/en/headless) |
| [**Deep Links**](https://code.claude.com/docs/en/deep-links) | `claude-cli://open?repo=…&q=…` |  |
| [**Dynamic Workflows**](https://code.claude.com/docs/en/workflows) | `/workflows`, `ultracode` keyword, `/effort ultracode`, `.claude/workflows/` | [Deep Research](https://code.claude.com/docs/en/workflows#run-a-bundled-workflow) |
| [**Agent Teams**](https://code.claude.com/docs/en/agent-teams) ![beta](!/tags/beta.svg) | built-in (env var) | [![Best Practice](!/tags/best-practice.svg)](https://x.com/bcherny/status/2019472394696683904) [![Implemented](!/tags/implemented.svg)](implementation/claude-agent-teams-implementation.md) |
| [**Agent View**](https://code.claude.com/docs/en/agent-view) ![beta](!/tags/beta.svg) | `claude agents`, `--bg`, `/bg` |  |
| [**Scheduled Tasks**](https://code.claude.com/docs/en/scheduled-tasks) | `/loop`, `/schedule`, cron tools | [![Best Practice](!/tags/best-practice.svg)](https://x.com/bcherny/status/2030193932404150413) [![Implemented](!/tags/implemented.svg)](implementation/claude-scheduled-tasks-implementation.md) [Desktop scheduled tasks](https://code.claude.com/docs/en/desktop-scheduled-tasks) · [Announcement](https://x.com/noahzweben/status/2036129220959805859) |
| [**Routines**](https://code.claude.com/docs/en/routines) ![beta](!/tags/beta.svg) | `claude.ai/code/routines`, `/schedule` | [Desktop Tasks](https://code.claude.com/docs/en/desktop-scheduled-tasks) |
| [**Tasks**](reports/claude-global-vs-project-settings.md#tasks-system) | `/tasks`, `~/.claude/tasks/` | [![Best Practice](!/tags/best-practice.svg)](reports/claude-global-vs-project-settings.md) [Ultrareview tracking](https://code.claude.com/docs/en/ultrareview#track-a-running-review) |
| [**Goal**](https://code.claude.com/docs/en/goal) | `/goal <condition>`, `/goal clear` | [![Implemented](!/tags/implemented.svg)](implementation/claude-goal-implementation.md) |
| [**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation) ![beta](!/tags/beta.svg) | `/voice` | [![Best Practice](!/tags/best-practice.svg)](https://x.com/trq212/status/2028628570692890800) |
| [**Bundled Skills**](https://code.claude.com/docs/en/skills#bundled-skills) | `/code-review`, `/batch` | [![Best Practice](!/tags/best-practice.svg)](https://x.com/bcherny/status/2027534984534544489) |
| [**Git Worktrees**](https://code.claude.com/docs/en/worktrees) | `--worktree`/`-w`, `.worktreeinclude`, `EnterWorktree`/`ExitWorktree`, `isolation: "worktree"`, `WorktreeCreate`/`WorktreeRemove` hooks | [![Best Practice](!/tags/best-practice.svg)](https://x.com/bcherny/status/2025007393290272904) |

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

<a id="orchestration-workflow"></a>

## <a href="orchestration-workflow/orchestration-workflow.md"><img src="!/tags/orchestration-workflow-hd.svg" alt="Orchestration Workflow"></a>

<img src="!/tags/c.svg" height="14"> **Command** → <img src="!/tags/a.svg" height="14"> **Agent** → <img src="!/tags/s.svg" height="14"> **Skill** 패턴의 구현 세부사항은 [orchestration-workflow](orchestration-workflow/orchestration-workflow.md)를 참고하세요.


<p align="center">
  <img src="orchestration-workflow/orchestration-workflow.svg" alt="Command Skill Agent Architecture Flow" width="100%">
</p>

<p align="center">
  <img src="orchestration-workflow/orchestration-workflow.gif" alt="Orchestration Workflow Demo" width="600">
</p>

![How to Use](!/tags/how-to-use.svg)

```bash
claude
/weather-orchestrator
```

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## ⚙️ DEVELOPMENT WORKFLOWS

모든 주요 워크플로우는 동일한 아키텍처 패턴으로 수렴합니다: **Research(조사) → Plan(계획) → Execute(실행) → Review(리뷰) → Ship(배포)**

| Name | ★ | Workflow | <img src="!/tags/a.svg" height="14"> | <img src="!/tags/c.svg" height="14"> | <img src="!/tags/s.svg" height="14"> |
|------|---|----------|---|---|---|
| [Superpowers](https://github.com/obra/superpowers) | 252k | <img src="https://img.shields.io/badge/brainstorming-ddf4ff" alt="brainstorming" align="middle"> → <img src="https://img.shields.io/badge/using--git--worktrees-ddf4ff" alt="using-git-worktrees" align="middle"> → <img src="https://img.shields.io/badge/writing--plans-ddf4ff" alt="writing-plans" align="middle"> → <img src="https://img.shields.io/badge/subagent--driven--development-ddf4ff" alt="subagent-driven-development" align="middle"> → <img src="https://img.shields.io/badge/dispatching--parallel--agents-fff3b0" alt="dispatching-parallel-agents" align="middle"> → <img src="https://img.shields.io/badge/executing--plans-fff3b0" alt="executing-plans" align="middle"> → <img src="https://img.shields.io/badge/test--driven--development-fff3b0" alt="test-driven-development" align="middle"> → <img src="https://img.shields.io/badge/verification--before--completion-fff3b0" alt="verification-before-completion" align="middle"> → <img src="https://img.shields.io/badge/requesting--code--review-ddf4ff" alt="requesting-code-review" align="middle"> → <img src="https://img.shields.io/badge/finishing--a--development--branch-ddf4ff" alt="finishing-a-development-branch" align="middle"> | 0 | 0 | 14 |
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | 228k | <img src="https://img.shields.io/badge/plan-ddf4ff" alt="plan" align="middle"> → <img src="https://img.shields.io/badge/plan--prd-ddf4ff" alt="plan-prd" align="middle"> → <img src="https://img.shields.io/badge/prp--implement-fff3b0" alt="prp-implement" align="middle"> → <img src="https://img.shields.io/badge/code--review-fff3b0" alt="code-review" align="middle"> → <img src="https://img.shields.io/badge/build--fix-fff3b0" alt="build-fix" align="middle"> → <img src="https://img.shields.io/badge/e2e--testing-ddf4ff" alt="e2e-testing" align="middle"> → <img src="https://img.shields.io/badge/prp--pr-ddf4ff" alt="prp-pr" align="middle"> | 67 | 139 | 278 |
| [Matt Pocock Skills](https://github.com/mattpocock/skills) | 165k | <img src="https://img.shields.io/badge/ask--matt-ddf4ff" alt="ask-matt" align="middle"> → <img src="https://img.shields.io/badge/grill--with--docs-ddf4ff" alt="grill-with-docs" align="middle"> → <img src="https://img.shields.io/badge/grill--me-fff3b0" alt="grill-me" align="middle"> → <img src="https://img.shields.io/badge/to--spec-ddf4ff" alt="to-spec" align="middle"> → <img src="https://img.shields.io/badge/to--tickets-ddf4ff" alt="to-tickets" align="middle"> → <img src="https://img.shields.io/badge/tdd-fff3b0" alt="tdd" align="middle"> → <img src="https://img.shields.io/badge/prototype-fff3b0" alt="prototype" align="middle"> → <img src="https://img.shields.io/badge/diagnosing--bugs-fff3b0" alt="diagnosing-bugs" align="middle"> → <img src="https://img.shields.io/badge/implement-ddf4ff" alt="implement" align="middle"> → <img src="https://img.shields.io/badge/code--review-ddf4ff" alt="code-review" align="middle"> → <img src="https://img.shields.io/badge/improve--codebase--architecture-ddf4ff" alt="improve-codebase-architecture" align="middle"> | 0 | 0 | 39 |
| [gstack](https://github.com/garrytan/gstack) | 121k | <img src="https://img.shields.io/badge/%2Foffice--hours-ddf4ff" alt="/office-hours" align="middle"> → <img src="https://img.shields.io/badge/%2Fplan--ceo--review-ddf4ff" alt="/plan-ceo-review" align="middle"> → <img src="https://img.shields.io/badge/%2Fplan--eng--review-ddf4ff" alt="/plan-eng-review" align="middle"> → <img src="https://img.shields.io/badge/%2Fplan--design--review-ddf4ff" alt="/plan-design-review" align="middle"> → <img src="https://img.shields.io/badge/%2Freview-ddf4ff" alt="/review" align="middle"> → <img src="https://img.shields.io/badge/%2Fqa-ddf4ff" alt="/qa" align="middle"> → <img src="https://img.shields.io/badge/%2Fship-ddf4ff" alt="/ship" align="middle"> → <img src="https://img.shields.io/badge/%2Fland--and--deploy-ddf4ff" alt="/land-and-deploy" align="middle"> | 0 | 0 | 53 |
| [Spec Kit](https://github.com/github/spec-kit) | 119k | <img src="https://img.shields.io/badge/%2Fspeckit.constitution-ddf4ff" alt="/speckit.constitution" align="middle"> → <img src="https://img.shields.io/badge/%2Fspeckit.specify-ddf4ff" alt="/speckit.specify" align="middle"> → <img src="https://img.shields.io/badge/%2Fspeckit.clarify-fff3b0" alt="/speckit.clarify" align="middle"> → <img src="https://img.shields.io/badge/%2Fspeckit.plan-ddf4ff" alt="/speckit.plan" align="middle"> → <img src="https://img.shields.io/badge/%2Fspeckit.tasks-ddf4ff" alt="/speckit.tasks" align="middle"> → <img src="https://img.shields.io/badge/%2Fspeckit.analyze-fff3b0" alt="/speckit.analyze" align="middle"> → <img src="https://img.shields.io/badge/%2Fspeckit.implement-ddf4ff" alt="/speckit.implement" align="middle"> → <img src="https://img.shields.io/badge/%2Fspeckit.converge-fff3b0" alt="/speckit.converge" align="middle"> → <img src="https://img.shields.io/badge/%2Fspeckit.checklist-ddf4ff" alt="/speckit.checklist" align="middle"> | 0 | 10 | 0 |
| [agent-skills](https://github.com/addyosmani/agent-skills) | 69k | <img src="https://img.shields.io/badge/%2Fspec-ddf4ff" alt="/spec" align="middle"> → <img src="https://img.shields.io/badge/%2Fplan-ddf4ff" alt="/plan" align="middle"> → <img src="https://img.shields.io/badge/%2Fbuild-ddf4ff" alt="/build" align="middle"> → <img src="https://img.shields.io/badge/%2Ftest-ddf4ff" alt="/test" align="middle"> → <img src="https://img.shields.io/badge/%2Freview-ddf4ff" alt="/review" align="middle"> → <img src="https://img.shields.io/badge/%2Fship-ddf4ff" alt="/ship" align="middle"> | 3 | 7 | 21 |
| [Get Shit Done](https://github.com/gsd-build/get-shit-done) | 65k | <img src="https://img.shields.io/badge/%2Fgsd--new--project-ddf4ff" alt="/gsd-new-project" align="middle"> → <img src="https://img.shields.io/badge/%2Fgsd--discuss--phase-ddf4ff" alt="/gsd-discuss-phase" align="middle"> → <img src="https://img.shields.io/badge/%2Fgsd--plan--phase-ddf4ff" alt="/gsd-plan-phase" align="middle"> → <img src="https://img.shields.io/badge/%2Fgsd--execute--phase-fff3b0" alt="/gsd-execute-phase" align="middle"> → <img src="https://img.shields.io/badge/%2Fgsd--verify--work-ddf4ff" alt="/gsd-verify-work" align="middle"> → <img src="https://img.shields.io/badge/%2Fgsd--ship-ddf4ff" alt="/gsd-ship" align="middle"> | 33 | 67 | 0 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | 60k | <img src="https://img.shields.io/badge/%2Fopsx:explore-ddf4ff" alt="/opsx:explore" align="middle"> → <img src="https://img.shields.io/badge/%2Fopsx:propose-ddf4ff" alt="/opsx:propose" align="middle"> → <img src="https://img.shields.io/badge/%2Fopsx:new-ddf4ff" alt="/opsx:new" align="middle"> → <img src="https://img.shields.io/badge/%2Fopsx:ff-fff3b0" alt="/opsx:ff" align="middle"> → <img src="https://img.shields.io/badge/%2Fopsx:apply-ddf4ff" alt="/opsx:apply" align="middle"> → <img src="https://img.shields.io/badge/%2Fopsx:verify-ddf4ff" alt="/opsx:verify" align="middle"> → <img src="https://img.shields.io/badge/%2Fopsx:archive-ddf4ff" alt="/opsx:archive" align="middle"> | 0 | 12 | 0 |
| [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) | 50k | <img src="https://img.shields.io/badge/bmad--brainstorming-ddf4ff" alt="bmad-brainstorming" align="middle"> → <img src="https://img.shields.io/badge/bmad--prfaq-ddf4ff" alt="bmad-prfaq" align="middle"> → <img src="https://img.shields.io/badge/bmad--prd-ddf4ff" alt="bmad-prd" align="middle"> → <img src="https://img.shields.io/badge/bmad--ux-ddf4ff" alt="bmad-ux" align="middle"> → <img src="https://img.shields.io/badge/bmad--technical--research-ddf4ff" alt="bmad-technical-research" align="middle"> → <img src="https://img.shields.io/badge/bmad--generate--project--context-ddf4ff" alt="bmad-generate-project-context" align="middle"> → <img src="https://img.shields.io/badge/bmad--create--story-ddf4ff" alt="bmad-create-story" align="middle"> → <img src="https://img.shields.io/badge/bmad--dev--auto-fff3b0" alt="bmad-dev-auto" align="middle"> → <img src="https://img.shields.io/badge/bmad--review--edge--case--hunter-fff3b0" alt="bmad-review-edge-case-hunter" align="middle"> | 6 | 0 | 47 |
| [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) | 38k | <img src="https://img.shields.io/badge/deep--interview-ddf4ff" alt="deep-interview" align="middle"> → <img src="https://img.shields.io/badge/team--plan-ddf4ff" alt="team-plan" align="middle"> → <img src="https://img.shields.io/badge/team--prd-ddf4ff" alt="team-prd" align="middle"> → <img src="https://img.shields.io/badge/team--exec-ddf4ff" alt="team-exec" align="middle"> → <img src="https://img.shields.io/badge/team--verify-fff3b0" alt="team-verify" align="middle"> → <img src="https://img.shields.io/badge/team--fix-fff3b0" alt="team-fix" align="middle"> | 19 | 0 | 40 |
| [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) | 23k | <img src="https://img.shields.io/badge/%2Fce--brainstorm-ddf4ff" alt="/ce-brainstorm" align="middle"> → <img src="https://img.shields.io/badge/%2Fce--plan-ddf4ff" alt="/ce-plan" align="middle"> → <img src="https://img.shields.io/badge/%2Fce--work-ddf4ff" alt="/ce-work" align="middle"> → <img src="https://img.shields.io/badge/%2Fce--simplify--code-fff3b0" alt="/ce-simplify-code" align="middle"> → <img src="https://img.shields.io/badge/%2Fce--code--review-ddf4ff" alt="/ce-code-review" align="middle"> → <img src="https://img.shields.io/badge/%2Fce--compound-ddf4ff" alt="/ce-compound" align="middle"> | 0 | 1 | 29 |
| [HumanLayer](https://github.com/humanlayer/humanlayer) | 11k | <img src="https://img.shields.io/badge/%2Fralph__research-fff3b0" alt="/ralph_research" align="middle"> → <img src="https://img.shields.io/badge/%2Fcreate__plan-ddf4ff" alt="/create_plan" align="middle"> → <img src="https://img.shields.io/badge/%2Fiterate__plan-fff3b0" alt="/iterate_plan" align="middle"> → <img src="https://img.shields.io/badge/%2Fvalidate__plan-ddf4ff" alt="/validate_plan" align="middle"> → <img src="https://img.shields.io/badge/%2Fimplement__plan-ddf4ff" alt="/implement_plan" align="middle"> → <img src="https://img.shields.io/badge/%2Fdescribe__pr-ddf4ff" alt="/describe_pr" align="middle"> → <img src="https://img.shields.io/badge/%2Fcommit-ddf4ff" alt="/commit" align="middle"> | 6 | 27 | 0 |

> *참고: 노란색 태그는 서브 루프(sub-loop)입니다 — 상위 단계 안에서 반복되는 단계(예: 태스크별, 스토리별, 또는 검증 조건이 통과할 때까지).*

### Others
- [RPI](development-workflows/rpi/rpi-workflow.md) [![Implemented](!/tags/implemented.svg)](development-workflows/rpi/rpi-workflow.md)
- [Ralph Wiggum Loop](https://www.youtube.com/watch?v=eAtvoGlpeRU) [![Implemented](!/tags/implemented.svg)](https://github.com/shanraisshan/ralph-wiggum-self-evolving-loop)
- [Andrej Karpathy (OpenAI 창립 멤버) 워크플로우](https://x.com/karpathy/status/2015883857489522876)
- [Peter Steinberger (OpenClaw 제작자) 워크플로우](https://youtu.be/8lF7HmQ_RgY?t=2582)
- Boris Cherny (Claude Code 제작자) 워크플로우 — [13 Tips](tips/claude-boris-13-tips-03-jan-26.md) · [10 Tips](tips/claude-boris-10-tips-01-feb-26.md) · [12 Tips](tips/claude-boris-12-tips-12-feb-26.md) · [2 Tips](tips/claude-boris-2-tips-25-mar-26.md) · [15 Tips](tips/claude-boris-15-tips-30-mar-26.md) · [6 Tips](tips/claude-boris-6-tips-16-apr-26.md) [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny)
- Thariq (Anthropic) 워크플로우 — [Skills](tips/claude-thariq-tips-17-mar-26.md) · [Session Management](tips/claude-thariq-tips-16-apr-26.md) [![Thariq](!/tags/thariq.svg)](https://x.com/trq212)

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 🔀 CROSS-MODEL WORKFLOWS

Claude Code를 다른 모델 — Codex, Gemini, GPT, Kimi, DeepSeek, 로컬 — 과 함께 사용하는 세 가지 메커니즘:

- **Plugin** — 다른 모델의 CLI가 Claude Code 안에서 실행됨 (`/codex:review` 같은 슬래시 커맨드)
- **MCP** — Claude Code가 Model Context Protocol을 통해 다른 모델을 도구로 호출
- **Router** — Claude Code의 API 엔드포인트를 다른 제공자로 교체

방법론: [Cross-Model (Claude Code + Codex) Workflow](development-workflows/cross-model-workflow/cross-model-workflow.md) [![Implemented](!/tags/implemented.svg)](development-workflows/cross-model-workflow/cross-model-workflow.md) — Claude에서 Plan, Codex에서 QA-Review를 수행하는 수동 2-터미널 흐름.

| 이름 | ★ | 유형 | 연결 대상 | 하는 일 |
|------|---|------|------------|--------------|
| [musistudio/claude-code-router](https://github.com/musistudio/claude-code-router) | 34k | Router | OpenRouter, DeepSeek, Ollama, Gemini, Kimi, Qwen, Groq, +more | Claude Code의 API를 호환되는 모든 제공자로 라우팅, 작업별 모델 선택 지원 |
| [router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) | 32k | Router | Gemini CLI, Codex, Claude Code, Antigravity | 각 CLI를 OpenAI/Gemini/Claude/Codex 호환 API 서비스로 래핑 |
| [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) | 18k | Plugin | Codex / GPT-5 | 공식 OpenAI 플러그인: Claude Code 안에서 `/codex:review`, `/codex:adversarial-review`, `/codex:rescue` |
| [BeehiveInnovations/pal-mcp-server](https://github.com/BeehiveInnovations/pal-mcp-server) | 12k | MCP | Gemini, OpenAI, Azure, Grok, Ollama, OpenRouter (50+ models) | 멀티모델 MCP 서버 (이전 `zen-mcp-server`) — 다른 모델을 Claude 도구로 호출 |

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 🧰 SKILL COLLECTIONS

주로 `SKILL.md` 파일의 큐레이션 라이브러리로 알려진 저장소들 (위의 전체 워크플로우 방법론과는 구별됨). 스타 내림차순 정렬.

| Name | ★ | <img src="!/tags/s.svg" height="14"> |
|------|---|---|
| [mattpocock/skills](https://github.com/mattpocock/skills) | 160k | 34 |
| [anthropics/skills](https://github.com/anthropics/skills) | 159k | 17 |
| [Egonex-AI/Understand-Anything](https://github.com/Egonex-AI/Understand-Anything) | 67k | 8 |
| [wshobson/agents](https://github.com/wshobson/agents) | 38k | 149 |
| [scientific-agent-skills](https://github.com/K-Dense-AI/scientific-agent-skills) | 30k | 148 |
| [awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) | 28k | 1,497+ (큐레이션 목록) |
| [impeccable](https://github.com/pbakaus/impeccable) | 27k | 1 (디자인 도메인 레퍼런스 7개 포함) |
| [agent-skills](https://github.com/addyosmani/agent-skills) | 27k | 21 |
| [claude-skills](https://github.com/alirezarezvani/claude-skills) | 15k | 246 (9개 도메인) |
| [shanraisshan/draw-json-architecture-skill](https://github.com/shanraisshan/draw-json-architecture-skill) | 3 | 1 |

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 🤖 AGENT COLLECTIONS

주로 서브에이전트 정의(`.claude/agents/*.md`)의 큐레이션 라이브러리로 알려진 저장소들. 스타 내림차순 정렬.

| Name | ★ | <img src="!/tags/a.svg" height="14"> |
|------|---|---|
| [msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents) | 130k | 254 |
| [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) | 23k | 156 |

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 💡 TIPS AND TRICKS (83)

🚫👶 = 방치해도 됨 (do not babysit — Claude에게 맡기고 지켜보지 않아도 되는 팁)

[프롬프팅](#tips-prompting) · [계획](#tips-planning) · [컨텍스트](#tips-context) · [세션](#tips-session) · [CLAUDE.md + .claude/rules](#tips-claudemd) · [에이전트](#tips-agents) · [커맨드](#tips-commands) · [스킬](#tips-skills) · [훅](#tips-hooks) · [워크플로우](#tips-workflows) · [고급](#tips-workflows-advanced) · [Git / PR](#tips-git-pr) · [디버깅](#tips-debugging) · [유틸리티](#tips-utilities) · [매일](#tips-daily)

![Community](!/tags/community.svg)

<a id="tips-prompting"></a>■ **프롬프팅 (3)**

| 팁 | 출처 |
|-----|--------|
| Claude에게 도전시키기 — "이 변경사항으로 나를 시험해봐, 통과 전엔 PR 만들지 마" 또는 "이게 동작한다는 걸 증명해봐"라고 하고 main과 브랜치를 diff시키기 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| 어설픈 수정 후 — "지금 알게 된 모든 걸 바탕으로, 이걸 버리고 우아한 해결책을 구현해봐" 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| Claude는 대부분 버그를 스스로 고침 — 버그를 붙여넣고 "고쳐"라고만, 방법을 세세히 관리하지 말 것 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742750473720121) |

<a id="tips-planning"></a>■ **계획/스펙 (7)**

| 팁 | 출처 |
|-----|--------|
| 항상 [plan mode](https://code.claude.com/docs/en/common-workflows)로 시작 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179845336527000) |
| 최소 스펙이나 프롬프트로 시작해 Claude가 [AskUserQuestion](https://code.claude.com/docs/en/cli-reference) 도구로 당신을 인터뷰하게 한 뒤, 새 세션에서 스펙을 실행 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2005315275026260309) |
| 항상 단계별 게이트 계획을 세우고, 각 단계마다 여러 테스트(단위·자동화·통합)를 둘 것 | [![Dex](!/tags/community-dex.svg)](videos/claude-dex-mlops-community-24-mar-26.md) [![Video](!/tags/video.svg)](https://youtu.be/YwZR6tc7qYg?t=1032) |
| PRD를 모든 계층(DB + 서비스 + UI)을 관통하는 수직 슬라이스(tracer bullets)로 쪼개기 — AI는 기본적으로 수평 단계(DB 단계 → API 단계 → 프론트엔드 단계)로 나눠 end-to-end 피드백을 마지막 단계까지 지연시킴. 《실용주의 프로그래머》에서 🚫👶 | [![Matt](!/tags/community-matt.svg)](videos/claude-matt-pocock-24-apr-26.md) [![Video](!/tags/video.svg)](https://youtu.be/-QFHIoCo-Ko) |
| 두 번째 Claude를 띄워 staff engineer처럼 계획을 리뷰하게 하거나, 리뷰에 [cross-model](development-workflows/cross-model-workflow/cross-model-workflow.md)을 활용 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742745365057733) |
| 작업을 넘기기 전에 상세한 스펙을 쓰고 모호함을 줄일 것 — 구체적일수록 결과가 좋아짐 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742752566632544) |
| 프로토타입 > PRD — 스펙을 쓰는 대신 20~30개 버전을 만들기, 만드는 비용이 낮으니 많이 시도할 것 | [![Boris](!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3630) [![Video](!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3630) |

<a id="tips-context"></a>■ **컨텍스트 (5)**

| 팁 | 출처 |
|-----|--------|
| 컨텍스트 부패(rot)는 1M 컨텍스트 모델에서 ~300-400k 토큰부터 시작 — 지능이 중요한 작업은 세션이 그 이상 흘러가지 않게 할 것 | [![Thariq](!/tags/thariq.svg)](tips/claude-thariq-tips-16-apr-26.md) |
| "dumb zone"(성능 저하 구간)은 컨텍스트 ~40%부터 시작 — "결과가 나빠지는 지점에 도달한다". 초보자: "40% 미만으로 유지하고, 60%에 이르면 마무리를 고민하라". 숙련자: "공격적으로 30% 미만 유지" — 간단한 작업만 60%까지. 작업 전환 시 [/compact](https://code.claude.com/docs/en/interactive-mode) 또는 [/clear](https://code.claude.com/docs/en/cli-reference)로 리셋 | [![Dex](!/tags/community-dex.svg)](videos/claude-dex-mlops-community-24-mar-26.md) [![Video](!/tags/video.svg)](https://youtu.be/YwZR6tc7qYg?t=1541) |
| 되감기 > 수정 — 실패한 시도+수정으로 컨텍스트를 오염시키는 대신, double-Esc나 [/rewind](https://code.claude.com/docs/en/checkpointing)로 실패 시점 이전으로 돌아가 배운 것을 담아 재프롬프트 🚫👶 | [![Thariq](!/tags/thariq.svg)](tips/claude-thariq-tips-16-apr-26.md) |
| [/compact](https://code.claude.com/docs/en/interactive-mode)에 힌트를 주는 것(/compact focus on the auth refactor, drop the test debugging)이 autocompact가 발동되게 두는 것보다 나음 — 컨텍스트 부패로 auto-compact될 때 모델은 가장 지능이 낮은 상태 | [![Thariq](!/tags/thariq.svg)](tips/claude-thariq-tips-16-apr-26.md) |
| 컨텍스트 관리에 서브에이전트를 활용 — "이 도구 출력을 다시 볼까, 아니면 결론만 필요할까?"를 자문할 것 — 20번의 파일 읽기 + 12번의 grep + 3번의 막다른 길은 자식 컨텍스트에 남고, 최종 리포트만 반환됨 🚫👶 | [![Thariq](!/tags/thariq.svg)](tips/claude-thariq-tips-16-apr-26.md) |

<a id="tips-session"></a>■ **세션 관리 (6)**

| 팁 | 출처 |
|-----|--------|
| 매 턴이 분기점 — Claude가 턴을 끝내면, 기존 컨텍스트를 얼마나 이어가야 하는지에 따라 Continue, /rewind, /clear, /compact, Subagent 중에서 선택 | [![Thariq](!/tags/thariq.svg)](tips/claude-thariq-tips-16-apr-26.md) |
| 새 작업 = 새 세션 — 관련 작업(예: 방금 만든 것의 문서 작성)은 효율을 위해 컨텍스트를 재사용할 수 있지만, 진짜 새로운 작업은 새 세션이 마땅함 | [![Thariq](!/tags/thariq.svg)](tips/claude-thariq-tips-16-apr-26.md) |
| 되감기 전에 "summarize from here"로 Claude가 인수인계 메시지를 쓰게 하기 — 미래의 Claude가 과거의 Claude에게 남기는 메모처럼 | [![Thariq](!/tags/thariq.svg)](tips/claude-thariq-tips-16-apr-26.md) |
| /compact vs /clear — compact는 손실이 있지만 흐름 유지에 좋음(작업 중, 세부가 흐릿해도 OK); /clear + 요약은 더 수고롭지만 무엇을 넘길지 정밀 제어(고위험 다음 단계) | [![Thariq](!/tags/thariq.svg)](tips/claude-thariq-tips-16-apr-26.md) |
| 장시간 세션엔 recaps 활용 — Claude가 한 일과 다음 할 일의 짧은 요약, 몇 분/몇 시간 뒤 돌아올 때 유용. /config의 recaps로 비활성화 | [![Boris](!/tags/boris-cherny.svg)](tips/claude-boris-6-tips-16-apr-26.md) |
| 중요한 세션은 [/rename](https://code.claude.com/docs/en/cli-reference)하고(예: [TODO - 리팩터 작업]) 나중에 [/resume](https://code.claude.com/docs/en/cli-reference) — 여러 Claude를 동시에 돌릴 때 각 인스턴스에 라벨링 | [![Cat](!/tags/cat-wu.svg)](https://every.to/podcast/how-to-use-claude-code-like-the-people-who-built-it) |

<a id="tips-claudemd"></a>■ **CLAUDE.md + .claude/rules (8)**  <!-- 카테고리명은 파일명이므로 원형 유지 -->

| 팁 | 출처 |
|-----|--------|
| [CLAUDE.md](https://code.claude.com/docs/en/memory)는 파일당 [200줄](https://code.claude.com/docs/en/memory#write-effective-instructions) 미만을 목표로. [humanlayer는 60줄](https://www.humanlayer.dev/blog/writing-a-good-claude-md) ([그래도 100% 보장은 아님](https://www.reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/)) | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179840848597422) [![Dex](!/tags/community-dex.svg)](https://www.humanlayer.dev/blog/writing-a-good-claude-md) |
| .claude/rules/*.md는 CLAUDE.md처럼 모든 세션에 자동 로드됨 — paths: YAML frontmatter를 추가하면 Claude가 glob에 매칭되는 파일을 만질 때만 lazy-load | [![Claude](!/tags/claude.svg)](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) |
| 도메인별 CLAUDE.md 규칙을 [\<important if="..."\> 태그](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md)로 감싸 파일이 길어져도 Claude가 무시하지 않도록 | [![Dex](!/tags/community-dex.svg)](https://www.hlyr.dev/blog/stop-claude-from-ignoring-your-claude-md) |
| 모노레포엔 [여러 CLAUDE.md](best-practice/claude-memory.md) 사용 — 조상 + 자손 로딩 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2016339448863355206) |
| 큰 지침은 [.claude/rules/](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules)로 분리 | [![Claude](!/tags/claude.svg)](https://code.claude.com/docs/en/memory#organize-rules-with-clauderules) |
| 아무 개발자나 Claude를 켜고 "테스트 돌려"라고 하면 첫 시도에 되어야 함 — 안 되면 CLAUDE.md에 필수 설정/빌드/테스트 명령이 빠진 것 | [![Dex](!/tags/community-dex.svg)](https://x.com/dexhorthy/status/2034713765401551053) |
| 코드베이스를 깨끗이 유지하고 마이그레이션을 끝낼 것 — 부분 마이그레이션된 프레임워크는 모델이 잘못된 패턴을 고르게 혼란시킴 | [![Boris](!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=1112) [![Video](!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=1112) |
| harness가 강제하는 동작(속성·권한·모델)엔 [settings.json](best-practice/claude-settings.md) 사용 — attribution.commit: ""이 결정론적인데 "NEVER add Co-Authored-By"를 CLAUDE.md에 넣지 말 것 | [![davila7](!/tags/community-davila7.svg)](https://x.com/dani_avila7/status/2036182734310195550) |

<a id="tips-agents"></a><img src="!/tags/a.svg" height="14"> **에이전트 (4)**

| 팁 | 출처 |
|-----|--------|
| 범용 QA·백엔드 엔지니어 대신, [스킬](https://code.claude.com/docs/en/skills)(점진적 공개)을 가진 기능별 [서브에이전트](https://code.claude.com/docs/en/sub-agents)(추가 컨텍스트)를 둘 것 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179850139000872) |
| "use subagents"라고 말해 문제에 더 많은 컴퓨트를 투입 — 작업을 위임해 메인 컨텍스트를 깨끗하고 집중되게 유지 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| 병렬 개발엔 [tmux 에이전트 팀](https://code.claude.com/docs/en/agent-teams)과 [git worktrees](https://x.com/bcherny/status/2025007393290272904) | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2025007393290272904) |
| [test time compute](https://code.claude.com/docs/en/sub-agents) 활용 — 분리된 컨텍스트 창이 결과를 개선; 한 에이전트가 버그를 만들고 다른(같은 모델) 에이전트가 그것을 찾을 수 있음 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031151689219321886) |

<a id="tips-commands"></a><img src="!/tags/c.svg" height="14"> **커맨드 (3)**

| 팁 | 출처 |
|-----|--------|
| 워크플로우엔 [서브에이전트](https://code.claude.com/docs/en/sub-agents) 대신 [커맨드](https://code.claude.com/docs/en/slash-commands) 사용 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| 하루에 여러 번 하는 모든 "inner loop" 워크플로우엔 [슬래시 커맨드](https://code.claude.com/docs/en/slash-commands) — 반복 프롬프팅을 절약, 커맨드는 .claude/commands/에 있고 git에 커밋됨 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179847949500714) |
| 하루 한 번 이상 하는 일은 [스킬](https://code.claude.com/docs/en/skills)이나 [커맨드](https://code.claude.com/docs/en/slash-commands)로 만들기 — /techdebt, context-dump, analytics 커맨드 구축 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742748984742078) |

<a id="tips-skills"></a><img src="!/tags/s.svg" height="14"> **스킬 (9)**

| 팁 | 출처 |
|-----|--------|
| [context: fork](https://code.claude.com/docs/en/skills)로 스킬을 격리된 서브에이전트에서 실행 — 메인 컨텍스트는 중간 도구 호출이 아니라 최종 결과만 봄. agent 필드로 서브에이전트 타입을 지정 | [![Lydia](!/tags/lydia.svg)](https://x.com/lydiahallie/status/2033603164398883042) |
| 모노레포엔 [하위 폴더 스킬](reports/claude-skills-for-larger-mono-repos.md) 사용 | [![Claude](!/tags/claude.svg)](https://code.claude.com/docs/en/skills) |
| 스킬은 파일이 아니라 폴더 — [점진적 공개](https://code.claude.com/docs/en/skills)를 위해 references/, scripts/, examples/ 하위 디렉토리 사용 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 모든 스킬에 Gotchas 섹션을 구축 — 가장 신호가 강한 콘텐츠, Claude의 실패 지점을 시간이 지나며 추가 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 스킬 description 필드는 요약이 아니라 트리거 — 모델을 위해 작성("나는 언제 발동해야 하나?") | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 스킬에 뻔한 내용을 쓰지 말 것 — Claude를 기본 동작에서 벗어나게 하는 것에 집중 🚫👶 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 스킬에서 Claude를 몰아붙이지 말 것 — 처방적 단계별 지침이 아니라 목표와 제약을 줄 것 🚫👶 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| 스킬에 스크립트와 라이브러리를 포함해 Claude가 보일러플레이트를 재구성하지 않고 조합하도록 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| SKILL.md에 !command를 임베드해 동적 셸 출력을 프롬프트에 주입 — Claude가 호출 시 실행하고 모델은 결과만 봄 | [![Lydia](!/tags/lydia.svg)](https://x.com/lydiahallie/status/2034337963820327017) |

<a id="tips-hooks"></a>■ **훅 (5)**

| 팁 | 출처 |
|-----|--------|
| 스킬에 [on-demand 훅](https://code.claude.com/docs/en/skills) 사용 — /careful은 파괴적 명령을 차단, /freeze는 디렉토리 밖 편집을 차단 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| PreToolUse 훅으로 [스킬 사용량을 측정](https://code.claude.com/docs/en/skills)해 인기 있거나 저발동하는 스킬을 찾기 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| [PostToolUse 훅](https://code.claude.com/docs/en/hooks)으로 코드 자동 포맷 — Claude가 잘 포맷된 코드를 생성하고, 훅이 CI 실패 방지를 위한 마지막 10%를 처리 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179852047335529) |
| 훅을 통해 [권한 요청](https://code.claude.com/docs/en/hooks)을 Opus로 라우팅 — 공격을 스캔하고 안전한 것은 자동 승인하게 하기 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742755737555434) |
| [Stop 훅](https://code.claude.com/docs/en/hooks)으로 턴 끝에 Claude가 계속 진행하거나 작업을 검증하도록 넛지 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701059253874861) |

<a id="tips-workflows"></a>■ **워크플로우 (5)**

| 팁 | 출처 |
|-----|--------|
| [/model](https://code.claude.com/docs/en/model-config)로 모델·추론 선택, [/context](https://code.claude.com/docs/en/interactive-mode)로 컨텍스트 사용량 확인, [/usage](https://code.claude.com/docs/en/costs)로 플랜 한도 확인, [/extra-usage](https://code.claude.com/docs/en/interactive-mode)로 초과 청구 설정, [/config](https://code.claude.com/docs/en/settings)로 설정 구성 — plan mode엔 Opus, 코드엔 Sonnet을 써서 둘의 장점을 모두 | [![Cat](!/tags/cat-wu.svg)](https://x.com/_catwu/status/1955694117264261609) |
| [/config](https://code.claude.com/docs/en/settings)에서 항상 [thinking mode](https://code.claude.com/docs/en/model-config) true(추론을 보기)와 [Output Style](https://code.claude.com/docs/en/output-styles) Explanatory(★ Insight 박스로 상세 출력 보기)를 사용해 Claude의 결정을 더 잘 이해 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179838864666847) |
| 프롬프트에 ultrathink 키워드를 써서 [고강도 추론](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking#tips-and-best-practices) | [![Claude](!/tags/claude.svg)](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking#tips-and-best-practices) |
| /focus 모드는 모든 중간 작업을 숨기고 최종 결과만 표시 — 모델이 올바른 명령을 실행하리라 믿고 결과만 보기 (/focus로 토글) | [![Boris](!/tags/boris-cherny.svg)](tips/claude-boris-6-tips-16-apr-26.md) |
| Opus 4.7의 적응형 사고로 effort 레벨을 조정 — 속도와 토큰 절약엔 low, 최고 지능엔 max (슬라이더: low · medium · high · xhigh · max) | [![Boris](!/tags/boris-cherny.svg)](tips/claude-boris-6-tips-16-apr-26.md) |

<a id="tips-workflows-advanced"></a>■ **워크플로우 고급 (9)**

| 팁 | 출처 |
|-----|--------|
| 아키텍처를 이해하기 위해 ASCII 다이어그램을 적극 활용 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742759218794768) |
| 로컬 반복 모니터링(최대 7일)엔 [/loop](https://code.claude.com/docs/en/scheduled-tasks) · 머신이 꺼져 있어도 도는 클라우드 기반 반복 작업엔 [/schedule](https://code.claude.com/docs/en/routines) | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038454341884154269) |
| 장시간 자율 작업엔 [Ralph Wiggum 플러그인](https://github.com/shanraisshan/ralph-wiggum-self-evolving-loop) | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179858435281082) |
| dangerously-skip-permissions 대신 와일드카드 문법을 쓴 [/permissions](https://code.claude.com/docs/en/permissions) (Bash(npm run *), Edit(/docs/**)) | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2007179854077407667) |
| [/sandbox](https://code.claude.com/docs/en/sandboxing)로 파일·네트워크 격리를 통해 권한 프롬프트 감소 — 내부적으로 84% 감소 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700506465579443) [![Cat](!/tags/cat-wu.svg)](https://creatoreconomy.so/p/inside-claude-code-how-an-ai-native-actually-works-cat-wu) |
| [제품 검증](https://code.claude.com/docs/en/skills) 스킬(signup-flow-driver, checkout-verifier)에 투자 — 일주일 들여 완성할 가치가 있음 | [![Thariq](!/tags/thariq.svg)](https://x.com/trq212/status/2033949937936085378) |
| dangerously-skip-permissions 대신 [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 사용 — 모델 기반 분류기가 각 명령의 안전성을 판단해 자동 승인하고, 위험하면 멈추고 물음. Shift+Tab으로 Ask → Plan → Auto 모드 순환 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](tips/claude-boris-6-tips-16-apr-26.md) |
| /less-permission-prompts 스킬로 반복해서 프롬프트되는 안전한 bash/MCP 명령을 세션 기록에서 스캔한 뒤, [settings](best-practice/claude-settings.md)에 붙여넣을 추천 allowlist를 받기 | [![Boris](!/tags/boris-cherny.svg)](tips/claude-boris-6-tips-16-apr-26.md) |
| (1) bash/브라우저/컴퓨터 사용으로 end-to-end 테스트 (2) /simplify 실행 (3) PR 올리기를 하는 /go 스킬을 구축 — 돌아왔을 때 코드가 동작함을 알 수 있도록 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](tips/claude-boris-6-tips-16-apr-26.md) |

<a id="tips-git-pr"></a>■ **Git / PR (5)**  <!-- Git/PR은 고유명사로 원형 유지 -->

| 팁 | 출처 |
|-----|--------|
| PR을 작고 집중되게 유지 — [p50 118줄](tips/claude-boris-2-tips-25-mar-26.md) (하루에 141개 PR, 4.5만 줄 변경), PR당 기능 하나, 리뷰·되돌리기가 쉬움 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| PR은 항상 [squash merge](tips/claude-boris-2-tips-25-mar-26.md) — 깨끗한 선형 히스토리, 기능당 커밋 하나, git revert·git bisect가 쉬움 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038552880018538749) |
| 자주 커밋 — 최소 시간당 한 번, 작업이 완료되는 즉시 커밋 | ![Shayan](!/tags/community-shayan.svg) |
| 동료의 PR에 [@claude](https://github.com/apps/claude)를 태그해 반복되는 리뷰 피드백에 대한 lint 규칙을 자동 생성 — 코드 리뷰에서 자신을 자동화 🚫👶 | [![Boris](!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=2715) [![Video](!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=2715) |
| 멀티에이전트 PR 분석엔 [/code-review](https://code.claude.com/docs/en/code-review) — 병합 전에 버그·보안 취약점·회귀를 포착 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2031089411820228645) |

<a id="tips-debugging"></a>■ **디버깅 (6)**

| 팁 | 출처 |
|-----|--------|
| 어떤 이슈에 막힐 때마다 스크린샷을 찍어 Claude와 공유하는 습관을 들일 것 | ![Shayan](!/tags/community-shayan.svg) |
| mcp([Claude in Chrome](https://code.claude.com/docs/en/chrome), [Playwright](https://github.com/microsoft/playwright-mcp), [Chrome DevTools](https://developer.chrome.com/blog/chrome-devtools-mcp))를 사용해 Claude가 크롬 콘솔 로그를 스스로 보게 하기 | [![Claude](!/tags/claude.svg)](https://code.claude.com/docs/en/chrome) |
| 더 나은 디버깅을 위해 (로그를 보고 싶은) 터미널은 항상 Claude에게 백그라운드 작업으로 실행하도록 요청 | ![Shayan](!/tags/community-shayan.svg) |
| [/doctor](https://code.claude.com/docs/en/cli-reference)로 설치·인증·설정 문제를 진단 | ![Shayan](!/tags/community-shayan.svg) |
| QA엔 [cross-model](development-workflows/cross-model-workflow/cross-model-workflow.md) 활용 — 예: 계획·구현 리뷰에 [Codex](https://github.com/shanraisshan/codex-cli-best-practice) | ![Shayan](!/tags/community-shayan.svg) |
| 에이전틱 검색(glob + grep)이 RAG를 능가 — 코드가 동기화에서 벗어나고 권한이 복잡하기 때문에 Claude Code는 벡터 DB를 시도했다 폐기함 | [![Boris](!/tags/boris-cherny.svg)](https://youtu.be/julbw1JuAz0?t=3095) [![Video](!/tags/video.svg)](https://youtu.be/julbw1JuAz0?t=3095) |

<a id="tips-utilities"></a>■ **유틸리티 (5)**

| 팁 | 출처 |
|-----|--------|
| IDE([VS Code](https://code.visualstudio.com/)/[Cursor](https://www.cursor.com/)) 대신 [iTerm](https://iterm2.com/)/[Ghostty](https://ghostty.org/)/[tmux](https://github.com/tmux/tmux) 터미널 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2017742753971769626) |
| 음성 프롬프팅엔 [/voice](https://code.claude.com/docs/en/voice-dictation)나 [Wispr Flow](https://wisprflow.ai) (10배 생산성) | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2038454362226467112) |
| Claude 피드백엔 [claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks) | ![Shayan](!/tags/community-shayan.svg) |
| 컨텍스트 인식과 빠른 압축엔 [status line](https://github.com/shanraisshan/claude-code-status-line) | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021700784019452195) ![Shayan](!/tags/community-shayan.svg) |
| 개인화된 경험을 위해 [Plans Directory](best-practice/claude-settings.md#plans-directory), [Spinner Verbs](best-practice/claude-settings.md#display--ux) 같은 [settings.json](best-practice/claude-settings.md) 기능을 탐색 | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny/status/2021701145023197516) |

<a id="tips-daily"></a>■ **매일 (2)**

| 팁 | 출처 |
|-----|--------|
| Claude Code를 매일 [업데이트](https://code.claude.com/docs/en/setup) | ![Shayan](!/tags/community-shayan.svg) |
| 하루를 [changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) 읽기로 시작 | ![Shayan](!/tags/community-shayan.svg) |

![Boris Cherny + Team](!/tags/claude.svg)

| Article / Tweet | Source |
|-----------------|--------|
| [6 Tips for Getting More Out of Opus 4.7 (Boris) \| 16/Apr/26](tips/claude-boris-6-tips-16-apr-26.md) | [Tweet](https://x.com/bcherny) |
| [Session Management & 1M Context (Thariq) \| 16/Apr/26](tips/claude-thariq-tips-16-apr-26.md) | [Tweet](https://x.com/trq212) |
| [15 Hidden & Under-Utilized Features in Claude Code (Boris) \| 30/Mar/26](tips/claude-boris-15-tips-30-mar-26.md) | [Tweet](https://x.com/bcherny/status/2038454336355999749) |
| [Squash Merging & PR Size Distribution (Boris) \| 25/Mar/26](tips/claude-boris-2-tips-25-mar-26.md) | [Tweet](https://x.com/bcherny/status/2038552880018538749) |
| [Lessons from Building Claude Code: How We Use Skills (Thariq) \| 17/Mar/26](tips/claude-thariq-tips-17-mar-26.md) | [Article](https://x.com/trq212/status/2033949937936085378) |
| [Code Review & Test Time Compute (Boris) \| 10/Mar/26](tips/claude-boris-2-tips-10-mar-26.md) | [Tweet](https://x.com/bcherny/status/2031089411820228645) |
| /loop — schedule recurring tasks for up to 3 days (Boris) \| 07 Mar 2026 | [Tweet](https://x.com/bcherny/status/2030193932404150413) |
| AskUserQuestion + ASCII Markdowns (Thariq) \| 28 Feb 2026 | [Tweet](https://x.com/trq212/status/2027543858289250472) |
| Seeing like an Agent - lessons from building Claude Code (Thariq) \| 28 Feb 2026 | [Article](https://x.com/trq212/status/2027463795355095314) |
| Git Worktrees - 5 ways how boris is using \| 21 Feb 2026 | [Tweet](https://x.com/bcherny/status/2025007393290272904) |
| Lessons from Building Claude Code: Prompt Caching Is Everything (Thariq) \| 20 Feb 2026 | [Article](https://x.com/trq212/status/2024574133011673516) |
| [12 ways how people are customizing their claudes (Boris) \| 12/Feb/26](tips/claude-boris-12-tips-12-feb-26.md) | [Tweet](https://x.com/bcherny/status/2021699851499798911) |
| [10 tips for using Claude Code from the team (Boris) \| 01/Feb/26](tips/claude-boris-10-tips-01-feb-26.md) | [Tweet](https://x.com/bcherny/status/2017742741636321619) |
| [How I use Claude Code — 13 tips from my surprisingly vanilla setup (Boris) \| 03/Jan/26](tips/claude-boris-13-tips-03-jan-26.md) | [Tweet](https://x.com/bcherny/status/2007179832300581177) |
| Ask Claude to interview you using AskUserQuestion tool (Thariq) \| 28/Dec/25 | [Tweet](https://x.com/trq212/status/2005315275026260309) |
| Always use plan mode, give Claude a way to verify, use /code-review (Boris) \| 27/Dec/25 | [Tweet](https://x.com/bcherny/status/2004711722926616680) |

#### Tips from Claude code CLI binary

[Spinner Verbs & Tips (extracted from CLI binary v2.1.121)](reports/claude-spinner-verbs-and-tips.md)

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 🎬 VIDEOS / PODCASTS

| 비디오 / 팟캐스트 | 출처 | YouTube |
|-----------------|--------|--------|
| From Vibe Coding to Agentic Engineering (Andrej) \| 02 May 2026 \| AI Engineer | [![Karpathy](!/tags/community-karpathy.svg)](https://x.com/karpathy) | [YouTube](https://www.youtube.com/watch?v=96jN2OCOfLs) |
| Full Walkthrough: Workflow for AI Coding (Matt) \| 24 Apr 2026 \| Matt Pocock | [![Matt](!/tags/community-matt.svg)](https://x.com/mattpocockuk) | [YouTube](https://youtu.be/-QFHIoCo-Ko) |
| Everything We Got Wrong About Research-Plan-Implement (Dex) \| 24 Mar 2026 \| MLOps Community | [![Dex](!/tags/community-dex.svg)](https://x.com/daborhyde) | [YouTube](https://youtu.be/YwZR6tc7qYg) |
| Building Claude Code with Boris Cherny (Boris) \| 04 Mar 2026 \| The Pragmatic Engineer | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/julbw1JuAz0) |
| Head of Claude Code: What happens after coding is solved (Boris) \| 19 Feb 2026 \| Lenny's Podcast | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/We7BZVKbCVw) |
| Inside Claude Code With Its Creator Boris Cherny (Boris) \| 17 Feb 2026 \| Y Combinator | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/PQU9o_5rHC4) |
| Boris Cherny (Creator of Claude Code) On What Grew His Career (Boris) \| 15 Dec 2025 \| Ryan Peterman | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/AmdLVWMdjOk) |
| The Secrets of Claude Code From the Engineers Who Built It (Cat) \| 29 Oct 2025 \| Every | [![Boris](!/tags/boris-cherny.svg)](https://x.com/bcherny) | [YouTube](https://youtu.be/IDSAMqip6ms) |

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 🔔 SUBSCRIBE

| 출처 | 이름 | 배지 |
|--------|------|-------|
| ![Reddit](https://img.shields.io/badge/-FF4500?style=flat&logo=reddit&logoColor=white) | [r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/), [r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/), [r/Anthropic](https://www.reddit.com/r/Anthropic/) | ![Boris + Team](!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Claude](https://x.com/claudeai), [Claude Devs](https://x.com/ClaudeDevs), [Anthropic](https://x.com/AnthropicAI), [Boris](https://x.com/bcherny), [Thariq](https://x.com/trq212), [Cat](https://x.com/_catwu), [Lydia](https://x.com/lydiahallie), [Noah](https://x.com/noahzweben), [Anthony](https://x.com/amorriscode), [Alex](https://x.com/alexalbert__), [Kenneth](https://x.com/neilhtennek) | ![Boris + Team](!/tags/claude.svg) |
| ![X](https://img.shields.io/badge/-000?style=flat&logo=x&logoColor=white) | [Jesse Kriss](https://x.com/obra) ([Superpowers](https://github.com/obra/superpowers)), [Affaan Mustafa](https://x.com/affaanmustafa) ([ECC](https://github.com/affaan-m/everything-claude-code)), [Garry Tan](https://x.com/garrytan) ([gstack](https://github.com/garrytan/gstack)), [Dex Horthy](https://x.com/dexhorthy) ([HumanLayer](https://github.com/humanlayer/humanlayer)), [Kieran Klaassen](https://x.com/kieranklaassen) ([Compound Eng](https://github.com/EveryInc/compound-engineering-plugin)), [Tabish Gilani](https://x.com/0xTab) ([OpenSpec](https://github.com/Fission-AI/OpenSpec)), [Brian McAdams](https://x.com/BMadCode) ([BMAD](https://github.com/bmad-code-org/BMAD-METHOD)), [Lex Christopherson](https://x.com/official_taches) ([GSD](https://github.com/gsd-build/get-shit-done)), [Matt Pocock](https://x.com/mattpocockuk) ([Skills](https://github.com/mattpocock/skills)), [Dani Avila](https://x.com/dani_avila7) ([CC Templates](https://github.com/davila7/claude-code-templates)), [Dan Shipper](https://x.com/danshipper) ([Every](https://every.to/)), [Andrej Karpathy](https://x.com/karpathy) ([AutoResearch](https://x.com/karpathy/status/2015883857489522876)), [Peter Steinberger](https://x.com/steipete) ([OpenClaw](https://x.com/openclaw)), [Sigrid Jin](https://x.com/realsigridjin) ([claw-code](https://github.com/ultraworkers/claw-code)), [Yeachan Heo](https://x.com/bellman_ych) ([oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)) | ![Community](!/tags/community.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Anthropic](https://www.youtube.com/@anthropic-ai) | ![Boris + Team](!/tags/claude.svg) |
| ![YouTube](https://img.shields.io/badge/-F00?style=flat&logo=youtube&logoColor=white) | [Lenny's Podcast](https://www.youtube.com/@LennysPodcast), [Y Combinator](https://www.youtube.com/@ycombinator), [The Pragmatic Engineer](https://www.youtube.com/@pragmaticengineer), [Ryan Peterman](https://www.youtube.com/@ryanlpeterman), [Every](https://www.youtube.com/@every_media), [MLOps Community](https://www.youtube.com/@MLOps) | ![Community](!/tags/community.svg) |

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## ☠️ STARTUPS / BUSINESSES

| Claude | 대체된 서비스 |
|-|-|
|[**Code Review**](https://code.claude.com/docs/en/code-review)|[Greptile](https://greptile.com), [CodeRabbit](https://coderabbit.ai), [Devin Review](https://devin.ai), [OpenDiff](https://opendiff.com), [Cursor BugBot](https://bugbot.dev)|
|[**Voice Dictation**](https://code.claude.com/docs/en/voice-dictation)|[Wispr Flow](https://wisprflow.ai), [SuperWhisper](https://superwhisper.com/)|
|[**Remote Control**](https://code.claude.com/docs/en/remote-control)|[OpenClaw](https://openclaw.ai/)
|[**Claude in Chrome**](https://code.claude.com/docs/en/chrome)|[Playwright MCP](https://github.com/microsoft/playwright-mcp), [Chrome DevTools MCP](https://developer.chrome.com/blog/chrome-devtools-mcp)|
|[**Computer Use**](https://docs.anthropic.com/en/docs/agents-and-tools/computer-use)|[OpenAI CUA](https://openai.com/index/computer-using-agent/)|
|[**Cowork**](https://claude.com/blog/cowork-research-preview)|[ChatGPT Agent](https://openai.com/chatgpt/agent/), [Perplexity Computer](https://www.perplexity.ai/computer/), [Manus](https://manus.im)|
|[**Tasks**](https://x.com/trq212/status/2014480496013803643)|[Beads](https://github.com/steveyegge/beads)
|[**Plan Mode**](https://code.claude.com/docs/en/common-workflows)|[Agent OS](https://github.com/buildermethods/agent-os)|
|[**Design**](https://claude.com/design)|[Figma](https://figma.com), [Framer](https://framer.com), [Sketch](https://sketch.com), [v0](https://v0.dev)|
|[**Agent SDK**](https://code.claude.com/docs/en/agent-sdk/overview)|[LangChain](https://langchain.com), [LangGraph](https://www.langchain.com/langgraph), [CrewAI](https://www.crewai.com), [AutoGen](https://github.com/microsoft/autogen), [OpenAI Assistants API](https://platform.openai.com/docs/assistants/overview)|
|[**Skills / Plugins**](https://code.claude.com/docs/en/plugins)|YC AI wrapper startups ([reddit](https://reddit.com/r/ClaudeAI/comments/1r6bh4d/claude_code_skills_are_basically_yc_ai_startup/))|

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

<a id="billion-dollar-questions"></a>
![Billion-Dollar Questions](!/tags/billion-dollar-questions.svg)

*답을 알고 계시면 shanraisshan@gmail.com 으로 알려주세요*

**메모리 & 지침 (4)**

1. CLAUDE.md 안에 정확히 무엇을 넣고 — 무엇을 빼야 하는가?
2. 이미 CLAUDE.md가 있다면, 별도의 constitution.md나 rules.md가 정말 필요한가?
3. CLAUDE.md를 얼마나 자주 갱신해야 하며, 언제 낡았는지(stale) 어떻게 아는가?
4. 왜 Claude는 여전히 CLAUDE.md 지침을 무시하는가 — 대문자로 MUST라고 써도? ([reddit](https://reddit.com/r/ClaudeCode/comments/1qn9pb9/claudemd_says_must_use_agent_claude_ignores_it_80/))

**에이전트, 스킬 & 워크플로우 (6)**

1. 커맨드 vs 에이전트 vs 스킬을 언제 써야 하며 — 언제는 그냥 기본 Claude Code가 더 나은가?
2. 모델이 개선됨에 따라 에이전트·커맨드·워크플로우를 얼마나 자주 갱신해야 하는가?
3. 제너럴리스트 서브에이전트를 둘 것인가, 기능별/역할별 에이전트를 둘 것인가? 서브에이전트에 상세한 페르소나를 주면 품질이 좋아지는가, 그리고 리서치/비전용 "완벽한 페르소나 프롬프트"는 어떤 모습인가?
4. Claude Code 내장 plan mode에 의존할 것인가 — 아니면 팀 워크플로우를 강제하는 자체 계획 커맨드/에이전트를 만들 것인가?
5. 개인 스킬(예: 자신의 코딩 스타일이 담긴 /implement)이 있을 때, 커뮤니티 스킬(예: /simplify)을 충돌 없이 어떻게 통합하는가 — 그리고 둘이 충돌하면 누가 이기는가?
6. 이제 다 왔는가? 기존 코드베이스를 스펙으로 변환하고, 코드를 지운 뒤, AI가 그 스펙만으로 정확히 같은 코드를 재생성할 수 있는가?

**스펙 & 문서화 (3)**

1. 저장소의 모든 기능은 마크다운 파일 형태의 스펙을 가져야 하는가?
2. 새 기능이 구현될 때 스펙이 낡지 않으려면 얼마나 자주 갱신해야 하는가?
3. 새 기능을 구현할 때, 다른 기능의 스펙에 미치는 파급 효과(ripple effect)를 어떻게 처리하는가?

### 🤔 [Does code matter?](https://github.com/shanraisshan/agentic-engineering)

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## REPORTS

<p align="center">
  <a href="reports/claude-agent-sdk-vs-cli-system-prompts.md"><img src="https://img.shields.io/badge/Agent_SDK_vs_CLI-555?style=for-the-badge" alt="Agent SDK vs CLI"></a>
  <a href="reports/claude-in-chrome-v-chrome-devtools-mcp.md"><img src="https://img.shields.io/badge/Browser_Automation_MCP-555?style=for-the-badge" alt="Browser Automation MCP"></a>
  <a href="reports/claude-global-vs-project-settings.md"><img src="https://img.shields.io/badge/Global_vs_Project_Settings-555?style=for-the-badge" alt="Global vs Project Settings"></a>
  <a href="reports/claude-skills-for-larger-mono-repos.md"><img src="https://img.shields.io/badge/Skills_in_Monorepos-555?style=for-the-badge" alt="Skills in Monorepos"></a>
  <br>
  <a href="reports/claude-agent-memory.md"><img src="https://img.shields.io/badge/Agent_Memory-555?style=for-the-badge" alt="Agent Memory"></a>
  <a href="reports/claude-advanced-tool-use.md"><img src="https://img.shields.io/badge/Advanced_Tool_Use-555?style=for-the-badge" alt="Advanced Tool Use"></a>
  <a href="reports/claude-usage-and-rate-limits.md"><img src="https://img.shields.io/badge/Usage_&_Rate_Limits-555?style=for-the-badge" alt="Usage & Rate Limits"></a>
  <a href="reports/claude-agent-command-skill.md"><img src="https://img.shields.io/badge/Agents_vs_Commands_vs_Skills-555?style=for-the-badge" alt="Agents vs Commands vs Skills"></a>
  <br>
  <a href="reports/llm-day-to-day-degradation.md"><img src="https://img.shields.io/badge/LLM_Degradation-555?style=for-the-badge" alt="LLM Degradation"></a>
  <a href="reports/why-harness-is-important.md"><img src="https://img.shields.io/badge/Why_Harness_is_Important-555?style=for-the-badge" alt="Why Harness is Important"></a>
  <a href="reports/claude-spinner-verbs-and-tips.md"><img src="https://img.shields.io/badge/Spinner_Verbs_&_Tips-555?style=for-the-badge" alt="Spinner Verbs & Tips"></a>
</p>

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

<a id="how-to-use"></a>

## <img src="!/tags/how-to-use-hd.svg" alt="How to Use">

다음 단계를 따라 이 저장소를 최대한 활용하세요:

1. **이 저장소를 워크플로우나 스킬이 아니라 "강의"로 읽으세요.** 우선은 레퍼런스 자료이고, 실행은 나중입니다.
2. **Claude를 챗봇처럼 쓰지 마세요.** primitives — 에이전트, 커맨드, 스킬, 훅 — 을 배우고, 이를 조합해 자신만의 워크플로우를 만드세요.
3. **[`/weather-orchestrator`](orchestration-workflow/orchestration-workflow.md)를 실행**해 완전한 command → agent → skill 흐름을 확인하세요. 계획부터 배포까지 어떤 개발 워크플로우에도 템플릿으로 활용하세요.
4. **작업 중 커스텀 훅 사운드를 들어보세요.** 그 구현은 전용 [Claude Code Hooks 저장소](https://github.com/shanraisshan/claude-code-hooks)에 있으며, [Agent Teams](implementation/claude-agent-teams-implementation.md) 같은 다른 패턴은 이 저장소의 `implementation/` 디렉토리에 있습니다.
5. **고급 주제와 그 구현을** [🔥 Hot](#-hot) 하위 표에서 배우세요 — 예를 들어 [Ralph Wiggum self-evolving loop](https://github.com/shanraisshan/ralph-wiggum-self-evolving-loop)는 클론해서 패턴을 처음부터 끝까지 볼 수 있는 완전한 동작 저장소입니다.
6. **당신의 실제 프로젝트에서 Claude에게 [tips and tricks](#-tips-and-tricks-83) 섹션을 가리키고** 수정 제안을 요청하세요 — 특히 `CLAUDE.md`를 어떻게 재구성할지. 모든 팁은 Claude 팀 또는 커뮤니티에서 나왔습니다.
7. **[Subscribe 섹션](#-subscribe)의 Reddit·YouTube 채널을 구독**해 커뮤니티를 따라가세요.

**🎬 비디오**

<a href="https://www.youtube.com/watch?v=AkAhkalkRY4"><img src="!/thumbnail/video-1.png" alt="Watch on YouTube" width="240"></a>
<a href="https://youtu.be/lPjhM6BBK0Q"><img src="!/thumbnail/video-2.png" alt="Watch on YouTube" width="240"></a>

**📊 프레젠테이션**

<a href="https://github.com/shanraisshan/claude-code-best-practice/tree/main/presentation/2026-04-25-gdg-kolachi-cli-claude-code-gemini"><img src="!/thumbnail/presentation-1.png" alt="Claude Code & Gemini CLI — GDG Kolachi" width="240"></a>

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

<p align="center">
  <a href="https://github.com/trending?since=monthly"><img src="!/root/github-trending.png" alt="GitHub Trending" width="1200"></a><br>
  ✨2026년 3월 GitHub 트렌딩✨
</p>

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=shanraisshan/claude-code-best-practice&type=Date&v=2)](https://star-history.com/#shanraisshan/claude-code-best-practice&Date)

<a href="https://github.com/shanraisshan/claude-code-best-practice/stargazers"><img src="https://img.shields.io/github/stars/shanraisshan/claude-code-best-practice?style=flat&label=%E2%98%85&labelColor=555&color=white" alt="GitHub Stars" align="center"></a> 스타 그리고 계속 증가 중

## Other Repos

<table>
<tr>
<td align="center" width="140">
  <a href="https://github.com/shanraisshan/claude-code-hooks"><img src="!/claude-speaking.svg" alt="Claude Code Hooks" width="64" height="64"></a><br>
  <a href="https://github.com/shanraisshan/claude-code-hooks"><strong>Claude Code<br>Hooks</strong></a>
</td>
<td align="center" width="140">
  <a href="https://github.com/shanraisshan/codex-cli-best-practice"><img src="!/codex-jumping.svg" alt="Codex CLI Best Practice" width="64" height="64"></a><br>
  <a href="https://github.com/shanraisshan/codex-cli-best-practice"><strong>Codex CLI<br>Best Practice</strong></a>
</td>
<td align="center" width="140">
  <a href="https://github.com/shanraisshan/codex-cli-hooks"><img src="!/codex-speaking.svg" alt="Codex CLI Hooks" width="64" height="64"></a><br>
  <a href="https://github.com/shanraisshan/codex-cli-hooks"><strong>Codex CLI<br>Hooks</strong></a>
</td>
<td align="center" width="140">
  <a href="https://github.com/shanraisshan/gemini-cli-best-practice"><img src="!/gemini-jumping.svg" alt="Gemini CLI Best Practice" width="64" height="64"></a><br>
  <a href="https://github.com/shanraisshan/gemini-cli-best-practice"><strong>Gemini CLI<br>Best Practice</strong></a>
</td>
<td align="center" width="140">
  <a href="https://github.com/shanraisshan/gemini-cli-hooks"><img src="!/gemini-speaking.svg" alt="Gemini CLI Hooks" width="64" height="64"></a><br>
  <a href="https://github.com/shanraisshan/gemini-cli-hooks"><strong>Gemini CLI<br>Hooks</strong></a>
</td>
</tr>
</table>

## Developed by

![Developed by](!/tags/developed-by.svg)

> | # | 워크플로우 | 설명 |
> |---|----------|-------------|
> | 1 | /workflows:development-workflows | 10개 워크플로우 저장소를 병렬로 조사해 DEVELOPMENT WORKFLOWS 표와 워크플로우 간 분석 리포트를 갱신 |
> | 2 | /workflows:skill-collections | 5개 스킬 컬렉션 저장소를 병렬로 조사해 SKILL COLLECTIONS 표를 갱신 |
> | 3 | /workflows:agent-collections | 모든 에이전트 컬렉션 저장소를 병렬로 조사해 AGENT COLLECTIONS 표를 갱신 |
> | 4 | /workflows:best-practice:workflow-concepts | 최신 Claude Code 기능·개념으로 README CONCEPTS 섹션을 갱신 |
> | 5 | /workflows:best-practice:workflow-claude-settings | Claude Code settings 리포트 변경을 추적하고 갱신이 필요한 부분을 찾기 |
> | 6 | /workflows:best-practice:workflow-claude-subagents | Claude Code subagents 리포트 변경을 추적하고 갱신이 필요한 부분을 찾기 |
> | 7 | /workflows:best-practice:workflow-claude-commands | Claude Code commands 리포트 변경을 추적하고 갱신이 필요한 부분을 찾기 |
> | 8 | /workflows:best-practice:workflow-claude-skills | Claude Code skills 리포트 변경을 추적하고 갱신이 필요한 부분을 찾기 |

## Extras

[![Claude for OSS](!/tags/claude-for-oss.svg)](https://claude.com/contact-sales/claude-for-oss)
[![Claude Community Ambassador](!/tags/claude-community-ambassador.svg)](https://claude.com/community/ambassadors)
[![Claude Certified Architect](!/tags/claude-certified-architect.svg)](https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request)
[![Anthropic Academy](!/tags/anthropic-academy.svg)](https://anthropic.skilljar.com/)
[![Join Claude Pakistan community on WhatsApp](!/tags/whatsapp-claude-pakistan.svg)](https://chat.whatsapp.com/BDUV2stIS0c7X5uY7RY6nS)

<p align="center">
  <img src="!/claude-jumping.svg" alt="section divider" width="60" height="50">
</p>

## <img src="!/tags/sponsor-heart.svg" width="22" height="22" align="center"> Sponsor My Work

제 작업이 마음에 드셨다면, 두드 파티(doodh patti, 밀크티) 🍵 한 잔 사주세요

<a href="https://buy.polar.sh/polar_cl_R6wjUESl8RiJD0iVaTyStBUV6WNuYvDmLJ0si1XXj4C"><img src="!/tags/polar.svg" alt="Polar" width="40" height="40" align="center"></a> <a href="https://buy.polar.sh/polar_cl_R6wjUESl8RiJD0iVaTyStBUV6WNuYvDmLJ0si1XXj4C"><strong>Polar</strong></a>

**헤더에 브랜드를 노출하고 싶으신가요?** 헤더 게재가 가능합니다 — [shanraisshan@gmail.com](mailto:shanraisshan@gmail.com)으로 이메일 주세요.
