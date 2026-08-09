<!--
  이 문서는 best-practice/claude-settings.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Settings Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Jul%2011%2C%202026%2010%3A43%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.207-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../.claude/settings.json)

Claude Code의 `settings.json` 파일에서 사용할 수 있는 모든 구성 옵션을 종합적으로 다루는 가이드입니다. v2.1.207 기준으로 Claude Code는 **80개 이상의 설정**과 **200개 이상의 환경 변수**를 노출합니다(래퍼 스크립트를 피하려면 `settings.json`의 `"env"` 필드를 사용하세요).

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## Table of Contents

1. [Settings Hierarchy](#settings-hierarchy)
2. [Core Configuration](#core-configuration)
3. [Permissions](#permissions)
4. [Hooks](#hooks)
5. [MCP Servers](#mcp-servers)
6. [Sandbox](#sandbox)
7. [Plugins](#plugins)
8. [Model Configuration](#model-configuration)
9. [Display & UX](#display--ux)
10. [AWS & Cloud Credentials](#aws--cloud-credentials)
11. [Environment Variables](#environment-variables-via-env)
12. [Useful Commands](#useful-commands)

---

## Settings Hierarchy

설정은 우선순위 순서(높음에서 낮음)로 적용됩니다:

| Priority | Location | Scope | Shared? | Purpose |
|----------|----------|-------|---------|---------|
| 1 | Managed settings | Organization | Yes (IT가 배포) | 재정의할 수 없는 보안 정책 |
| 2 | Command line arguments | Session | N/A | 임시 단일 세션 재정의 |
| 3 | `.claude/settings.local.json` | Project | No (git-ignored) | 개인용 프로젝트별 설정 |
| 4 | `.claude/settings.json` | Project | Yes (커밋됨) | 팀 공유 설정 |
| 5 | `~/.claude/settings.json` | User | N/A | 전역 개인 기본값 |

**Managed settings**는 조직이 강제하며 명령줄 인수를 포함한 다른 어떤 수준으로도 재정의할 수 없습니다. 전달 방식:
- **Server-managed** 설정(원격 전달)
- **MDM 프로필** — macOS plist, `com.anthropic.claudecode`
- **레지스트리 정책** — Windows `HKLM\SOFTWARE\Policies\ClaudeCode`(관리자) 및 `HKCU\SOFTWARE\Policies\ClaudeCode`(사용자 수준, 정책 우선순위 가장 낮음)
- **파일** — `managed-settings.json` 및 `managed-mcp.json`(macOS: `/Library/Application Support/ClaudeCode/`, Linux/WSL: `/etc/claude-code/`, Windows: `C:\Program Files\ClaudeCode\`)
- **드롭인 디렉터리** — 독립적인 정책 조각을 위해 `managed-settings.json` 옆에 두는 `managed-settings.d/`(v2.1.83). systemd 관례를 따라 `managed-settings.json`이 기본(base)으로 먼저 병합되고, 그다음 드롭인 디렉터리의 모든 `*.json` 파일이 알파벳순으로 정렬되어 그 위에 병합됩니다. 스칼라 값의 경우 나중 파일이 이전 파일을 재정의하며, 배열은 연결(concatenate) 후 중복 제거되고, 객체는 딥 병합됩니다. `.`으로 시작하는 숨김 파일은 무시됩니다. 병합 순서를 제어하려면 숫자 접두사를 사용하세요(예: `10-telemetry.json`, `20-security.json`)

관리 티어 내에서 우선순위는 다음과 같습니다: server-managed > MDM/OS 수준 정책 > 파일 기반(`managed-settings.d/*.json` + `managed-settings.json`) > HKCU 레지스트리(Windows 전용). 관리 소스는 하나만 사용되며, 티어를 가로질러 병합되지 않습니다. 파일 기반 티어 내에서는 드롭인 파일과 기본 파일이 함께 병합됩니다.

> **Note:** v2.1.75 기준으로 더 이상 사용되지 않는 Windows 대체 경로 `C:\ProgramData\ClaudeCode\managed-settings.json`이 제거되었습니다. 대신 `C:\Program Files\ClaudeCode\managed-settings.json`을 사용하세요.

> **Note (v2.1.126):** `/config`는 이제 변경 사항을 메모리에만 보관하는 대신 `~/.claude/settings.json`에 영구 저장합니다. 대화형 Config UI를 통한 편집은 재시작 후에도 유지됩니다.

**Managed 전용 정책 키:**

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `parentSettingsBehavior` | string | `"first-wins"` | 임베딩 호스트 프로세스(SDK 부모)가 프로그래밍 방식으로 제공한 관리 설정이, 관리자가 배포한 관리 티어가 함께 존재할 때 적용되는지를 제어합니다. `"first-wins"`: 부모가 제공한 설정은 폐기되고 관리자 티어만 적용됩니다. `"merge"`: 부모가 제공한 설정이 관리자 티어 아래에 적용되며, 정책을 **강화**할 수는 있지만 완화할 수는 없도록 필터링됩니다. v2.1.133+ 필요 |
| `policyHelper` | object | - | 시작 시 관리 설정을 동적으로 계산하는 관리자 배포 실행 파일. 객체 형태: 헬퍼 바이너리를 가리키는 `{path: string}`. MDM 또는 시스템 `managed-settings.json` 파일에서만 존중됩니다(사용자/프로젝트 설정에서는 절대 안 됨). 헬퍼 출력은 매 시작마다 관리 티어에 병합됩니다. v2.1.136+ 필요 |
| `requiredMinimumVersion` | string | - | **(Managed only)** 설치된 버전이 이 하한보다 낮으면 Claude Code 시작을 막습니다. CLI는 사용자에게 업그레이드를 요구하는 오류와 함께 종료됩니다. `minimumVersion`(자동 업데이트 하한 제어)을 보완하며, 이것은 시작 시 강제합니다. 예: `"2.1.163"` |
| `requiredMaximumVersion` | string | - | **(Managed only)** 설치된 버전이 이 상한을 초과하면 Claude Code 시작을 막습니다. 버전이 너무 새로우면 CLI가 오류와 함께 종료됩니다. 관리 환경에서 특정 버전 범위를 고정하려면 `requiredMinimumVersion`과 함께 사용하세요. 예: `"2.1.165"` |
| `browserExternalPageTools` | string | - | **(Managed only)** `"disabled"`로 설정하면 Claude가 데스크톱 앱의 Browser 창에서 외부 페이지를 읽거나 조작하는 도구를 사용하지 못하도록 막습니다. 사용자는 여전히 직접 외부 사이트로 이동할 수 있으며, 로컬 개발 서버 미리보기는 영향을 받지 않습니다 |

**Important**:
- `deny` 규칙은 가장 높은 안전 우선순위를 가지며 우선순위가 낮은 allow/ask 규칙으로 재정의할 수 없습니다.
- 관리 설정은 로컬 파일이 다른 값을 지정하더라도 로컬 동작을 잠그거나 재정의할 수 있습니다.
- 배열 설정(예: `permissions.allow`)은 스코프 전반에 걸쳐 **연결 후 중복 제거**됩니다 — 모든 수준의 항목이 대체되지 않고 결합됩니다.

---

## Core Configuration

### General Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `$schema` | string | - | IDE 검증 및 자동 완성을 위한 JSON Schema URL(예: `"https://json.schemastore.org/claude-code-settings.json"`) |
| `model` | string | `"default"` | 기본 모델을 재정의합니다. 별칭(`sonnet`, `opus`, `haiku`) 또는 전체 모델 ID를 허용합니다 |
| `agent` | string | - | 메인 대화의 기본 에이전트를 설정합니다. 값은 `.claude/agents/`의 에이전트 이름입니다. `--agent` CLI 플래그로도 사용 가능합니다 |
| `language` | string | `"english"` | Claude가 선호하는 응답 언어. 음성 받아쓰기 언어와 터미널 탭 제목도 설정합니다(v2.1.121) |
| `claudeMdExcludes` | array | - | [memory](https://code.claude.com/docs/en/memory) 로드 시 건너뛸 `CLAUDE.md` 파일의 glob 패턴 또는 절대 경로. 패턴은 절대 파일 경로에 대해 매칭됩니다. 사용자·프로젝트·로컬 메모리에만 적용되며, 관리 정책 파일은 제외할 수 없습니다. 예: `["**/vendor/**/CLAUDE.md"]` |
| `claudeMd` | string | - | **(Managed only)** 조직 관리 [memory](https://code.claude.com/docs/en/memory)로 주입되는 CLAUDE.md 스타일 지침. 관리 또는 정책 설정에서 설정한 경우에만 존중되며, 사용자·프로젝트·로컬 설정에서는 무시됩니다. 예: `"Always run make lint before committing."` |
| `cleanupPeriodDays` | number | `30` | 시작 시 정리 스윕의 나이 컷오프(최소 1). 비활성 세션 기록과 고아 서브에이전트 워크트리가 삭제되며, v2.1.117 기준으로 스윕은 `~/.claude/tasks/`, `~/.claude/shell-snapshots/`, `~/.claude/backups/`도 포함합니다. `0`으로 설정하면 검증 오류로 거부됩니다. 비대화형 모드(`-p`)에서 기록 쓰기를 비활성화하려면 `--no-session-persistence` 또는 SDK 옵션 `persistSession: false`를 사용하세요 |
| `autoUpdatesChannel` | string | `"latest"` | 릴리스 채널: `"stable"` 또는 `"latest"` |
| `minimumVersion` | string | - | 자동 업데이터가 특정 버전 아래로 다운그레이드하지 못하도록 막습니다. stable 채널로 전환하고 stable이 따라잡을 때까지 현재 버전에 머무르기로 선택하면 자동으로 설정됩니다. `autoUpdatesChannel`과 함께 사용됩니다 |
| `alwaysThinkingEnabled` | boolean | `false` | 모든 세션에서 확장 사고(extended thinking)를 기본적으로 활성화합니다 |
| `thinkingBudgetTokens` | number | - | 응답당 확장 사고에 대한 고정 토큰 예산을 설정합니다. 설정하면 사고 토큰 수를 지정한 값으로 제한합니다. 설정하지 않으면 Claude Code는 모델과 노력 수준에 기반한 동적 예산을 사용합니다. `alwaysThinkingEnabled`와 함께 동작합니다 |
| `skipWebFetchPreflight` | boolean | `false` | 가져오기 전에 요청된 각 호스트명을 `api.anthropic.com`으로 전송하는 WebFetch 도메인 안전성 검사를 건너뜁니다. Bedrock, Vertex AI, 제한적 이그레스가 있는 Foundry 배포처럼 Anthropic으로의 아웃바운드 트래픽을 차단하는 환경에서 `true`로 설정하세요 |
| `availableModels` | array | - | `/model`, `--model`, Config 도구 또는 `ANTHROPIC_MODEL`을 통해 사용자가 선택할 수 있는 모델을 제한합니다. Default 옵션에는 영향을 주지 않습니다. v2.1.172 기준으로 서브에이전트 디스패치용 모델 선택기와 `advisorModel` 선택기도 제한합니다. Default 모델 옵션까지 추가로 제한하려면 `enforceAvailableModels: true`를 사용하세요. 예: `["sonnet", "haiku"]` |
| `enforceAvailableModels` | boolean | `false` | **(Managed only)** `true`이면 `availableModels` 허용 목록이 Default 모델 옵션도 제한합니다 — 사용자는 Default 슬롯을 통해서도 허용 목록 밖의 모델을 선택할 수 없습니다. 이 플래그가 없으면 `availableModels`는 Default 옵션을 제한하지 않습니다. 완전한 모델 잠금을 위해 `availableModels`와 함께 사용하세요(v2.1.175) |
| `fastModePerSessionOptIn` | boolean | `false` | 사용자가 세션마다 fast mode에 옵트인하도록 요구합니다 |
| `defaultShell` | string | `"bash"` | 입력 상자 `!` 명령을 위한 기본 셸. `"bash"`(기본값) 또는 `"powershell"`을 허용합니다. `"powershell"`로 설정하면 Windows에서 대화형 `!` 명령을 PowerShell을 통해 라우팅합니다. `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` 필요(v2.1.84). **v2.1.120:** PowerShell을 사용할 수 있으면 Git for Windows가 설치되어 있지 않아도 Windows에서 대체 셸로 사용됩니다. **v2.1.126:** PowerShell이 활성화되면 Bash로 기본 설정되는 대신 *주* 셸로 취급됩니다. PowerShell 7 감지는 이제 Microsoft Store 설치, PATH에 없는 MSI 설치, `.NET` 전역 도구 설치도 포함합니다 |
| `includeGitInstructions` | boolean | `true` | 내장 커밋/PR 워크플로 지침과 git 상태 스냅샷을 Claude의 시스템 프롬프트에 포함합니다. `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` 환경 변수가 설정된 경우 이 설정보다 우선합니다 |
| `voice` | object | - | 음성 받아쓰기 구성. 세 개의 필드를 가진 객체: `enabled`(boolean — 푸시투토크 on/off), `mode`(string — 눌러서 말하기는 `"hold"`, 탭해서 토글은 `"tap"`), `autoSubmit`(boolean — 받아쓰기 종료 시 전사를 즉시 제출). `/voice`를 실행하면 자동으로 기록됩니다. Claude.ai 계정 필요(v2.1.118에서 구조 확장) |
| `voiceEnabled` | boolean | - | **DEPRECATED** — `voice.enabled`의 레거시 별칭입니다. `mode`와 `autoSubmit` 제어를 얻으려면 대신 `voice` 객체를 사용하세요 |
| `showClearContextOnPlanAccept` | boolean | `false` | plan 수락 화면에 "clear context" 옵션을 표시합니다. 옵션을 복원하려면 `true`로 설정하세요(v2.1.81부터 기본적으로 숨김) |
| `viewMode` | string | - | 시작 시 기본 기록 보기 모드: `"default"`, `"verbose"`, `"focus"`. 설정하면 고정된 Ctrl+O 선택을 재정의합니다 |
| `disableDeepLinkRegistration` | string | - | `"disable"`로 설정하면 Claude Code가 시작 시 운영체제에 `claude-cli://` 프로토콜 핸들러를 등록하지 못하도록 막습니다. 딥 링크를 사용하면 외부 도구가 `claude-cli://open?q=...`를 통해 미리 채워진 프롬프트로 Claude Code 세션을 열 수 있습니다. `q` 매개변수는 URL 인코딩된 줄바꿈(`%0A`)을 사용하여 여러 줄 프롬프트를 지원합니다. 프로토콜 핸들러 등록이 제한되거나 별도로 관리되는 환경에 유용합니다 |
| `showThinkingSummaries` | boolean | `false` | 대화형 세션에서 확장 사고 요약을 표시합니다. 설정되지 않거나 `false`(대화형 모드 기본값)이면 사고 블록이 API에 의해 편집(redact)되어 접힌 스텁으로 표시됩니다. 편집은 모델이 생성하는 내용이 아니라 보이는 내용만 바꿉니다 — 사고 비용을 줄이려면 대신 예산을 낮추거나 사고를 비활성화하세요. 비대화형 모드(`-p`)와 SDK 호출자는 이 설정과 무관하게 항상 요약을 받습니다 |
| `disableSkillShellExecution` | boolean | `false` | 사용자·프로젝트·플러그인·추가 디렉터리 소스의 스킬 및 커스텀 명령에서 `` !`...` `` 및 `` ```! `` 블록의 인라인 셸 실행을 비활성화합니다. 명령은 실행되는 대신 `[shell command execution disabled by policy]`로 대체됩니다. 번들 및 관리 스킬은 영향을 받지 않습니다(v2.1.91) |
| `maxSkillDescriptionChars` | number | `1536` | Claude가 매 턴 보는 [skill listing](https://code.claude.com/docs/en/skills)에서 스킬당 결합된 `description` 및 `when_to_use` 텍스트의 문자 상한. 이보다 긴 텍스트는 잘립니다(v2.1.105) |
| `skillListingBudgetFraction` | number | `0.01` | Claude가 매 턴 보는 [skill listing](https://code.claude.com/docs/en/skills)을 위해 예약된 모델 컨텍스트 창의 비율(`0.01` = 1%). 목록이 예산을 초과하면, 가장 적게 사용된 스킬의 설명이 이름만으로 축소되어 Claude가 여전히 호출할 수는 있지만 이유는 볼 수 없게 됩니다(v2.1.105) |
| `forceRemoteSettingsRefresh` | boolean | `false` | **(Managed only)** 원격 관리 설정이 새로 가져와질 때까지 CLI 시작을 차단합니다. 가져오기가 실패하면 CLI가 종료됩니다(fail-closed). 어떤 세션이 시작되기 전에 정책 집행이 최신이어야 하는 엔터프라이즈 환경에서 사용하세요(v2.1.92) |
| `wslInheritsWindowsSettings` | boolean | `false` | **(Windows managed settings only)** `true`이면 WSL의 Claude Code가 `/etc/claude-code`에 더해 Windows 정책 체인(HKLM 레지스트리 + `C:\Program Files\ClaudeCode\managed-settings.json`)에서 관리 설정을 읽으며, Windows 소스가 우선합니다. HKLM 레지스트리 키 또는 `C:\Program Files\ClaudeCode\managed-settings.json`에서 설정한 경우에만 존중되며, 둘 다 쓰려면 Windows 관리자 권한이 필요합니다. HKCU 정책도 WSL에 적용되려면 이 플래그가 HKCU 자체에도 추가로 설정되어야 합니다. 네이티브 Windows에는 영향이 없습니다(v2.1.118) |
| `tui` | string | `"default"` | 렌더링 모드: `"fullscreen"` 또는 `"default"`. 깜빡임 없는 대체 화면 렌더링을 위해 `/tui fullscreen`으로 설정합니다(v2.1.110) |
| `awaySummaryEnabled` | boolean | `true` | 사용자가 자리를 비운 후 돌아오면 "away summary"(유휴 세션 요약)를 생성합니다. 옵트아웃하려면 `false`로 설정하세요. `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` 환경 변수와 짝을 이룹니다(v2.1.110) |
| `autoCompactEnabled` | boolean | `true` | 컨텍스트가 모델 한도에 근접하면 대화를 자동 압축합니다. 자동 압축을 비활성화하고 컨텍스트를 수동으로 관리하려면 `false`로 설정하세요. `DISABLE_AUTO_COMPACT` 환경 변수로도 비활성화할 수 있습니다 |
| `skillOverrides` | object | - | 스킬 이름을 키로 하는 스킬별 가시성 재정의. 값은 `"on"`(전체), `"name-only"`(보이지만 자동 설명되지 않음), `"user-invocable-only"`(모델 발견에서는 숨겨지지만 여전히 슬래시로 호출 가능), `"off"`(완전히 숨김)입니다. 예: `{"legacy-context": "name-only", "deploy": "off"}`(v2.1.129) |
| `disableRemoteControl` | boolean | `false` | [Remote Control](https://code.claude.com/docs/en/remote-control)을 비활성화합니다: `claude remote-control`, `--remote-control` 플래그, 자동 시작, 세션 내 토글을 차단합니다. 일반적으로 디바이스별 MDM 집행을 위해 관리 설정에 배치하지만 모든 스코프에서 작동합니다(v2.1.128) |
| `agentPushNotifEnabled` | boolean | `false` | Claude가 푸시하기로 결정할 때(예: 작업 완료) [Remote Control](https://code.claude.com/docs/en/remote-control)에 사전 푸시 알림을 보냅니다. `/config`에 **Push when Claude decides**로 표시됩니다 |
| `inputNeededNotifEnabled` | boolean | `false` | 권한 프롬프트나 질문이 사용자 입력을 기다릴 때 [Remote Control](https://code.claude.com/docs/en/remote-control)에 푸시 알림을 보냅니다. `/config`에 **Push when actions required**로 표시됩니다 |
| `remoteControlAtStartup` | boolean/null | - | 시작 시 [Remote Control](https://code.claude.com/docs/en/remote-control)을 자동 연결합니다. `true`는 항상 자동 연결, `false`는 절대 자동 연결 안 함, 미설정은 조직 기본값을 사용합니다. 모든 스코프에서 설정할 수 있습니다(v2.1.119+) |
| `disableAgentView` | boolean | `false` | `true`로 설정하면 [background agents 및 agent view](https://code.claude.com/docs/en/agent-view)를 끕니다: `claude agents`, `--bg`, `/background`, 온디맨드 슈퍼바이저. 모든 스코프에서 설정할 수 있지만 일반적으로 관리 설정에 배치합니다. `CLAUDE_CODE_DISABLE_AGENT_VIEW` 환경 변수를 `1`로 설정하는 것과 동등합니다 |
| `disableWorkflows` | boolean | `false` | `true`로 설정하면 [dynamic workflows](https://code.claude.com/docs/en/workflows)(`/workflows`)와 번들된 워크플로 슬래시 명령을 비활성화합니다. 모든 스코프에서 설정할 수 있습니다. `CLAUDE_CODE_DISABLE_WORKFLOWS` 환경 변수와 동등합니다. 워크플로는 v2.1.154에서 도입되었습니다 |
| `workflowKeywordTriggerEnabled` | boolean | `true` | 프롬프트에 "ultracode"라는 단어를 입력하면 [dynamic workflow](https://code.claude.com/docs/en/workflows)를 트리거하는지 여부. 명시적 `/workflows` 호출을 요구하려면 `false`로 설정하세요. Ultracode, `/workflows`, 저장된 워크플로 명령은 이 설정에 영향을 받지 않습니다. `/config`에 **Workflow keyword trigger**로 표시됩니다(v2.1.157; 트리거 키워드가 v2.1.160에서 workflow→ultracode로 이름 변경) |
| `ultracode` | boolean | - | **(Session-only — 영구 저장 안 됨)** `true`이면 하니스가 토큰 비용과 무관하게 철저함을 극대화하여 모든 실질적 작업에 대해 기본적으로 워크플로를 작성하고 실행합니다. 공식 "Available settings" 목록에 나타나지만 세션 범위입니다: `settings.json`에 기록되는 대신 `/effort ultracode`, `--settings`, 또는 SDK를 통해 설정합니다(v2.1.154) |
| `dynamicWorkflowSize` | string | - | [dynamic workflow](https://code.claude.com/docs/en/workflows)에서 생성되는 에이전트 수에 대한 권고 지침. 값: `"small"`, `"medium"`, `"large"`. 설정하면 워크플로 하니스가 작업에 따라 확대·축소하기 전 기본 함대 크기로 이 값을 사용합니다. `/config`에서 **Workflow size**로 설정합니다(v2.1.202; 값은 v2.1.205에서 공식화됨) |
| `disableBundledSkills` | boolean | `false` | Claude Code의 내장 기능(번들 스킬)을 모델로부터 숨깁니다. `true`이면 모델이 내장 스킬을 호출할 수 없습니다. `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` 환경 변수와 짝을 이룹니다. 엄격한 플러그인 전용 커스터마이징이 필요할 때 유용합니다(v2.1.169) |
| `disableArtifact` | boolean | `false` | Artifact 웹 게시 도구를 비활성화합니다. `true`이면 Claude가 웹 아티팩트를 생성하거나 게시할 수 없습니다. 모든 스코프에서 설정할 수 있습니다 |
| `enableArtifact` | boolean | - | **(v2.1.196+)** Artifact 웹 게시 도구에 대한 사용자 수준 옵트인. `true`로 설정하면 어떤 조직 정책도 요구하지 않을 때에도 사용자에게 Artifact를 활성화합니다. `disableArtifact: true`가 우선하여 이 설정을 재정의합니다 |
| `feedbackSurveyRate` | number | - | 자격이 있을 때 세션 품질 설문이 나타날 확률(0–1). 엔터프라이즈 관리자가 설문 표시 빈도를 제어할 수 있습니다. 예: `0.05` = 자격 있는 세션의 5% |
| `advisorModel` | string | - | 서버 측 advisor 도구용 모델. 모델 별칭(`opus`, `sonnet`, `fable`) 또는 전체 모델 ID를 허용합니다. 설정하지 않으면 advisor가 세션 모델을 사용합니다. v2.1.98+ 필요 |
| `respondToBashCommands` | boolean | `true` | `!` 셸 명령이 완료된 후 Claude가 자동으로 응답하는지 여부. `!` bash 명령이 끝날 때 자동 후속 응답을 비활성화하려면 `false`로 설정하세요(v2.1.186) |
| `askUserQuestionTimeout` | string | `"never"` | 응답되지 않은 AskUserQuestion 대화상자가 사용자 없이 자동 진행되기까지 대기하는 시간. 값: `"60s"`, `"5m"`, `"10m"`, `"never"`(자동 진행 안 함 — 기본값). `/config`에서 **Ask user question timeout**으로 설정합니다. `CLAUDE_AFK_TIMEOUT_MS` 환경 변수와 짝을 이루며, 환경 변수는 이 설정이 지속 시간으로 설정된 경우에만 적용됩니다.(v2.1.200) |

**Example:**
```json
{
  "model": "opus",
  "agent": "code-reviewer",
  "language": "japanese",
  "cleanupPeriodDays": 60,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true
}
```

### Plans & Memory Directories

계획 및 자동 메모리 파일을 커스텀 위치에 저장합니다.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `plansDirectory` | string | `~/.claude/plans` | `/plan` 출력이 저장되는 디렉터리 |
| `autoMemoryDirectory` | string | - | 자동 메모리 저장을 위한 커스텀 디렉터리. `~/`로 확장되는 경로를 허용합니다. 메모리 쓰기가 민감한 위치로 재지정되는 것을 방지하기 위해 프로젝트 설정(`.claude/settings.json`)에서는 허용되지 않으며, 정책·로컬·사용자 설정에서는 허용됩니다 |
| `autoMemoryEnabled` | boolean | `true` | [auto memory](https://code.claude.com/docs/en/memory)를 활성화합니다. `false`이면 Claude가 자동 메모리 디렉터리에서 읽거나 쓰지 않습니다. 세션 중 `/memory`로도 토글할 수 있으며, `CLAUDE_CODE_DISABLE_AUTO_MEMORY` 환경 변수로 비활성화할 수 있습니다 |
| `fileCheckpointingEnabled` | boolean | `true` | 각 편집 전에 파일을 스냅샷하여 `/rewind`로 복원할 수 있게 합니다. 체크포인팅을 건너뛰고 디스크 공간을 절약하려면 `false`로 설정하세요. `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` 환경 변수로도 비활성화할 수 있습니다 |

**Example:**
```json
{
  "plansDirectory": "./my-plans"
}
```

**Use Case:** 계획 아티팩트를 Claude의 내부 파일과 별도로 정리하거나, 계획을 공유된 팀 위치에 보관하는 데 유용합니다.

### Worktree Settings

`--worktree`가 git 워크트리를 생성하고 관리하는 방식을 구성합니다. 대규모 모노레포에서 디스크 사용량과 시작 시간을 줄이는 데 유용합니다.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `worktree.symlinkDirectories` | array | `[]` | 큰 디렉터리를 디스크에 중복 저장하지 않도록 메인 저장소에서 각 워크트리로 심볼릭 링크할 디렉터리 |
| `worktree.sparsePaths` | array | `[]` | git sparse-checkout(cone 모드)을 통해 각 워크트리에서 체크아웃할 디렉터리. 나열된 경로만 디스크에 기록됩니다 |
| `worktree.baseRef` | string | `"fresh"` | 새 워크트리가 분기하는 ref. `"fresh"`는 `origin/<default-branch>`에서 분기하여 원격과 일치하는 깨끗한 트리를 만듭니다. `"head"`는 현재 로컬 `HEAD`에서 분기하며, 커밋되지 않았지만 추적되는 변경 사항을 포함합니다(v2.1.133) |
| `worktree.bgIsolation` | string | `"worktree"` | [background sessions](https://code.claude.com/docs/en/agent-view)의 격리 모드. `"worktree"`(기본값)는 `EnterWorktree`가 호출될 때까지 메인 체크아웃에서 `Edit`/`Write`를 차단합니다. `"none"`은 백그라운드 작업이 작업 사본을 직접 편집하게 합니다(v2.1.143) |

**Example:**
```json
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache"],
    "sparsePaths": ["packages/my-app", "shared/utils"]
  }
}
```

### Attribution Settings

git 커밋과 풀 리퀘스트에 대한 귀속(attribution) 메시지를 커스터마이징합니다.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `attribution.commit` | string | Co-authored-by | Git 커밋 귀속(트레일러 지원) |
| `attribution.pr` | string | Generated message | 풀 리퀘스트 설명 귀속 |
| `attribution.sessionUrl` | boolean | `true` | 웹 세션 또는 Remote Control 세션에서 생성된 커밋과 PR에 claude.ai 세션 URL 링크를 포함합니다. 링크를 생략하려면 `false`로 설정하세요. 로컬 CLI 세션에는 영향이 없습니다(v2.1.183) |
| `prUrlTemplate` | string | - | 커밋 귀속의 "PR" 배지가 풀 리퀘스트 UI로 연결되는 방식을 제어하는 URL 템플릿. 저장소 호스트, 소유자, 저장소, PR 번호에 대한 플레이스홀더를 지원합니다. 기본 `https://github.com/...` URL이 적용되지 않는 자체 호스팅 GitLab/Bitbucket/GitHub Enterprise 인스턴스에 유용합니다(v2.1.119) |
| `includeCoAuthoredBy` | boolean | `true` | **DEPRECATED** - 대신 `attribution`을 사용하세요 |

**Example:**
```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: Claude <noreply@anthropic.com>",
    "pr": "Generated with Claude Code"
  }
}
```

**Note:** 귀속을 완전히 숨기려면 빈 문자열(`""`)로 설정하세요.

### Authentication Helpers

동적 인증 토큰 생성을 위한 스크립트.

| Key | Type | Description |
|-----|------|-------------|
| `apiKeyHelper` | string | 인증 토큰을 출력하는 셸 스크립트 경로(`X-Api-Key` 및 `Authorization: Bearer` 헤더 둘 다로 전송됨) |
| `forceLoginMethod` | string | 로그인을 `"claudeai"`, `"console"`, 또는 `"gateway"` 계정으로 제한합니다. 조직 관리 Claude 게이트웨이 배포에는 `"gateway"`를 사용하세요. **(Managed only)** |
| `forceLoginOrgUUID` | string \| array | 로그인이 특정 조직에 속하도록 요구합니다. 단일 UUID 문자열(로그인 중 해당 조직을 미리 선택하기도 함) 또는 나열된 조직 중 하나이면 미리 선택 없이 허용되는 UUID 배열을 허용합니다. 관리 설정에서 설정하면 인증된 계정이 나열된 조직에 속하지 않을 때 로그인이 실패합니다. 빈 배열은 fail-closed로 잘못된 구성 메시지와 함께 로그인을 차단합니다 |
| `forceLoginGatewayUrl` | string | **(Managed only)** `forceLoginMethod`가 `"gateway"`일 때 로그인 화면에 Claude 게이트웨이 URL을 미리 채웁니다. 사용자가 게이트웨이 URL을 수동으로 입력할 필요를 없애줍니다. 일반적으로 `forceLoginMethod: "gateway"`와 함께 설정합니다 |
| `gcpAuthRefresh` | string | GCP Application Default Credentials가 만료되거나 로드할 수 없을 때 이를 갱신하는 커스텀 스크립트. Claude Code가 인증을 재시도하기 전에 실행합니다. ADC가 수명이 짧고 갱신에 조직별 헬퍼가 필요할 때 유용합니다. 예: `"gcloud auth application-default login"` |

**Example:**
```json
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh",
  "forceLoginMethod": "console",
  "forceLoginOrgUUID": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"]
}
```

### Company Announcements

시작 시 사용자에게 커스텀 공지를 표시합니다(무작위로 순환).

| Key | Type | Description |
|-----|------|-------------|
| `companyAnnouncements` | array | 시작 시 표시되는 문자열 배열 |

**Example:**
```json
{
  "companyAnnouncements": [
    "Welcome to Acme Corp!",
    "Remember to run tests before committing!",
    "Check the wiki for coding standards"
  ]
}
```

---

## Permissions

Claude가 수행할 수 있는 도구와 작업을 제어합니다.

### Permission Structure

```json
{
  "permissions": {
    "allow": [],
    "ask": [],
    "deny": [],
    "additionalDirectories": [],
    "defaultMode": "acceptEdits",
    "disableBypassPermissionsMode": "disable"
  }
}
```

### Permission Keys

| Key | Type | Description |
|-----|------|-------------|
| `permissions.allow` | array | 프롬프트 없이 도구 사용을 허용하는 규칙 |
| `permissions.ask` | array | 사용자 확인을 요구하는 규칙 |
| `permissions.deny` | array | 도구 사용을 차단하는 규칙(가장 높은 우선순위) |
| `permissions.additionalDirectories` | array | Claude가 접근할 수 있는 추가 디렉터리 |
| `permissions.defaultMode` | string | 기본 권한 모드. 유효한 값: `"default"`, `"manual"`(`"default"`의 별칭, v2.1.200), `"acceptEdits"`, `"dontAsk"`, `"bypassPermissions"`, `"auto"`, `"plan"`. Remote 환경에서는 `acceptEdits`와 `plan`만 존중됩니다(v2.1.70+) |
| `permissions.disableBypassPermissionsMode` | string | bypass 모드 활성화를 방지합니다 |
| `permissions.skipDangerousModePermissionPrompt` | boolean | `--dangerously-skip-permissions` 또는 `defaultMode: "bypassPermissions"`를 통해 bypass 권한 모드로 진입하기 전에 표시되는 확인 프롬프트를 건너뜁니다. 신뢰할 수 없는 저장소가 프롬프트를 자동 우회하는 것을 방지하기 위해 프로젝트 설정(`.claude/settings.json`)에서 설정하면 무시됩니다 |
| `allowManagedPermissionRulesOnly` | boolean | **(Managed only)** 관리 권한 규칙만 적용됩니다. 사용자/프로젝트의 `allow`, `ask`, `deny` 규칙은 무시됩니다 |
| `autoMode` | object | [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 분류기가 차단하고 허용하는 것을 커스터마이징합니다. `environment`(신뢰할 수 있는 인프라 설명), `allow`(차단 규칙의 예외), `soft_deny`(차단 규칙), `hard_deny`(무조건 차단 규칙 — `allow` 예외나 `$defaults` 센티널로 재정의할 수 없음, v2.1.136)를 포함하며 — 모두 산문 문자열 배열입니다. `classifyAllShell`(boolean, 기본값 `false`)도 허용합니다: `true`이면 임의 코드 실행 패턴만이 아니라 모든 Bash/PowerShell 명령을 auto-mode 분류기를 통해 라우팅하여, 분류기 호출이 늘어나는 대가로 더 엄격한 커버리지를 제공합니다(v2.1.193). 저장소 인젝션을 방지하기 위해 **공유 프로젝트 설정**(`.claude/settings.json`) **또는 로컬 설정**(`.claude/settings.local.json`)에서는 읽지 않습니다(v2.1.207에서 로컬 설정 스코프 제거). 사용자 및 관리 설정에서만 사용 가능하며, `~/.claude/settings.json`에서 구성하세요. `allow` 또는 `soft_deny`를 설정하면, 배열에 리터럴 문자열 `"$defaults"`를 포함하지 않는 한 해당 섹션의 전체 기본 목록을 **대체**합니다 — 센티널은 그 위치에서 내장 규칙을 상속하므로 커스텀 항목이 그 옆에 추가됩니다(v2.1.118). 커스터마이징 전에 `claude auto-mode defaults`를 실행하여 내장 규칙을 확인하세요 |
| `disableAutoMode` | string | `"disable"`로 설정하면 [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode)가 활성화되지 못하도록 막습니다. `Shift+Tab` 순환에서 `auto`를 제거하고 시작 시 `--permission-mode auto`를 거부합니다. 모든 설정 수준에서 설정할 수 있으며, 사용자가 재정의할 수 없는 관리 설정에서 가장 유용합니다 |
| `useAutoModeDuringPlan` | boolean | auto mode를 사용할 수 있을 때 plan 모드가 auto mode 의미론을 사용하는지 여부. 기본값: `true`. 공유 프로젝트 설정(`.claude/settings.json`)에서는 읽지 않습니다. `/config`에 "Use auto mode during plan"으로 표시됩니다 |

### Permission Modes

| Mode | Behavior |
|------|----------|
| `"default"` | 프롬프트를 동반한 표준 권한 검사 |
| `"manual"` | v2.1.200에서 도입된 `"default"`의 별칭 — 프롬프트를 동반한 동일한 표준 권한 검사입니다. 이름 변경은 CLI, VS Code, JetBrains UI 전반의 명확성을 개선합니다. `"default"`가 허용되는 모든 곳에서 허용됩니다 |
| `"acceptEdits"` | 작업 디렉터리 또는 `additionalDirectories`의 경로에 대해 파일 편집 **및 일반적인 파일시스템 명령**(`mkdir`, `touch`, `mv`, `cp` 등)을 자동으로 수락합니다. **v2.1.160:** 코드 실행을 허용하는 빌드 도구 구성 파일(`.npmrc`, `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`, `.devcontainer/` 등)을 쓰기 전과, 셸 시작 파일(`.zshenv`, `.zlogin`, `.bash_login`) 및 `~/.config/git/`에 쓰기 전에는 항상 프롬프트합니다 |
| `"dontAsk"` | `/permissions` 또는 `permissions.allow` 규칙을 통해 사전 승인되지 않은 도구를 자동으로 거부합니다 |
| `"bypassPermissions"` | 모든 권한 검사를 건너뜁니다(위험). 모든 경로 기반 프롬프트를 건너뜁니다 — `.git`, `.config/git`, `.claude`, `.vscode`, `.idea`, `.husky`, `.cargo`, `.devcontainer`, `.yarn`, `.mvn`에 대한 쓰기가 더 이상 프롬프트하지 않습니다(**v2.1.121**에서 `.claude/commands/`, `.claude/agents/`, `.claude/skills/`, `.claude/worktrees/`를 예외 처리; **v2.1.126**에서 남은 모든 경로 기반 프롬프트 제거). 파일시스템 루트나 홈 디렉터리를 대상으로 하는 삭제(`rm -rf /`, `rm -rf ~`)만 모델 오류에 대한 회로 차단기로서 여전히 프롬프트합니다 |
| `"auto"` | 작업이 요청과 일치하는지 검증하는 백그라운드 안전성 검사와 함께 도구 호출을 자동 승인합니다. 리서치 프리뷰. 분류기는 읽기 전용과 파일 편집을 자동 승인하고, 그 외 모든 것은 안전성 검사를 통해 전송합니다. 연속 3회 또는 총 20회 차단 후 프롬프트로 되돌아갑니다. v2.1.111부터 기본 `Shift+Tab` 권한 모드 순환에 포함됩니다(`--enable-auto-mode` 플래그는 v2.1.111에서 제거됨 — `--permission-mode auto`로 이 모드에서 시작). `autoMode` 설정으로 구성합니다 |
| `"plan"` | 읽기 전용 탐색 모드. v2.1.136 기준으로 일치하는 `Edit(...)` allow 규칙이 존재하더라도 파일 쓰기가 차단됩니다 — 이제 plan 모드는 읽기 전용 보장을 유지하기 위해 명시적 allow 규칙을 재정의합니다 |

### Tool Permission Syntax

| Tool | Syntax | Examples |
|------|--------|----------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`, `Bash(* install)`, `Bash(git * main)` |
| `PowerShell` | `PowerShell(cmd *)` | `PowerShell(Get-ChildItem *)`, `PowerShell(git commit *)` — Bash와 동일한 형태이며, 일반적인 별칭이 정규화되고(`gci`/`ls`/`dir` → `Get-ChildItem`) PowerShell AST가 파싱되어 `|`/`;`/`&&`/`||` 체인의 각 하위 명령이 매칭되어야 합니다 |
| `Read` | `Read(path pattern)` | `Read(.env)`, `Read(./secrets/**)` |
| `Edit` | `Edit(path pattern)` | `Edit(src/**)`, `Edit(*.ts)` |
| `Write` | `Write(path pattern)` | `Write(*.md)`, `Write(./docs/**)` |
| `NotebookEdit` | `NotebookEdit(pattern)` | `NotebookEdit(*)` |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | 전역 웹 검색 |
| `Task` | `Task(agent-name)` | `Task(Explore)`, `Task(my-agent)` |
| `Agent` | `Agent(name)` | `Agent(researcher)`, `Agent(*)` — 권한이 서브에이전트 생성으로 범위 지정됨 |
| `Skill` | `Skill(skill-name)` 또는 `Skill(prefix *)` | `Skill(weather-fetcher)`, `Skill(weather *)`는 `weather-fetcher`/`weather-svg-creator`와 매칭(v2.1.139) |
| `MCP` | `mcp__server__tool` 또는 `MCP(server:tool)` | `mcp__memory__*`, `MCP(github:*)` |
| `Tool` | `Tool(param:value)` | `Agent(model:opus)`, `Bash(cmd:npm run *)` — 도구의 입력 매개변수에 대해 권한 규칙을 매칭; 값 위치에서 `*` 와일드카드 지원(v2.1.178) |
| `Cd` | `Cd(path pattern)` | `Cd(/home/*)`, `Cd(~/projects/*)` — `/cd` 명령이 이동할 수 있는 디렉터리를 제어 |

**Evaluation order:** 규칙은 순서대로 평가됩니다: deny 규칙 먼저, 그다음 ask, 그다음 allow. 첫 번째로 매칭되는 규칙이 이깁니다.

**Deny rule glob patterns (v2.1.166):** `deny` 규칙에서 도구 이름 위치에 `"*"`를 사용하면 모든 도구와 매칭됩니다 — 전역 deny와 동등합니다. 예를 들어 deny 배열의 `"*"`는 모든 도구 호출을 차단합니다. 이를 통해 접근을 완전히 잠그고 특정 allow/ask 예외를 새겨넣을 수 있습니다.

**Read/Edit path patterns:** `Read`, `Edit`, `Write`에 대한 권한 규칙은 네 가지 접두사 유형을 가진 gitignore 스타일 패턴을 지원합니다:

| Prefix | Meaning | Example |
|--------|---------|---------|
| `//` | 파일시스템 루트로부터의 절대 경로 | `Read(//Users/alice/file)` |
| `~/` | 홈 디렉터리 기준 | `Read(~/.zshrc)` |
| `/` | 프로젝트 루트 기준 | `Edit(/src/**)` |
| `./` 또는 없음 | 상대 경로(현재 디렉터리) | `Read(.env)`, `Read(*.ts)` |

**Symlink resolution:** 권한 규칙은 심볼릭 링크 경로와 해결된 대상 둘 다를 검사합니다. **Allow** 규칙은 심볼릭 링크와 그 대상 *둘 다* 매칭될 때만 적용됩니다 — 허용된 디렉터리 안에 있지만 밖을 가리키는 심볼릭 링크는 여전히 프롬프트합니다. **Deny** 규칙은 심볼릭 링크 *또는* 그 대상 중 하나라도 매칭되면 적용됩니다 — 거부된 파일로의 심볼릭 링크는 그 자체로 거부됩니다.

**Bash wildcard notes:**
- `*`는 **어느 위치에나** 나타날 수 있습니다: 접두사(`Bash(* install)`), 접미사(`Bash(npm *)`), 또는 중간(`Bash(git * main)`)
- **단어 경계:** `Bash(ls *)`(`*` 앞에 공백)는 `ls -la`와 매칭되지만 `lsof`와는 매칭되지 않습니다. `Bash(ls*)`(공백 없음)는 둘 다 매칭됩니다
- `Bash(*)`는 `Bash`와 동등하게 취급됩니다(모든 bash 명령과 매칭)
- 권한 규칙은 출력 리다이렉션을 지원합니다: `Bash(python:*)`는 `python script.py > output.txt`와 매칭됩니다
- 레거시 `:*` 접미사 구문(예: `Bash(npm:*)`)은 ` *`와 동등하지만 더 이상 사용되지 않습니다
- **복합 명령:** 셸 연산자(`&&`, `||`, `;`, `|`, `|&`, `&`, 줄바꿈)는 명령을 분할하며 각 하위 명령이 독립적으로 매칭되어야 합니다 — `Bash(safe-cmd *)`는 `safe-cmd && other-cmd`를 **승인하지 않습니다**
- **프로세스 래퍼:** `timeout`, `time`, `nice`, `nohup`, `stdbuf`는 매칭 전에 제거됩니다(그래서 `Bash(npm test *)`는 `timeout 30 npm test`와도 매칭). 플래그 없는 `xargs`도 제거됩니다. Exec 래퍼 `watch`, `setsid`, `ionice`, `flock`, 그리고 `-exec`/`-delete`가 있는 `find`는 항상 프롬프트하며 접두사 규칙으로 승인할 수 없습니다

**Example:**
```json
{
  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__*"
    ],
    "ask": [
      "Bash(rm *)",
      "Bash(git push *)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)",
      "Bash(curl *)"
    ],
    "additionalDirectories": ["../shared-libs/"]
  }
}
```

---

## Hooks

Hook 구성(이벤트, 속성, 매처, 종료 코드, 환경 변수, HTTP hook)은 전용 저장소에서 관리됩니다:

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** — 사운드 알림 시스템, 25개의 모든 hook 이벤트, HTTP hook, 매처 패턴, 종료 코드, 환경 변수를 포함한 완전한 hook 참조.

Hook 관련 설정 키(`hooks`, `disableAllHooks`(커스텀 상태 표시줄도 비활성화), `allowManagedHooksOnly`, `allowedHttpHookUrls`, `httpHookAllowedEnvVars`)는 거기에 문서화되어 있습니다.

공식 hooks 참조는 [Claude Code Hooks Documentation](https://code.claude.com/docs/en/hooks)을 확인하세요.

---

## MCP Servers

확장 기능을 위한 Model Context Protocol 서버를 구성합니다.

> **OAuth (v2.1.111):** OAuth를 통해 인증하는 MCP 서버는 보호된 리소스 메타데이터 검색을 위해 [RFC 9728](https://datatracker.ietf.org/doc/rfc9728/)을 따릅니다. 규격에 맞는 서버는 `/.well-known/oauth-protected-resource` 아래에 인증 엔드포인트를 노출하며, Claude Code가 OAuth 흐름을 자동으로 완료합니다 — 규격 준수 서버에는 수동 `apiKeyHelper`나 `headersHelper` 스크립트가 필요 없습니다.

> **Reserved server names (v2.1.128+):** `workspace`, `Claude Browser`, `Claude Preview`는 예약된 MCP 서버 이름입니다. 이 이름을 가진 사용자 정의 서버는 로드 시 건너뛰어지며 세션 로그에 경고가 기록됩니다. 충돌을 피하려면 이 이름을 사용하는 기존 서버의 이름을 변경하세요.

> **`.mcp.json` hot-reload (v2.1.139):** `/mcp` Reconnect 작업은 이제 재연결 전에 디스크에서 `.mcp.json`을 다시 읽으므로, 서버를 추가하거나 편집할 때 더 이상 세션 재시작이 필요하지 않습니다. Claude Code는 또한 stdio로 실행되는 MCP 서버 환경에 `CLAUDE_PROJECT_DIR`을 주입하여(v2.1.139) 서버가 프로젝트 루트 기준으로 경로를 해결할 수 있게 합니다.

> **Per-server timeout floor (v2.1.162):** 1000ms 미만의 서버별 `timeout` 값은 무시되고 전역 `MCP_TOOL_TIMEOUT` 기본값이 대신 적용됩니다. 1000ms 이상의 값은 이전과 같이 존중됩니다.

### MCP Settings

| Key | Type | Scope | Description |
|-----|------|-------|-------------|
| `enableAllProjectMcpServers` | boolean | Any | 모든 `.mcp.json` 서버를 자동 승인합니다. **Security note (v2.1.196):** `.mcp.json` 서버는 더 이상 자체 승인되지 않으며, 이제 `enableAllProjectMcpServers: true` 또는 `enabledMcpjsonServers`를 통한 명시적 옵트인이 필요합니다 |
| `enabledMcpjsonServers` | array | Any | 특정 서버 이름을 허용 목록에 추가 |
| `disabledMcpjsonServers` | array | Any | 특정 서버 이름을 차단 목록에 추가 |
| `allowedMcpServers` | array | Managed only | 이름/명령/URL 매칭 허용 목록 |
| `deniedMcpServers` | array | Managed only | 매칭 차단 목록 |
| `allowManagedMcpServersOnly` | boolean | Managed only | 관리 허용 목록에 명시적으로 나열된 MCP 서버만 허용 |
| `channelsEnabled` | boolean | Managed only | Team 및 Enterprise 사용자에게 [channels](https://code.claude.com/docs/en/channels)를 허용합니다. 설정되지 않거나 `false`이면 `--channels` 플래그와 무관하게 채널 메시지 전달이 차단됩니다 |
| `allowedChannelPlugins` | array | Managed only | 메시지를 푸시할 수 있는 채널 플러그인의 허용 목록. 설정하면 기본 Anthropic 허용 목록을 대체합니다. 미정의 = 기본값으로 되돌아감, 빈 배열 = 모든 채널 플러그인 차단. `channelsEnabled: true` 필요. 각 항목은 `marketplace`와 `plugin` 필드를 가진 객체입니다(v2.1.84) |
| `allowAllClaudeAiMcps` | boolean | Managed only | `managed-mcp.json`과 함께 claude.ai 클라우드 MCP 커넥터를 로드합니다. 활성화하면 관리자 배포 관리 MCP 서버에 더해 claude.ai 호스팅 MCP 커넥터를 사용할 수 있게 됩니다 |
| `disableClaudeAiConnectors` | boolean | Any | claude.ai MCP 커넥터의 자동 가져오기를 비활성화합니다. `true`이면 어떤 `allowAllClaudeAiMcps` 정책과 무관하게 claude.ai 클라우드 커넥터가 로드되지 않습니다(v2.1.182) |

### MCP Server Matching (Managed Settings)

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverCommand": "npx @modelcontextprotocol/*" },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" }
  ]
}
```

### Per-Server Tool Loading (`alwaysLoad`, v2.1.121)

기본적으로 MCP 도구 정의는 지연 로드됩니다(도구 검색을 통해 필요할 때 컨텍스트에 로드). `.mcp.json`(또는 인라인 `mcpServers`)의 개별 MCP 서버 항목에 `alwaysLoad: true`를 설정하면 해당 서버를 지연 로드에서 제외합니다 — 그러면 해당 서버의 모든 도구가 `ENABLE_TOOL_SEARCH`와 무관하게 세션 시작 시 미리 로드됩니다. 모든 서버 유형에서 사용 가능하며, Claude Code v2.1.121+가 필요합니다. 매 턴 필요한 소수의 도구에만 사용하세요 — 미리 로드되는 각 도구는 대화에 사용될 수 있는 컨텍스트를 소비합니다.

```json
{
  "mcpServers": {
    "always-on-server": {
      "type": "http",
      "url": "https://mcp.example.com",
      "alwaysLoad": true
    }
  }
}
```

MCP 서버는 도구의 `_meta` 객체에 `"anthropic/alwaysLoad": true`를 포함하여 개별 도구를 항상 로드되도록 표시할 수도 있습니다 — 서버 도구의 일부만 지연 로드를 우회해야 할 때 유용합니다.

**Example:**
```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": ["memory", "github", "filesystem"],
  "disabledMcpjsonServers": ["experimental-server"]
}
```

---

## Sandbox

보안을 위한 bash 명령 샌드박싱을 구성합니다.

### Sandbox Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `sandbox.enabled` | boolean | `false` | bash 샌드박싱 활성화 |
| `sandbox.failIfUnavailable` | boolean | `false` | 샌드박스가 활성화되었지만 시작할 수 없을 때, 샌드박스 없이 실행하는 대신 오류와 함께 종료합니다. 엄격한 샌드박싱을 요구하는 엔터프라이즈 정책에 유용합니다(v2.1.83) |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | 샌드박스 상태일 때 bash를 자동 승인합니다. v2.1.139 기준으로 셸 확장 형태(`$VAR`, `$(cmd)`)가 올바르게 인식되어, 변수 치환을 포함하는 명령이 샌드박스 자동 승인이 활성화된 경우 더 이상 프롬프트로 되돌아가지 않습니다 |
| `sandbox.excludedCommands` | array | `[]` | 샌드박스 밖에서 실행할 명령 |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | `dangerouslyDisableSandbox`를 허용합니다. `false`로 설정하면 탈출구가 완전히 비활성화되며 모든 명령이 샌드박스에서 실행되어야 합니다(또는 `excludedCommands`에 있어야 함). 엄격한 샌드박싱을 요구하는 엔터프라이즈 정책에 유용합니다 |
| `sandbox.ignoreViolations` | object | `{}` | 명령 패턴을 경로 배열에 매핑 — 위반 경고를 억제 *(JSON 스키마에 있음, 공식 설정 페이지에는 없음)* |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | **(Linux and WSL2 only)** 권한 없는 Docker 환경을 위한 약한 샌드박스를 활성화합니다(보안 감소) |
| `sandbox.network.allowUnixSockets` | array | `[]` | **(macOS only)** 샌드박스에서 접근 가능한 특정 Unix 소켓 경로. Linux와 WSL2에서는 seccomp 필터가 소켓 경로를 검사할 수 없으므로 무시됩니다. 대신 `allowAllUnixSockets`를 사용하세요 |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | 모든 Unix 소켓을 허용합니다(`allowUnixSockets`를 재정의). Linux와 WSL2에서는 `socket(AF_UNIX, ...)` 호출을 차단하는 seccomp 필터를 건너뛰므로, 이것이 Unix 소켓을 허용하는 유일한 방법입니다 |
| `sandbox.network.allowLocalBinding` | boolean | `false` | localhost 포트에 바인딩 허용(macOS) |
| `sandbox.network.allowedDomains` | array | `[]` | 샌드박스용 네트워크 도메인 허용 목록 |
| `sandbox.network.deniedDomains` | array | `[]` | bash 샌드박스용 네트워크 도메인 거부 목록. `allowedDomains`의 와일드카드보다 우선합니다. glob 패턴을 지원합니다(예: `"*.example.com"`)(v2.1.113) |
| `sandbox.network.httpProxyPort` | number | - | HTTP 프록시 포트 1-65535(커스텀 프록시) |
| `sandbox.network.socksProxyPort` | number | - | SOCKS5 프록시 포트 1-65535(커스텀 프록시) |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | 관리 허용 목록의 도메인만 허용(관리 설정) |
| `sandbox.network.allowMachLookup` | array | `[]` | (macOS only) 샌드박스가 조회할 수 있는 추가 XPC/Mach 서비스 이름. 접두사 매칭을 위해 단일 후행 `*`를 지원합니다. iOS Simulator나 Playwright처럼 XPC를 통해 통신하는 도구에 필요합니다. 예: `["com.apple.coresimulator.*"]` |
| `sandbox.filesystem.allowWrite` | array | `[]` | 샌드박스된 명령이 쓸 수 있는 추가 경로. 배열은 모든 설정 스코프 전반에 걸쳐 병합됩니다. `Edit(...)` allow 권한 규칙의 경로와도 병합됩니다. 접두사: `/`(절대), `~/`(홈), `./` 또는 없음(프로젝트 설정에서는 프로젝트 상대, 사용자 설정에서는 `~/.claude` 상대). 절대 경로에 대한 이전 `//` 접두사도 여전히 작동합니다. **Note:** 이는 절대에 `//`, 프로젝트 상대에 `/`를 사용하는 [Read/Edit 권한 규칙](#tool-permission-syntax)과 다릅니다 |
| `sandbox.filesystem.denyWrite` | array | `[]` | 샌드박스된 명령이 쓸 수 없는 경로. 배열은 모든 설정 스코프 전반에 걸쳐 병합됩니다. `Edit(...)` deny 권한 규칙의 경로와도 병합됩니다. `allowWrite`와 동일한 경로 접두사 규칙 |
| `sandbox.filesystem.denyRead` | array | `[]` | 샌드박스된 명령이 읽을 수 없는 경로. 배열은 모든 설정 스코프 전반에 걸쳐 병합됩니다. `Read(...)` deny 권한 규칙의 경로와도 병합됩니다. `allowWrite`와 동일한 경로 접두사 규칙 |
| `sandbox.filesystem.allowRead` | array | `[]` | `denyRead` 영역 내에서 읽기 접근을 다시 허용할 경로. `denyRead`보다 우선합니다. 배열은 모든 설정 스코프 전반에 걸쳐 병합됩니다. `allowWrite`와 동일한 경로 접두사 규칙 |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **(Managed only)** 관리 설정의 `allowRead` 경로만 존중됩니다. 사용자·프로젝트·로컬 설정의 `allowRead` 항목은 무시됩니다 |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | (macOS only) 시스템 TLS 신뢰(`com.apple.trustd.agent`) 접근을 허용합니다. 보안을 감소시킵니다 |
| `sandbox.bwrapPath` | string | - | **(Managed only, Linux/WSL2)** bubblewrap(`bwrap`) 바이너리의 절대 경로. 자동 `PATH` 감지를 재정의합니다. 관리 설정에서만 존중되며 사용자·프로젝트 설정에서는 안 됩니다. 예: `/opt/admin/bwrap`(v2.1.133) |
| `sandbox.socatPath` | string | - | **(Managed only, Linux/WSL2)** 샌드박스 네트워크 프록시에 사용되는 `socat` 바이너리의 절대 경로. 자동 `PATH` 감지를 재정의합니다. 관리 설정에서만 존중됩니다. 예: `/opt/admin/socat`(v2.1.133) |
| `sandbox.allowAppleEvents` | boolean | `false` | **(macOS only)** 샌드박스된 명령이 Apple Events를 보내도록 옵트인합니다. `open`, `osascript`, 또는 Apple Events IPC에 의존하는 브라우저 인증 흐름을 사용하는 도구에 필요합니다(v2.1.181) |
| `sandbox.credentials` | object | — | 샌드박스된 하위 프로세스 환경에서 어떤 자격 증명 파일과 환경 변수가 차단되는지에 대한 세밀한 제어. `files`(샌드박스 읽기에서 차단할 파일 경로 배열)와 `envVars`(하위 프로세스 환경에서 제거할 환경 변수 이름 배열)를 포함합니다. `files` 또는 `envVars`의 개별 잘못된 항목은 경고와 함께 제거되고 유효한 부분집합이 적용됩니다(v2.1.187; v2.1.191+에서 하위 배열을 가진 객체로 확장됨) |

**Example:**
```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["git", "docker", "gh"],
    "allowUnsandboxedCommands": false,
    "network": {
      "allowUnixSockets": ["/var/run/docker.sock"],
      "allowLocalBinding": true
    }
  }
}
```

---

## Plugins

Claude Code 플러그인과 마켓플레이스를 구성합니다.

### Plugin Settings

| Key | Type | Scope | Description |
|-----|------|-------|-------------|
| `enabledPlugins` | object | Any | 특정 플러그인 활성화/비활성화 |
| `extraKnownMarketplaces` | object | Project | 커스텀 플러그인 마켓플레이스 추가(`.claude/settings.json`을 통한 팀 공유) |
| `strictKnownMarketplaces` | array | Managed only | 허용된 마켓플레이스의 허용 목록 |
| `strictPluginOnlyCustomization` | boolean \| array | Managed only | 사용자 및 프로젝트 소스의 스킬, 에이전트, hook, MCP 서버를 차단하여 플러그인이나 관리 설정에서만 올 수 있게 합니다. `true`는 네 개의 표면을 모두 잠급니다. `["skills", "hooks"]` 같은 배열은 이름이 지정된 것만 잠급니다 |
| `pluginSuggestionMarketplaces` | array | Managed only | 세션 중 상황별 설치 제안으로 플러그인이 나타날 수 있는 마켓플레이스 이름의 허용 목록. "이 플러그인이 필요할 수 있습니다" 프롬프트를 표면화할 수 있는 마켓플레이스를 제한합니다(v2.1.152) |
| `skippedMarketplaces` | array | Any | 사용자가 설치를 거부한 마켓플레이스 *(JSON 스키마에 있음, 공식 설정 페이지에는 없음)* |
| `skippedPlugins` | array | Any | 사용자가 설치를 거부한 플러그인 *(JSON 스키마에 있음, 공식 설정 페이지에는 없음)* |
| `pluginConfigs` | object | Any | 플러그인별 MCP 서버 구성(`plugin@marketplace`를 키로 함) *(JSON 스키마에 있음, 공식 설정 페이지에는 없음)* |
| `blockedMarketplaces` | array | Managed only | 특정 플러그인 마켓플레이스를 차단합니다. 각 항목은 소스 문자열, `hostPattern`, 또는 `pathPattern`으로 매칭할 수 있습니다 — v2.1.119 기준으로 `hostPattern`과 `pathPattern` 매처가 다운로드가 파일시스템에 닿기 전에 올바르게 집행되어, 차단된 마켓플레이스가 절대 디스크에 도달하지 않습니다 |
| `pluginTrustMessage` | string | Managed only | 사용자에게 플러그인 신뢰를 프롬프트할 때 표시되는 커스텀 메시지 |
| `disableSideloadFlags` | boolean | Managed only | `--plugin-dir`, `--plugin-url`, `--agents`, `--mcp-config` 시작 플래그를 거부합니다. `true`이면 사용자가 실행 시 사이드로드 플래그를 전달하여 `strictKnownMarketplaces`를 우회할 수 없습니다. 마켓플레이스 전용 플러그인 배포를 강제하기 위해 관리 환경에서 사용하세요(v2.1.193) |

**Marketplace source types:** `github`, `git`, `directory`, `hostPattern`, `settings`, `url`, `npm`, `file`. 호스팅된 마켓플레이스 저장소를 설정하지 않고 소수의 플러그인을 인라인으로 선언하려면 `source: 'settings'`를 사용하세요.

**Example:**
```json
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true,
    "experimental@acme-tools": false
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    },
    "inline-tools": {
      "source": {
        "source": "settings",
        "name": "inline-tools",
        "plugins": [
          {
            "name": "code-formatter",
            "source": { "source": "github", "repo": "acme-corp/code-formatter" }
          }
        ]
      }
    }
  }
}
```

---

## Model Configuration

### Model Aliases

| Alias | Description |
|-------|-------------|
| `"default"` | 계정 유형에 권장됨 |
| `"sonnet"` | 최신 Sonnet 모델(Anthropic API에서는 Claude Sonnet 5 — 네이티브 1M 토큰 컨텍스트, v2.1.197에서 도입; Bedrock/Vertex/Foundry에서는 Claude Sonnet 4.6) |
| `"opus"` | 최신 Opus 모델(v2.1.154 기준 Anthropic API에서 Claude Opus 4.8; v2.1.207 기준 Bedrock, Vertex, AWS의 Claude Platform에서도 Opus 4.8이 기본값 — 이전에는 4.6이었음). v2.1.142부터 fast-mode 기본값이기도 합니다. Opus 4.8은 `high` 노력을 기본값으로 하며 `/effort xhigh`를 지원합니다 |
| `"haiku"` | 빠른 Haiku 모델 |
| `"sonnet[1m]"` | 1M 토큰 컨텍스트를 가진 Sonnet |
| `"opus[1m]"` | 1M 토큰 컨텍스트를 가진 Opus(v2.1.75부터 Max, Team, Enterprise에서 기본값) |
| `"opusplan"` | 계획에는 Opus, 실행에는 Sonnet |
| `"fable"` | Claude Fable 5 — 장기 추론 모델. Anthropic API 전용(v2.1.170+). Fable 5는 기본적으로 1M 컨텍스트를 포함합니다. `[1m]` 접미사는 자동으로 제거되므로 `fable[1m]`은 중복입니다(v2.1.173) |

**Example:**
```json
{
  "model": "opus"
}
```

> **Note (v2.1.144):** `/model`은 **현재 세션에 대해서만** 모델을 변경합니다. `/model` 선택기에서 `d`를 누르면 선택을 기본값으로도 설정합니다. `model` 설정과 `ANTHROPIC_MODEL`은 계속해서 영구 기본값을 제어합니다.

### Model Overrides

Bedrock, Vertex, 또는 Foundry 배포를 위해 Anthropic 모델 ID를 공급자별 모델 ID에 매핑합니다.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `effortLevel` | string | - | 세션 전반에 걸쳐 노력 수준을 영구 저장합니다. `"low"`, `"medium"`, `"high"`, 또는 `"xhigh"`(Opus 4.7 및 4.8, v2.1.111)를 허용합니다. `/effort low`, `/effort medium`, `/effort high`, 또는 `/effort xhigh`를 실행하면 자동으로 기록됩니다. Opus 4.6, Sonnet 4.6, Opus 4.7, Opus 4.8(`high`가 기본값)에서 지원됩니다. 지원되지 않는 수준은 활성 모델에서 지원되는 가장 높은 수준으로 되돌아갑니다 |
| `fallbackModel` | array | - | 기본 모델을 사용할 수 없을 때(예: 속도 제한 또는 용량 문제) 순차적으로 시도되는 최대 3개의 대체 모델 ID. 각 항목은 모델 ID 또는 별칭입니다. Claude Code는 먼저 기본 모델을 시도하고, 실패하면 각 대체를 순서대로 시도합니다. 첫 번째 성공 응답에서 멈춥니다(v2.1.166) |
| `modelOverrides` | object | - | 모델 선택기 항목을 공급자별 ID(예: Bedrock 추론 프로필 ARN)에 매핑합니다. 각 키는 모델 선택기 항목 이름이고, 각 값은 공급자 모델 ID입니다 |

**Example:**
```json
{
  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0",
    "claude-sonnet-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-sonnet-4-6-v1:0"
  }
}
```

### Effort Level

`/model` 명령은 모델이 응답당 적용하는 추론량을 조정하는 **노력 수준(effort level)** 제어를 노출합니다. `/model` UI에서 ← → 화살표 키를 사용하여 노력 수준을 순환하세요.

| Effort Level | Description |
|-------------|-------------|
| Max | 최대 추론 깊이, Opus 4.6 전용 |
| XHigh | 확장 고추론 깊이, Opus 4.7 및 4.8(모든 플랜의 Opus 4.7에서 기본값, v2.1.111; Opus 4.8에서는 사용 가능하지만 기본값은 `high`, v2.1.154) |
| High (Opus 4.6/Sonnet 4.6에서 기본값) | 전체 추론 깊이, 복잡한 작업에 최적 |
| Medium | 균형 잡힌 추론, 일상적 작업에 적합 |
| Low | 최소 추론, 가장 빠른 응답 |

**How to use:**
1. `/effort low`, `/effort medium`, 또는 `/effort high`를 실행하여 직접 설정(v2.1.76+)
2. 또는 `/model` 실행 → 모델 선택 → **← →** 화살표 키로 조정
3. 설정은 `settings.json`의 `effortLevel` 키를 통해 유지됩니다

**Note:** 노력 수준은 Max 및 Team 플랜의 Opus 4.6, Sonnet 4.6, Opus 4.7, Opus 4.8에서 사용 가능합니다. 기본값은 v2.1.68에서 High에서 Medium으로 변경되었다가, v2.1.94에서 API 키, Bedrock/Vertex/Foundry, Team, Enterprise 사용자에 대해 다시 **High**로 변경되었습니다. v2.1.117에서는 Opus 4.6 및 Sonnet 4.6의 Pro/Max 구독자에 대해서도 기본값이 `medium`에서 `high`로 상향되어, 모든 등급이 `high`로 정렬되었습니다. v2.1.111은 **`xhigh`**(당시 Opus 4.7 전용)를 도입하고 모든 플랜의 Opus 4.7에서 이를 기본 노력 수준으로 만들었습니다. **v2.1.154**는 Anthropic API의 최신 Opus로 **Opus 4.8**을 추가했으며, `xhigh`를 지원하지만 기본값은 `high`입니다. v2.1.75 기준으로 Opus 4.6의 1M 컨텍스트 창은 Max, Team, Enterprise 플랜에서 기본적으로 사용 가능합니다.

**Effort env propagation:** 스킬 파일 내부에서는 `${CLAUDE_EFFORT}`를 사용하여 현재 노력 수준을 참조합니다(v2.1.120). v2.1.133 기준으로 동일한 `$CLAUDE_EFFORT` 변수가 Bash 도구 하위 프로세스와 hook 핸들러의 환경에도 주입되어, 셸 스크립트와 hook 명령이 별도의 구성 파일을 읽지 않고도 활성 노력 등급에 따라 동작을 조정할 수 있습니다.

### Model Environment Variables

`env` 키를 통해 구성:

```json
{
  "env": {
    "ANTHROPIC_MODEL": "sonnet",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "custom-haiku-model",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "custom-sonnet-model",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "custom-opus-model",
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku",
    "MAX_THINKING_TOKENS": "10000"
  }
}
```

---

## Display & UX

### Display Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `statusLine` | object | - | 커스텀 상태 표시줄 구성 |
| `outputStyle` | string | `"default"` | 출력 스타일(예: `"Explanatory"`) |
| `spinnerTipsEnabled` | boolean | `true` | 대기 중 팁 표시 |
| `spinnerVerbs` | object | - | `mode`("append" 또는 "replace")와 `verbs` 배열을 가진 커스텀 스피너 동사 |
| `spinnerTipsOverride` | object | - | `tips`(문자열 배열)와 선택적 `excludeDefault`(boolean)를 가진 커스텀 스피너 팁. `excludeDefault`가 `true`이면 커스텀 팁만 표시되고, `false`이거나 없으면 커스텀 팁이 내장 팁과 병합됩니다. v2.1.121 기준으로 `excludeDefault: true`는 시간 기반 스피너 팁도 억제합니다 |
| `respectGitignore` | boolean | `true` | 파일 선택기에서 .gitignore 존중 |
| `prefersReducedMotion` | boolean | `false` | UI에서 애니메이션과 모션 효과 감소 |
| `axScreenReader` | boolean | `false` | 스크린 리더 친화적 출력 모드를 활성화합니다. `true`이면 Claude가 장식용 상자 그리기 문자, 색상, 기타 터미널 UI 요소 없이 평면 텍스트를 출력합니다. `/config`에 **Screen reader mode**로 표시됩니다. User 스코프에서만 적용되며 프로젝트나 관리 설정에서는 읽지 않습니다. `--ax-screen-reader` CLI 플래그로도 사용 가능합니다(v2.1.181) |
| `syntaxHighlightingDisabled` | boolean | `false` | diff, 코드 블록, 파일 미리보기에서 구문 강조를 비활성화합니다. diff 출력만 관장하는 `CLAUDE_CODE_SYNTAX_HIGHLIGHT` 환경 변수와는 구별됩니다 |
| `fileSuggestion` | object | - | 커스텀 파일 제안 명령(아래 File Suggestion Configuration 참조) |
| `autoScrollEnabled` | boolean | `true` | 전체 화면 모드에서 대화를 자동 스크롤합니다. 자동 스크롤을 비활성화하려면 `false`로 설정하세요(v2.1.110). v2.1.119 이전 버전은 이를 `~/.claude.json`에 저장했습니다 |
| `editorMode` | string | `"normal"` | 입력 프롬프트의 키 바인딩 모드: `"normal"` 또는 `"vim"`. `/config`에 **Editor mode**로 표시됩니다. v2.1.119 이전 버전은 이를 `~/.claude.json`에 저장했습니다 |
| `showTurnDuration` | boolean | `true` | 응답 후 턴 지속 시간 메시지를 표시합니다(예: "Cooked for 1m 6s"). v2.1.119 이전 버전은 이를 `~/.claude.json`에 저장했습니다 |
| `teammateMode` | string | `"in-process"` | [agent team](https://code.claude.com/docs/en/agent-teams) 팀메이트가 표시되는 방식: `"auto"`(tmux나 iTerm2에서는 분할 창을 선택, 그 외에는 in-process), `"in-process"`(v2.1.179부터 기본값), `"tmux"`, 또는 `"iterm2"`(자동 감지와 무관하게 iTerm2 분할 창을 강제, v2.1.186). [choose a display mode](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode)를 참조하세요. v2.1.119 이전 버전은 이를 `~/.claude.json`에 저장했습니다 |
| `terminalProgressBarEnabled` | boolean | `true` | 지원되는 터미널(ConEmu, Ghostty 1.2.0+, iTerm2 3.6.6+)에서 터미널 진행 표시줄을 표시합니다. `/config`에 **Terminal progress bar**로 표시됩니다. v2.1.119 이전 버전은 이를 `~/.claude.json`에 저장했습니다 |
| `preferredNotifChannel` | string | `"auto"` | 작업 완료 및 권한 프롬프트 알림 방법. 값: `"auto"`, `"terminal_bell"`, `"iterm2"`, `"iterm2_with_bell"`, `"kitty"`, `"ghostty"`, `"notifications_disabled"`. 기본값 `"auto"`는 iTerm2, Ghostty, Kitty에서 데스크톱 알림을 보내고 다른 터미널에서는 아무것도 하지 않습니다. 모든 터미널에서 벨 문자를 울리려면 `"terminal_bell"`로 설정하세요. `/config`에 **Notifications**로 표시됩니다. [Get a terminal bell or notification](https://code.claude.com/docs/en/terminal-config#get-a-terminal-bell-or-notification)을 참조하세요 |
| `wheelScrollAccelerationEnabled` | boolean | `true` | 전체 화면 모드에서 마우스 휠 스크롤 가속을 비활성화합니다. OS 수준 가속 곡선 대신 고정된 틱당 스크롤 단계를 사용하려면 `false`로 설정하세요(v2.1.174) |
| `footerLinksRegexes` | array | - | 푸터 행에 링크 배지로 표시하기 위해 URL에 대해 매칭되는 정규식 패턴. 매칭되는 각 URL은 채팅 UI 하단에 클릭 가능한 배지를 생성합니다(v2.1.176) |

### Global Config Settings (`~/.claude.json`)

이 IDE 관련 기본 설정은 `settings.json`이 **아니라** `~/.claude.json`에 저장됩니다.

> **v2.1.119 migration note:** v2.1.119 기준으로 `autoScrollEnabled`, `editorMode`, `showTurnDuration`, `teammateMode`, `terminalProgressBarEnabled`가 `settings.json`으로 이동했으며 위의 Display Settings 표에 문서화되어 있습니다. 이전 버전은 여기에 저장했습니다.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `autoConnectIde` | boolean | `false` | Claude Code가 외부 터미널에서 시작될 때 실행 중인 IDE에 자동으로 연결합니다. VS Code나 JetBrains 터미널 밖에서 실행할 때 `/config`에 **Auto-connect to IDE (external terminal)**로 표시됩니다 |
| `autoInstallIdeExtension` | boolean | `true` | VS Code 터미널에서 실행할 때 Claude Code IDE 확장을 자동으로 설치합니다. `/config`에 **Auto-install IDE extension**으로 표시됩니다. `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` 환경 변수로도 비활성화할 수 있습니다 |
| `externalEditorContext` | boolean | `false` | `Ctrl+G`로 외부 편집기를 열 때 Claude의 이전 응답을 `#`으로 주석 처리된 컨텍스트로 앞에 추가합니다. 활성화하려면 `true`로 설정하세요 |
| `teammateDefaultModel` | string | `null` | 리드가 디스패치할 때 [agent-team](https://code.claude.com/docs/en/agent-teams) 팀메이트의 기본 모델. `null`은 리드의 모델을 상속합니다. 공식 설정 페이지의 "Global config settings" 아래에 나열되어 있습니다 |

### Workspace & Teams

| Key | Type | Description |
|-----|------|-------------|
| `sshConfigs` | object[] | Desktop에서 드롭다운으로 표면화되는 SSH 연결 정의. 각 항목은 `id`, `name`, `sshHost`를 반드시 포함해야 하며, 선택적으로 `sshPort`, `sshIdentityFile`, `startDirectory`를 포함합니다 |

**Field reference:**

| Field | Required | Description |
|-------|----------|-------------|
| `id` | yes | SSH 연결 항목의 고유 식별자 |
| `name` | yes | Desktop 드롭다운에 표시되는 표시 이름 |
| `sshHost` | yes | SSH 호스트(예: `user@dev.example.com` 또는 `dev.example.com`) |
| `sshPort` | no | SSH 포트 번호 |
| `sshIdentityFile` | no | SSH 아이덴티티 파일(개인 키) 경로 |
| `startDirectory` | no | 연결 후 초기 작업 디렉터리 |

**Example:**
```json
{
  "sshConfigs": [
    {
      "id": "dev-vm",
      "name": "Dev VM",
      "sshHost": "user@dev.example.com",
      "sshPort": 22,
      "sshIdentityFile": "~/.ssh/id_ed25519",
      "startDirectory": "/home/user/project"
    }
  ]
}
```

### Status Line Configuration

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 2,
    "refreshInterval": 5
  }
}
```

| Field | Description |
|-------|-------------|
| `type` | 셸 스크립트를 실행하려면 `"command"`로 설정 |
| `command` | 상태 표시줄 출력을 생성하는 셸 명령 또는 스크립트 경로 |
| `padding` | 상태 표시줄 콘텐츠에 추가되는 추가 수평 간격(문자 단위). 기본값은 `0`. 인터페이스의 내장 간격을 넘어서는 상대적 들여쓰기를 제어합니다 |
| `refreshInterval` | 이벤트 기반 업데이트에 더해 N초마다 명령을 다시 실행합니다. 최소는 `1`. 상태 표시줄이 시간 기반 데이터(예: 시계)를 표시하거나, 메인 세션이 유휴 상태일 때 백그라운드 서브에이전트가 git 상태를 변경할 때 유용합니다. 이벤트에서만 실행하려면 설정하지 마세요(v2.1.97) |

**Status Line Input Fields:**

상태 표시줄 명령은 stdin으로 JSON 객체를 받습니다. 전체 JSON 스키마와 예제는 [Status Line Documentation](https://code.claude.com/docs/en/statusline)을 참조하세요.

| Field | Description |
|-------|-------------|
| `model.id`, `model.display_name` | 현재 모델 식별자와 표시 이름 |
| `cwd`, `workspace.current_dir` | 현재 작업 디렉터리(둘 다 동일한 값을 포함; `workspace.current_dir` 선호) |
| `workspace.project_dir` | Claude Code가 실행된 디렉터리(작업 디렉터리가 바뀌면 `cwd`와 다를 수 있음) |
| `workspace.added_dirs` | `/add-dir` 또는 `--add-dir`로 추가된 추가 디렉터리 |
| `workspace.git_worktree` | `git worktree add`로 생성된 연결된 워크트리 내부에 있을 때의 Git 워크트리 이름. 메인 작업 트리에서는 없음(v2.1.97) |
| `cost.total_cost_usd` | 총 세션 비용(USD) |
| `cost.total_duration_ms` | 세션 시작 이후 총 벽시계 시간(밀리초) |
| `cost.total_api_duration_ms` | API 응답 대기에 소요된 총 시간(밀리초) |
| `cost.total_lines_added`, `cost.total_lines_removed` | 세션 중 변경된 코드 줄 수 |
| `context_window.total_input_tokens`, `context_window.total_output_tokens` | 세션 전반의 누적 토큰 수 |
| `context_window.context_window_size` | 최대 컨텍스트 창 크기(토큰)(기본값 200000, 확장 컨텍스트는 1000000) |
| `context_window.used_percentage` | 미리 계산된 컨텍스트 창 사용 비율 |
| `context_window.remaining_percentage` | 미리 계산된 컨텍스트 창 잔여 비율 |
| `context_window.current_usage` | 마지막 API 호출의 토큰 수(입력, 출력, 캐시 토큰) |
| `exceeds_200k_tokens` | 가장 최근 API 응답의 총 토큰이 200k를 초과하는지 여부(고정 임계값) |
| `rate_limits.five_hour.used_percentage` | 5시간 속도 제한 사용 비율(v2.1.80+) |
| `rate_limits.five_hour.resets_at` | 5시간 속도 제한 재설정 타임스탬프(Unix epoch 초) |
| `rate_limits.seven_day.used_percentage` | 7일 속도 제한 사용 비율 |
| `rate_limits.seven_day.resets_at` | 7일 속도 제한 재설정 타임스탬프(Unix epoch 초) |
| `session_id` | 고유 세션 식별자 |
| `session_name` | `--name` 또는 `/rename`으로 설정한 커스텀 세션 이름. 커스텀 이름이 없으면 없음 |
| `transcript_path` | 대화 기록 파일 경로 |
| `version` | Claude Code 버전 |
| `output_style.name` | 현재 출력 스타일 이름 |
| `vim.mode` | vim 모드가 활성화된 경우 현재 vim 모드(`NORMAL` 또는 `INSERT`) |
| `agent.name` | `--agent` 플래그 또는 에이전트 설정으로 실행할 때의 에이전트 이름 |
| `effort.level` | 현재 추론 노력(`low`, `medium`, `high`, `xhigh`, 또는 `max`). 세션 중간의 `/effort` 변경을 포함한 실시간 세션 값을 반영합니다. 현재 모델이 노력 매개변수를 지원하지 않으면 없음(v2.1.121) |
| `thinking.enabled` | 세션에서 확장 사고가 활성화되었는지 여부(v2.1.121) |
| `worktree.name` | 활성 워크트리 이름(`--worktree` 세션 중에만 존재) |
| `worktree.path` | 워크트리 디렉터리의 절대 경로 |
| `worktree.branch` | 워크트리의 Git 브랜치 이름. hook 기반 워크트리에서는 없음 |
| `worktree.original_cwd` | 워크트리 진입 전 디렉터리 |
| `worktree.original_branch` | 워크트리 진입 전 체크아웃된 Git 브랜치. hook 기반 워크트리에서는 없음 |
| `github` | 감지된 경우 현재 브랜치의 GitHub 저장소 및 풀 리퀘스트 정보 — 저장소 아이덴티티와 연관된 PR(v2.1.145) |

### File Suggestion Configuration

파일 제안 스크립트는 stdin으로 JSON 객체를 받고(예: `{"query": "src/comp"}`) 최대 15개의 파일 경로(한 줄에 하나씩)를 출력해야 합니다.

```json
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  },
  "respectGitignore": true
}
```

**Example:**
```json
{
  "statusLine": {
    "type": "command",
    "command": "git branch --show-current 2>/dev/null || echo 'no-branch'"
  },
  "spinnerTipsEnabled": true,
  "spinnerVerbs": {
    "mode": "replace",
    "verbs": ["Cooking", "Brewing", "Crafting", "Conjuring"]
  },
  "spinnerTipsOverride": {
    "tips": ["Use /compact at ~50% context", "Start with plan mode for complex tasks"],
    "excludeDefault": true
  }
}
```

---

## AWS & Cloud Credentials

### AWS Settings

| Key | Type | Description |
|-----|------|-------------|
| `awsAuthRefresh` | string | AWS 인증을 갱신하는 스크립트(`.aws` 디렉터리를 수정) |
| `awsCredentialExport` | string | AWS 자격 증명이 포함된 JSON을 출력하는 스크립트 |

**Example:**
```json
{
  "awsAuthRefresh": "aws sso login --profile myprofile",
  "awsCredentialExport": "/bin/generate_aws_grant.sh"
}
```

### OpenTelemetry

| Key | Type | Description |
|-----|------|-------------|
| `otelHeadersHelper` | string | 동적 OpenTelemetry 헤더를 생성하는 스크립트 |

**Example:**
```json
{
  "otelHeadersHelper": "/bin/generate_otel_headers.sh"
}
```

---

## Environment Variables (via `env`)

모든 Claude Code 세션에 대해 환경 변수를 설정합니다.

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "...",
    "NODE_ENV": "development",
    "DEBUG": "true"
  }
}
```

### Common Environment Variables

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | 인증용 API 키 |
| `ANTHROPIC_AUTH_TOKEN` | OAuth 토큰 |
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude.ai 인증을 위한 OAuth 액세스 토큰. SDK 및 자동화 환경에서 `/login`의 대안입니다. 키체인에 저장된 자격 증명보다 우선합니다 |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | Claude.ai 인증을 위한 OAuth 리프레시 토큰. 설정하면 `claude auth login`이 브라우저를 여는 대신 이 토큰을 직접 교환합니다. `CLAUDE_CODE_OAUTH_SCOPES`가 필요합니다 |
| `CLAUDE_CODE_OAUTH_SCOPES` | 리프레시 토큰이 발급된 공백으로 구분된 OAuth 스코프(예: `"user:profile user:inference user:sessions:claude_code"`). `CLAUDE_CODE_OAUTH_REFRESH_TOKEN`이 설정되면 필요합니다 |
| `ANTHROPIC_WORKSPACE_ID` | [workload identity federation](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation)용 워크스페이스 ID. 페더레이션 규칙이 둘 이상의 워크스페이스로 범위 지정되어 토큰 교환이 어느 워크스페이스를 대상으로 할지 알아야 할 때 설정합니다(v2.1.141) |
| `ANTHROPIC_BASE_URL` | 커스텀 API 엔드포인트 |
| `ANTHROPIC_BEDROCK_BASE_URL` | Bedrock 엔드포인트 URL 재정의 |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | Bedrock Mantle 엔드포인트 URL 재정의. [Mantle endpoint](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint)를 참조하세요 |
| `ANTHROPIC_BEDROCK_SERVICE_TIER` | Bedrock 서비스 등급: `default`, `flex`, 또는 `priority`. 모든 요청에 `X-Amzn-Bedrock-Service-Tier` 헤더로 전송됩니다. [Amazon Bedrock service tiers](https://code.claude.com/docs/en/amazon-bedrock#service-tiers)를 참조하세요(v2.1.122) |
| `ANTHROPIC_AWS_API_KEY` | AWS의 Claude Platform용 워크스페이스 API 키 |
| `ANTHROPIC_AWS_BASE_URL` | AWS의 Claude Platform 엔드포인트 URL 재정의 |
| `ANTHROPIC_AWS_WORKSPACE_ID` | AWS의 Claude Platform에 필요한 워크스페이스 ID |
| `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST` | Claude Code를 임베드하고 사용자를 대신하여 모델 공급자 라우팅을 관리하는 호스트 플랫폼이 설정합니다. 설정되면 `settings.json`의 공급자 선택/엔드포인트/인증 환경 변수(예: `CLAUDE_CODE_USE_BEDROCK`, `ANTHROPIC_BASE_URL`, `ANTHROPIC_API_KEY`)가 무시되어 사용자 설정이 호스트의 라우팅을 재정의할 수 없습니다. Bedrock/Vertex/Foundry에 대한 자동 텔레메트리 옵트아웃도 건너뛰어져 텔레메트리는 표준 `DISABLE_TELEMETRY` 옵트아웃을 따릅니다(v2.1.126) |
| `ANTHROPIC_VERTEX_BASE_URL` | Vertex AI 엔드포인트 URL 재정의 |
| `ANTHROPIC_BETAS` | 쉼표로 구분된 Anthropic 베타 헤더 값 |
| `ANTHROPIC_VERTEX_PROJECT_ID` | Vertex AI용 GCP 프로젝트 ID |
| `GCLOUD_PROJECT` | Vertex AI 요청용 GCP 프로젝트 ID(`ANTHROPIC_VERTEX_PROJECT_ID`를 재정의) |
| `GOOGLE_APPLICATION_CREDENTIALS` | Vertex AI 인증용 GCP 서비스 계정 자격 증명 파일 경로 |
| `GOOGLE_CLOUD_PROJECT` | Vertex AI 요청용 GCP 프로젝트 ID(`ANTHROPIC_VERTEX_PROJECT_ID`를 재정의) |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | `/model` 선택기에 커스텀 항목으로 추가할 모델 ID. 내장 별칭을 대체하지 않고 비표준 또는 게이트웨이별 모델을 선택 가능하게 만드는 데 사용합니다 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | `/model` 선택기에서 커스텀 모델 항목의 표시 이름. 설정하지 않으면 모델 ID로 기본 설정됩니다 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | `/model` 선택기에서 커스텀 모델 항목의 표시 설명. 설정하지 않으면 `Custom model (<model-id>)`로 기본 설정됩니다 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_SUPPORTED_CAPABILITIES` | 커스텀 모델 항목의 기능 감지를 재정의합니다. 쉼표로 구분된 값(예: `effort,thinking`). 커스텀 모델이 자동 감지가 확인할 수 없는 기능을 지원할 때 필요합니다. [model configuration](https://code.claude.com/docs/en/model-config#customize-pinned-model-display-and-capabilities)을 참조하세요 |
| `ANTHROPIC_MODEL` | 사용할 모델 이름. 별칭(`sonnet`, `opus`, `haiku`) 또는 전체 모델 ID를 허용합니다. `model` 설정을 재정의합니다 |
| `INIT_PROMPT` | 세션 초기화 시 주입되는 커스텀 시스템 프롬프트 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Haiku 모델 별칭을 커스텀 모델 ID로 재정의합니다(예: 서드파티 배포용) |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` | Bedrock/Vertex/Foundry에서 고정 모델을 사용할 때 `/model` 선택기의 Haiku 항목 레이블을 커스터마이징합니다. 기본값은 모델 ID |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION` | `/model` 선택기의 Haiku 항목 설명을 커스터마이징합니다. 기본값은 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES` | 고정 Haiku 모델의 기능 감지를 재정의합니다. 쉼표로 구분된 값(예: `effort,thinking`). 고정 모델이 자동 감지가 확인할 수 없는 기능을 지원할 때 필요합니다 |
| `CLAUDECODE` | Claude Code가 생성하는 셸 환경(Bash 도구, tmux 세션)에서 `1`로 설정됩니다. hook이나 상태 표시줄 명령에서는 설정되지 않습니다. 스크립트가 Claude Code 셸 내부에서 실행 중인지 감지하는 데 사용합니다 |
| `CLAUDE_CODE_CHILD_SESSION` | Claude Code가 Bash, PowerShell, Monitor 도구, hook 명령, 상태 표시줄 명령을 통해 생성하는 하위 프로세스에서 `1`로 설정됩니다. stdio MCP 서버 하위 프로세스에서는 설정되지 않습니다. `CLAUDECODE`와 달리 이것은 Claude Code 자체의 생성 경로에서만 설정되므로(IDE 확장 아님), 중첩된 `claude` 세션을 IDE 통합 터미널에서 시작된 최상위 `claude`와 안정적으로 구별합니다. 중첩된 대화형 TUI 세션은 `--resume`, `--continue`, 위쪽 화살표 기록, `claude agents`에서 자동으로 제외됩니다. 비대화형 `claude -p` 세션은 여전히 유지됩니다. 이 제외를 재정의하려면 `CLAUDE_CODE_FORCE_SESSION_PERSISTENCE=1`을 설정하세요(v2.1.172) |
| `CLAUDE_CODE_FORCE_SESSION_PERSISTENCE` | 중첩된 대화형 TUI 세션을 `--resume`, `--continue`, 위쪽 화살표 기록, `claude agents`에서 자동 제외하는 것을 재정의하려면 `1`로 설정합니다. 기본적으로 중첩 세션(`CLAUDE_CODE_CHILD_SESSION=1`)은 기록을 오염시키지 않도록 제외됩니다. 중첩 세션을 추적하고 싶을 때 유지를 강제하려면 이것을 설정하세요 |
| `CLAUDE_CODE_SESSION_ID` | 읽기 전용. Bash 및 PowerShell 도구 하위 프로세스에서 현재 세션 ID로 자동 설정됩니다. hook에 전달되는 `session_id` 필드와 일치합니다. `/clear` 시 업데이트됩니다. 스크립트와 외부 도구를 그것을 시작한 Claude Code 세션과 연관 짓는 데 사용합니다(v2.1.132). `--resume` 시 stdio MCP 서버 환경에도 주입됩니다(v2.1.163 changelog) *(v2.1.163 changelog에 있음; 공식 env-vars 페이지에는 아직 없음 — 읽기 전용)* |
| `AI_AGENT` | Claude Code가 하위 프로세스 환경(Bash 도구, hook, MCP stdio 서버)에서 자동으로 설정합니다. 부모 프로세스를 AI 에이전트로 식별하는 일반 플래그 — `CLAUDECODE` 같은 각 에이전트별 변수를 확인하는 대신 어떤 AI 에이전트에서든 호출될 때 동작을 조정하는 도구에 유용합니다 *(v2.1.120 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` | 조직 상태 확인이 네트워크 오류로 실패할 때 fast mode를 허용하려면 `1`로 설정합니다. 회사 프록시가 상태 엔드포인트를 차단할 때 유용합니다 |
| `CLAUDE_CODE_USE_BEDROCK` | AWS Bedrock 사용(활성화하려면 `1`) |
| `CLAUDE_CODE_USE_VERTEX` | Google Vertex AI 사용(활성화하려면 `1`) |
| `CLAUDE_CODE_USE_FOUNDRY` | Microsoft Foundry 사용(활성화하려면 `1`) |
| `CLAUDE_CODE_USE_MANTLE` | Bedrock [Mantle endpoint](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) 사용(활성화하려면 `1`) |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | Windows에서 PowerShell 도구를 활성화하려면 `1`로 설정(옵트인 프리뷰). 활성화하면 Claude가 Git Bash를 통해 라우팅하는 대신 PowerShell 명령을 네이티브로 실행할 수 있습니다. 네이티브 Windows에서만 지원되며 WSL에서는 지원되지 않습니다(v2.1.84) |
| `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY` | 도구 호출, hook, 상태 표시줄 명령을 위해 PowerShell을 생성할 때 Claude Code가 `-ExecutionPolicy Bypass`를 전달하지 않도록 하여 대신 머신의 유효 실행 정책을 존중하게 하려면 `1`로 설정합니다. 기본적으로 Claude Code는 기본 Restricted Windows에서 `.ps1` 스크립트와 모듈 가져오기가 작동하도록 프로세스 스코프에서 실행 정책을 우회합니다. Group Policy `MachinePolicy`/`UserPolicy`는 절대 재정의하지 않습니다(v2.1.143) |
| `CLAUDE_CODE_REMOTE` | 읽기 전용. Claude Code가 클라우드 세션으로 실행될 때 `true`로 자동 설정됩니다. hook이나 설정 스크립트에서 이를 읽어 클라우드 환경에 있는지 감지하세요 |
| `CLAUDE_CODE_REMOTE_SESSION_ID` | 읽기 전용. 클라우드 세션에서 현재 세션의 ID로 자동 설정됩니다. 세션 기록으로 돌아가는 링크를 구성하려면 이를 읽으세요 |
| `CLAUDE_CODE_BRIDGE_SESSION_ID` | 읽기 전용. 세션에 활성 Remote Control 연결이 있는 동안 Bash 도구 및 hook 명령 하위 프로세스에서 자동 설정됩니다. 값은 `session_` 형태의 세션 ID입니다(세션의 `claude.ai/code` URL과 동일한 식별자). 연결이 끝나면 제거됩니다. 스크립트가 그것을 실행한 세션으로 돌아가는 링크를 만들 수 있게 합니다. 클라우드 세션에서는 대신 `CLAUDE_CODE_REMOTE_SESSION_ID`를 읽으세요(v2.1.199) |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | 자동 생성되는 Remote Control 세션 이름의 접두사. 기본값은 머신 호스트명 |
| `CLAUDE_CLIENT_PRESENCE_FILE` | 존재할 때 활성 클라이언트를 신호하고 Remote Control의 모바일 푸시 알림을 억제하는 파일 경로. 데스크톱 클라이언트가 항상 실행 중이고 모바일 핑을 원하지 않는 환경에서 유용합니다 |
| `CLAUDE_CODE_DISABLE_NOTIFICATION_PRESENCE_CHECK` | 사용자가 세션에서 적극적으로 입력하는 중에도 푸시 알림을 보내려면 `1`로 설정합니다. 기본적으로 활성 사용자 존재가 감지되면 알림이 억제됩니다(v2.1.193) |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | 텔레메트리 활성화/비활성화(`0` 또는 `1`) |
| `DISABLE_ERROR_REPORTING` | 오류 보고 비활성화(비활성화하려면 `1`) |
| `DISABLE_AUTOUPDATER` | npm 레지스트리에 대한 자동 업데이트 확인을 비활성화하려면 `1`로 설정합니다. 시작 시 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables)를 참조하세요 |
| `DISABLE_UPDATES` | 모든 업데이트 경로 — 자동 확인, 알림, 수동 `claude update` — 를 완전히 차단하려면 `1`로 설정합니다. 백그라운드 확인만 비활성화하는 `DISABLE_AUTOUPDATER`보다 엄격합니다. 명시적으로 다시 활성화될 때까지 모든 업데이트를 차단해야 하는 환경에서 사용하세요 *(v2.1.118 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE` | 새 버전을 사용할 수 있을 때 Claude Code가 백그라운드에서 패키지 관리자의 업그레이드 명령을 실행하도록 하려면 `1`로 설정합니다. Homebrew 및 WinGet 설치에 적용됩니다. 다른 패키지 관리자는 계속해서 업그레이드 명령을 실행하지 않고 표시합니다. [Auto updates](https://code.claude.com/docs/en/setup#auto-updates)를 참조하세요(v2.1.129) |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` | `ANTHROPIC_BASE_URL`이 LiteLLM, Kong, 또는 내부 프록시 같은 Anthropic 호환 게이트웨이를 가리킬 때 게이트웨이의 `/v1/models` 엔드포인트에서 `/model` 선택기를 채우려면 `1`로 설정합니다. 공유 API 키로 뒷받침되는 게이트웨이는 그렇지 않으면 키가 접근할 수 있는 모든 모델을 노출하므로 기본적으로 꺼져 있습니다. 발견된 모델은 여전히 `availableModels` 허용 목록으로 필터링됩니다(v2.1.129, 이전 자동 발견에서 옵트인으로 변경) |
| `DISABLE_TELEMETRY` | 텔레메트리 비활성화(비활성화하려면 `1`) |
| `DO_NOT_TRACK` | 표준 옵트아웃 변수; 텔레메트리 수집을 옵트아웃하려면 `1`로 설정합니다. `DISABLE_TELEMETRY`에 의해 존중됩니다 |
| `MCP_TIMEOUT` | MCP 시작 타임아웃(ms) |
| `CLAUDE_CODE_MCP_ALLOWLIST_ENV` | 상속된 대부분의 환경 변수를 제거하여 신뢰할 수 없는 서버 프로세스로 자격 증명이 누출되는 것을 방지하고, stdio MCP 서버를 안전한 기준 환경으로만 생성합니다 |
| `MAX_MCP_OUTPUT_TOKENS` | 최대 MCP 출력 토큰(기본값: 25000). 출력이 10,000 토큰을 초과하면 경고가 표시됩니다 |
| `API_TIMEOUT_MS` | API 요청 타임아웃(ms)(기본값: 600000) |
| `API_FORCE_IDLE_TIMEOUT` | 스트리밍 연결에 대한 5분 유휴 타임아웃을 재정의합니다. 유휴 타임아웃을 완전히 비활성화하려면 `0`, 모든 연결에 강제하려면 `1`, 또는 기본값을 위해 설정하지 마세요(자주 멈추는 느리거나 불안정한 게이트웨이에서 자동 활성화됨). 느린 API 게이트웨이에 유용합니다(v2.1.169) |
| `CLAUDE_CODE_CONNECT_TIMEOUT_MS` | 스트리밍 API 요청의 연결, TLS, 응답 헤더 단계에 대한 타임아웃(밀리초)(기본값: `60000` / 60초). 이 창 내에 응답 헤더가 도착하지 않으면 요청이 중단되고 재시도됩니다. 비활성화하고 `API_TIMEOUT_MS`에만 의존하려면 `0`으로 설정하세요 |
| `CLAUDE_AFK_TIMEOUT_MS` | `askUserQuestionTimeout`이 지속 시간으로 설정된 경우, 응답되지 않은 AskUserQuestion 대화상자가 자동 진행되기까지의 밀리초. v2.1.200 기준으로 기본값은 `askUserQuestionTimeout`이 제어하는 `"never"`(자동 진행 안 함)입니다. 이 환경 변수는 타임아웃 지속 시간이 활성일 때만 적용됩니다. `0`으로 설정하면 대화상자가 즉시 닫힙니다. 질문을 열어두려면 `askUserQuestionTimeout: "never"`를 선호하세요(v2.1.198; 기본값 v2.1.200에서 변경) |
| `CLAUDE_AFK_COUNTDOWN_MS` | 응답되지 않은 AskUserQuestion 대화상자에 화면상 카운트다운이 나타나기까지 자동 진행 전 밀리초(기본값: `20000` / 20초)(v2.1.198) |
| `BASH_MAX_TIMEOUT_MS` | Bash 명령 타임아웃 |
| `BASH_MAX_OUTPUT_LENGTH` | 최대 bash 출력 길이 |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 자동 압축 임계값 비율(1-100). 기본값은 ~95%. 압축을 더 일찍 트리거하려면 더 낮게 설정하세요(예: `50`). 95%를 초과하는 값은 효과가 없습니다. 현재 사용량을 모니터링하려면 `/context`를 사용하세요. 예: `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50 claude` |
| `CLAUDE_CODE_MAX_CONTEXT_TOKENS` | Claude Code가 활성 모델에 대해 가정하는 컨텍스트 창 크기를 재정의합니다. `DISABLE_COMPACT`도 설정된 경우에만 적용됩니다. 이름의 컨텍스트 창이 내장 크기와 일치하지 않는 모델로 `ANTHROPIC_BASE_URL`을 통해 라우팅할 때 사용하세요 |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | bash 호출 간 cwd 유지(활성화하려면 `1`) |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | 백그라운드 작업 비활성화(비활성화하려면 `1`) |
| `CLAUDE_CODE_DISABLE_BG_SHELL_PRESSURE_REAP` | 유휴 백그라운드 셸 명령의 자동 메모리 압박 회수를 비활성화하려면 `1`로 설정합니다. 설정하지 않으면 Claude Code가 리소스를 확보하기 위해 메모리 압박 하에서 유휴 백그라운드 셸을 자동으로 회수합니다(v2.1.193 changelog, 공식 env-vars 페이지에는 아직 없음) |
| `CLAUDE_CODE_DISABLE_BG_EXIT_HANDOFF` | 슈퍼바이저가 세션 프로세스를 중지, 재시작, 또는 업데이트할 때 백그라운드 세션의 실행 중인 백그라운드 셸 명령, 동적 워크플로, 백그라운드 서브에이전트를 중지하려면 `1`로 설정합니다. 기본적으로 이들은 세션의 다음 프로세스로 인계됩니다(v2.1.198) |
| `CLAUDE_CODE_DISABLE_ADVISOR_TOOL` | advisor 도구와 `/advisor` 명령을 비활성화하려면 `1`로 설정합니다. advisor 사용을 생략하는 것의 환경 변수 등가물입니다. advisor 구성을 위해 `advisorModel`과 함께 사용하세요(최소 v2.1.98) |
| `CLAUDE_CODE_DISABLE_AGENT_VIEW` | 백그라운드 에이전트와 agent view(`claude agents`, `--bg`, `/background`, 온디맨드 슈퍼바이저)를 끄려면 `1`로 설정합니다. `disableAgentView` 설정의 환경 변수 등가물입니다 *(공식 설정 페이지에서 참조됨; env-vars 페이지에는 나열되지 않음)* |
| `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS` | 내장 Explore 및 Plan 서브에이전트를 비활성화하려면 `1`로 설정합니다. Claude는 대신 검색 도구나 general-purpose 서브에이전트로 탐색합니다. Plan 모드는 Explore 및 Plan 에이전트를 시작하는 대신 파일을 직접 읽습니다(v2.1.198) |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | 실험적 agent teams 기능 활성화(활성화하려면 `1`). 세션 내에서 협조하는 서브에이전트 팀을 생성할 수 있게 합니다. 시작 시 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables)를 참조하세요 |
| `CLAUDE_CODE_DISABLE_WORKFLOWS` | [dynamic workflows](https://code.claude.com/docs/en/workflows)(`/workflows`)와 번들된 워크플로 슬래시 명령을 비활성화하려면 `1`로 설정합니다. `disableWorkflows` 설정의 환경 변수 등가물입니다 |
| `CLAUDE_CODE_ENABLE_AUTO_MODE` | **v2.1.207 기준으로 auto mode는 Bedrock, Vertex AI, Foundry에서 기본적으로 사용 가능하며 — 이 옵트인은 더 이상 필요하지 않습니다.** 이전(v2.1.158–v2.1.206)에는: 해당 공급자에서 [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode)를 사용 가능하게 하려면 `1`로 설정했습니다. auto mode를 끄려면 설정에서 `disableAutoMode: "disable"`을 사용하세요 |
| `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` | Claude Code의 내장 기능(번들 스킬)을 모델로부터 숨기려면 `1`로 설정합니다. `disableBundledSkills` 설정의 환경 변수 등가물입니다(v2.1.169) |
| `CLAUDE_CODE_DISABLE_ARTIFACT` | 세션 출력을 claude.ai의 비공개 웹 페이지로 게시하는 [Artifact](https://code.claude.com/docs/en/artifacts) 도구를 비활성화하려면 `1`로 설정합니다. `disableArtifact` 설정과 동등합니다 |
| `CLAUDE_CODE_ARTIFACT_AUTO_OPEN` | 새 아티팩트가 게시될 때 Claude Code가 브라우저를 자동으로 여는 것을 멈추려면 `0`으로 설정합니다. 기존 아티팩트의 재게시는 이 설정과 무관하게 브라우저를 열지 않습니다 |
| `ENABLE_TOOL_SEARCH` | MCP 도구 검색 임계값(예: `auto:5`) |
| `ENABLE_PROMPT_CACHING_1H` | 1시간 프롬프트 캐시 TTL 옵트인. 더 이상 사용되지 않는 `ENABLE_PROMPT_CACHING_1H_BEDROCK`을 대체합니다 *(v2.1.108 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `FORCE_PROMPT_CACHING_5M` | 5분 프롬프트 캐시 TTL 강제 *(v2.1.108 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` | away summary / 유휴 세션 요약 옵트아웃. 비활성화하려면 `0`으로 설정합니다. `awaySummaryEnabled` 설정과 짝을 이룹니다(v2.1.110) |
| `DISABLE_PROMPT_CACHING` | 모든 프롬프트 캐싱 비활성화(비활성화하려면 `1`) |
| `DISABLE_PROMPT_CACHING_HAIKU` | Haiku 프롬프트 캐싱 비활성화 |
| `DISABLE_PROMPT_CACHING_SONNET` | Sonnet 프롬프트 캐싱 비활성화 |
| `DISABLE_PROMPT_CACHING_OPUS` | Opus 프롬프트 캐싱 비활성화 |
| `ENABLE_PROMPT_CACHING_1H_BEDROCK` | Bedrock에서 1시간 캐시 TTL 요청(활성화하려면 `1`) *(공식 문서에 없음 — 미검증; v2.1.108 changelog는 더 이상 사용되지 않으며 `ENABLE_PROMPT_CACHING_1H`로 대체되었다고 명시)* |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | 실험적 베타 기능 비활성화(비활성화하려면 `1`) |
| `CLAUDE_CODE_SHELL` | 자동 셸 감지 재정의 |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | 기본 파일 읽기 토큰 한도 재정의 |
| `CLAUDE_CODE_GLOB_HIDDEN` | Claude가 Glob 도구를 호출할 때 결과에서 dotfile을 제외하려면 `false`로 설정합니다. 기본적으로 포함됩니다. `@` 파일 자동 완성, `ls`, Grep, Read에는 영향을 주지 않습니다 |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | Glob 도구가 `.gitignore` 패턴을 존중하게 하려면 `false`로 설정합니다. 기본적으로 Glob은 gitignore된 파일을 포함하여 매칭되는 모든 파일을 반환합니다. 자체 `respectGitignore` 설정을 가진 `@` 파일 자동 완성에는 영향을 주지 않습니다 |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Glob 파일 발견의 타임아웃(초) |
| `CLAUDE_CODE_ENABLE_TASKS` | 세션이 구조화된 Task 도구(`TaskCreate`, `TaskUpdate`, `TaskGet`, `TaskList`)를 사용할지 레거시 `TodoWrite` 도구를 사용할지 제어합니다. v2.1.142 기준으로 Task 도구가 모든 모드에서 기본값입니다. `TodoWrite`로 되돌리려면 `0`으로 설정하세요 |
| `CLAUDE_CODE_SIMPLE` | 최소한의 시스템 프롬프트와 Bash, 파일 읽기, 파일 편집 도구만으로 실행하려면 `1`로 설정합니다. 시작 시 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables)를 참조하세요 |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | 유휴 지속 시간(ms) 후 SDK 모드 자동 종료 |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | 적응형 사고 비활성화(비활성화하려면 `1`) |
| `CLAUDE_CODE_DISABLE_THINKING` | 확장 사고 강제 비활성화(비활성화하려면 `1`) |
| `DISABLE_INTERLEAVED_THINKING` | interleaved-thinking 베타 헤더 전송 방지(비활성화하려면 `1`) |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | 1M 토큰 컨텍스트 창 비활성화(비활성화하려면 `1`) |
| `CLAUDE_CODE_ACCOUNT_UUID` | 인증용 계정 UUID 재정의 |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | git 관련 시스템 프롬프트 지침 비활성화 |
| `CLAUDE_CODE_ATTRIBUTION_HEADER` | 시스템 프롬프트에서 Claude Code 귀속 블록을 생략하려면 `0`으로 설정 |
| `CLAUDE_CODE_NEW_INIT` | `/init`이 대화형 설정 흐름을 실행하게 하려면 `true`로 설정합니다. 코드베이스를 탐색하기 전에 어떤 파일(CLAUDE.md, 스킬, hook)을 생성할지 묻습니다. 이것 없이는 `/init`이 CLAUDE.md를 자동으로 생성합니다 |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | 하나 이상의 읽기 전용 플러그인 시드 디렉터리 경로, Unix에서는 `:`, Windows에서는 `;`로 구분. 컨테이너 이미지에 미리 채워진 플러그인을 번들합니다. Claude Code는 시작 시 이 디렉터리에서 마켓플레이스를 등록하고 다시 클론하지 않고 미리 캐시된 플러그인을 사용합니다 |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | Claude.ai MCP 서버 활성화 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 노력 수준 설정: `low`, `medium`, `high`, `xhigh`(Opus 4.7 및 4.8, v2.1.111), `max`(Opus 4.6 전용), 또는 `auto`(모델 기본값 사용). `/effort`와 `effortLevel` 설정보다 우선합니다. 시작 시 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables)를 참조하세요 |
| `CLAUDE_EFFORT` | 읽기 전용. 셸 스크립트와 hook이 현재 등급에 적응할 수 있도록 활성 노력 수준과 함께 Bash 도구 하위 프로세스와 hook 핸들러에 주입됩니다(`CLAUDE_CODE_EFFORT_LEVEL`의 동반; v2.1.133). 스킬 파일 내부에서는 `${CLAUDE_EFFORT}`를 사용하세요 *(changelog에 있음, 공식 env-vars 페이지에는 없음 — 읽기 전용, 사용자 구성 불가)* |
| `CLAUDE_CODE_ALWAYS_ENABLE_EFFORT` | 일반적으로 노력 수준 선택을 지원하지 않는 모델을 포함한 모든 모델에서 노력 매개변수를 강제 활성화하려면 `1`로 설정합니다. `/effort`와 `effortLevel` 설정이 표준 노력 지원 세트 밖의 모델에서 적용될 수 있게 합니다(v2.1.154) |
| `CLAUDE_CODE_MAX_TURNS` | 중지 전 최대 에이전트 턴 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | `DISABLE_AUTOUPDATER`, `DISABLE_FEEDBACK_COMMAND`, `DISABLE_ERROR_REPORTING`, `DISABLE_TELEMETRY`를 설정하는 것과 동등 |
| `CLAUDE_CODE_SKIP_SETTINGS_SETUP` | 첫 실행 설정 설정 흐름 건너뛰기 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_PROMPT_CACHING_ENABLED` | 프롬프트 캐싱 동작 재정의 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_DISABLE_TOOLS` | 비활성화할 도구의 쉼표로 구분된 목록 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_DISABLE_MCP` | 모든 MCP 서버 비활성화(비활성화하려면 `1`) *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | 응답당 최대 출력 토큰. 기본값: 32,000(v2.1.77 기준 Opus 4.6은 64,000). 상한: 64,000(v2.1.77 기준 Opus 4.6 및 Sonnet 4.6은 128,000) |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | fast mode 완전 비활성화(비활성화하려면 `1`) |
| `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` | **v2.1.160에서 제거됨** — 이제 환경 변수는 no-op입니다. fast mode는 이 변수와 무관하게 기본 모델에서 실행됩니다. 이전에는 [fast mode](https://code.claude.com/docs/en/fast-mode)를 기본값 대신 Claude Opus 4.6에 고정했습니다(v2.1.142–v2.1.159) |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | 스트리밍 요청이 스트림 도중 실패할 때 비스트리밍 대체를 비활성화하려면 `1`로 설정합니다. 스트리밍 오류가 대체 대신 재시도 계층으로 전파됩니다. 프록시나 게이트웨이가 대체로 인해 중복 도구 실행을 유발할 때 유용합니다(v2.1.83) |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | 멈춘 스트림을 중단하는 스트림 유휴 워치독. v2.1.163 기준으로 기본적으로 활성화됨; 비활성화하려면 `0`으로 설정 |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | Anthropic API에서 기본적으로 활성화됨(v2.1.139+); 옵트아웃하려면 `0`으로 설정 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | 자동 메모리 비활성화(비활성화하려면 `1`) |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | `/rewind`용 파일 체크포인팅 비활성화(비활성화하려면 `1`) |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | 첨부 파일 처리 비활성화(비활성화하려면 `1`) |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | CLAUDE.md 파일 로드 방지(비활성화하려면 `1`) |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | 시작 시 `--add-dir`로 지정된 추가 디렉터리에서 CLAUDE.md 메모리 파일 로드(활성화하려면 `1`). 시작 시 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables)를 참조하세요 |
| `CLAUDE_CODE_DISABLE_POLICY_SKILLS` | 시스템 전역 관리 스킬 디렉터리에서 스킬 로드 건너뛰기(비활성화하려면 `1`) |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` | 이전 세션이 턴 도중 종료된 경우 자동 재개(활성화하려면 `1`) |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | 프롬프트 기록과 세션 기록을 디스크에 쓰지 않으려면 `1`로 설정합니다. 이 변수가 설정된 상태로 시작된 세션은 `--resume`, `--continue`, 또는 위쪽 화살표 기록에 나타나지 않습니다. 임시 스크립트 세션에 유용합니다 |
| `CLAUDE_CODE_USER_EMAIL` | 인증을 위해 사용자 이메일을 동기적으로 제공 |
| `CLAUDE_CODE_ORGANIZATION_UUID` | 인증을 위해 조직 UUID를 동기적으로 제공 |
| `CLAUDE_CONFIG_DIR` | 커스텀 구성 디렉터리(기본 `~/.claude`를 재정의) |
| `CLAUDE_CODE_TMPDIR` | 내부 임시 파일에 사용되는 임시 디렉터리 재정의. Claude Code는 이 경로에 `/claude/`를 추가합니다. 기본값: Unix/macOS에서 `/tmp`, Windows에서 `os.tmpdir()` |
| `ANTHROPIC_CUSTOM_HEADERS` | API 요청용 커스텀 헤더(`Name: Value` 형식, 여러 헤더는 줄바꿈으로 구분) |
| `CLAUDE_CODE_EXTRA_BODY` | 모든 API 요청 본문의 최상위에 병합할 JSON 객체. 공급자별 필드(예: 커스텀 게이트웨이용 라우팅 힌트)를 주입하는 데 사용합니다 |
| `CLAUDE_CODE_PROPAGATE_TRACEPARENT` | 커스텀 프록시를 통해 라우팅할 때 요청을 통해 W3C `traceparent` 헤더를 전파하여 Claude Code 트레이스를 상위 텔레메트리에 연결하려면 `1`로 설정합니다 |
| `ANTHROPIC_FOUNDRY_API_KEY` | Microsoft Foundry 인증용 API 키 |
| `ANTHROPIC_FOUNDRY_BASE_URL` | Foundry 리소스용 기본 URL |
| `ANTHROPIC_FOUNDRY_RESOURCE` | Foundry 리소스 이름 |
| `AWS_BEARER_TOKEN_BEDROCK` | 인증용 Bedrock API 키 |
| `ANTHROPIC_SMALL_FAST_MODEL` | **DEPRECATED** — 대신 `ANTHROPIC_DEFAULT_HAIKU_MODEL`을 사용하세요 |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | 더 이상 사용되지 않는 Haiku급 모델 재정의를 위한 AWS 리전 |
| `CLAUDE_CODE_SHELL_PREFIX` | bash 명령 앞에 추가되는 명령 접두사 |
| `BASH_DEFAULT_TIMEOUT_MS` | 기본 bash 명령 타임아웃(ms) |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | Bedrock에 대한 AWS 인증 건너뛰기(건너뛰려면 `1`) |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | Foundry에 대한 Azure 인증 건너뛰기(건너뛰려면 `1`) |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | Bedrock Mantle에 대한 AWS 인증 건너뛰기(예: LLM 게이트웨이 사용 시) |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | Vertex에 대한 Google 인증 건너뛰기(건너뛰려면 `1`) |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | 프록시가 DNS 해석을 수행하도록 허용 |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | `apiKeyHelper`의 자격 증명 갱신 간격(ms) |
| `CLAUDE_CODE_CLIENT_CERT` | mTLS용 클라이언트 인증서 경로 |
| `CLAUDE_CODE_CLIENT_KEY` | mTLS용 클라이언트 개인 키 경로 |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | 암호화된 mTLS 키의 암호구 |
| `CLAUDE_CODE_CERT_STORE` | TLS 연결용 CA 인증서 소스의 쉼표로 구분된 목록: `bundled`(Claude Code와 함께 배포되는 Mozilla CA 세트) 및/또는 `system`(OS 신뢰 저장소). 기본값: `bundled,system`. 시스템 저장소 통합에는 네이티브 바이너리 배포판이 필요하며, Node.js 런타임에서는 이 값과 무관하게 번들 세트만 사용됩니다(v2.1.101) |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | 플러그인 마켓플레이스 git 클론 타임아웃(ms)(기본값: 120000) |
| `CLAUDE_CODE_PLUGIN_PREFER_HTTPS` | GitHub `owner/repo` 축약 플러그인 소스를 SSH 대신 HTTPS로 클론하려면 `1`로 설정합니다. 플러그인 설치/업데이트와 `/plugin marketplace add`/`update`에 적용됩니다. `github.com`용 SSH 키가 구성되지 않은 CI 러너나 컨테이너에 유용합니다(v2.1.141) |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | 플러그인 루트 디렉터리 재정의 |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | 공식 마켓플레이스 자동 추가 건너뛰기(비활성화하려면 `1`) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` | 첫 쿼리 전에 플러그인 설치 완료 대기(활성화하려면 `1`) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL_TIMEOUT_MS` | 동기 플러그인 설치 타임아웃(ms) |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | `git pull`이 실패할 때 지우고 다시 클론하는 대신 기존 마켓플레이스 캐시를 유지하려면 `1`로 설정합니다. 다시 클론하면 동일하게 실패할 오프라인 또는 에어갭 환경에서 유용합니다 |
| `CLAUDE_CODE_ENABLE_BACKGROUND_PLUGIN_REFRESH` | 백그라운드 설치 완료 후 세션 턴 경계에서 플러그인 상태 새로 고침(활성화하려면 `1`). 이것 없이는 새로 설치된 플러그인이 다음 세션에 적용됩니다 |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | UI에서 이메일/조직 정보 숨기기 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_DISABLE_CRON` | 예약/cron 작업 비활성화(비활성화하려면 `1`) |
| `DISABLE_INSTALLATION_CHECKS` | 설치 경고 비활성화 |
| `DISABLE_FEEDBACK_COMMAND` | `/feedback` 명령 비활성화. 이전 이름 `DISABLE_BUG_COMMAND`도 허용됩니다 |
| `DISABLE_DOCTOR_COMMAND` | `/doctor` 명령 숨기기(비활성화하려면 `1`) |
| `DISABLE_LOGIN_COMMAND` | `/login` 명령 숨기기(비활성화하려면 `1`) |
| `DISABLE_LOGOUT_COMMAND` | `/logout` 명령 숨기기(비활성화하려면 `1`) |
| `DISABLE_UPGRADE_COMMAND` | `/upgrade` 명령 숨기기(비활성화하려면 `1`) |
| `DISABLE_EXTRA_USAGE_COMMAND` | `/extra-usage` 명령 숨기기 — v2.1.144에서 `/usage-credits`로 이름 변경되었으나 이 환경 변수 이름은 변경되지 않음(비활성화하려면 `1`) |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | `/install-github-app` 명령 숨기기(비활성화하려면 `1`) |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | 플레이버 텍스트와 비필수 모델 호출 비활성화 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | 디버그 로그 파일 경로 재정의. 이름과 달리 이것은 디렉터리가 아니라 파일 경로입니다. `--debug`, `/debug`, 또는 `DEBUG` 환경 변수를 통해 별도로 디버그 모드가 활성화되어야 하며, 이 변수만 설정해도 로깅이 활성화되지 않습니다. 디버그 모드를 활성화하고 경로를 한 번에 설정하려면 `--debug-file`을 사용하세요. 기본값: `~/.claude/debug/<session-id>.txt` |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | 최소 디버그 로그 수준 |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | 긴 작업의 자동 백그라운드화 강제(활성화하려면 `1`) |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | Opus 4.0/4.1을 최신 모델로 재매핑하는 것 방지(비활성화하려면 `1`) |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | 기본값만이 아니라 모든 기본 모델에 대해 대체 모델 트리거(활성화하려면 `1`) |
| `CCR_FORCE_BUNDLE` | GitHub 접근이 가능하더라도 `claude --remote`가 로컬 저장소를 번들하고 업로드하도록 강제하려면 `1`로 설정합니다. 시작 시 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables)를 참조하세요 |
| `CLAUDE_CODE_GIT_BASH_PATH` | Windows 전용: Git Bash 실행 파일(`bash.exe`) 경로. Git Bash가 설치되었지만 PATH에 없을 때 사용하세요 |
| `DISABLE_COST_WARNINGS` | 비용 경고 메시지 비활성화 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 서브에이전트용 모델 재정의(예: `haiku`, `sonnet`) |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | 하위 프로세스 환경(Bash 도구, hook, MCP stdio 서버)에서 Anthropic 및 클라우드 공급자 자격 증명을 제거하려면 `1`로 설정합니다. 하위 프로세스가 API 키를 상속하지 않아야 하는 심층 방어에 사용하세요(v2.1.83) |
| `CLAUDE_CODE_SCRIPT_CAPS` | `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB`이 설정되었을 때 특정 스크립트가 세션당 호출될 수 있는 횟수를 제한하는 JSON 객체. 키는 명령 텍스트에 대해 매칭되는 부분 문자열이고, 값은 정수 호출 한도입니다. 예를 들어 `{"deploy.sh": 2}`는 `deploy.sh`가 최대 두 번 호출되도록 허용합니다. 매칭은 부분 문자열 기반이며, `xargs`나 `find -exec`를 통한 런타임 팬아웃은 감지되지 않습니다 — 이것은 심층 방어 제어입니다 |
| `CLAUDE_CODE_PERFORCE_MODE` | Perforce 인식 쓰기 보호를 활성화하려면 `1`로 설정합니다. 설정되면 대상 파일에 소유자 쓰기 비트가 없을 때 Edit, Write, NotebookEdit이 `p4 edit <file>` 힌트와 함께 실패하는데, Perforce는 `p4 edit`이 파일을 열 때까지 동기화된 파일에서 이를 지웁니다. Claude Code가 Perforce 변경 추적을 우회하는 것을 방지합니다(v2.1.98) |
| `CLAUDE_CODE_MAX_RETRIES` | API 요청 재시도 횟수 재정의(기본값: 10) |
| `CLAUDE_CODE_RETRY_WATCHDOG` | 비용량 API 오류에 대한 재시도 횟수를 300으로 상향합니다. 표준 재시도 상한(`CLAUDE_CODE_MAX_RETRIES`, 기본값: 10)은 용량 오류에 여전히 적용됩니다 *(v2.1.199 changelog에 있음, 공식 env-vars 페이지에는 없음)* |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | 최대 병렬 읽기 전용 도구(기본값: 10) |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | SDK 모드에서 내장 서브에이전트 유형 비활성화(비활성화하려면 `1`) |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | SDK 모드에서 MCP 도구에 대한 `mcp__<server>__` 접두사 건너뛰기(활성화하려면 `1`) |
| `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS` | 백그라운드 서브에이전트의 멈춤 타임아웃(ms)(기본값: 600000 / 10분). 이 지속 시간 동안 출력을 생성하지 않으면 서브에이전트가 종료됩니다 |
| `MCP_CONNECTION_NONBLOCKING` | `-p` 모드에서 MCP 연결 대기를 완전히 건너뛰려면 `true`로 설정합니다. 가장 느린 서버에서 차단하는 대신 `--mcp-config` 서버 연결을 5초로 제한합니다 *(v2.1.89 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd hook 타임아웃(ms)(하드 1.5초 한도를 대체) |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | 피드백 설문 프롬프트 비활성화(비활성화하려면 `1`) |
| `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL` | Anthropic 대상 비필수 트래픽이 차단될 때 세션 품질 설문을 자체 OpenTelemetry 컬렉터로 라우팅하려면 `1`로 설정합니다. 설문 평점은 구성된 컬렉터에 OTEL 이벤트로만 방출되며 — 설문 데이터는 Anthropic으로 전송되지 않습니다. `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`, `DISABLE_TELEMETRY`, 또는 `DO_NOT_TRACK`이 설정된 경우 적용되며, 그렇지 않으면 효과가 없습니다. `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY`와 조직 제품 피드백 정책이 우선합니다(v2.1.136) |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | 터미널 제목 업데이트 비활성화(비활성화하려면 `1`) |
| `CLAUDE_CODE_TMUX_TRUECOLOR` | tmux 내부에서 24비트 트루컬러 출력을 허용하려면 `1`로 설정합니다. 기본적으로 Claude Code는 `$TMUX`가 설정되면 256색으로 제한하는데, tmux는 구성되지 않는 한 트루컬러 이스케이프 시퀀스를 통과시키지 않기 때문입니다. `~/.tmux.conf`에 `set -ga terminal-overrides ',*:Tc'`를 추가한 후 이를 설정하세요 |
| `CLAUDE_CODE_NO_FLICKER` | 깜빡임 없는 대체 화면 렌더링을 활성화하려면 `1`로 설정합니다. 전체 화면 재그리기 중 시각적 깜빡임을 제거합니다(v2.1.88) |
| `CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT` | 전체 화면 렌더링에서 매 프레임 전체 화면을 다시 칠하려면 `1`로 설정합니다. 특이한 터미널 에뮬레이터에서 부분 재그리기가 시각적 아티팩트를 생성할 때 사용하세요 |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` | 전체 화면 렌더링을 비활성화하고 클래식 메인 화면 렌더러를 사용하려면 `1`로 설정합니다. 대화가 터미널의 네이티브 스크롤백에 유지되어 `Cmd+f`와 tmux 복사 모드가 평소대로 작동합니다. `CLAUDE_CODE_NO_FLICKER`와 `tui` 설정보다 우선합니다. `/tui default`로도 전환할 수 있습니다(v2.1.132) |
| `CLAUDE_CODE_FORCE_SYNC_OUTPUT` | 터미널이 지원하지만 자동 감지되지 않을 때 DEC 사설 모드 2026 동기화 출력을 강제 활성화하려면 `1`로 설정합니다. BSU/ESU를 구현하지만 기능 프로브에 응답하지 않는 Emacs `eat` 같은 에뮬레이터에 유용합니다. tmux 하에서는 효과가 없습니다(v2.1.129) |
| `CLAUDE_CODE_SCROLL_SPEED` | 전체 화면 렌더링용 마우스 휠 스크롤 배수. 더 빠른 스크롤을 위해 늘리고, 더 세밀한 제어를 위해 줄이세요 |
| `CLAUDE_CODE_DISABLE_VIRTUAL_SCROLL` | 전체 화면 렌더링에서 가상 스크롤을 비활성화하고 기록의 모든 메시지를 렌더링하려면 `1`로 설정합니다. 전체 화면 모드에서 스크롤 시 메시지가 나타나야 할 곳에 빈 영역이 표시되면 사용하세요 |
| `CLAUDE_CODE_DISABLE_MOUSE` | 전체 화면 렌더링에서 마우스 추적을 비활성화하려면 `1`로 설정합니다. 마우스 이벤트가 터미널 멀티플렉서나 접근성 도구를 방해할 때 유용합니다 |
| `CLAUDE_CODE_DISABLE_MOUSE_CLICKS` | 휠 스크롤은 유지하면서 전체 화면 렌더링에서 마우스 클릭 상호작용을 비활성화하려면 `1`로 설정합니다. 클릭 이벤트 억제를 전체 마우스 추적 비활성화(`CLAUDE_CODE_DISABLE_MOUSE`)와 구별합니다. 터미널 선택이나 외부 접근성 도구가 앱 내 마우스 클릭과 충돌할 때 유용합니다 *(v2.1.195 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_HIDE_CWD` | Claude Code 시작 로고 배너에서 현재 작업 디렉터리를 숨기려면 `1`로 설정합니다. CWD 경로가 호스트나 프로젝트 레이아웃에 대한 정보를 누출하는 화면 녹화, 데모, 또는 공유 세션에 유용합니다(v2.1.119) |
| `CLAUDE_CODE_ACCESSIBILITY` | 스크린 리더와 접근성 도구를 위해 네이티브 터미널 커서를 계속 보이게 하려면 `1`로 설정 |
| `CLAUDE_AX_SCREEN_READER` | 스크린 리더 친화적 출력을 렌더링하려면 `1`로 설정합니다: 장식용 테두리나 애니메이션 없는 평면 텍스트. `axScreenReader` 설정이 `true`여도 스크린 리더 모드를 강제로 끄려면 `0`으로 설정합니다. `--ax-screen-reader` CLI 플래그가 우선합니다(v2.1.181+) |
| `CLAUDE_CODE_NATIVE_CURSOR` | Claude Code의 커스텀 커서 문자 대신 입력 캐럿 위치에 터미널 자체 커서를 표시하려면 `1`로 설정 |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | diff 출력에서 구문 강조를 비활성화하려면 `0`으로 설정 |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | 자동 IDE 확장 설치 건너뛰기(건너뛰려면 `1`) |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | 자동 IDE 연결 동작 재정의 |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | 연결을 위한 IDE 호스트 주소 재정의 |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | IDE 잠금 파일 검증 건너뛰기(건너뛰려면 `1`) |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | OTel 헤더 헬퍼 스크립트의 디바운스 간격(ms) |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS` | OpenTelemetry 플러시 타임아웃(ms) |
| `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS` | OpenTelemetry 종료 타임아웃(ms) |
| `CLAUDE_ENABLE_BYTE_WATCHDOG` | 바이트 수준 스트리밍 유휴 워치독을 강제 활성화하려면 `1`, 강제 비활성화하려면 `0`으로 설정합니다. 설정하지 않으면 Anthropic API 연결에 대해 워치독이 기본적으로 활성화됩니다. 바이트 워치독은 `CLAUDE_STREAM_IDLE_TIMEOUT_MS`로 설정된 지속 시간(최소 5분) 동안 회선에 바이트가 도착하지 않으면 이벤트 수준 워치독과 독립적으로 연결을 중단합니다 |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | 스트리밍 유휴 워치독의 타임아웃(ms). 두 개의 워치독이 적용됩니다: **바이트 수준**(기본값 및 최소 `300000` / 5분, 회선에 바이트가 도착하지 않으면 중단)과 **이벤트 수준**(기본값 `90000` / 90초, 최소 없음, SSE 이벤트가 도착하지 않으면 중단). 바이트 워치독은 Anthropic API 연결에 대해 기본적으로 활성화됩니다. `CLAUDE_ENABLE_BYTE_WATCHDOG`으로 제어하세요. 오래 실행되는 도구나 느린 네트워크가 조기 타임아웃 오류를 유발하면 이벤트 타임아웃을 늘리세요 |
| `OTEL_LOG_TOOL_DETAILS` | OpenTelemetry 이벤트에 `tool_parameters`를 포함하려면 `1`로 설정합니다. 개인정보 보호를 위해 기본적으로 생략됩니다 *(v2.1.85 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `OTEL_LOG_RAW_API_BODIES` | 전체 API 요청 및 응답 본문을 OpenTelemetry 로그 이벤트로 방출하려면 `1`로 설정합니다. 개인정보 보호와 페이로드 크기를 위해 기본적으로 생략됩니다. 게이트웨이나 프록시에서 디버깅에 유용합니다 *(v2.1.111 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `OTEL_RESOURCE_ATTRIBUTES` | Claude Code가 방출하는 모든 OpenTelemetry 메트릭 데이터 포인트에 리소스 속성으로 추가되는 쉼표로 구분된 `key=value` 쌍. 모든 메트릭에 나타나 컬렉터에서 필터링할 수 있는 환경 또는 배포 레이블(예: `environment=production,team=platform`)을 붙이는 데 사용합니다(v2.1.162) |
| `OTEL_LOG_USER_PROMPTS` | OpenTelemetry LLM 요청 스팬에 `user_system_prompt` 필드를 포함하려면 `1`로 설정합니다. 개인정보 보호를 위해 기본적으로 생략됩니다 — 사용자 프롬프트는 민감한 데이터를 포함할 수 있으므로, OTel 컬렉터를 제어하고 정책을 갖춘 경우에만 옵트인하세요 *(v2.1.121 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `OTEL_LOG_ASSISTANT_RESPONSES` | OpenTelemetry 로그 이벤트에 모델 응답 텍스트를 포함하려면 `1`로 설정합니다. 개인정보 보호와 페이로드 크기를 위해 기본적으로 생략됩니다. OTel 컬렉터를 제어하고 모델 출력 처리를 위한 정책을 갖춘 경우에만 사용하세요 *(v2.1.193 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | 메트릭 및 로그용 OpenTelemetry 컬렉터 엔드포인트 URL. [Monitoring](https://code.claude.com/docs/en/monitoring-usage)을 참조하세요 |
| `OTEL_EXPORTER_OTLP_HEADERS` | 컬렉터 인증을 위한 OpenTelemetry 익스포터 헤더(`Name=Value` 형식, 쉼표로 구분) |
| `OTEL_LOG_TOOL_CONTENT` | 전체 도구 입력 및 출력을 OpenTelemetry 로그 이벤트로 방출하려면 `1`로 설정합니다. 개인정보 보호를 위해 기본적으로 생략됩니다 |
| `OTEL_METRICS_EXPORTER` | OpenTelemetry 메트릭 익스포터 유형(예: `otlp`). [Monitoring](https://code.claude.com/docs/en/monitoring-usage)을 참조하세요 |
| `OTEL_TRACES_EXPORTER` | OpenTelemetry 트레이스 익스포터 유형(예: `otlp`). [Monitoring](https://code.claude.com/docs/en/monitoring-usage)을 참조하세요 |
| `OTEL_METRICS_INCLUDE_ENTRYPOINT` | 모든 OpenTelemetry 메트릭 데이터 포인트에 세션 진입점(예: 대화형 vs `-p` vs SDK)을 레이블로 포함하려면 `1`로 설정합니다. Claude Code가 어떻게 호출되었는지로 메트릭을 분류하는 데 유용합니다(v2.1.161 changelog) *(v2.1.161 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_FORK_SUBAGENT` | 외부 빌드(Anthropic 서명되지 않은 배포판)에서 포크된 서브에이전트를 활성화하려면 `1`로 설정합니다. 포크된 서브에이전트는 메인 에이전트의 컨텍스트를 공유하는 대신 격리된 자식 프로세스에서 실행됩니다 *(v2.1.117 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_MCP_SERVER_NAME` | MCP 서버의 이름, `headersHelper` 스크립트에 환경 변수로 전달되어 서버별 인증 헤더를 생성할 수 있게 합니다 *(v2.1.85 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_MCP_SERVER_URL` | MCP 서버의 URL, `CLAUDE_CODE_MCP_SERVER_NAME`과 함께 `headersHelper` 스크립트에 환경 변수로 전달됩니다 *(v2.1.85 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | Opus 모델 별칭 재정의(예: `claude-opus-4-6[1m]`) |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_NAME` | Bedrock/Vertex/Foundry에서 고정 모델을 사용할 때 `/model` 선택기의 Opus 항목 레이블을 커스터마이징합니다. 기본값은 모델 ID |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_DESCRIPTION` | `/model` 선택기의 Opus 항목 설명을 커스터마이징합니다. 기본값은 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_OPUS_MODEL_SUPPORTED_CAPABILITIES` | 고정 Opus 모델의 기능 감지를 재정의합니다. 쉼표로 구분된 값(예: `effort,thinking`). 고정 모델이 자동 감지가 확인할 수 없는 기능을 지원할 때 필요합니다 |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | Sonnet 모델 별칭 재정의(예: `claude-sonnet-4-6`) |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_NAME` | Bedrock/Vertex/Foundry에서 고정 모델을 사용할 때 `/model` 선택기의 Sonnet 항목 레이블을 커스터마이징합니다. 기본값은 모델 ID |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_DESCRIPTION` | `/model` 선택기의 Sonnet 항목 설명을 커스터마이징합니다. 기본값은 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL_SUPPORTED_CAPABILITIES` | 고정 Sonnet 모델의 기능 감지를 재정의합니다. 쉼표로 구분된 값(예: `effort,thinking`). 고정 모델이 자동 감지가 확인할 수 없는 기능을 지원할 때 필요합니다 |
| `ANTHROPIC_DEFAULT_FABLE_MODEL` | Fable 모델 별칭 재정의(예: `claude-fable-5`) |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_NAME` | Bedrock/Vertex/Foundry에서 고정 모델을 사용할 때 `/model` 선택기의 Fable 항목 레이블을 커스터마이징합니다. 기본값은 모델 ID |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_DESCRIPTION` | `/model` 선택기의 Fable 항목 설명을 커스터마이징합니다. 기본값은 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_FABLE_MODEL_SUPPORTED_CAPABILITIES` | 고정 Fable 모델의 기능 감지를 재정의합니다. 쉼표로 구분된 값(예: `effort,thinking`). 고정 모델이 자동 감지가 확인할 수 없는 기능을 지원할 때 필요합니다 |
| `MAX_THINKING_TOKENS` | 응답당 최대 확장 사고 토큰. Anthropic API에서 확장 사고를 완전히 비활성화하려면 `0`으로 설정합니다(`--thinking disabled`와 동등). 고정 사고 예산을 사용할 때만 적용됩니다 — 적응형 사고 모델(Opus 4.7+)에서는 노력 수준이 사고 깊이를 대신 제어합니다 |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | 자동 압축 계산에 사용되는 토큰 단위 컨텍스트 용량을 설정합니다. 기본값은 모델의 컨텍스트 창(표준 200K, 확장 컨텍스트 모델은 1M). 1M 모델에서 압축을 위해 500K로 취급하려면 더 낮은 값(예: `500000`)을 사용하세요. 실제 컨텍스트 창으로 제한됩니다. `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`는 이 값의 백분율로 적용됩니다. 이를 설정하면 압축 임계값이 상태 표시줄의 `used_percentage`에서 분리됩니다 |
| `DISABLE_AUTO_COMPACT` | 자동 컨텍스트 압축 비활성화(비활성화하려면 `1`). 수동 `/compact`는 여전히 작동합니다 *(공식 문서에 없음 — 미검증)* |
| `DISABLE_COMPACT` | 모든 압축 비활성화 — 자동 및 수동 둘 다(비활성화하려면 `1`) |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | 프롬프트 제안 활성화 |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED` | 세션에 plan 모드 요구 |
| `CLAUDE_CODE_TEAM_NAME` | agent teams용 팀 이름 |
| `CLAUDE_CODE_TASK_LIST_ID` | 작업 통합용 작업 목록 ID |
| `CLAUDE_ENV_FILE` | 커스텀 환경 파일 경로 |
| `FORCE_AUTOUPDATE_PLUGINS` | 플러그인 자동 업데이트 강제(활성화하려면 `1`) |
| `HTTP_PROXY` | 네트워크 요청용 HTTP 프록시 URL |
| `HTTPS_PROXY` | 네트워크 요청용 HTTPS 프록시 URL |
| `NO_PROXY` | 프록시를 우회하는 호스트의 쉼표로 구분된 목록 |
| `MCP_TOOL_TIMEOUT` | MCP 도구 실행 타임아웃(ms) |
| `MCP_CLIENT_SECRET` | MCP OAuth 클라이언트 시크릿 |
| `MCP_OAUTH_CALLBACK_PORT` | MCP OAuth 콜백 포트 |
| `IS_DEMO` | 데모 모드 활성화 |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | 슬래시 명령 도구 출력의 문자 예산 |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | Claude 3.5 Haiku용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | Claude 3.7 Sonnet용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | Claude 4.0 Opus용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | Claude 4.0 Sonnet용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | Claude 4.1 Opus용 Vertex AI 리전 재정의 |

---

## Useful Commands

| Command | Description |
|---------|-------------|
| `/model` | 모델 전환 및 노력 수준 조정(Opus 4.7 및 4.8) |
| `/effort` | 노력 수준 직접 설정: `low`, `medium`, `high`, `xhigh`(Opus 4.7 및 4.8, v2.1.111), 또는 `max`(Opus 4.6 전용)(v2.1.76+) |
| `/config` | 대화형 구성 UI; 프롬프트 기반 설정을 위해 `key=value` 구문도 허용: `/config model=sonnet`(v2.1.181) |
| `/memory` | 모든 메모리 파일 보기/편집 |
| `/agents` | 서브에이전트 관리 |
| `/mcp` | MCP 서버 관리 |
| `/hooks` | 구성된 hook 보기 |
| `/plugin` | 플러그인 관리 |
| `claude plugin tag` | 배포를 위해 마켓플레이스의 플러그인 버전에 태그를 지정합니다. 플러그인 이름과 버전으로 마켓플레이스 저장소에서 실행하세요(v2.1.118) |
| `claude plugin prune` | 마켓플레이스 소스가 더 이상 존재하지 않는 플러그인을 제거합니다(예: 마켓플레이스 삭제 또는 `extraKnownMarketplaces` 항목 제거). 로컬 캐시를 정리하고 고아 플러그인을 비활성화합니다(v2.1.121) |
| `claude plugin details <plugin>` | 플러그인의 구성 요소 인벤토리(명령, 에이전트, 스킬, hook)와 세션당 추가하는 컨텍스트 토큰 비용을 표시합니다. 관리 환경에서 플러그인을 활성화하기 전에 토큰 지출을 감사하는 데 유용합니다(v2.1.139) |
| `/keybindings` | 커스텀 키보드 단축키 구성 |
| `/skills` | 스킬 보기 및 관리 |
| `/permissions` | 권한 규칙 보기 및 관리 |
| `/usage-credits` | 남은 사용 크레딧과 한도 보기. v2.1.144에서 `/extra-usage`에서 이름 변경(이전 이름도 여전히 작동) |
| `claude gateway` | 조직 관리 배포를 위한 Claude Gateway 연결 관리. 관리 설정에서 `forceLoginMethod: "gateway"` 필요(v2.1.195) |
| `--doctor` | 구성 문제 진단 |
| `--debug` | hook 실행 세부 정보가 있는 디버그 모드 |

---

## Quick Reference: Complete Example

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "sonnet",
  "advisorModel": "fable",
  "agent": "code-reviewer",
  "language": "english",
  "cleanupPeriodDays": 30,
  "autoUpdatesChannel": "stable",
  "alwaysThinkingEnabled": true,
  "showThinkingSummaries": true,
  "viewMode": "default",
  "tui": "fullscreen",
  "awaySummaryEnabled": false,
  "includeGitInstructions": true,
  "defaultShell": "bash",
  "plansDirectory": "./plans",
  "claudeMdExcludes": ["**/vendor/**/CLAUDE.md"],
  "effortLevel": "high",
  "maxSkillDescriptionChars": 1536,
  "skillListingBudgetFraction": 0.01,
  "disableAgentView": false,
  "disableWorkflows": false,
  "workflowKeywordTriggerEnabled": true,
  "syntaxHighlightingDisabled": false,

  "worktree": {
    "symlinkDirectories": ["node_modules"],
    "sparsePaths": ["packages/my-app", "shared/utils"],
    "baseRef": "fresh",
    "bgIsolation": "worktree"
  },

  "skillOverrides": {
    "legacy-context": "name-only",
    "deploy": "off"
  },

  "modelOverrides": {
    "claude-opus-4-6": "arn:aws:bedrock:us-east-1:123456789:inference-profile/anthropic.claude-opus-4-6-v1:0"
  },

  "autoMode": {
    "classifyAllShell": false,
    "environment": [
      "Source control: github.example.com/acme-corp and all repos under it",
      "Trusted internal domains: *.internal.example.com"
    ],
    "soft_deny": ["$defaults", "Never run terraform apply"],
    "hard_deny": ["Never run rm -rf on directories outside the project"]
  },

  "permissions": {
    "allow": [
      "Edit(*)",
      "Write(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__*",
      "Agent(*)"
    ],
    "deny": [
      "Read(.env)",
      "Read(./secrets/**)"
    ],
    "additionalDirectories": ["../shared/"],
    "defaultMode": "acceptEdits"
  },

  "enableAllProjectMcpServers": true,

  "mcpServers": {
    "always-on-server": {
      "type": "http",
      "url": "https://mcp.example.com",
      "alwaysLoad": true
    }
  },

  "sshConfigs": [
    {
      "id": "dev-vm",
      "name": "Dev VM",
      "sshHost": "user@dev.example.com"
    }
  ],

  "sandbox": {
    "enabled": true,
    "excludedCommands": ["git", "docker"],
    "filesystem": {
      "denyRead": ["./secrets/"],
      "denyWrite": ["./.env"]
    }
  },

  "attribution": {
    "commit": "Generated with Claude Code",
    "pr": ""
  },
  "prUrlTemplate": "https://gitlab.example.com/{owner}/{repo}/-/merge_requests/{number}",

  "statusLine": {
    "type": "command",
    "command": "git branch --show-current"
  },

  "spinnerTipsEnabled": true,
  "spinnerTipsOverride": {
    "tips": ["Custom tip 1", "Custom tip 2"],
    "excludeDefault": false
  },
  "prefersReducedMotion": false,
  "preferredNotifChannel": "terminal_bell",

  "env": {
    "NODE_ENV": "development",
    "CLAUDE_CODE_EFFORT_LEVEL": "medium",
    "ANTHROPIC_BEDROCK_SERVICE_TIER": "priority"
  }
}
```

---

## Sources

- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings)
- [Claude Code Settings JSON Schema](https://json.schemastore.org/claude-code-settings.json)
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code GitHub Settings Examples](https://github.com/feiskyer/claude-code-settings)
- [Claude Code Environment Variables Reference](https://code.claude.com/docs/en/env-vars)
- [Claude Code Permissions Reference](https://code.claude.com/docs/en/permissions)
