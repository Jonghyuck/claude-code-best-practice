<!--
  이 문서는 best-practice/claude-subagents.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Sub-agents Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Jul%2011%2C%202026%2011%3A36%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.207-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-subagents-implementation.md)

Claude Code 서브에이전트 — frontmatter 필드와 공식 내장 에이전트 유형.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter Fields (16)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | 소문자와 하이픈을 사용하는 고유 식별자 |
| `description` | string | Yes | 언제 호출할지. Claude가 자동으로 호출하게 하려면 `"PROACTIVELY"`를 사용 |
| `tools` | string/list | No | 쉼표로 구분된 도구 허용 목록(예: `Read, Write, Edit, Bash`). 생략하면 모든 도구를 상속. 스폰 가능한 서브에이전트를 제한하는 `Agent(agent_type)` 문법을 지원하며, 이전 별칭인 `Task(agent_type)`도 여전히 동작 |
| `disallowedTools` | string/list | No | 거부할 도구. 상속되거나 지정된 목록에서 제거됨 |
| `model` | string | No | 사용할 모델: `sonnet`, `opus`, `haiku`, 전체 모델 ID(예: `claude-opus-4-6`), 또는 `inherit`(기본값: `inherit`) |
| `permissionMode` | string | No | 권한 모드: `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, 또는 `plan` |
| `maxTurns` | integer | No | 서브에이전트가 멈추기 전까지 수행할 에이전트 턴의 최대 횟수 |
| `skills` | list | No | 시작 시 에이전트 컨텍스트에 미리 로드할 스킬 이름(단순히 사용 가능하게 만드는 것이 아니라 전체 내용이 주입됨) |
| `mcpServers` | list | No | 이 서브에이전트를 위한 MCP 서버 — 서버 이름 문자열 또는 인라인 `{name: config}` 객체 |
| `hooks` | object | No | 이 서브에이전트에 한정된 라이프사이클 훅. 모든 훅 이벤트를 지원하며, `PreToolUse`, `PostToolUse`, `Stop`이 가장 흔히 쓰임 |
| `memory` | string | No | 영속 메모리 범위: `user`, `project`, 또는 `local` |
| `background` | boolean | No | 항상 백그라운드 작업으로 실행하려면 `true`로 설정(기본값: `false`) |
| `effort` | string | No | 이 서브에이전트가 활성일 때의 노력 수준 재정의: `low`, `medium`, `high`, `xhigh`, `max`(Opus 4.6 전용). 기본값: 세션에서 상속 |
| `isolation` | string | No | 임시 git worktree에서 실행하려면 `"worktree"`로 설정(변경이 없으면 자동 정리됨) |
| `initialPrompt` | string | No | 이 에이전트가 메인 세션 에이전트로 실행될 때(`--agent` 또는 `agent` 설정을 통해) 첫 사용자 턴으로 자동 제출됨. 커맨드와 스킬이 처리됨. 사용자가 제공한 프롬프트 앞에 추가됨 |
| `color` | string | No | 작업 목록과 트랜스크립트에서 서브에이전트를 표시할 색상: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, 또는 `cyan` |

---

## ![Official](../!/tags/official.svg) **(5)**

| # | Agent | Model | Tools | Description |
|---|-------|-------|-------|-------------|
| 1 | `general-purpose` | inherit | All | 복잡한 다단계 작업 — 리서치, 코드 검색, 자율 작업을 위한 기본 에이전트 유형 |
| 2 | `Explore` | haiku | Read-only (no Write, Edit) | 빠른 코드베이스 검색 및 탐색 — 파일 찾기, 코드 검색, 코드베이스 질문 답변에 최적화됨 |
| 3 | `Plan` | inherit | Read-only (no Write, Edit) | 플랜 모드에서의 사전 계획 리서치 — 코드를 작성하기 전에 코드베이스를 탐색하고 구현 접근 방식을 설계 |
| 4 | `statusline-setup` | sonnet | Read, Edit | 사용자의 Claude Code 상태 표시줄 설정을 구성 |
| 5 | `claude-code-guide` | haiku | Glob, Grep, Read, WebFetch, WebSearch | Claude Code 기능, Agent SDK, Claude API에 관한 질문에 답변 |

---

## Sources

- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
- [CLI reference — Claude Code Docs](https://code.claude.com/docs/en/cli-reference)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
