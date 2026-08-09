<!--
  이 문서는 reports/claude-agent-memory.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Claude Code: Agent Memory Frontmatter

서브에이전트를 위한 영속 메모리 — 에이전트가 세션을 넘나들며 학습하고, 기억하고, 지식을 쌓을 수 있게 합니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Overview

**Claude Code v2.1.33** (2026년 2월)에서 도입된 `memory` frontmatter 필드는 각 서브에이전트에 자체적인 markdown 기반 영속 지식 저장소를 제공합니다. 이전에는 모든 에이전트 호출이 매번 백지 상태에서 시작했습니다.

```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Write, Edit, Bash
model: sonnet
memory: user
---

You are a code reviewer. As you review code, update your agent memory with
patterns, conventions, and recurring issues you discover.
```

---

## Memory Scopes

| Scope | Storage Location | Version Controlled | Shared | Best For |
|-------|-----------------|-------------------|--------|----------|
| `user` | `~/.claude/agent-memory/<agent-name>/` | 아니요 | 아니요 | 프로젝트 전반에 걸친 지식 (권장 기본값) |
| `project` | `.claude/agent-memory/<agent-name>/` | 예 | 예 | 팀이 공유해야 하는 프로젝트 고유 지식 |
| `local` | `.claude/agent-memory-local/<agent-name>/` | 아니요 (git-ignored) | 아니요 | 개인적인 프로젝트 고유 지식 |

이 스코프들은 설정 계층 구조(`~/.claude/settings.json` → `.claude/settings.json` → `.claude/settings.local.json`)를 그대로 반영합니다.

---

## How It Works

1. **시작 시**: `MEMORY.md`의 첫 200줄이 에이전트의 시스템 프롬프트에 주입됩니다
2. **도구 접근**: 에이전트가 자신의 메모리를 관리할 수 있도록 `Read`, `Write`, `Edit`가 자동으로 활성화됩니다
3. **실행 중**: 에이전트는 자신의 메모리 디렉터리를 자유롭게 읽고 씁니다
4. **정리(Curation)**: `MEMORY.md`가 200줄을 초과하면 에이전트가 세부 내용을 주제별 파일로 옮깁니다

```
~/.claude/agent-memory/code-reviewer/     # user scope example
├── MEMORY.md                              # Primary file (first 200 lines loaded)
├── react-patterns.md                      # Topic-specific file
└── security-checklist.md                  # Topic-specific file
```

---

## Agent Memory vs Other Memory Systems

| System | Who Writes | Who Reads | Scope |
|--------|-----------|-----------|-------|
| **CLAUDE.md** | 사용자 (수동) | 메인 Claude + 모든 에이전트 | 프로젝트 |
| **Auto-memory** | 메인 Claude (자동) | 메인 Claude만 | 프로젝트별·사용자별 |
| **`/memory` command** | 사용자 (에디터를 통해) | 메인 Claude만 | 프로젝트별·사용자별 |
| **Agent memory** | 에이전트 자신 | 해당 에이전트만 | 설정 가능 (user/project/local) |

이 시스템들은 **상호 보완적**입니다 — 에이전트는 CLAUDE.md(프로젝트 컨텍스트)와 자신의 메모리(에이전트 고유 지식)를 모두 읽습니다.

---

## Practical Example

```yaml
---
name: api-developer
description: Implement API endpoints following team conventions
tools: Read, Write, Edit, Bash
model: sonnet
memory: project
skills:
  - api-conventions
  - error-handling-patterns
---

Implement API endpoints. Follow the conventions from your preloaded skills.
As you work, save architectural decisions and patterns to your memory.
```

이는 **skills**(시작 시점의 정적 지식)와 **memory**(시간이 지나며 쌓이는 동적 지식)를 결합한 것입니다.

---

## Tips

- **메모리 사용을 프롬프트로 지시하기** — 명시적인 지침을 포함하세요: `"Before starting, review your memory. After completing, update your memory with what you learned."`
- 에이전트를 호출할 때 **메모리 확인을 요청하기**: `"Review this PR, and check your memory for patterns you've seen before."`
- **올바른 스코프 선택하기** — 프로젝트 전반에는 `user`, 팀 공유에는 `project`, 개인용에는 `local`

---

## Sources

- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
- [Manage Claude's memory — Claude Code Docs](https://code.claude.com/docs/en/memory)
- [Claude Code v2.1.33 Release Notes](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
