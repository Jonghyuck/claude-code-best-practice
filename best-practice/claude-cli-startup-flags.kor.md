<!--
  이 문서는 best-practice/claude-cli-startup-flags.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# CLI Startup Flags Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026-white?style=flat&labelColor=555)

터미널에서 Claude Code를 실행할 때 사용하는 시작 플래그, 최상위 서브커맨드, 시작 환경 변수에 대한 참고 문서입니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Table of Contents

1. [Session Management](#session-management)
2. [Model & Configuration](#model--configuration)
3. [Permissions & Security](#permissions--security)
4. [Output & Format](#output--format)
5. [System Prompt](#system-prompt)
6. [Agent & Subagent](#agent--subagent)
7. [MCP & Plugins](#mcp--plugins)
8. [Directory & Workspace](#directory--workspace)
9. [Budget & Limits](#budget--limits)
10. [Integration](#integration)
11. [Initialization & Maintenance](#initialization--maintenance)
12. [Debug & Diagnostics](#debug--diagnostics)
13. [Settings Override](#settings-override)
14. [Version & Help](#version--help)
15. [Subcommands](#subcommands)
16. [Environment Variables](#environment-variables)

---

## Session Management

| Flag | Short | Description |
|------|-------|-------------|
| `--continue` | `-c` | 현재 디렉터리에서 가장 최근 대화를 이어감 |
| `--resume` | `-r` | ID나 이름으로 특정 세션을 재개하거나 대화형 선택기를 표시 |
| `--from-pr <NUMBER\|URL>` | | 특정 GitHub PR에 연결된 세션을 재개 |
| `--fork-session` | | 세션을 재개할 때 새 세션 ID를 생성 (`--resume` 또는 `--continue`와 함께 사용) |
| `--session-id <UUID>` | | 특정 세션 ID를 사용 (유효한 UUID여야 함) |
| `--no-session-persistence` | | 세션 지속성을 비활성화 (print 모드 전용) |
| `--remote` | | claude.ai에 새 웹 세션을 생성 |
| `--teleport` | | 웹 세션을 로컬 터미널에서 재개 |

---

## Model & Configuration

| Flag | Short | Description |
|------|-------|-------------|
| `--model <NAME>` | | 별칭(`sonnet`, `opus`, `haiku`) 또는 전체 모델 ID로 모델을 설정 |
| `--fallback-model <NAME>` | | 기본 모델이 과부하일 때 자동으로 대체할 모델 (print 모드 전용) |
| `--betas <LIST>` | | API 요청에 포함할 베타 헤더 (API 키 사용자 전용) |

---

## Permissions & Security

| Flag | Short | Description |
|------|-------|-------------|
| `--dangerously-skip-permissions` | | 모든 권한 프롬프트를 건너뜀. 각별히 주의해서 사용 |
| `--allow-dangerously-skip-permissions` | | 권한 우회를 활성화하지는 않고 옵션으로만 사용 가능하게 함 |
| `--permission-mode <MODE>` | | 지정한 권한 모드로 시작: `default`, `plan`, `acceptEdits`, `bypassPermissions` |
| `--allowedTools <TOOLS>` | | 프롬프트 없이 실행되는 도구 (권한 규칙 문법) |
| `--disallowedTools <TOOLS>` | | 모델 컨텍스트에서 완전히 제거되는 도구 |
| `--tools <TOOLS>` | | Claude가 사용할 수 있는 내장 도구를 제한 (모두 비활성화하려면 `""` 사용) |
| `--permission-prompt-tool <TOOL>` | | 비대화형 모드에서 권한 프롬프트를 처리할 MCP 도구를 지정 |

---

## Output & Format

| Flag | Short | Description |
|------|-------|-------------|
| `--print` | `-p` | 대화형 모드 없이 응답만 출력 (헤드리스/SDK 모드) |
| `--output-format <FORMAT>` | | 출력 형식: `text`, `json`, `stream-json` |
| `--input-format <FORMAT>` | | 입력 형식: `text`, `stream-json` |
| `--json-schema <SCHEMA>` | | 스키마에 맞는 검증된 JSON을 얻음 (print 모드 전용) |
| `--include-partial-messages` | | 부분 스트리밍 이벤트를 포함 (`--print`와 `--output-format=stream-json` 필요) |
| `--verbose` | | 턴 단위 전체 출력을 포함한 상세 로깅을 활성화 |

---

## System Prompt

| Flag | Short | Description |
|------|-------|-------------|
| `--system-prompt <TEXT>` | | 전체 시스템 프롬프트를 사용자 지정 텍스트로 교체 |
| `--system-prompt-file <PATH>` | | 파일에서 시스템 프롬프트를 불러와 기본값을 교체 (print 모드 전용) |
| `--append-system-prompt <TEXT>` | | 기본 시스템 프롬프트에 사용자 지정 텍스트를 추가 |
| `--append-system-prompt-file <PATH>` | | 파일 내용을 기본 프롬프트에 추가 (print 모드 전용) |

---

## Agent & Subagent

| Flag | Short | Description |
|------|-------|-------------|
| `--agent <NAME>` | | 현재 세션에 사용할 에이전트를 지정 |
| `--agents <JSON>` | | JSON으로 사용자 지정 서브에이전트를 동적으로 정의 |
| `--teammate-mode <MODE>` | | 에이전트 팀 표시 방식을 설정: `auto`, `in-process`, `tmux` |

---

## MCP & Plugins

| Flag | Short | Description |
|------|-------|-------------|
| `--mcp-config <PATH\|JSON>` | | JSON 파일 또는 문자열에서 MCP 서버를 불러옴 |
| `--strict-mcp-config` | | `--mcp-config`의 MCP 서버만 사용하고 나머지는 모두 무시 |
| `--plugin-dir <PATH>` | | 이 세션에 한해 디렉터리에서 플러그인을 불러옴 (반복 사용 가능) |

---

## Directory & Workspace

| Flag | Short | Description |
|------|-------|-------------|
| `--add-dir <PATH>` | | Claude가 접근할 추가 작업 디렉터리를 지정 |
| `--worktree` | `-w` | 격리된 git worktree에서 Claude를 시작 (HEAD에서 분기) |

---

## Budget & Limits

| Flag | Short | Description |
|------|-------|-------------|
| `--max-budget-usd <AMOUNT>` | | 중단하기 전 API 호출에 쓸 최대 금액(달러) (print 모드 전용) |
| `--max-turns <NUMBER>` | | 에이전트 턴 수를 제한 (print 모드 전용) |

---

## Integration

| Flag | Short | Description |
|------|-------|-------------|
| `--chrome` | | 웹 자동화를 위한 Chrome 브라우저 연동을 활성화 |
| `--no-chrome` | | 이 세션에서 Chrome 브라우저 연동을 비활성화 |
| `--ide` | | 유효한 IDE가 정확히 하나 있을 때 시작 시 자동으로 IDE에 연결 |

---

## Initialization & Maintenance

| Flag | Short | Description |
|------|-------|-------------|
| `--init` | | 초기화 훅을 실행하고 대화형 모드를 시작 |
| `--init-only` | | 초기화 훅을 실행하고 종료 (대화형 세션 없음) |
| `--maintenance` | | 유지보수 훅을 실행하고 종료 |

---

## Debug & Diagnostics

| Flag | Short | Description |
|------|-------|-------------|
| `--debug <CATEGORIES>` | | 선택적 카테고리 필터링과 함께 디버그 모드를 활성화 (예: `"api,hooks"`) |

---

## Settings Override

| Flag | Short | Description |
|------|-------|-------------|
| `--settings <PATH\|JSON>` | | 불러올 설정 JSON 파일의 경로 또는 JSON 문자열 |
| `--setting-sources <LIST>` | | 불러올 소스의 쉼표 구분 목록: `user`, `project`, `local` |
| `--disable-slash-commands` | | 이 세션의 모든 스킬과 슬래시 커맨드를 비활성화 |

---

## Version & Help

| Flag | Short | Description |
|------|-------|-------------|
| `--version` | `-v` | 버전 번호를 출력 |
| `--help` | `-h` | 도움말 정보를 표시 |

---

## Subcommands

다음은 `claude <subcommand>` 형태로 실행하는 최상위 커맨드입니다:

| Subcommand | Description |
|------------|-------------|
| `claude` | 대화형 REPL을 시작 |
| `claude "query"` | 초기 프롬프트와 함께 REPL을 시작 |
| `claude agents` | 구성된 에이전트를 나열 |
| `claude auth` | Claude Code 인증을 관리 |
| `claude doctor` | 커맨드라인에서 진단을 실행 |
| `claude install` | Claude Code 네이티브 빌드를 설치하거나 전환 |
| `claude mcp` | MCP 서버를 구성 (`add`, `remove`, `list`, `get`, `enable`) |
| `claude plugin` | Claude Code 플러그인을 관리 |
| `claude remote-control` | 원격 제어 세션을 관리 |
| `claude setup-token` | 구독 사용을 위한 장기 토큰을 생성 |
| `claude update` / `claude upgrade` | 최신 버전으로 업데이트 |

---

## Environment Variables

다음은 Claude Code를 실행하기 전 셸에서 설정하는 시작 전용 환경 변수입니다(`settings.json`으로는 구성할 수 없음):

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | 실험적 에이전트 팀을 활성화. env로도 구성 가능 — [Settings Reference](./claude-settings.md#environment-variables) 참고 |
| `CLAUDE_CODE_TMPDIR` | 내부 파일용 임시 디렉터리를 재정의. `env` 키로도 구성 가능 — [Settings Reference](./claude-settings.md#environment-variables-via-env) 참고 |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | 추가 디렉터리의 CLAUDE.md 로딩을 활성화. `env` 키로도 구성 가능 — [Settings Reference](./claude-settings.md#environment-variables-via-env) 참고 |
| `DISABLE_AUTOUPDATER=1` | 자동 업데이트를 비활성화. `env` 키로도 구성 가능 — [Settings Reference](./claude-settings.md#environment-variables-via-env) 참고 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 사고 깊이를 제어 — [Settings Reference](./claude-settings.md#environment-variables-via-env) 참고 |
| `USE_BUILTIN_RIPGREP=0` | 내장 ripgrep 대신 시스템 ripgrep을 사용 (Alpine Linux) |
| `CLAUDE_CODE_SIMPLE` | 심플 모드를 활성화 (Bash + Edit 도구만 사용). `env` 키로도 구성 가능 — [Settings Reference](./claude-settings.md#environment-variables-via-env) 참고 |
| `CLAUDE_BASH_NO_LOGIN=1` | BashTool에서 로그인 셸을 건너뜀 |
| `CCR_FORCE_BUNDLE=1` | `claude --remote` 사용 시 로컬 저장소를 강제로 번들링/업로드. `env` 키로도 구성 가능 — [Settings Reference](./claude-settings.md#environment-variables-via-env) 참고 |

`settings.json`의 `"env"` 키로 구성 가능한 환경 변수(`MAX_THINKING_TOKENS`, `CLAUDE_CODE_SHELL`, `CLAUDE_CODE_ENABLE_TASKS`, `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`, `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` 등)에 대해서는 [Claude Settings Reference](./claude-settings.md#environment-variables-via-env)를 참고하세요.

---

## Sources

- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Claude Code Headless Mode](https://code.claude.com/docs/en/headless)
- [Claude Code Setup](https://code.claude.com/docs/en/setup)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code Common Workflows](https://code.claude.com/docs/en/common-workflows)
