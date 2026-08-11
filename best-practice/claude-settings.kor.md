<!--
  이 문서는 best-practice/claude-settings.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Settings Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Aug%2007%2C%202026%2010%3A48%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.224-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../.claude/settings.json)

Claude Code의 `settings.json` 파일에서 사용할 수 있는 모든 설정 옵션에 대한 종합 가이드입니다. v2.1.224 기준으로 Claude Code는 **127개 이상의 설정**과 **311개의 환경 변수**를 노출합니다(래퍼 스크립트를 피하려면 `settings.json`의 `"env"` 필드를 사용하세요).

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

설정은 우선순위 순서(높은 것에서 낮은 것)로 적용됩니다:

| Priority | Location | Scope | Shared? | Purpose |
|----------|----------|-------|---------|---------|
| 1 | Managed settings | Organization | Yes (deployed by IT) | 재정의할 수 없는 보안 정책 |
| 2 | Command line arguments | Session | N/A | 단일 세션 임시 재정의 |
| 3 | `.claude/settings.local.json` | Project | No (git-ignored) | 개인용 프로젝트별 설정 |
| 4 | `.claude/settings.json` | Project | Yes (committed) | 팀 공유 설정 |
| 5 | `~/.claude/settings.json` | User | N/A | 전역 개인 기본값 |

**Managed settings**는 조직에서 강제하며, 명령줄 인수를 포함한 어떤 하위 레벨로도 재정의할 수 없습니다. 전달 방식:
- **Server-managed** 설정(원격 전달)
- **MDM 프로파일** — macOS의 `com.anthropic.claudecode` plist
- **레지스트리 정책** — Windows `HKLM\SOFTWARE\Policies\ClaudeCode`(관리자) 및 `HKCU\SOFTWARE\Policies\ClaudeCode`(사용자 레벨, 가장 낮은 정책 우선순위)
- **파일** — `managed-settings.json` 및 `managed-mcp.json`(macOS: `/Library/Application Support/ClaudeCode/`, Linux/WSL: `/etc/claude-code/`, Windows: `C:\Program Files\ClaudeCode\`)
- **Drop-in 디렉터리** — 독립적인 정책 조각을 위한 `managed-settings.json` 옆의 `managed-settings.d/`(v2.1.83). systemd 관례를 따라 `managed-settings.json`이 먼저 베이스로 병합되고, 그다음 drop-in 디렉터리의 모든 `*.json` 파일이 알파벳순으로 정렬되어 위에 병합됩니다. 스칼라 값은 나중 파일이 앞선 파일을 재정의하며, 배열은 연결되고 중복 제거되며, 객체는 깊은 병합(deep-merge)됩니다. `.`으로 시작하는 숨김 파일은 무시됩니다. 병합 순서를 제어하려면 숫자 접두사를 사용하세요(예: `10-telemetry.json`, `20-security.json`)

managed 계층 내에서 우선순위는: server-managed > MDM/OS 레벨 정책 > 파일 기반(`managed-settings.d/*.json` + `managed-settings.json`) > HKCU 레지스트리(Windows 전용) 순입니다. 대부분의 키에 대해 하나의 managed 소스만 우선하며, 소스는 계층을 넘어 병합되지 않습니다. 파일 기반 계층 내에서는 drop-in 파일과 베이스 파일이 함께 병합됩니다.

**예외 — admin-source union 키:** 다음 키들은 *어떤* admin 제어 managed 소스가 설정하든 존중되며, 우선하는 소스에만 국한되지 않습니다: `sandbox.network.allowManagedDomainsOnly`, `sandbox.network.allowedDomains`(`allowManagedDomainsOnly`가 설정된 경우), `sandbox.filesystem.allowManagedReadPathsOnly`, `sandbox.filesystem.allowRead`(`allowManagedReadPathsOnly`가 설정된 경우), `allowAllClaudeAiMcps`, `sandbox.bwrapPath`, `sandbox.socatPath`, `forceRemoteSettingsRefresh`, 그리고 `env`(가장 높은 우선순위 소스의 전체 객체를 취하는 대신 모든 admin 소스에 걸쳐 **키 단위로** 병합됨).

> **Note:** v2.1.75 기준으로 더 이상 사용되지 않는 Windows 폴백 경로 `C:\ProgramData\ClaudeCode\managed-settings.json`이 제거되었습니다. 대신 `C:\Program Files\ClaudeCode\managed-settings.json`을 사용하세요.

> **Note (v2.1.126):** `/config`는 이제 변경 사항을 메모리에만 유지하지 않고 `~/.claude/settings.json`에 영구 저장합니다. 대화형 Config UI를 통해 수행한 편집은 재시작 후에도 유지됩니다.

**Managed 전용 정책 키:**

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `parentSettingsBehavior` | string | `"first-wins"` | 임베딩 호스트 프로세스(SDK 부모)가 프로그램적으로 제공한 managed 설정이, admin이 배포한 managed 계층도 함께 존재할 때 적용될지 여부를 제어합니다. `"first-wins"`: 부모 제공 설정은 폐기되고 admin 계층만 적용됩니다. `"merge"`: 부모 제공 설정이 admin 계층 아래에서 적용되며, 정책을 **강화**할 수는 있지만 완화할 수는 없도록 필터링됩니다. v2.1.133+ 필요 |
| `policyHelper` | object | - | 시작 시 managed 설정을 동적으로 계산하는 admin 배포 실행 파일. 객체 형태: `{path: string, timeoutMs?: number, refreshIntervalMs?: number}` — `path`는 헬퍼 바이너리를 가리키고, `timeoutMs`는 대기 시간을 제한하며(생략 또는 `0`이면 타임아웃 없음), `refreshIntervalMs`는 세션 시작 시 재실행을 제어합니다(생략 시 시작 시에만, `0`이면 비활성화, 그 외에는 `60000` 이상이어야 함). MDM 또는 시스템 `managed-settings.json` 파일에서만 존중됩니다(사용자/프로젝트 설정에서는 절대 안 됨). **구성되면 `policyHelper` 출력이 유일한 활성 managed 소스가 되며, 해당 실행에 대해 다른 managed 계층 소스(server-managed, MDM, 파일)는 무시됩니다.** v2.1.136+ 필요 |
| `requiredMinimumVersion` | string | - | **(Managed 전용)** 설치된 버전이 이 하한선보다 낮으면 Claude Code 시작을 차단합니다. CLI가 오류와 함께 종료되며 사용자에게 업그레이드를 안내합니다. `minimumVersion`(자동 업데이트 하한 제어)을 보완하며, 이 키는 시작 시점에 강제합니다. 예: `"2.1.163"` |
| `requiredMaximumVersion` | string | - | **(Managed 전용)** 설치된 버전이 이 상한선을 초과하면 Claude Code 시작을 차단합니다. 버전이 너무 새로우면 CLI가 오류와 함께 종료됩니다. managed 환경에서 특정 버전 범위를 고정하려면 `requiredMinimumVersion`과 함께 사용하세요. 예: `"2.1.165"` |
| `browserExternalPageTools` | string | - | **(Managed 전용)** `"disabled"`로 설정하면 Claude가 데스크톱 앱의 Browser 창에서 외부 페이지를 읽거나 조작하는 도구를 사용하지 못하게 합니다. 사용자는 여전히 외부 사이트를 직접 탐색할 수 있으며, 로컬 개발 서버 미리보기는 영향을 받지 않습니다 |
| `disableBrowserExternalNavigation` | boolean | - | **(Managed 전용)** `true`(JSON boolean만 — 문자열 `"true"`는 조용히 무시됨)로 설정하면 Claude가 데스크톱 앱의 Browser 창을 외부 URL로 탐색하지 못하게 합니다. JSON boolean `true`만 존중됩니다 |
| `disableMobileSimulatorTools` | boolean | - | **(Managed 전용)** `true`(JSON boolean만 — 잘못된 값은 경고를 로깅함)로 설정하면 모바일 시뮬레이터 도구를 Claude에서 제거합니다. JSON boolean `true`만 존중됩니다 |

**중요**:
- `deny` 규칙은 가장 높은 안전 우선순위를 가지며, 우선순위가 낮은 allow/ask 규칙으로 재정의할 수 없습니다.
- Managed 설정은 로컬 파일이 다른 값을 지정하더라도 로컬 동작을 잠그거나 재정의할 수 있습니다.
- 배열 설정(예: `permissions.allow`)은 스코프 전반에 걸쳐 **연결되고 중복 제거**됩니다 — 모든 레벨의 항목이 대체되지 않고 결합됩니다. **예외:** `fallbackModel`과 `availableModels`는 병합되지 **않습니다** — 이를 정의하는 가장 높은 우선순위의 설정 파일이 전체 값을 제공합니다. `availableModels`의 경우, managed 소스 값이 그대로 적용되며 하위 스코프가 이를 확장할 수 없습니다.

---

## Core Configuration

### General Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `$schema` | string | - | IDE 검증 및 자동 완성을 위한 JSON Schema URL(예: `"https://json.schemastore.org/claude-code-settings.json"`) |
| `model` | string | `"default"` | 기본 모델을 재정의합니다. 별칭(`sonnet`, `opus`, `haiku`) 또는 전체 모델 ID를 허용합니다 |
| `agent` | string | - | 메인 대화의 기본 에이전트를 설정합니다. 값은 `.claude/agents/`의 에이전트 이름입니다. `--agent` CLI 플래그로도 사용할 수 있습니다 |
| `language` | string | `"english"` | Claude가 선호하는 응답 언어. 음성 받아쓰기 언어와 **자동 생성 세션 제목**(v2.1.121; v2.1.176 이후로 언어가 설정되지 않으면 제목이 대화 언어와 일치)도 설정합니다 |
| `claudeMdExcludes` | array | - | [memory](https://code.claude.com/docs/en/memory)를 로드할 때 건너뛸 `CLAUDE.md` 파일의 glob 패턴 또는 절대 경로. 패턴은 절대 파일 경로와 매칭됩니다. 사용자, 프로젝트, 로컬 메모리에만 적용되며 managed 정책 파일은 제외할 수 없습니다. 예: `["**/vendor/**/CLAUDE.md"]` |
| `claudeMd` | string | - | **(Managed 전용)** 조직 관리형 [memory](https://code.claude.com/docs/en/memory)로 주입되는 CLAUDE.md 스타일 지침. managed 또는 정책 설정에서 설정된 경우에만 존중되며, 사용자·프로젝트·로컬 설정에서는 무시됩니다. 예: `"Always run make lint before committing."` |
| `cleanupPeriodDays` | number | `30` | 시작 시 정리 스윕의 나이 컷오프(최소 1). 비활성 세션 트랜스크립트와 고아 서브에이전트 worktree가 삭제되며, v2.1.117 기준으로 스윕은 `~/.claude/tasks/`, `~/.claude/shell-snapshots/`, `~/.claude/backups/`도 포함합니다. `0`으로 설정하면 검증 오류로 거부됩니다. 비대화형 모드(`-p`)에서 트랜스크립트 쓰기를 비활성화하려면 `--no-session-persistence` 또는 SDK 옵션 `persistSession: false`를 사용하세요 |
| `autoUpdatesChannel` | string | `"latest"` | 릴리스 채널: `"stable"` 또는 `"latest"` |
| `minimumVersion` | string | - | 자동 업데이터가 특정 버전 아래로 다운그레이드하는 것을 방지합니다. stable 채널로 전환하면서 stable이 따라잡을 때까지 현재 버전에 머무르기로 선택하면 자동으로 설정됩니다. `autoUpdatesChannel`과 함께 사용됩니다 |
| `alwaysThinkingEnabled` | boolean | `false` | 모든 세션에 대해 기본적으로 확장 사고(extended thinking)를 활성화합니다 |
| `thinkingBudgetTokens` | number | - | 응답당 확장 사고에 대한 고정 토큰 예산을 설정합니다. 설정되면 사고 토큰 수를 지정된 값으로 제한합니다. 설정되지 않으면 Claude Code는 모델과 effort 레벨에 기반한 동적 예산을 사용합니다. `alwaysThinkingEnabled`와 함께 작동합니다 *(공식 설정 페이지에 없음 — 미검증)* |
| `skipWebFetchPreflight` | boolean | `false` | 가져오기 전에 각 요청된 호스트명을 `api.anthropic.com`으로 보내는 WebFetch 도메인 안전 검사를 건너뜁니다. Bedrock, Vertex AI, 또는 제한적 egress를 가진 Foundry 배포처럼 Anthropic으로의 아웃바운드 트래픽을 차단하는 환경에서는 `true`로 설정하세요 |
| `availableModels` | array | - | 사용자가 `/model`, `--model`, Config 도구, 또는 `ANTHROPIC_MODEL`을 통해 선택할 수 있는 모델을 제한합니다. Default 옵션에는 영향을 주지 않습니다. v2.1.172 기준으로 서브에이전트 디스패치 및 `advisorModel` 선택기의 모델 선택기도 제약합니다. Default 모델 옵션까지 추가로 제약하려면 `enforceAvailableModels: true`를 사용하세요. 예: `["sonnet", "haiku"]` |
| `enforceAvailableModels` | boolean | `false` | **(Managed 전용)** `true`이면 `availableModels` 허용 목록이 Default 모델 옵션도 제약합니다 — 사용자는 Default 슬롯을 통해서도 허용 목록 밖의 모델을 선택할 수 없습니다. 이 플래그가 없으면 `availableModels`는 Default 옵션을 제한하지 않습니다. 완전한 모델 잠금을 위해 `availableModels`와 함께 사용하세요(v2.1.175) |
| `fastModePerSessionOptIn` | boolean | `false` | 사용자가 매 세션마다 fast 모드에 옵트인하도록 요구합니다 |
| `fastMode` | boolean | `false` | 모든 세션에 fast 모드를 활성화합니다. `true`이면 Claude Code가 기본적으로 더 빠른 모델 계층을 사용합니다. 사용자는 `/fast`로 세션별로 fast 모드를 토글할 수도 있습니다. `fastModePerSessionOptIn`과 함께 사용됩니다 |
| `defaultShell` | string | `"bash"` | 입력창 `!` 명령의 기본 셸. `"bash"`(기본값) 또는 `"powershell"`을 허용합니다. `"powershell"`로 설정하면 Windows에서 대화형 `!` 명령을 PowerShell로 라우팅합니다. `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`이 필요합니다(v2.1.84). **v2.1.120:** PowerShell을 사용할 수 있으면 Git for Windows가 설치되지 않았더라도 Windows에서 폴백 셸로 사용됩니다. **v2.1.126:** PowerShell이 활성화되면 Bash로 기본 설정하는 대신 *주* 셸로 취급됩니다. PowerShell 7 감지는 이제 Microsoft Store 설치, PATH에 없는 MSI 설치, `.NET` 전역 도구 설치도 포함합니다 |
| `includeGitInstructions` | boolean | `true` | 내장 커밋 및 PR 워크플로 지침과 git 상태 스냅샷을 Claude의 시스템 프롬프트에 포함합니다. `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` 환경 변수가 설정되면 이 설정보다 우선합니다 |
| `voice` | object | - | 음성 받아쓰기 구성. 세 필드를 가진 객체: `enabled`(boolean — push-to-talk 켜기/끄기), `mode`(string — hold-to-talk의 경우 `"hold"` 또는 tap-to-toggle의 경우 `"tap"`), `autoSubmit`(boolean — 받아쓰기가 끝나면 즉시 트랜스크립트 제출). `/voice`를 실행하면 자동으로 작성됩니다. Claude.ai 계정이 필요합니다(v2.1.118에서 구조 확장) |
| `voiceEnabled` | boolean | - | **DEPRECATED** — `voice.enabled`의 레거시 별칭. `mode`와 `autoSubmit` 제어를 얻으려면 `voice` 객체를 대신 사용하세요 |
| `showClearContextOnPlanAccept` | boolean | `false` | plan 수락 화면에 "clear context" 옵션을 표시합니다. 옵션을 복원하려면 `true`로 설정하세요(v2.1.81 이후 기본적으로 숨김) |
| `viewMode` | string | - | 시작 시 기본 트랜스크립트 보기 모드: `"default"`, `"verbose"`, 또는 `"focus"`. 설정되면 sticky `/focus` 토글 선택을 재정의합니다 |
| `disableDeepLinkRegistration` | string | - | `"disable"`로 설정하면 Claude Code가 시작 시 운영 체제에 `claude-cli://` 프로토콜 핸들러를 등록하지 못하게 합니다. 딥링크를 사용하면 외부 도구가 `claude-cli://open?q=...`를 통해 미리 채워진 프롬프트로 Claude Code 세션을 열 수 있습니다. `q` 매개변수는 URL 인코딩된 개행(`%0A`)을 사용해 여러 줄 프롬프트를 지원합니다. 프로토콜 핸들러 등록이 제한되거나 별도로 관리되는 환경에서 유용합니다 |
| `showThinkingSummaries` | boolean | `false` | 대화형 세션에서 확장 사고 요약을 표시합니다. 설정되지 않았거나 `false`이면(대화형 모드 기본값), 사고 블록은 API에 의해 편집되어 접힌 스텁으로 표시됩니다. 편집은 모델이 생성하는 내용이 아니라 보이는 내용만 바꿉니다 — 사고 비용을 줄이려면 예산을 낮추거나 사고를 비활성화하세요. 비대화형 모드(`-p`)와 SDK 호출자는 이 설정과 관계없이 항상 요약을 받습니다 |
| `disableSkillShellExecution` | boolean | `false` | 사용자, 프로젝트, 플러그인, 또는 추가 디렉터리 소스의 스킬과 커스텀 명령에서 `` !`...` `` 및 `` ```! `` 블록의 인라인 셸 실행을 비활성화합니다. 명령은 실행되는 대신 `[shell command execution disabled by policy]`로 대체됩니다. 번들 및 managed 스킬은 영향을 받지 않습니다(v2.1.91) |
| `skillListingMaxDescChars` | number | `1536` | Claude가 매 턴 보는 [skill listing](https://code.claude.com/docs/en/skills)에서 결합된 `description`과 `when_to_use` 텍스트에 대한 스킬별 문자 상한. 이보다 긴 텍스트는 잘립니다(v2.1.105) |
| `skillListingBudgetFraction` | number | `0.01` | Claude가 매 턴 보는 [skill listing](https://code.claude.com/docs/en/skills)을 위해 예약된 모델 컨텍스트 윈도의 비율(`0.01` = 1%). 목록이 예산을 초과하면, 가장 적게 사용된 스킬의 설명이 이름만으로 축소되어 Claude가 여전히 호출할 수는 있지만 이유는 볼 수 없게 됩니다(v2.1.105) |
| `forceRemoteSettingsRefresh` | boolean | `false` | **(Managed 전용)** 원격 managed 설정이 새로 가져와질 때까지 CLI 시작을 차단합니다. 가져오기가 실패하면 CLI가 종료됩니다(fail-closed). 모든 세션이 시작되기 전에 정책 강제가 최신이어야 하는 엔터프라이즈 환경에서 사용하세요(v2.1.92) |
| `wslInheritsWindowsSettings` | boolean | `false` | **(Windows managed settings 전용)** `true`이면 WSL의 Claude Code가 `/etc/claude-code`에 더해 Windows 정책 체인(HKLM 레지스트리 + `C:\Program Files\ClaudeCode\managed-settings.json`)에서 managed 설정을 읽으며, Windows 소스가 우선합니다. HKLM 레지스트리 키 또는 `C:\Program Files\ClaudeCode\managed-settings.json`에서 설정된 경우에만 존중되며, 둘 다 쓰기에 Windows 관리자 권한이 필요합니다. HKCU 정책도 WSL에 적용되려면 플래그가 HKCU 자체에도 추가로 설정되어야 합니다. 네이티브 Windows에는 영향을 주지 않습니다(v2.1.118) |
| `tui` | string | `"default"` | 렌더링 모드: `"fullscreen"` 또는 `"default"`. 깜빡임 없는 alt-screen 렌더링을 위해 `/tui fullscreen`으로 설정합니다(v2.1.110) |
| `awaySummaryEnabled` | boolean | `true` | 사용자가 자리를 비운 후 돌아왔을 때 "away summary"(유휴 세션 요약)를 생성합니다. 옵트아웃하려면 `false`로 설정하세요. `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` 환경 변수와 함께 사용됩니다(v2.1.110) |
| `autoCompactEnabled` | boolean | `true` | 컨텍스트가 모델 한계에 접근하면 대화를 자동 압축합니다. 자동 압축을 비활성화하고 컨텍스트를 수동으로 관리하려면 `false`로 설정하세요. `DISABLE_AUTO_COMPACT` 환경 변수로도 비활성화할 수 있습니다 |
| `autoCompactWindow` | number | model-tuned | 자동 압축이 트리거되기 전 컨텍스트 윈도가 얼마나 차는지(토큰 단위, 100,000–1,000,000). 설정되지 않으면 Claude Code가 모델에 맞춰 조정된 윈도를 사용합니다. `/autocompact` 명령(사용자 설정에 기록) 또는 `--autocompact` CLI 플래그로 설정하세요. `CLAUDE_CODE_AUTO_COMPACT_WINDOW` 환경 변수와 함께 사용됩니다(v2.1.221+) |
| `skillOverrides` | object | - | 스킬 이름으로 키가 지정되는 스킬별 가시성 재정의. 값은 `"on"`(전체), `"name-only"`(보이지만 자동 설명 안 됨), `"user-invocable-only"`(모델 발견에서 숨겨지지만 슬래시로는 여전히 호출 가능), 또는 `"off"`(완전히 숨김)입니다. 예: `{"legacy-context": "name-only", "deploy": "off"}`(v2.1.129) |
| `disableRemoteControl` | boolean | `false` | [Remote Control](https://code.claude.com/docs/en/remote-control) 비활성화: `claude remote-control`, `--remote-control` 플래그, 자동 시작, 세션 내 토글을 차단합니다. 일반적으로 기기별 MDM 강제를 위해 managed 설정에 배치되지만, 어떤 스코프에서도 작동합니다(v2.1.128) |
| `agentPushNotifEnabled` | boolean | `false` | Claude가 푸시하기로 결정할 때(예: 작업 완료) [Remote Control](https://code.claude.com/docs/en/remote-control)에 사전 푸시 알림을 보냅니다. `/config`에 **Push when Claude decides**로 표시됩니다 |
| `inputNeededNotifEnabled` | boolean | `false` | 권한 프롬프트나 질문이 사용자 입력을 기다릴 때 [Remote Control](https://code.claude.com/docs/en/remote-control)에 푸시 알림을 보냅니다. `/config`에 **Push when actions required**로 표시됩니다 |
| `remoteControlAtStartup` | boolean/null | - | 시작 시 [Remote Control](https://code.claude.com/docs/en/remote-control)을 자동 연결합니다. `true`는 항상 자동 연결, `false`는 절대 자동 연결 안 함, 미설정은 조직 기본값을 사용합니다. **스코프 예외:** 프로젝트 또는 로컬 설정의 `false`는 managed `true`에 대해서도 적용됩니다(사용자 및 프로젝트 설정이 옵트아웃할 수 있음). 사용자 설정, `--settings`, managed 설정만 `true`로 설정할 수 있습니다(v2.1.119+) |
| `disableAgentView` | boolean | `false` | `true`로 설정하면 [백그라운드 에이전트 및 agent view](https://code.claude.com/docs/en/agent-view)를 끕니다: `claude agents`, `--bg`, `/background`, 온디맨드 슈퍼바이저. 어떤 스코프에서도 설정할 수 있지만 일반적으로 managed 설정에 배치됩니다. `CLAUDE_CODE_DISABLE_AGENT_VIEW` 환경 변수를 `1`로 설정하는 것과 동일합니다 |
| `disableWorkflows` | boolean | `false` | `true`로 설정하면 [동적 워크플로](https://code.claude.com/docs/en/workflows)(`/workflows`)와 번들 워크플로 슬래시 명령을 비활성화합니다. 어떤 스코프에서도 설정할 수 있습니다. `CLAUDE_CODE_DISABLE_WORKFLOWS` 환경 변수와 동일합니다. 워크플로는 v2.1.154에서 도입되었습니다 |
| `workflowKeywordTriggerEnabled` | boolean | `true` | 프롬프트에 "ultracode"라는 단어를 입력하면 [동적 워크플로](https://code.claude.com/docs/en/workflows)를 트리거할지 여부. 명시적 `/workflows` 호출을 요구하려면 `false`로 설정하세요. Ultracode, `/workflows`, 저장된 워크플로 명령은 이 설정의 영향을 받지 않습니다. `/config`에 **Ultracode keyword trigger**로 표시됩니다(v2.1.157; 트리거 키워드가 v2.1.160에서 workflow→ultracode로 이름 변경됨) |
| `ultracode` | boolean | - | **(세션 전용 — 영구 저장 안 됨)** `true`이면 하니스가 토큰 비용과 관계없이 철저함을 극대화하여 모든 실질적 작업에 대해 기본적으로 워크플로를 작성하고 실행합니다. 공식 "Available settings" 목록에 나타나지만 세션 범위입니다: `settings.json`에 기록하는 대신 `/effort ultracode`, `--settings`, 또는 SDK로 설정합니다(v2.1.154) |
| `dynamicWorkflowSize` | string | - | [동적 워크플로](https://code.claude.com/docs/en/workflows)에서 스폰되는 에이전트 수에 대한 권고 지침. 값: `"small"`, `"medium"`, `"large"`. 설정되면 워크플로 하니스가 작업에 따라 규모를 늘리거나 줄이기 전 기본 플릿 크기로 이를 사용합니다. `/config`에서 **Workflow size**로 설정(v2.1.202; v2.1.205에서 값 공식화) *(공식 문서에 없음 — 미검증; v2.1.219에서 `workflowSizeGuideline`으로 대체됨)* |
| `workflowSizeGuideline` | string | `"medium"` | 어떤 설정 파일에서도 설정 가능한 동적 워크플로 플릿 크기에 대한 권고 지침. 값: `"small"`, `"medium"`(v2.1.219 기준 기본값), `"large"`, `"unrestricted"`. 기본 플릿 크기가 이제 실행 중인 워크플로 상태 표시줄에 렌더링됩니다. `dynamicWorkflowSize` 대신 이 키를 사용하세요 — managed 또는 사용자 설정에서 전파됩니다(v2.1.219) |
| `disableBundledSkills` | boolean | `false` | Claude Code의 내장 기능(번들 스킬)을 모델로부터 숨깁니다. `true`이면 모델이 내장 스킬을 호출할 수 없습니다. `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` 환경 변수와 함께 사용됩니다. 엄격한 플러그인 전용 커스터마이징이 필요할 때 유용합니다(v2.1.169) |
| `disableArtifact` | boolean | `false` | Artifact 웹 게시 도구를 비활성화합니다. `true`이면 Claude가 웹 아티팩트를 생성하거나 게시할 수 없습니다. 어떤 스코프에서도 설정할 수 있습니다 |
| `enableArtifact` | boolean | - | **(v2.1.196+)** Artifact 웹 게시 도구에 대한 사용자 레벨 옵트인. `true`로 설정하면 조직 정책이 요구하지 않더라도 사용자에게 Artifact를 활성화합니다. `disableArtifact: true`가 우선하며 이 설정을 재정의합니다 |
| `feedbackSurveyRate` | number | - | 자격이 될 때 세션 품질 설문이 나타날 확률(0–1). 엔터프라이즈 관리자는 설문이 표시되는 빈도를 제어할 수 있습니다. 예: `0.05` = 자격 세션의 5% |
| `advisorModel` | string | - | 서버 측 advisor 도구용 모델. 모델 별칭(`opus`, `sonnet`) 또는 전체 모델 ID를 허용합니다. 설정되지 않으면 advisor는 세션 모델을 사용합니다. v2.1.98+ 필요. **v2.1.210+:** `"fable"`로 설정해도 더 이상 advisor가 연결되지 않습니다 — Fable 5는 advisor 선택기에서 일시적으로 사용할 수 없습니다; 대신 `"opus"` 또는 `"sonnet"`을 사용하세요 |
| `respondToBashCommands` | boolean | `true` | `!` 셸 명령이 완료된 후 Claude가 자동으로 응답할지 여부. `!` bash 명령이 끝났을 때 자동 후속 응답을 비활성화하려면 `false`로 설정하세요(v2.1.186) |
| `askUserQuestionTimeout` | string | `"never"` | 응답되지 않은 AskUserQuestion 대화 상자가 사용자 없이 자동 계속되기 전 대기 시간. 값: `"60s"`, `"5m"`, `"10m"`, `"never"`(자동 계속 안 함 — 기본값). `/config`에서 **Question auto-continue timeout**으로 설정합니다. `CLAUDE_AFK_TIMEOUT_MS` 환경 변수와 함께 사용됩니다; 환경 변수는 이 설정이 duration으로 설정된 경우에만 적용됩니다. 프로젝트 및 로컬 설정에서만 존중됩니다 — managed 및 사용자 설정 값은 무시됩니다.(v2.1.200) |
| `theme` | string | `"dark"` | UI 색상 테마. 값: `"auto"`(OS를 따름), `"dark"`, `"light"`, `"dark-daltonized"`, `"light-daltonized"`, `"dark-ansi"`, `"light-ansi"`, `"custom:<slug>"`, `"custom:<plugin>:<slug>"`. 플러그인 제공 테마는 `custom:` 접두사를 사용합니다 |
| `verbose` | boolean | `false` | 잘린 요약 대신 전체 도구 출력을 표시합니다. `--verbose`로 실행하는 것과 동일하며, verbose 보기를 세션 간에 유지합니다 |
| `switchModelsOnFlag` | boolean | `true` | 안전 분류기가 요청을 플래그할 때 자동으로 폴백 모델로 전환합니다. `false`이면 플래그된 요청이 재라우팅되지 않고 차단됩니다(v2.1.170) |
| `processWrapper` | string | - | 백그라운드 프로세스에 사용되는 기업 런처 명령(예: 자격 증명 주입 래퍼). managed 설정, 사용자 설정(`~/.claude/settings.json`), 또는 `--settings`에서만 존중됩니다; 신뢰할 수 없는 저장소가 백그라운드 프로세스 시작을 가로채는 것을 방지하기 위해 프로젝트 및 로컬 설정에서는 무시됩니다(v2.1.210) |
| `remote.defaultEnvironmentId` | string | - | 명시적 environment ID 없이 `--remote`를 통해 원격 세션 또는 백그라운드 에이전트를 시작할 때 사용할 기본 environment ID. 설정되면 Claude Code가 프롬프트하는 대신 이 환경을 자동 선택합니다. 사용자 및 managed 설정에서만 존중됩니다(v2.1.200+) |

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

plan 및 auto-memory 파일을 커스텀 위치에 저장합니다.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `plansDirectory` | string | `~/.claude/plans` | `/plan` 출력이 저장되는 디렉터리 |
| `autoMemoryDirectory` | string | - | auto-memory 저장을 위한 커스텀 디렉터리. `~/`로 확장되는 경로를 허용합니다. 메모리 쓰기를 민감한 위치로 리디렉션하는 것을 방지하기 위해 프로젝트 설정(`.claude/settings.json`)에서는 허용되지 않으며, 정책·로컬·사용자 설정에서 허용됩니다 |
| `autoMemoryEnabled` | boolean | `true` | [auto memory](https://code.claude.com/docs/en/memory)를 활성화합니다. `false`이면 Claude가 auto-memory 디렉터리를 읽거나 쓰지 않습니다. 세션 중 `/memory`로도 토글할 수 있으며, `CLAUDE_CODE_DISABLE_AUTO_MEMORY` 환경 변수로 비활성화할 수 있습니다 |
| `fileCheckpointingEnabled` | boolean | `true` | `/rewind`로 복원할 수 있도록 각 편집 전에 파일을 스냅샷합니다. 체크포인팅을 건너뛰고 디스크 공간을 절약하려면 `false`로 설정하세요. `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` 환경 변수로도 비활성화할 수 있습니다 |

**Example:**
```json
{
  "plansDirectory": "./my-plans"
}
```

**Use Case:** 계획 아티팩트를 Claude의 내부 파일과 별도로 정리하거나, 팀 공유 위치에 계획을 보관하는 데 유용합니다.

### Worktree Settings

`--worktree`가 git worktree를 생성하고 관리하는 방식을 구성합니다. 대규모 모노레포에서 디스크 사용량과 시작 시간을 줄이는 데 유용합니다.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `worktree.symlinkDirectories` | array | `[]` | 큰 디렉터리를 디스크에 중복 생성하지 않도록 메인 저장소에서 각 worktree로 심링크할 디렉터리 |
| `worktree.sparsePaths` | array | `[]` | git sparse-checkout(cone mode)을 통해 각 worktree에서 체크아웃할 디렉터리. 나열된 경로만 디스크에 기록됩니다 |
| `worktree.baseRef` | string | `"fresh"` | 새 worktree가 분기하는 ref. `"fresh"`는 원격과 일치하는 깨끗한 트리를 위해 `origin/<default-branch>`에서 분기합니다. `"head"`는 커밋되지 않았지만 추적되는 변경을 포함하여 현재 로컬 `HEAD`에서 분기합니다(v2.1.133) |
| `worktree.bgIsolation` | string | `"worktree"` | [백그라운드 세션](https://code.claude.com/docs/en/agent-view)의 격리 모드. `"worktree"`(기본값)는 `EnterWorktree`가 호출될 때까지 메인 체크아웃에서 `Edit`/`Write`를 차단합니다; `"none"`은 백그라운드 작업이 작업 사본을 직접 편집하도록 허용합니다(v2.1.143) |

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

git 커밋 및 pull request에 대한 attribution 메시지를 커스터마이징합니다.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `attribution.commit` | string | Co-authored-by | Git 커밋 attribution(트레일러 지원) |
| `attribution.pr` | string | Generated message | Pull request 설명 attribution |
| `attribution.sessionUrl` | boolean | `true` | 웹 세션 또는 Remote Control 세션에서 생성된 커밋과 PR에 claude.ai 세션 URL 링크를 포함합니다. 링크를 생략하려면 `false`로 설정하세요. 로컬 CLI 세션에는 영향을 주지 않습니다(v2.1.183) |
| `prUrlTemplate` | string | - | 커밋 attribution의 "PR" 배지가 pull request UI로 링크되는 방식을 제어하는 URL 템플릿. 저장소 호스트, 소유자, 저장소, PR 번호에 대한 플레이스홀더를 지원합니다. 기본 `https://github.com/...` URL이 적용되지 않는 자체 호스팅 GitLab/Bitbucket/GitHub Enterprise 인스턴스에 유용합니다(v2.1.119) |
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

**Note:** attribution을 완전히 숨기려면 빈 문자열(`""`)로 설정하세요.

### Authentication Helpers

동적 인증 토큰 생성을 위한 스크립트.

| Key | Type | Description |
|-----|------|-------------|
| `apiKeyHelper` | string | 인증 토큰을 출력하는 셸 스크립트 경로(`X-Api-Key`와 `Authorization: Bearer` 헤더 양쪽으로 전송됨) |
| `forceLoginMethod` | string | 로그인을 `"claudeai"`, `"console"`, 또는 `"gateway"` 계정으로 제한합니다. 조직 관리형 Claude 게이트웨이 배포에는 `"gateway"`를 사용하세요. **(Managed 전용)** |
| `forceLoginOrgUUID` | string \| array | 로그인이 특정 조직에 속하도록 요구합니다. 단일 UUID 문자열(로그인 중 해당 조직을 사전 선택하기도 함) 또는 나열된 조직 중 어느 것이든 사전 선택 없이 허용되는 UUID 배열을 받습니다. managed 설정에서 설정되면, 인증된 계정이 나열된 조직에 속하지 않으면 로그인이 실패합니다; 빈 배열은 fail closed로 오설정 메시지와 함께 로그인을 차단합니다 |
| `forceLoginGatewayUrl` | string | **(Managed 전용)** `forceLoginMethod`가 `"gateway"`일 때 로그인 화면에 Claude 게이트웨이 URL을 미리 채웁니다. 사용자가 게이트웨이 URL을 수동 입력할 필요를 없앱니다. 일반적으로 `forceLoginMethod: "gateway"`와 함께 설정됩니다 |
| `gcpAuthRefresh` | string | GCP Application Default Credentials가 만료되거나 로드될 수 없을 때 이를 갱신하는 커스텀 스크립트. 인증을 재시도하기 전에 Claude Code가 실행합니다. ADC가 수명이 짧고 갱신에 조직별 헬퍼가 필요할 때 유용합니다. 예: `"gcloud auth application-default login"` |

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
| `permissions.defaultMode` | string | 기본 권한 모드. 유효 값: `"default"`, `"manual"`(`"default"`의 별칭, v2.1.200), `"acceptEdits"`, `"dontAsk"`, `"bypassPermissions"`, `"auto"`, `"plan"`. Remote 환경에서는 `acceptEdits`와 `plan`만 존중됩니다(v2.1.70+). **Note (v2.1.142):** `"auto"`는 프로젝트(`.claude/settings.json`) 또는 로컬 설정에서 설정되면 무시됩니다 — 저장소가 스스로에게 auto 모드를 부여할 수 없습니다; 대신 `~/.claude/settings.json`을 사용하세요 |
| `permissions.disableBypassPermissionsMode` | string | bypass 모드 활성화를 방지 |
| `permissions.skipDangerousModePermissionPrompt` | boolean | `--dangerously-skip-permissions` 또는 `defaultMode: "bypassPermissions"`를 통해 bypass 권한 모드에 진입하기 전에 표시되는 확인 프롬프트를 건너뜁니다. 신뢰할 수 없는 저장소가 프롬프트를 자동 우회하는 것을 방지하기 위해 프로젝트 설정(`.claude/settings.json`)에서 설정되면 무시됩니다 |
| `allowManagedPermissionRulesOnly` | boolean | **(Managed 전용)** managed 권한 규칙만 적용됩니다; 사용자/프로젝트 `allow`, `ask`, `deny` 규칙은 무시됩니다 |
| `autoMode` | object | [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 분류기가 차단하고 허용하는 것을 커스터마이징합니다. `environment`(신뢰할 수 있는 인프라 설명), `allow`(차단 규칙에 대한 예외), `soft_deny`(차단 규칙), `hard_deny`(무조건 차단 규칙 — `allow` 예외나 `$defaults` 센티널로 재정의 불가, v2.1.136)를 포함하며 — 모두 산문 문자열의 배열입니다. `classifyAllShell`(boolean, 기본값 `false`)도 허용합니다: `true`이면 임의 코드 실행 패턴만이 아니라 모든 Bash/PowerShell 명령을 auto-mode 분류기로 라우팅하여, 더 많은 분류기 호출을 대가로 더 엄격한 커버리지를 제공합니다(v2.1.193). **repo 인젝션을 방지하기 위해 공유 프로젝트 설정**(`.claude/settings.json`) **또는 로컬 설정**(`.claude/settings.local.json`)**에서 읽지 않습니다**(v2.1.207에서 로컬 설정 스코프 제거). 사용자 및 managed 설정에서만 사용 가능합니다; `~/.claude/settings.json`에서 구성하세요. `allow` 또는 `soft_deny`를 설정하면, 배열에 리터럴 문자열 `"$defaults"`를 포함하지 않는 한 해당 섹션의 전체 기본 목록을 **대체**합니다 — 센티널은 해당 위치에서 내장 규칙을 상속하므로 커스텀 항목이 그와 함께 추가됩니다(v2.1.118). 커스터마이징하기 전에 내장 규칙을 보려면 `claude auto-mode defaults`를 실행하세요 |
| `disableAutoMode` | string | `"disable"`로 설정하면 [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 활성화를 방지합니다. `Shift+Tab` 순환에서 `auto`를 제거하고 시작 시 `--permission-mode auto`를 거부합니다. 어떤 설정 레벨에서도 설정할 수 있지만, 사용자가 재정의할 수 없는 managed 설정에서 가장 유용합니다 |
| `useAutoModeDuringPlan` | boolean | auto 모드를 사용할 수 있을 때 plan 모드가 auto 모드 시맨틱을 사용할지 여부. 기본값: `true`. 공유 프로젝트 설정(`.claude/settings.json`)에서 읽지 않습니다. `/config`에 "Use auto mode during plan"으로 표시됩니다 |

### Permission Modes

| Mode | Behavior |
|------|----------|
| `"default"` | 프롬프트가 있는 표준 권한 검사 |
| `"manual"` | v2.1.200에서 도입된 `"default"`의 별칭 — 프롬프트가 있는 동일한 표준 권한 검사. 이름 변경은 CLI, VS Code, JetBrains UI 전반의 명료성을 개선합니다. `"default"`가 허용되는 모든 곳에서 허용됩니다 |
| `"acceptEdits"` | 작업 디렉터리 또는 `additionalDirectories`의 경로에 대해 파일 편집 **및 일반 파일시스템 명령**(`mkdir`, `touch`, `mv`, `cp` 등)을 자동으로 수락합니다. **v2.1.160:** 코드 실행을 부여하는 빌드 도구 설정 파일(`.npmrc`, `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`, `.devcontainer/` 등)을 쓰기 전과, 셸 시작 파일(`.zshenv`, `.zlogin`, `.bash_login`) 및 `~/.config/git/`에 쓰기 전에 항상 프롬프트합니다 |
| `"dontAsk"` | `/permissions` 또는 `permissions.allow` 규칙을 통해 사전 승인되지 않은 도구를 자동 거부합니다. MCP 서버 매니페스트에 `requiresUserInteraction`으로 표시된 도구와 조직이 ask로 설정한 커넥터 도구는 명시적으로 허용했더라도 **거부**됩니다 |
| `"bypassPermissions"` | 모든 권한 검사를 건너뜁니다(위험). 모든 경로 기반 프롬프트를 건너뜁니다 — `.git`, `.config/git`, `.claude`, `.vscode`, `.idea`, `.husky`, `.cargo`, `.devcontainer`, `.yarn`, `.mvn`에 대한 쓰기가 더 이상 프롬프트하지 않습니다(**v2.1.121**이 `.claude/commands/`, `.claude/agents/`, `.claude/skills/`, `.claude/worktrees/`를 면제; **v2.1.126**이 남은 모든 경로 기반 프롬프트를 제거). 세 가지 부류는 여전히 프롬프트합니다: (1) 파일시스템 루트 또는 홈 디렉터리를 대상으로 하는 제거(`rm -rf /`, `rm -rf ~`, `$(...)`·백틱 치환·`<(...)` 프로세스 치환을 사용하는 명령 포함), (2) MCP 서버 매니페스트에 `requiresUserInteraction`으로 표시된 도구, (3) 조직이 ask로 설정한 커넥터 도구. 명시적 `permissions.ask` 규칙은 여전히 프롬프트합니다 — 내장 권한 검사만 우회됩니다 |
| `"auto"` | 작업이 요청과 일치하는지 검증하는 백그라운드 안전 검사와 함께 도구 호출을 자동 승인합니다. 리서치 프리뷰. 분류기가 읽기 전용 및 파일 편집을 자동 승인하고; 그 외 모든 것은 안전 검사를 통과시킵니다. 연속 3회 또는 총 20회 차단 후 프롬프트로 폴백합니다. v2.1.111 이후 기본 `Shift+Tab` 권한 모드 순환에 포함됩니다(v2.1.111에서 `--enable-auto-mode` 플래그 제거됨 — `--permission-mode auto`로 이 모드에서 시작). `autoMode` 설정으로 구성합니다 |
| `"plan"` | 읽기 전용 탐색 모드. v2.1.136 기준으로 매칭되는 `Edit(...)` allow 규칙이 있더라도 파일 쓰기가 차단됩니다 — plan 모드는 이제 읽기 전용 보장을 유지하기 위해 명시적 allow 규칙을 재정의합니다 |

### Tool Permission Syntax

| Tool | Syntax | Examples |
|------|--------|----------|
| `Bash` | `Bash(command pattern)` | `Bash(npm run *)`, `Bash(* install)`, `Bash(git * main)` |
| `PowerShell` | `PowerShell(cmd *)` | `PowerShell(Get-ChildItem *)`, `PowerShell(git commit *)` — Bash와 동일한 형태; 일반 별칭이 정규화되고(`gci`/`ls`/`dir` → `Get-ChildItem`) PowerShell AST가 파싱되어 `|`/`;`/`&&`/`||` 체인의 각 하위 명령이 매칭되어야 합니다 |
| `Read` | `Read(path pattern)` | `Read(.env)`, `Read(./secrets/**)` |
| `Edit` | `Edit(path pattern)` | `Edit(src/**)`, `Edit(*.ts)` |
| `Write` | `Write(path pattern)` | `Write(*.md)`, `Write(./docs/**)` — **`deny` 및 `ask` 규칙에서만 유효**; `allow` 규칙에서는 파싱 시 허용되지만 절대 참조되지 않습니다(아래 note 참고) |
| `NotebookEdit` | `NotebookEdit(pattern)` | `NotebookEdit(*)` — **`deny` 및 `ask` 규칙에서만 유효**; `allow` 규칙에서는 파싱 시 허용되지만 절대 참조되지 않습니다(아래 note 참고) |
| `WebFetch` | `WebFetch(domain:pattern)` | `WebFetch(domain:example.com)` |
| `WebSearch` | `WebSearch` | 전역 웹 검색 |
| `Task` | `Task(agent-name)` | `Task(Explore)`, `Task(my-agent)` — `Agent`의 레거시 별칭; 새 구성에서는 `Agent(name)`을 선호하세요 |
| `Agent` | `Agent(name)` | `Agent(researcher)`, `Agent(*)` — 서브에이전트 스폰으로 스코프가 지정된 권한 |
| `Skill` | `Skill(skill-name)` 또는 `Skill(prefix *)` | `Skill(weather-fetcher)`, `Skill(weather *)`는 `weather-fetcher`/`weather-svg-creator`와 매칭됩니다(v2.1.139) *(공식 권한 문서에 없음 — 미검증)* |
| `MCP` | `mcp__server__tool` | `mcp__memory__*`, `mcp__github__*` — 공식 문서는 이중 밑줄 형태만 보여줍니다; `MCP(server:tool)` 축약형 *(공식 권한 문서에 없음 — 미검증)* |
| `Tool` | `Tool(param:value)` | `Agent(model:opus)`, `Agent(isolation:worktree)`, `Bash(run_in_background:true)` — 도구의 입력 매개변수에 대해 **deny 및 ask 규칙**을 매칭합니다; 값 위치에서 `*` 와일드카드를 지원합니다. **allow 규칙은 이 구문을 사용하지 않습니다** — 한 매개변수 값에 대한 allow 규칙이 전체 안전성을 확립하지 못하므로, allow 규칙은 계속 각 도구 자체의 지정자 구문을 사용합니다. 도구의 주 콘텐츠 필드 매칭(예: `Bash(command:rm *)`)도 금지되며 시작 경고를 트리거합니다(v2.1.178) |
| `Cd` | `Cd(path pattern)` | `Cd(/home/*)`, `Cd(~/projects/*)` — `/cd` 명령이 탐색할 수 있는 디렉터리를 제어합니다. **Allowlist 모드:** *어떤* `Cd` allow 규칙이라도 추가하면 `/cd`가 allowlist 모드로 전환됩니다 — 명시적으로 허용되지 않은 모든 경로가 거부됩니다. 단독 `Cd` deny는 `/cd`를 완전히 비활성화합니다. **와일드카드 시맨틱이 Read/Edit와 다릅니다:** `*`는 정확히 하나의 경로 세그먼트와 매칭됩니다; `**`는 세그먼트를 넘습니다. 후행 `/**`는 명명된 루트 자체와도 매칭됩니다. 이 규칙들은 gitignore 스타일 매칭을 따르지 **않습니다** |

> **v2.1.210:** **allow** 규칙의 `Write(path)`, `NotebookEdit(path)`, `Glob(path)`는 파싱 시 허용되지만 **절대 참조되지 않습니다** — Claude Code의 allow 규칙 평가는 `Edit(path)`와 `Read(path)`만 확인합니다. 시작 경고가 이러한 항목을 표시하고 올바른 대안을 권장합니다. **deny** 및 **ask** 규칙에서는 `Write(path)`, `NotebookEdit(path)`, `Glob(path)`가 예상대로 계속 작동합니다.

**평가 순서:** 규칙은 순서대로 평가됩니다: deny 규칙 먼저, 그다음 ask, 그다음 allow. 첫 번째로 매칭되는 규칙이 우선합니다.

**Deny 규칙 glob 패턴(v2.1.166):** `deny` 규칙에서 도구 이름 위치에 `"*"`를 사용하면 모든 도구와 매칭됩니다 — 전역 deny와 동일합니다. 예를 들어 deny 배열의 `"*"`는 모든 도구 호출을 차단합니다. 이를 통해 접근을 완전히 잠그고 특정 allow/ask 예외를 만들 수 있습니다.

**Allow 규칙 glob 제한:** `allow` 규칙에서 도구 이름 glob은 **리터럴 `mcp__<server>__` 접두사 뒤에서만** 허용됩니다 — server 세그먼트는 glob이 없어야 합니다. `"*"`, `"B*"`, `"mcp__*"` 같은 앵커되지 않은 allow glob은 **시작 경고와 함께 건너뛰어지며** 아무것도 자동 승인하지 않습니다. 예를 들어 `"allow": ["mcp__github__*"]`는 작동하지만 `"allow": ["mcp__*"]`는 작동하지 않습니다. 대신 `"deny": ["*"]`에 특정 allow 규칙을 더해 허용 목록으로 만드세요.

**Read/Edit 경로 패턴:** `Read`, `Edit`, `Write`에 대한 권한 규칙은 네 가지 접두사 유형의 gitignore 스타일 패턴을 지원합니다:

| Prefix | Meaning | Example |
|--------|---------|---------|
| `//` | 파일시스템 루트로부터의 절대 경로 | `Read(//Users/alice/file)` |
| `~/` | 홈 디렉터리 기준 | `Read(~/.zshrc)` |
| `/` | 프로젝트 루트 기준 | `Edit(/src/**)` |
| `./` 또는 없음 | 상대 경로(현재 디렉터리) | `Read(.env)`, `Read(*.ts)` |

**심링크 해석:** 권한 규칙은 심링크 경로와 해석된 대상 모두를 확인합니다. **Allow** 규칙은 심링크와 그 대상이 *둘 다* 매칭될 때만 적용됩니다 — 허용된 디렉터리 안에 있지만 그 밖을 가리키는 심링크는 여전히 프롬프트합니다. **Deny** 규칙은 심링크 또는 그 대상 *중 하나라도* 매칭되면 적용됩니다 — 거부된 파일에 대한 심링크는 그 자체가 거부됩니다.

**Bash 와일드카드 참고:**
- `*`는 **어느 위치에든** 나타날 수 있습니다: 접두사(`Bash(* install)`), 접미사(`Bash(npm *)`), 또는 중간(`Bash(git * main)`)
- **단어 경계:** `Bash(ls *)`(`*` 앞에 공백)는 `ls -la`와 매칭되지만 `lsof`와는 안 됩니다; `Bash(ls*)`(공백 없음)는 둘 다 매칭됩니다
- `Bash(*)`는 `Bash`와 동등하게 취급됩니다(모든 bash 명령과 매칭)
- 권한 규칙은 출력 리디렉션을 지원합니다: `Bash(python:*)`는 `python script.py > output.txt`와 매칭됩니다
- `:*` 접미사 구문(예: `Bash(npm:*)`)은 후행 와일드카드를 쓰는 동등한 방식입니다 — **더 이상 사용되지 않는 것은 아니지만** 끝에서만 인식됩니다(예: `Bash(git:* push)`는 콜론을 리터럴로 취급). 권한 대화 상자는 공백 형태로 씁니다
- **복합 명령:** 셸 연산자(`&&`, `||`, `;`, `|`, `|&`, `&`, 개행)가 명령을 분할하며 각 하위 명령이 독립적으로 매칭되어야 합니다 — `Bash(safe-cmd *)`는 `safe-cmd && other-cmd`를 승인하지 **않습니다**
- **프로세스 래퍼:** `timeout`, `time`, `nice`, `nohup`, `stdbuf`는 매칭 전에 제거됩니다(따라서 `Bash(npm test *)`는 `timeout 30 npm test`와도 매칭됨); 단독 `xargs`(플래그 없음)도 제거됩니다. Exec 래퍼 `watch`, `setsid`, `ionice`, `flock`, 그리고 `-exec`/`-delete`가 있는 `find`는 항상 프롬프트하며 접두사 규칙으로 승인할 수 없습니다

**Example:**
```json
{
  "permissions": {
    "allow": [
      "Edit(*)",
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__memory__*"
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

Hook 구성(이벤트, 속성, 매처, 종료 코드, 환경 변수, HTTP hooks)은 전용 저장소에서 관리됩니다:

> **[claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks)** — 사운드 알림 시스템, 26개 hook 이벤트 전부, HTTP hooks, 매처 패턴, 종료 코드, 환경 변수를 포함한 완전한 hook 레퍼런스.

Hook 관련 설정 키(`hooks`, `disableAllHooks`(커스텀 상태 표시줄도 비활성화함), `allowManagedHooksOnly`, `allowedHttpHookUrls`, `httpHookAllowedEnvVars`)가 그곳에 문서화되어 있습니다.

공식 hooks 레퍼런스는 [Claude Code Hooks Documentation](https://code.claude.com/docs/en/hooks)을 참고하세요.

---

## MCP Servers

확장 기능을 위한 Model Context Protocol 서버를 구성합니다.

> **OAuth (v2.1.111):** OAuth로 인증하는 MCP 서버는 protected-resource 메타데이터 발견을 위해 [RFC 9728](https://datatracker.ietf.org/doc/rfc9728/)을 따릅니다. 규격을 준수하는 서버는 `/.well-known/oauth-protected-resource` 아래에 인증 엔드포인트를 노출하며, Claude Code는 OAuth 흐름을 자동으로 완료합니다 — 규격 준수 서버에는 수동 `apiKeyHelper` 또는 `headersHelper` 스크립트가 필요 없습니다.

> **예약된 서버 이름(v2.1.128+):** `workspace`, `Claude Browser`, `Claude Preview`는 예약된 MCP 서버 이름입니다. 이 이름을 가진 사용자 정의 서버는 로드 시점에 건너뛰어지며 세션 로그에 경고가 기록됩니다. 충돌을 피하려면 이 이름을 사용하는 기존 서버의 이름을 변경하세요.

> **`.mcp.json` 핫 리로드(v2.1.139):** `/mcp` Reconnect 작업은 이제 재연결 전에 디스크에서 `.mcp.json`을 다시 읽으므로, 서버를 추가하거나 편집하는 데 더 이상 세션 재시작이 필요 없습니다. Claude Code는 서버가 프로젝트 루트를 기준으로 경로를 해석할 수 있도록 stdio로 시작된 MCP 서버 환경에 `CLAUDE_PROJECT_DIR`도 주입합니다(v2.1.139).

> **서버별 타임아웃 하한(v2.1.162):** 1000ms 미만의 서버별 `timeout` 값은 무시되고 전역 `MCP_TOOL_TIMEOUT` 기본값이 대신 적용됩니다. 1000ms 이상 값은 이전처럼 존중됩니다.

### MCP Settings

| Key | Type | Scope | Description |
|-----|------|-------|-------------|
| `enableAllProjectMcpServers` | boolean | Any | 모든 `.mcp.json` 서버를 자동 승인합니다. **보안 참고(v2.1.196):** `.mcp.json` 서버는 더 이상 스스로 승인되지 않습니다 — 이제 `enableAllProjectMcpServers: true` 또는 `enabledMcpjsonServers`를 통한 명시적 옵트인이 필요합니다 |
| `enabledMcpjsonServers` | array | Any | 특정 서버 이름을 허용 목록에 추가 |
| `disabledMcpjsonServers` | array | Any | 특정 서버 이름을 차단 목록에 추가 |
| `allowedMcpServers` | array | Managed only | 이름/명령/URL 매칭이 있는 허용 목록 |
| `deniedMcpServers` | array | Managed only | 매칭이 있는 차단 목록 |
| `allowManagedMcpServersOnly` | boolean | Managed only | managed 허용 목록에 명시적으로 나열된 MCP 서버만 허용 |
| `channelsEnabled` | boolean | Managed only | Team 및 Enterprise 사용자를 위한 [channels](https://code.claude.com/docs/en/channels)를 허용합니다. 미설정 또는 `false`이면 `--channels` 플래그와 관계없이 채널 메시지 전달이 차단됩니다 |
| `allowedChannelPlugins` | array | Managed only | 메시지를 푸시할 수 있는 채널 플러그인의 허용 목록. 설정되면 기본 Anthropic 허용 목록을 대체합니다. Undefined = 기본값으로 폴백, 빈 배열 = 모든 채널 플러그인 차단. `channelsEnabled: true`가 필요합니다. 각 항목은 `marketplace`와 `plugin` 필드를 가진 객체입니다(v2.1.84) |
| `allowAllClaudeAiMcps` | boolean | Managed only | `managed-mcp.json`과 함께 claude.ai 클라우드 MCP 커넥터를 로드합니다. 활성화되면 admin 배포 managed MCP 서버에 더해 claude.ai 호스팅 MCP 커넥터가 사용 가능해집니다 |
| `disableClaudeAiConnectors` | boolean | Any | claude.ai MCP 커넥터의 자동 가져오기를 비활성화합니다. `true`이면 claude.ai 클라우드 커넥터가 로드되지 않습니다. **제한적 값 예외:** `true`는 managed `false`에 대해서도 프로젝트 설정을 포함한 어떤 스코프에서든 적용됩니다 — 우선순위가 낮은 스코프는 옵트아웃할 수 있지만 옵트인할 수는 없습니다(v2.1.182) |

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

> **`${VAR}` 보간(v2.1.219):** `allowedMcpServers`와 `deniedMcpServers` 항목은 `${VAR}` 플레이스홀더를 포함할 수 있습니다. Claude Code는 로드 시점에 시작 환경과 managed 설정 `env` 블록에서 이를 해석합니다. managed 설정에 하드코딩하지 않고 환경별 서버 이름이나 URL을 참조하는 데 사용하세요(예: `{ "serverUrl": "https://${MCP_HOST}/*" }`).

### Per-Server Tool Loading (`alwaysLoad`, v2.1.121)

기본적으로 MCP 도구 정의는 지연됩니다(도구 검색을 통해 필요 시 컨텍스트에 로드됨). `.mcp.json`(또는 인라인 `mcpServers`)의 개별 MCP 서버 항목에 `alwaysLoad: true`를 설정하면 해당 서버를 지연에서 면제합니다 — 그러면 해당 서버의 모든 도구가 `ENABLE_TOOL_SEARCH`와 관계없이 세션 시작 시 사전에 로드됩니다. 모든 서버 유형에서 사용 가능하며; Claude Code v2.1.121+가 필요합니다. 매 턴 필요한 소수의 도구에만 사용하세요 — 각 사전 로드 도구는 대화에 사용될 컨텍스트를 소비합니다.

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

MCP 서버는 도구의 `_meta` 객체에 `"anthropic/alwaysLoad": true`를 포함하여 개별 도구를 always-loaded로 표시할 수도 있습니다 — 서버 도구의 일부만 지연을 우회해야 할 때 유용합니다.

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
| `sandbox.failIfUnavailable` | boolean | `false` | 샌드박스가 활성화되었지만 시작할 수 없을 때 언샌드박스로 실행하는 대신 오류와 함께 종료합니다. 엄격한 샌드박싱을 요구하는 엔터프라이즈 정책에 유용합니다(v2.1.83) |
| `sandbox.autoAllowBashIfSandboxed` | boolean | `true` | 샌드박스일 때 bash를 자동 승인합니다. v2.1.139 기준으로 셸 확장 형태(`$VAR`, `$(cmd)`)가 올바르게 인식되어, 샌드박스 자동 승인이 활성화되면 변수 치환을 포함한 명령이 더 이상 프롬프트로 폴백하지 않습니다 |
| `sandbox.excludedCommands` | array | `[]` | 샌드박스 밖에서 실행할 명령 |
| `sandbox.allowUnsandboxedCommands` | boolean | `true` | `dangerouslyDisableSandbox`를 허용합니다. `false`로 설정하면 탈출구가 완전히 비활성화되고 모든 명령이 샌드박스로 실행되어야 합니다(또는 `excludedCommands`에 있어야 함). 엄격한 샌드박싱을 요구하는 엔터프라이즈 정책에 유용합니다 |
| `sandbox.filesystem.disabled` | boolean | `false` | 네트워크 egress 제어는 유지하면서 파일시스템 격리를 건너뜁니다. `true`이면 샌드박스가 파일 접근을 제한하지 않지만 네트워크 egress는 `sandbox.network.allowedDomains`로 계속 제한됩니다. **사용자 설정, managed 설정, 또는 `--settings`에서만 존중됩니다** — `.claude/settings.json`과 `.claude/settings.local.json`에서는 무시됩니다(v2.1.216) |
| `sandbox.ignoreViolations` | object | `{}` | 명령 패턴을 경로 배열에 매핑 — 위반 경고 억제 *(공식 설정 페이지가 아닌 JSON 스키마에 있음)* |
| `sandbox.enableWeakerNestedSandbox` | boolean | `false` | **(Linux 및 WSL2 전용)** 비특권 Docker 환경을 위한 더 약한 샌드박스 활성화(보안 감소) |
| `sandbox.network.allowUnixSockets` | array | `[]` | **(macOS 전용)** 샌드박스에서 접근 가능한 특정 Unix 소켓 경로. Linux 및 WSL2에서는 seccomp 필터가 소켓 경로를 검사할 수 없으므로 무시됩니다; 대신 `allowAllUnixSockets`를 사용하세요 |
| `sandbox.network.allowAllUnixSockets` | boolean | `false` | 모든 Unix 소켓 허용(`allowUnixSockets`를 재정의). Linux 및 WSL2에서는 `socket(AF_UNIX, ...)` 호출을 차단하는 seccomp 필터를 건너뛰므로 Unix 소켓을 허용하는 유일한 방법입니다 |
| `sandbox.network.allowLocalBinding` | boolean | `false` | localhost 포트 바인딩 허용(macOS) |
| `sandbox.network.allowedDomains` | array | `[]` | 샌드박스용 네트워크 도메인 허용 목록 |
| `sandbox.network.strictAllowlist` | boolean | `false` | `true`이면 `allowedDomains`에 **없는** 호스트로의 모든 네트워크 접근을 사용자에게 프롬프트하지 않고 거부합니다. 기본적으로는 나열되지 않은 호스트가 권한을 프롬프트하는데, 이 설정은 허용 목록을 하드 차단으로 만듭니다. 유효 허용 목록에는 `WebFetch(domain:...)` allow 규칙도 포함됩니다. 샌드박스 bash 명령의 아웃바운드 연결에만 적용됩니다 — Claude Code 자체의 API 및 WebFetch 호출은 영향을 받지 않습니다. **사용자 설정, managed 설정, 또는 `--settings`에서만 존중됩니다** — `.claude/settings.json`과 `.claude/settings.local.json`에서는 무시됩니다(v2.1.219) |
| `sandbox.network.deniedDomains` | array | `[]` | bash 샌드박스용 네트워크 도메인 차단 목록. `allowedDomains`의 와일드카드보다 우선합니다. glob 패턴을 지원합니다(예: `"*.example.com"`)(v2.1.113) |
| `sandbox.network.httpProxyPort` | number | - | HTTP 프록시 포트 1-65535(커스텀 프록시) |
| `sandbox.network.socksProxyPort` | number | - | SOCKS5 프록시 포트 1-65535(커스텀 프록시) |
| `sandbox.network.allowManagedDomainsOnly` | boolean | `false` | managed 허용 목록의 도메인만 허용(managed 설정) |
| `sandbox.network.allowMachLookup` | array | `[]` | (macOS 전용) 샌드박스가 조회할 수 있는 추가 XPC/Mach 서비스 이름. 접두사 매칭을 위한 단일 후행 `*`을 지원합니다. iOS Simulator나 Playwright처럼 XPC를 통해 통신하는 도구에 필요합니다. 예: `["com.apple.coresimulator.*"]` |
| `sandbox.filesystem.allowWrite` | array | `[]` | 샌드박스 명령이 쓸 수 있는 추가 경로. 배열은 모든 설정 스코프에 걸쳐 병합됩니다. `Edit(...)` allow 권한 규칙의 경로와도 병합됩니다. 접두사: `/`(절대), `~/`(홈), `./` 또는 없음(프로젝트 설정에서는 프로젝트 기준, 사용자 설정에서는 `~/.claude` 기준). 오래된 `//` 접두사(절대 경로)도 여전히 작동합니다. **Note:** 이는 절대에 `//`, 프로젝트 기준에 `/`를 사용하는 [Read/Edit 권한 규칙](#tool-permission-syntax)과 다릅니다 |
| `sandbox.filesystem.denyWrite` | array | `[]` | 샌드박스 명령이 쓸 수 없는 경로. 배열은 모든 설정 스코프에 걸쳐 병합됩니다. `Edit(...)` deny 권한 규칙의 경로와도 병합됩니다. `allowWrite`와 동일한 경로 접두사 규칙 |
| `sandbox.filesystem.denyRead` | array | `[]` | 샌드박스 명령이 읽을 수 없는 경로. 배열은 모든 설정 스코프에 걸쳐 병합됩니다. `Read(...)` deny 권한 규칙의 경로와도 병합됩니다. `allowWrite`와 동일한 경로 접두사 규칙 |
| `sandbox.filesystem.allowRead` | array | `[]` | `denyRead` 영역 내에서 읽기 접근을 재허용하는 경로. 매칭되는 `denyRead` glob 항목을 재정의하지만, 정확한 경로의 `denyRead` 규칙은 여전히 정확한 매칭을 차단합니다 — `allowRead`가 모든 `denyRead` 패턴을 보편적으로 이기지는 않습니다. 배열은 모든 설정 스코프에 걸쳐 병합됩니다. `allowWrite`와 동일한 경로 접두사 규칙 |
| `sandbox.filesystem.allowManagedReadPathsOnly` | boolean | `false` | **(Managed 전용)** managed 설정의 `allowRead` 경로만 존중됩니다. 사용자, 프로젝트, 로컬 설정의 `allowRead` 항목은 무시됩니다 |
| `sandbox.enableWeakerNetworkIsolation` | boolean | `false` | (macOS 전용) 시스템 TLS 신뢰(`com.apple.trustd.agent`)에 대한 접근을 허용; 보안 감소 |
| `sandbox.bwrapPath` | string | - | **(Managed 전용, Linux/WSL2)** bubblewrap(`bwrap`) 바이너리의 절대 경로. 자동 `PATH` 감지를 재정의합니다. managed 설정에서만 존중되며, 사용자 또는 프로젝트 설정에서는 안 됩니다. 예: `/opt/admin/bwrap`(v2.1.133) |
| `sandbox.socatPath` | string | - | **(Managed 전용, Linux/WSL2)** 샌드박스 네트워크 프록시에 사용되는 `socat` 바이너리의 절대 경로. 자동 `PATH` 감지를 재정의합니다. managed 설정에서만 존중됩니다. 예: `/opt/admin/socat`(v2.1.133) |
| `sandbox.allowAppleEvents` | boolean | `false` | **(macOS 전용)** 샌드박스 명령이 Apple Events를 보내도록 하는 옵트인. `open`, `osascript`, 또는 Apple Events IPC에 의존하는 브라우저 인증 흐름을 사용하는 도구에 필요합니다. **경고:** 이를 활성화하면 코드 실행 격리가 제거됩니다 — Apple Events는 다른 애플리케이션에서 코드를 실행하는 데 사용될 수 있습니다(v2.1.181) |
| `sandbox.credentials` | object | — | 어떤 자격 증명 파일과 환경 변수가 샌드박스 하위 프로세스 환경에서 차단되는지에 대한 세분화된 제어. 두 배열을 가진 객체: `files`(자격 증명 파일 항목 배열 — 아래 하위 키 참고)와 `envVars`(`{name: string, mode: string, injectHosts?: string[]}` 항목 배열 — `mode`는 `"deny"`(기본값, 변수를 제거) 또는 `"mask"`(플레이스홀더로 치환, 사용자 설정/managed/`--settings`에서만 존중; 같은 변수가 두 mode로 나타나면 `deny`가 우선)이며 `injectHosts`가 값을 나열된 호스트에만 선택적으로 노출). 개별 유효하지 않은 항목은 경고와 함께 제거되고; 유효한 부분집합이 강제됩니다.(v2.1.187; 항목별 객체 형태는 v2.1.191 이후; 항목별 `mode` 및 `injectHosts`는 v2.1.199 이후) |
| `sandbox.credentials.files[].path` | string | — | 보호할 자격 증명 파일의 절대 또는 `~/` 접두사 경로 |
| `sandbox.credentials.files[].mode` | string | `"deny"` | `"deny"`는 파일의 샌드박스 읽기를 차단합니다; `"mask"`는 읽기를 가로채 플레이스홀더 값으로 치환하여 명령이 여전히 실행될 수 있지만 원시 자격 증명을 유출할 수 없게 합니다. **`mask` 모드는 사용자 설정, managed 설정, 또는 `--settings`에서만 존중됩니다**(v2.1.187 deny; v2.1.221 mask) |
| `sandbox.credentials.files[].extract` | string | — | 마스킹을 위해 파일 내 자격 증명 값을 찾는 데 사용되는 단일 캡처 그룹이 있는 정규식. `mode`가 `"mask"`이고 파일이 잘 알려진 형식이 아닐 때 필요합니다. 예: `"oauth_token:\\s*(\\S+)"`(v2.1.221) |
| `sandbox.credentials.files[].injectHosts` | array | — | `mode`가 `"mask"`일 때도 TLS 종료 프록시를 통해 실제 자격 증명 값을 받는 호스트명. `sandbox.network.tlsTerminate`가 구성되어야 합니다(v2.1.199) |
| `sandbox.credentials.files[].onExtractNoMatch` | string | `"warn"` | `extract` 정규식이 아무것도 매칭하지 못할 때의 동작: `"warn"`(경고 로깅, 읽기 허용), `"deny"`(읽기 차단), 또는 `"error"`(샌드박스 명령 중단)(v2.1.221) |
| `sandbox.credentials.files[].maskDuplicates` | boolean | `false` | `true`이면 같은 샌드박스 명령 중에 읽힌 다른 파일과 환경 변수에 나타나는 추출된 자격 증명 값의 발생도 마스킹합니다(v2.1.221) |
| `sandbox.network.tlsTerminate` | object | — | 샌드박스 bash 명령을 위한 TLS 종료 구성. 설정되면 Claude Code가 샌드박스에서 나가는 아웃바운드 HTTPS에 대해 TLS man-in-the-middle 역할을 하여 자격 증명 마스킹(`sandbox.credentials` `mask` 모드)을 가능하게 합니다. 세션용 임시 인증 기관을 생성하려면 `{}`로 설정하거나, 자체 CA를 제공하려면 `caCertPath`와 `caKeyPath`를 설정하세요. **사용자 설정, managed 설정, 또는 `--settings`에서만 존중됩니다** — `.claude/settings.json`과 `.claude/settings.local.json`에서는 무시됩니다. 실험적(v2.1.199) |
| `sandbox.credentials.allowPlaintextInject` | boolean | `false` | `mask` 자격 증명 치환이 **평문 HTTP 요청**(업스트림 신원이 검증되지 않고 자격 증명이 평문으로 이동하는 경우)에도 적용되도록 허용합니다. `false`(기본값)이면 `injectHosts`를 통한 자격 증명 주입이 TLS 종료된 HTTPS 연결로만 제한되어, 자격 증명이 암호화되지 않은 채널로 전송되는 것을 방지합니다. 평문 노출이 허용되는 신뢰할 수 있는 로컬 환경에서만 사용하세요(v2.1.199) |

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
| `strictKnownMarketplaces` | boolean | Managed only | `true`이면 공식 Anthropic 마켓플레이스만 허용됩니다; 추가 또는 커스텀 마켓플레이스를 설치할 수 없습니다 |
| `strictPluginOnlyCustomization` | boolean \| array | Managed only | 사용자 및 프로젝트 소스의 스킬, 에이전트, hook, MCP 서버를 차단하여 플러그인 또는 managed 설정에서만 올 수 있게 합니다. `true`는 네 가지 표면을 모두 잠급니다; `["skills", "hooks"]` 같은 배열은 명명된 것만 잠급니다 |
| `pluginSuggestionMarketplaces` | array | Managed only | 세션 중에 플러그인이 상황별 설치 제안으로 나타날 수 있는 마켓플레이스 이름의 허용 목록. "이 플러그인이 필요할 수 있습니다" 프롬프트를 표면화할 수 있는 마켓플레이스를 제한합니다(v2.1.152) |
| `skippedMarketplaces` | array | Any | 사용자가 설치를 거부한 마켓플레이스 *(공식 설정 페이지가 아닌 JSON 스키마에 있음)* |
| `skippedPlugins` | array | Any | 사용자가 설치를 거부한 플러그인 *(공식 설정 페이지가 아닌 JSON 스키마에 있음)* |
| `pluginConfigs` | object | Managed / User / --settings | 플러그인별 MCP 서버 구성(`plugin@marketplace`로 키 지정). v2.1.207 기준으로 더 이상 프로젝트 레벨 `.claude/settings.json` 또는 `.claude/settings.local.json`에서 읽지 않습니다; 사용자, `--settings`, managed 설정만 존중됩니다 |
| `blockedMarketplaces` | array | Managed only | 특정 플러그인 마켓플레이스를 차단합니다. 각 항목은 소스 문자열, `hostPattern`, 또는 `pathPattern`으로 매칭할 수 있습니다 — v2.1.119 기준으로 `hostPattern`과 `pathPattern` 매처가 다운로드가 파일시스템에 닿기 전에 올바르게 강제되므로, 차단된 마켓플레이스는 절대 디스크에 도달하지 않습니다 |
| `pluginTrustMessage` | string | Managed only | 사용자에게 플러그인 신뢰를 프롬프트할 때 표시되는 커스텀 메시지 |
| `disableSideloadFlags` | boolean | Managed only | `--plugin-dir`, `--plugin-url`, `--agents`, `--mcp-config` 시작 플래그를 거부합니다. `true`이면 사용자가 시작 시 사이드로드 플래그를 전달하여 `strictKnownMarketplaces`를 우회할 수 없습니다. managed 환경에서 마켓플레이스 전용 플러그인 배포를 강제하는 데 사용하세요(v2.1.193) |

**마켓플레이스 소스 유형:** `github`, `git`, `directory`, `settings`, `url`, `npm`, `file`, `archive`. 호스팅된 마켓플레이스 저장소를 설정하지 않고 소수의 플러그인을 인라인으로 선언하려면 `source: 'settings'`를 사용하세요. SHA-256 고정이 있는 zip 기반 플러그인 설치에는 `source: 'archive'`를 사용하세요(v2.1.224). 참고: `hostPattern`은 소스 유형이 아니라 `blockedMarketplaces`용 *매처* 필드입니다.

**소유자 와일드카드 항목(v2.1.223):** `strictKnownMarketplaces`와 `blockedMarketplaces`는 이제 특정 조직 또는 사용자의 모든 저장소와 매칭하기 위한 `"owner/*"` 와일드카드 항목을 허용합니다.

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
| `"opus"` | 최신 Opus 모델(v2.1.219 기준 Anthropic API에서는 **Claude Opus 5**(`claude-opus-5`) — 네이티브 1M 토큰 컨텍스트; v2.1.207 기준 Bedrock, Vertex, AWS의 Claude Platform에서는 Opus 4.8). Opus 5와 Opus 4.8이 fast-mode 모델입니다(v2.1.219에서 Opus 4.7이 fast 모드에서 제거됨). Opus 5는 `ultracode` effort를 지원합니다; Opus 4.8은 기본값이 `high`이고 `xhigh`를 지원합니다 |
| `"haiku"` | 빠른 Haiku 모델 |
| `"sonnet[1m]"` | 1M 토큰 컨텍스트의 Sonnet |
| `"opus[1m]"` | 1M 토큰 컨텍스트의 Opus(v2.1.75 이후 Max, Team, Enterprise에서 기본값) |
| `"opusplan"` | 계획에는 Opus, 실행에는 Sonnet |
| `"fable"` | Claude Fable 5 — 장기 지평 추론 모델. Anthropic API 전용(v2.1.170+). Fable 5는 기본적으로 1M 컨텍스트를 포함합니다; `[1m]` 접미사는 자동으로 제거되므로 `fable[1m]`은 불필요합니다(v2.1.173) |

**Example:**
```json
{
  "model": "opus"
}
```

> **Note (v2.1.144):** `/model`은 **현재 세션에만** 모델을 변경합니다. `/model` 선택기에서 `d`를 누르면 선택을 기본값으로도 설정합니다. `model` 설정과 `ANTHROPIC_MODEL`은 계속 영구 기본값을 제어합니다.

### Model Overrides

Bedrock, Vertex, 또는 Foundry 배포를 위해 Anthropic 모델 ID를 제공자별 모델 ID에 매핑합니다.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `effortLevel` | string | - | 세션 간에 effort 레벨을 유지합니다. `"low"`, `"medium"`, `"high"`, `"xhigh"`(Fable 5, Opus 5, Sonnet 5, Opus 4.7, Opus 4.8, v2.1.111)를 허용합니다. **`"max"`와 `"ultracode"`는 세션 전용이며 여기서 허용되지 않습니다** — `/effort` 또는 `--settings`를 통해 단일 세션에 설정하되 `settings.json`에는 쓰지 마세요. `/effort <level>`을 실행하면 자동으로 기록됩니다. 기본 effort는 effort를 지원하는 모든 모델에서 `high`이며, 예외로 Opus 4.7은 기본값이 `xhigh`입니다. 지원되지 않는 레벨은 활성 모델에서 지원되는 최고 레벨로 폴백됩니다 |
| `fallbackModel` | array | - | 주 모델을 사용할 수 없을 때(예: 속도 제한 또는 용량 문제) 순차적으로 시도되는 최대 3개의 폴백 모델 ID. 각 항목은 모델 ID 또는 별칭입니다; `"default"`는 계정 기본값으로 확장됩니다. Claude Code는 먼저 주 모델을 시도하고; 실패하면 각 폴백을 순서대로 시도합니다. 첫 성공 응답에서 멈춥니다. **대부분의 배열 설정과 달리 이 키는 설정 파일 간에 병합되지 않습니다** — 이를 정의하는 가장 높은 우선순위의 파일이 전체 체인을 제공합니다; (중복 제거 후) 3개를 초과하는 항목은 조용히 무시됩니다(v2.1.166) |
| `modelOverrides` | object | - | 모델 선택기 항목을 제공자별 ID(예: Bedrock inference profile ARN)에 매핑합니다. 각 키는 모델 선택기 항목 이름이고, 각 값은 제공자 모델 ID입니다 |

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

`/model` 명령은 모델이 응답당 적용하는 추론량을 조정하는 **effort level** 제어를 노출합니다. `/model` UI에서 ← → 화살표 키를 사용하여 effort 레벨을 순환하세요.

| Effort Level | Description |
|-------------|-------------|
| Ultracode | 세션 전용: xhigh 추론 깊이에 더해 ultracode 워크플로를 트리거합니다. `/effort ultracode`로 활성화 — **settings.json의 유효한 `effortLevel` 값이 아님**(v2.1.203+) |
| Max | 세션 전용: 지원하는 모델(Fable 5, Opus 5, Sonnet 5, Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 4.6)에서 최대 추론 깊이. **settings.json의 유효한 `effortLevel` 값이 아님** |
| XHigh | 확장된 높은 추론 깊이. Fable 5, Opus 5, Sonnet 5, Opus 4.7, Opus 4.8에서 사용 가능(v2.1.111). 모든 플랜에서 Opus 4.7의 기본값; 다른 지원 모델에서는 기본값이 `high` |
| High(Opus 4.7을 제외한 모든 effort 지원 모델의 기본값) | 전체 추론 깊이, 복잡한 작업에 최적 |
| Medium | 균형 잡힌 추론, 일상 작업에 적합 |
| Low | 최소 추론, 가장 빠른 응답 |

**사용 방법:**
1. `/effort low`, `/effort medium`, 또는 `/effort high`를 실행하여 직접 설정(v2.1.76+)
2. 또는 `/model` 실행 → 모델 선택 → **← →** 화살표 키로 조정
3. 설정은 `settings.json`의 `effortLevel` 키를 통해 유지됩니다

**Note:** effort 레벨은 Max 및 Team 플랜에서 Opus 4.6, Sonnet 4.6, Opus 4.7, Opus 4.8에 사용 가능합니다. 기본값은 v2.1.68에서 High에서 Medium으로 변경되었다가, v2.1.94에서 API 키, Bedrock/Vertex/Foundry, Team, Enterprise 사용자에 대해 **High**로 다시 변경되었습니다. v2.1.117에서는 Opus 4.6과 Sonnet 4.6의 Pro/Max 구독자에 대해서도 기본값이 `medium`에서 `high`로 상향되어 모든 계층이 `high`로 정렬되었습니다. v2.1.111은 **`xhigh`**(당시 Opus 4.7 전용)를 도입하고 모든 플랜에서 Opus 4.7의 기본 effort 레벨로 만들었습니다. **v2.1.154**는 Anthropic API에서 최신 Opus로 **Opus 4.8**을 추가했습니다; `xhigh`를 지원하지만 기본값은 `high`입니다. v2.1.75 기준으로 Opus 4.6의 1M 컨텍스트 윈도는 Max, Team, Enterprise 플랜에서 기본적으로 사용 가능합니다.

**Effort 환경 전파:** 스킬 파일 내에서 `${CLAUDE_EFFORT}`를 사용하여 현재 effort 레벨을 참조하세요(v2.1.120). v2.1.133 기준으로 동일한 `$CLAUDE_EFFORT` 변수가 Bash 도구 하위 프로세스와 hook 핸들러의 환경에도 주입되므로, 셸 스크립트와 hook 명령이 별도 설정 파일을 읽지 않고도 활성 effort 계층에 따라 동작을 조정할 수 있습니다.

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
| `spinnerTipsOverride` | object | - | `tips`(문자열 배열)와 선택적 `excludeDefault`(boolean)를 가진 커스텀 스피너 팁. `excludeDefault`가 `true`이면 커스텀 팁만 표시됩니다; `false`이거나 없으면 커스텀 팁이 내장 팁과 병합됩니다. v2.1.121 기준으로 `excludeDefault: true`는 시간 기반 스피너 팁도 억제합니다 |
| `respectGitignore` | boolean | `true` | 파일 선택기에서 .gitignore 존중 |
| `prefersReducedMotion` | boolean | `false` | UI에서 애니메이션과 모션 효과 감소 |
| `axScreenReader` | boolean | `false` | 스크린 리더 친화적 출력 모드를 활성화합니다. `true`이면 Claude가 장식용 박스 그리기 문자, 색상, 기타 터미널 UI 요소 없이 평문을 출력합니다. `/config`에 **Screen reader mode**로 표시됩니다. User 스코프에만 적용됩니다 — 프로젝트 또는 managed 설정에서 읽지 않습니다. `--ax-screen-reader` CLI 플래그로도 사용할 수 있습니다(v2.1.181) |
| `syntaxHighlightingDisabled` | boolean | `false` | diff, 코드 블록, 파일 미리보기에서 구문 강조를 비활성화합니다. diff 출력만 관장하는 `CLAUDE_CODE_SYNTAX_HIGHLIGHT` 환경 변수와 구별됩니다 |
| `fileSuggestion` | object | - | 커스텀 파일 제안 명령(아래 File Suggestion Configuration 참고) |
| `autoScrollEnabled` | boolean | `true` | 전체 화면 모드에서 대화를 자동 스크롤합니다. 자동 스크롤을 비활성화하려면 `false`로 설정하세요(v2.1.110). v2.1.119 이전 버전은 이를 `~/.claude.json`에 저장했습니다 |
| `editorMode` | string | `"normal"` | 입력 프롬프트의 키 바인딩 모드: `"normal"` 또는 `"vim"`. `/config`에 **Editor mode**로 표시됩니다. v2.1.119 이전 버전은 이를 `~/.claude.json`에 저장했습니다 |
| `vimInsertModeRemaps` | object | - | vim insert 모드에 적용되는 커스텀 키 재매핑. 키가 정확히 두 개의 인쇄 가능한 문자(입력 시퀀스)이고 값이 대체 시퀀스인 객체 — 유일한 유효 대상 값은 `<Esc>`입니다. `editorMode`가 `"vim"`일 때만 적용됩니다. 예: `{"jk": "<Esc>", "jj": "<Esc>"}`(v2.1.208) |
| `showTurnDuration` | boolean | `true` | 응답 후 턴 소요 시간 메시지 표시(예: "Cooked for 1m 6s"). v2.1.119 이전 버전은 이를 `~/.claude.json`에 저장했습니다 |
| `teammateMode` | string | `"in-process"` | [agent team](https://code.claude.com/docs/en/agent-teams) 팀메이트가 표시되는 방식: `"auto"`(tmux 또는 iTerm2에서는 분할 창, 그 외에는 in-process 선택), `"in-process"`(v2.1.179 이후 기본값), `"tmux"`, 또는 `"iterm2"`(자동 감지와 관계없이 iTerm2 분할 창 강제, v2.1.186). [choose a display mode](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode)를 참고하세요. v2.1.119 이전 버전은 이를 `~/.claude.json`에 저장했습니다 |
| `terminalProgressBarEnabled` | boolean | `true` | 지원되는 터미널(ConEmu, Ghostty 1.2.0+, iTerm2 3.6.6+)에서 터미널 진행 표시줄을 표시합니다. `/config`에 **Terminal progress bar**로 표시됩니다. v2.1.119 이전 버전은 이를 `~/.claude.json`에 저장했습니다 |
| `preferredNotifChannel` | string | `"auto"` | 작업 완료 및 권한 프롬프트 알림 방식. 값: `"auto"`, `"terminal_bell"`, `"iterm2"`, `"iterm2_with_bell"`, `"kitty"`, `"ghostty"`, `"notifications_disabled"`. 기본값 `"auto"`는 iTerm2, Ghostty, Kitty에서 데스크톱 알림을 보내고 다른 터미널에서는 아무것도 하지 않습니다. 어떤 터미널에서든 벨 문자를 울리려면 `"terminal_bell"`을 설정하세요. `/config`에 **Notifications**로 표시됩니다. [Get a terminal bell or notification](https://code.claude.com/docs/en/terminal-config#get-a-terminal-bell-or-notification)을 참고하세요 |
| `wheelScrollAccelerationEnabled` | boolean | `true` | 전체 화면 모드에서 마우스 휠 스크롤 가속을 비활성화합니다. OS 레벨 가속 곡선 대신 틱당 고정 스크롤 단계를 사용하려면 `false`로 설정하세요(v2.1.174) |
| `footerLinksRegexes` | array | - | **턴 출력**(도구 결과, 파일 내용, 가져온 페이지, Claude의 응답)에 대해 매칭되어 푸터 행에 링크 배지로 표시되는 객체 배열. 각 항목은 `{type, pattern, url, label}`이며 `pattern`은 명명된 캡처 그룹이 있는 정규식이고 `url`/`label`이 이 그룹을 참조할 수 있습니다. 매칭된 패턴은 채팅 UI 하단에 클릭 가능한 배지를 생성합니다. 턴당 5개 배지로 제한; URL 최대 2048자; 허용 스킴: `http`, `https`, `vscode`, `cursor`, `windsurf`, `zed`, `jetbrains`, `idea`, `slack`, `linear`, `notion`, `figma`, `vscode-insiders`. User/`--settings`/managed 전용(v2.1.176) |
| `emojiCompletionEnabled` | boolean | `true` | 프롬프트 입력에서 이모지 단축코드 자동 완성을 활성화합니다(예: `:tada:` → 🎉). 비활성화하려면 `false`로 설정하세요. v2.1.217+ 필요 |

### Global Config Settings (`~/.claude.json`)

이 IDE 관련 환경 설정은 `settings.json`이 **아니라** `~/.claude.json`에 저장됩니다.

> **v2.1.119 마이그레이션 참고:** v2.1.119 기준으로 `autoScrollEnabled`, `editorMode`, `showTurnDuration`, `teammateMode`, `terminalProgressBarEnabled`가 `settings.json`으로 이동했으며 위 Display Settings 표에 문서화되어 있습니다. 이전 버전은 이를 여기에 저장했습니다.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `autoConnectIde` | boolean | `false` | 외부 터미널에서 Claude Code가 시작될 때 실행 중인 IDE에 자동 연결합니다. VS Code 또는 JetBrains 터미널 밖에서 실행 시 `/config`에 **Auto-connect to IDE (external terminal)**로 표시됩니다 |
| `autoInstallIdeExtension` | boolean | `true` | VS Code 터미널에서 실행할 때 Claude Code IDE 확장을 자동 설치합니다. `/config`에 **Auto-install IDE extension**으로 표시됩니다. `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` 환경 변수로도 비활성화할 수 있습니다 |
| `externalEditorContext` | boolean | `false` | `Ctrl+G`로 외부 편집기를 열 때 Claude의 이전 응답을 `#` 주석 처리된 컨텍스트로 앞에 붙입니다. 활성화하려면 `true`로 설정하세요 |
| `teammateDefaultModel` | string | `null` | 리드가 디스패치할 때 [agent-team](https://code.claude.com/docs/en/agent-teams) 팀메이트의 기본 모델. `null`은 리드의 모델을 상속합니다. 공식 설정 페이지의 "Global config settings" 아래에 나열됩니다 |
| `diffTool` | string | - | 파일 diff를 볼 때 호출되는 외부 diff 도구 명령. 설정되면 Claude Code가 내장 diff 보기를 렌더링하는 대신 두 파일 경로를 인수로 이 명령을 스폰합니다 |
| `permissionExplainerEnabled` | boolean | `true` | 권한 프롬프트와 함께 왜 권한이 요청되는지에 대한 AI 생성 자연어 설명을 표시합니다. 설명을 억제하고 원시 도구 호출만 표시하려면 `false`로 설정하세요 |

### Workspace & Teams

| Key | Type | Description |
|-----|------|-------------|
| `sshConfigs` | object[] | Desktop에서 드롭다운으로 표면화되는 SSH 연결 정의. 각 항목은 `id`, `name`, `sshHost`를 포함해야 하며; 선택적으로 `sshPort`, `sshIdentityFile`, `startDirectory`를 포함합니다. managed 및 사용자 설정에서만 존중됩니다 — 프로젝트 설정 값은 무시됩니다 |

**필드 레퍼런스:**

| Field | Required | Description |
|-------|----------|-------------|
| `id` | yes | SSH 연결 항목의 고유 식별자 |
| `name` | yes | Desktop 드롭다운에 표시되는 이름 |
| `sshHost` | yes | SSH 호스트(예: `user@dev.example.com` 또는 `dev.example.com`) |
| `sshPort` | no | SSH 포트 번호 |
| `sshIdentityFile` | no | SSH identity 파일(개인 키) 경로 |
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
| `padding` | 상태 표시줄 콘텐츠에 추가되는 추가 수평 간격(문자 단위). 기본값 `0`. 인터페이스 내장 간격을 넘는 상대 들여쓰기를 제어 |
| `refreshInterval` | 이벤트 기반 업데이트에 더해 N초마다 명령을 재실행합니다. 최소값은 `1`입니다. 상태 표시줄이 시간 기반 데이터(예: 시계)를 표시하거나 백그라운드 서브에이전트가 메인 세션이 유휴일 때 git 상태를 변경할 때 유용합니다. 이벤트 시에만 실행하려면 미설정으로 두세요(v2.1.97) |
| `hideVimModeIndicator` | `true`이면 커스텀 상태 표시줄 명령이 대신 렌더링할 수 있도록 상태 영역에서 내장 vim 모드 표시기를 억제합니다. `editorMode`가 `"vim"`일 때만 효과가 있습니다 |

**Status Line 입력 필드:**

상태 표시줄 명령은 stdin에서 JSON 객체를 받습니다. 전체 JSON 스키마와 예제는 [Status Line Documentation](https://code.claude.com/docs/en/statusline)을 참고하세요.

| Field | Description |
|-------|-------------|
| `model.id`, `model.display_name` | 현재 모델 식별자와 표시 이름 |
| `cwd`, `workspace.current_dir` | 현재 작업 디렉터리(둘 다 같은 값을 포함; `workspace.current_dir` 선호) |
| `workspace.project_dir` | Claude Code가 시작된 디렉터리(작업 디렉터리가 변경되면 `cwd`와 다를 수 있음) |
| `workspace.added_dirs` | `/add-dir` 또는 `--add-dir`로 추가된 추가 디렉터리 |
| `workspace.git_worktree` | `git worktree add`로 생성된 링크된 worktree 내부일 때의 Git worktree 이름. 메인 작업 트리에서는 없음(v2.1.97) |
| `cost.total_cost_usd` | 총 세션 비용(USD) |
| `cost.total_duration_ms` | 세션 시작 이후 총 실제 경과 시간(밀리초) |
| `cost.total_api_duration_ms` | API 응답 대기에 소요된 총 시간(밀리초) |
| `cost.total_lines_added`, `cost.total_lines_removed` | 세션 중 변경된 코드 라인 수 |
| `context_window.total_input_tokens`, `context_window.total_output_tokens` | 세션 전반의 누적 토큰 수 |
| `context_window.context_window_size` | 최대 컨텍스트 윈도 크기(토큰 단위, 기본 200000, 확장 컨텍스트는 1000000) |
| `context_window.used_percentage` | 사전 계산된 컨텍스트 윈도 사용 비율 |
| `context_window.remaining_percentage` | 사전 계산된 컨텍스트 윈도 잔여 비율 |
| `context_window.current_usage` | 마지막 API 호출의 토큰 수(input, output, cache 토큰) |
| `exceeds_200k_tokens` | 가장 최근 API 응답의 총 토큰이 200k를 초과하는지 여부(고정 임계값) |
| `rate_limits.five_hour.used_percentage` | 5시간 속도 제한 사용 비율(v2.1.80+) |
| `rate_limits.five_hour.resets_at` | 5시간 속도 제한 리셋 타임스탬프(Unix epoch 초) |
| `rate_limits.seven_day.used_percentage` | 7일 속도 제한 사용 비율 |
| `rate_limits.seven_day.resets_at` | 7일 속도 제한 리셋 타임스탬프(Unix epoch 초) |
| `session_id` | 고유 세션 식별자 |
| `session_name` | `--name` 또는 `/rename`으로 설정한 커스텀 세션 이름. 커스텀 이름이 없으면 없음 |
| `transcript_path` | 대화 트랜스크립트 파일 경로 |
| `version` | Claude Code 버전 |
| `output_style.name` | 현재 출력 스타일 이름 |
| `vim.mode` | vim 모드가 활성화된 경우 현재 vim 모드(`NORMAL` 또는 `INSERT`) |
| `agent.name` | `--agent` 플래그 또는 에이전트 설정으로 실행 시 에이전트 이름 |
| `effort.level` | 현재 추론 effort(`low`, `medium`, `high`, `xhigh`, 또는 `max`). 세션 중 `/effort` 변경을 포함한 실시간 세션 값을 반영합니다. 현재 모델이 effort 매개변수를 지원하지 않으면 없음(v2.1.121) |
| `thinking.enabled` | 세션에 대해 확장 사고가 활성화되어 있는지 여부(v2.1.121) |
| `worktree.name` | 활성 worktree 이름(`--worktree` 세션 중에만 존재) |
| `worktree.path` | worktree 디렉터리의 절대 경로 |
| `worktree.branch` | worktree의 Git 브랜치 이름. hook 기반 worktree에서는 없음 |
| `worktree.original_cwd` | worktree에 진입하기 전의 디렉터리 |
| `worktree.original_branch` | worktree에 진입하기 전 체크아웃된 Git 브랜치. hook 기반 worktree에서는 없음 |
| `github` | 감지된 경우 현재 브랜치의 GitHub 저장소 및 pull request 정보 — 저장소 신원과 연관된 PR(v2.1.145) |

### File Suggestion Configuration

파일 제안 스크립트는 stdin에서 JSON 객체(예: `{"query": "src/comp"}`)를 받고 최대 15개의 파일 경로(한 줄에 하나씩)를 출력해야 합니다.

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
| `awsCredentialExport` | string | AWS 자격 증명이 담긴 JSON을 출력하는 스크립트 |

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

모든 Claude Code 세션에 대한 환경 변수를 설정합니다.

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
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude.ai 인증용 OAuth 액세스 토큰. SDK 및 자동화 환경에서 `/login`의 대안. 키체인에 저장된 자격 증명보다 우선합니다 |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN` | Claude.ai 인증용 OAuth 리프레시 토큰. 설정되면 `claude auth login`이 브라우저를 열지 않고 이 토큰을 직접 교환합니다. `CLAUDE_CODE_OAUTH_SCOPES`가 필요합니다 |
| `CLAUDE_CODE_OAUTH_SCOPES` | 리프레시 토큰이 발급된 공백 구분 OAuth 스코프(예: `"user:profile user:inference user:sessions:claude_code"`). `CLAUDE_CODE_OAUTH_REFRESH_TOKEN`이 설정되면 필요합니다 |
| `ANTHROPIC_WORKSPACE_ID` | [workload identity federation](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation)용 workspace ID. federation 규칙이 둘 이상의 workspace로 스코프될 때 토큰 교환이 대상 workspace를 알 수 있도록 설정합니다(v2.1.141) |
| `ANTHROPIC_BASE_URL` | 커스텀 API 엔드포인트 |
| `ANTHROPIC_BEDROCK_BASE_URL` | Bedrock 엔드포인트 URL 재정의 |
| `ANTHROPIC_BEDROCK_MANTLE_BASE_URL` | Bedrock Mantle 엔드포인트 URL 재정의. [Mantle endpoint](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) 참고 |
| `ANTHROPIC_BEDROCK_SERVICE_TIER` | Bedrock 서비스 계층: `default`, `flex`, 또는 `priority`. 모든 요청에 `X-Amzn-Bedrock-Service-Tier` 헤더로 전송됩니다. [Amazon Bedrock service tiers](https://code.claude.com/docs/en/amazon-bedrock#service-tiers) 참고(v2.1.122) |
| `ANTHROPIC_AWS_API_KEY` | AWS의 Claude Platform용 workspace API 키 |
| `ANTHROPIC_AWS_BASE_URL` | AWS의 Claude Platform 엔드포인트 URL 재정의 |
| `ANTHROPIC_AWS_WORKSPACE_ID` | AWS의 Claude Platform용 필수 workspace ID |
| `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST` | Claude Code를 임베딩하고 사용자 대신 모델 제공자 라우팅을 관리하는 호스트 플랫폼에 의해 설정됩니다. 설정되면 `settings.json`의 제공자 선택/엔드포인트/인증 환경 변수(예: `CLAUDE_CODE_USE_BEDROCK`, `ANTHROPIC_BASE_URL`, `ANTHROPIC_API_KEY`)가 무시되어 사용자 설정이 호스트의 라우팅을 재정의할 수 없습니다. Bedrock/Vertex/Foundry에 대한 자동 텔레메트리 옵트아웃도 건너뛰어지므로, 텔레메트리는 표준 `DISABLE_TELEMETRY` 옵트아웃을 따릅니다(v2.1.126) |
| `ANTHROPIC_VERTEX_BASE_URL` | Vertex AI 엔드포인트 URL 재정의 |
| `ANTHROPIC_BETAS` | 쉼표로 구분된 Anthropic beta 헤더 값 |
| `ANTHROPIC_VERTEX_PROJECT_ID` | Vertex AI용 GCP 프로젝트 ID |
| `GCLOUD_PROJECT` | Vertex AI 요청용 GCP 프로젝트 ID(`ANTHROPIC_VERTEX_PROJECT_ID`를 재정의) |
| `GOOGLE_APPLICATION_CREDENTIALS` | Vertex AI 인증용 GCP 서비스 계정 자격 증명 파일 경로 |
| `GOOGLE_CLOUD_PROJECT` | Vertex AI 요청용 GCP 프로젝트 ID(`ANTHROPIC_VERTEX_PROJECT_ID`를 재정의) |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` | `/model` 선택기에 커스텀 항목으로 추가할 모델 ID. 내장 별칭을 대체하지 않고 비표준 또는 게이트웨이별 모델을 선택 가능하게 하는 데 사용 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_NAME` | `/model` 선택기의 커스텀 모델 항목 표시 이름. 설정되지 않으면 모델 ID로 기본 설정됩니다 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_DESCRIPTION` | `/model` 선택기의 커스텀 모델 항목 표시 설명. 설정되지 않으면 `Custom model (<model-id>)`로 기본 설정됩니다 |
| `ANTHROPIC_CUSTOM_MODEL_OPTION_SUPPORTED_CAPABILITIES` | 커스텀 모델 항목의 기능 감지를 재정의합니다. 쉼표로 구분된 값(예: `effort,thinking`). 커스텀 모델이 자동 감지가 확인할 수 없는 기능을 지원할 때 필요합니다. [model configuration](https://code.claude.com/docs/en/model-config#customize-pinned-model-display-and-capabilities) 참고 |
| `ANTHROPIC_MODEL` | 사용할 모델 이름. 별칭(`sonnet`, `opus`, `haiku`) 또는 전체 모델 ID를 허용합니다. `model` 설정을 재정의합니다 |
| `INIT_PROMPT` | 세션 초기화 시 주입되는 커스텀 시스템 프롬프트 |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Haiku 모델 별칭을 커스텀 모델 ID로 재정의(예: 서드파티 배포용) |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_NAME` | Bedrock/Vertex/Foundry에서 고정 모델을 사용할 때 `/model` 선택기의 Haiku 항목 레이블을 커스터마이징합니다. 기본값은 모델 ID |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_DESCRIPTION` | `/model` 선택기의 Haiku 항목 설명을 커스터마이징합니다. 기본값은 `Custom model (<model-id>)` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL_SUPPORTED_CAPABILITIES` | 고정 Haiku 모델의 기능 감지를 재정의합니다. 쉼표로 구분된 값(예: `effort,thinking`). 고정 모델이 자동 감지가 확인할 수 없는 기능을 지원할 때 필요합니다 |
| `CLAUDE_PID` | 읽기 전용. Claude Code가 스폰하는 모든 하위 프로세스(Bash 도구, hook, 상태 표시줄 명령, MCP stdio 서버)에서 Claude Code 프로세스의 PID로 설정됩니다. 부모 Claude Code 프로세스에 신호를 보내거나 Claude Code 계보를 감지하는 스크립트에 사용하세요. v2.1.214에서 도입 |
| `CLAUDECODE` | Claude Code가 스폰하는 셸 환경(Bash 도구, tmux 세션)에서 `1`로 설정됩니다. hook이나 상태 표시줄 명령에서는 설정되지 않습니다. 스크립트가 Claude Code 셸 내부에서 실행 중인지 감지하는 데 사용하세요 |
| `CLAUDE_CODE_CHILD_SESSION` | Claude Code가 Bash, PowerShell, Monitor 도구, hook 명령, 상태 표시줄 명령을 통해 스폰하는 하위 프로세스에서 `1`로 설정됩니다. stdio MCP 서버 하위 프로세스에는 설정되지 않습니다. `CLAUDECODE`와 달리 이것은 Claude Code 자체의 스폰 경로에서만 설정되므로(IDE 확장이 아님), IDE 통합 터미널에서 시작된 최상위 `claude`와 중첩된 `claude` 세션을 안정적으로 구별합니다. 중첩된 대화형 TUI 세션은 `--resume`, `--continue`, 위쪽 화살표 기록, `claude agents`에서 자동으로 제외됩니다. 비대화형 `claude -p` 세션은 여전히 유지됩니다. 이 제외를 재정의하려면 `CLAUDE_CODE_FORCE_SESSION_PERSISTENCE=1`을 설정하세요(v2.1.172) |
| `CLAUDE_CODE_FORCE_SESSION_PERSISTENCE` | 중첩된 대화형 TUI 세션을 `--resume`, `--continue`, 위쪽 화살표 기록, `claude agents`에서 자동 제외하는 것을 재정의하려면 `1`로 설정하세요. 기본적으로 중첩 세션(`CLAUDE_CODE_CHILD_SESSION=1`인 경우)은 기록을 오염시키지 않도록 제외됩니다. 중첩 세션을 추적하려면 이를 설정하여 유지를 강제하세요 |
| `CLAUDE_CODE_SESSION_ID` | 읽기 전용. Bash 및 PowerShell 도구 하위 프로세스에서 현재 세션 ID로 자동 설정됩니다. hook에 전달되는 `session_id` 필드와 일치합니다. `/clear` 시 업데이트됩니다. 스크립트와 외부 도구를 이를 시작한 Claude Code 세션과 상관시키는 데 사용하세요(v2.1.132). `--resume` 시 stdio MCP 서버 환경에도 주입됩니다(v2.1.163 changelog) *(v2.1.163 changelog에 있음; 공식 env-vars 페이지에는 아직 없음 — 읽기 전용)* |
| `AI_AGENT` | 하위 프로세스 환경(Bash 도구, hook, MCP stdio 서버)에서 Claude Code에 의해 자동 설정됩니다. 부모 프로세스를 AI 에이전트로 식별하는 일반 플래그 — `CLAUDECODE` 같은 에이전트별 변수를 각각 확인하는 대신 어떤 AI 에이전트에서 호출되든 동작을 조정하는 도구에 유용 *(v2.1.120 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS` | 네트워크 오류로 조직 상태 검사가 실패할 때 fast 모드를 허용하려면 `1`로 설정하세요. 기업 프록시가 상태 엔드포인트를 차단할 때 유용합니다 |
| `CLAUDE_CODE_USE_BEDROCK` | AWS Bedrock 사용(`1`로 활성화) |
| `CLAUDE_CODE_USE_VERTEX` | Google Vertex AI 사용(`1`로 활성화) |
| `CLAUDE_CODE_USE_FOUNDRY` | Microsoft Foundry 사용(`1`로 활성화) |
| `CLAUDE_CODE_USE_MANTLE` | Bedrock [Mantle endpoint](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) 사용(`1`로 활성화) |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | Windows에서 PowerShell 도구를 활성화하려면 `1`로 설정하세요(옵트인 프리뷰). 활성화되면 Claude가 Git Bash를 통해 라우팅하는 대신 PowerShell 명령을 네이티브로 실행할 수 있습니다. 네이티브 Windows에서만 지원되며 WSL에서는 안 됩니다(v2.1.84) |
| `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY` | 도구 호출, hook, 상태 표시줄 명령에 PowerShell을 스폰할 때 Claude Code가 `-ExecutionPolicy Bypass`를 전달하는 것을 중단하고 대신 머신의 유효 실행 정책을 존중하려면 `1`로 설정하세요. 기본적으로 Claude Code는 기본 Restricted Windows에서 `.ps1` 스크립트와 모듈 가져오기가 작동하도록 프로세스 스코프에서 실행 정책을 우회합니다. Group Policy `MachinePolicy`/`UserPolicy`는 절대 재정의하지 않습니다(v2.1.143) |
| `CLAUDE_CODE_REMOTE` | 읽기 전용. Claude Code가 클라우드 세션으로 실행될 때 `true`로 자동 설정됩니다. hook 또는 설정 스크립트에서 이를 읽어 클라우드 환경에 있는지 감지하세요 |
| `CLAUDE_CODE_REMOTE_SESSION_ID` | 읽기 전용. 클라우드 세션에서 현재 세션의 ID로 자동 설정됩니다. 세션 트랜스크립트로의 링크를 구성하려면 이를 읽으세요 |
| `CLAUDE_CODE_BRIDGE_SESSION_ID` | 읽기 전용. 세션에 활성 Remote Control 연결이 있는 동안 Bash 도구 및 hook 명령 하위 프로세스에서 자동 설정됩니다. 값은 `session_` 형태의 세션 ID입니다(세션의 `claude.ai/code` URL과 동일한 식별자). 연결이 끝나면 제거됩니다. 스크립트가 이를 실행한 세션으로 다시 링크할 수 있게 합니다. 클라우드 세션에서는 대신 `CLAUDE_CODE_REMOTE_SESSION_ID`를 읽으세요(v2.1.199) |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | 자동 생성된 Remote Control 세션 이름의 접두사. 기본값은 머신 호스트명 |
| `CLAUDE_CLIENT_PRESENCE_FILE` | 존재할 때 활성 클라이언트를 신호하고 Remote Control의 모바일 푸시 알림을 억제하는 파일 경로. 데스크톱 클라이언트가 항상 실행 중이고 모바일 핑을 원하지 않는 환경에서 유용합니다 |
| `CLAUDE_CODE_DISABLE_NOTIFICATION_PRESENCE_CHECK` | 사용자가 세션에 활발히 입력하는 동안에도 푸시 알림을 보내려면 `1`로 설정하세요. 기본적으로 활성 사용자 존재가 감지되면 알림이 억제됩니다(v2.1.193) |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | 텔레메트리 활성화/비활성화(`0` 또는 `1`) |
| `DISABLE_ERROR_REPORTING` | 오류 보고 비활성화(`1`로 비활성화) |
| `DISABLE_AUTOUPDATER` | npm 레지스트리에 대한 자동 업데이트 검사를 비활성화하려면 `1`로 설정하세요. 시작 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) 참고 |
| `DISABLE_UPDATES` | 모든 업데이트 경로를 완전히 차단하려면 `1`로 설정하세요 — 자동 검사, 알림, 수동 `claude update`. 백그라운드 검사만 비활성화하는 `DISABLE_AUTOUPDATER`보다 엄격합니다. 명시적으로 다시 활성화될 때까지 모든 업데이트를 차단해야 하는 환경에서 사용하세요 *(v2.1.118 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE` | 새 버전이 있을 때 Claude Code가 백그라운드에서 패키지 관리자의 업그레이드 명령을 실행하도록 하려면 `1`로 설정하세요. Homebrew 및 WinGet 설치에 적용됩니다. 다른 패키지 관리자는 실행하지 않고 계속 업그레이드 명령을 표시합니다. [Auto updates](https://code.claude.com/docs/en/setup#auto-updates) 참고(v2.1.129) |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` | `ANTHROPIC_BASE_URL`이 LiteLLM, Kong, 또는 내부 프록시 같은 Anthropic 호환 게이트웨이를 가리킬 때 게이트웨이의 `/v1/models` 엔드포인트에서 `/model` 선택기를 채우려면 `1`로 설정하세요. 공유 API 키로 뒷받침되는 게이트웨이는 키가 접근할 수 있는 모든 모델을 노출하므로 기본적으로 꺼져 있습니다. 발견된 모델은 여전히 `availableModels` 허용 목록으로 필터링됩니다(v2.1.129, 이전 자동 발견에서 옵트인 방식으로 변경) |
| `DISABLE_TELEMETRY` | 텔레메트리 비활성화(`1`로 비활성화) |
| `DO_NOT_TRACK` | 표준 옵트아웃 변수; 텔레메트리 수집에서 옵트아웃하려면 `1`로 설정하세요. `DISABLE_TELEMETRY`에 의해 존중됩니다 |
| `MCP_TIMEOUT` | MCP 시작 타임아웃(ms) |
| `MCP_CONNECT_TIMEOUT_MS` | MCP 서버 TCP/Unix 연결 단계의 타임아웃(ms)(기본값: `5000`). `MCP_TIMEOUT`(시작) 및 `MCP_TOOL_TIMEOUT`(호출별)과 구별됩니다 |
| `MCP_SERVER_CONNECTION_BATCH_SIZE` | 시작 시 병렬로 연결할 로컬(stdio) MCP 서버 수. 서버가 많은 머신의 시작 부하를 조절하는 데 사용 |
| `MCP_REMOTE_SERVER_CONNECTION_BATCH_SIZE` | 시작 시 병렬로 연결할 원격(HTTP/SSE) MCP 서버 수. 네트워크 지연을 고려하여 로컬 배치 크기와 분리됨 |
| `CLAUDE_CODE_MCP_ALLOWLIST_ENV` | 안전한 기준 환경만으로 stdio MCP 서버를 스폰하여, 신뢰할 수 없는 서버 프로세스로의 자격 증명 누출을 방지하기 위해 대부분의 상속된 환경 변수를 제거합니다 |
| `MAX_MCP_OUTPUT_TOKENS` | 최대 MCP 출력 토큰(기본값: 25000). 출력이 10,000 토큰을 초과하면 경고 표시 |
| `API_TIMEOUT_MS` | API 요청 타임아웃(ms)(기본값: 600000) |
| `API_FORCE_IDLE_TIMEOUT` | 스트리밍 연결의 5분 유휴 타임아웃을 재정의합니다. 유휴 타임아웃을 완전히 비활성화하려면 `0`, 모든 연결에 강제하려면 `1`, 또는 기본값(자주 멈추는 느리거나 불안정한 게이트웨이에서 자동 활성화)을 위해 미설정으로 두세요. 느린 API 게이트웨이에 유용합니다(v2.1.169) |
| `CLAUDE_CODE_CONNECT_TIMEOUT_MS` | **v2.1.186에서 제거됨.** 이 변수는 no-op입니다. 대신 `API_TIMEOUT_MS`를 사용하세요. 이전에는 스트리밍 API 요청의 연결, TLS, 응답 헤더 단계 타임아웃을 제어했습니다 |
| `CLAUDE_AFK_TIMEOUT_MS` | `askUserQuestionTimeout`이 duration으로 설정된 경우, 응답되지 않은 AskUserQuestion 대화 상자가 자동 계속되기까지의 밀리초. v2.1.200 기준으로 기본값은 `askUserQuestionTimeout`이 제어하는 `"never"`(자동 계속 안 함)입니다; 이 환경 변수는 타임아웃 duration이 활성일 때만 적용됩니다. `0`으로 설정하면 대화 상자가 즉시 닫힙니다. 질문을 열어두려면 `askUserQuestionTimeout: "never"`를 선호하세요(v2.1.198; 기본값 v2.1.200에서 변경) |
| `CLAUDE_AFK_COUNTDOWN_MS` | 응답되지 않은 AskUserQuestion 대화 상자에서 자동 계속 전 화면 카운트다운이 나타나기까지의 밀리초(기본값: `20000` / 20초)(v2.1.198) |
| `BASH_MAX_TIMEOUT_MS` | Bash 명령 타임아웃 |
| `BASH_MAX_OUTPUT_LENGTH` | 최대 bash 출력 길이 |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 자동 압축 임계값 비율(1-100). 기본값은 ~95%. 압축을 더 일찍 트리거하려면 더 낮게 설정하세요(예: `50`). 95%를 초과하는 값은 효과가 없습니다. 현재 사용량을 모니터링하려면 `/context`를 사용하세요. 예: `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50 claude` |
| `CLAUDE_CODE_MAX_CONTEXT_TOKENS` | Claude Code가 활성 모델에 대해 가정하는 컨텍스트 윈도 크기를 재정의합니다. `DISABLE_COMPACT`도 설정된 경우에만 적용됩니다. `ANTHROPIC_BASE_URL`을 통해 이름에 대한 내장 크기와 컨텍스트 윈도가 일치하지 않는 모델로 라우팅할 때 사용하세요 |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | 각 Bash 또는 PowerShell 명령 후 **원래 작업 디렉터리로 돌아가려면** `1`로 설정하세요. 기본적으로 명령 내부에서 수행한 디렉터리 변경은 다음 명령까지 유지됩니다 — 이 변수는 각 호출 후 원래 cwd를 복원합니다 |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | 백그라운드 작업 비활성화(`1`로 비활성화) |
| `CLAUDE_CODE_DISABLE_BG_SHELL_PRESSURE_REAP` | 유휴 백그라운드 셸 명령의 메모리 압박 자동 회수를 비활성화하려면 `1`로 설정하세요. 미설정 시 Claude Code는 리소스를 확보하기 위해 메모리 압박 하에서 유휴 백그라운드 셸을 자동으로 회수합니다(v2.1.193 changelog, 공식 env-vars 페이지에는 아직 없음) |
| `CLAUDE_CODE_DISABLE_BG_EXIT_HANDOFF` | 슈퍼바이저가 세션 프로세스를 중지, 재시작, 또는 업데이트할 때 백그라운드 세션의 실행 중인 백그라운드 셸 명령, 동적 워크플로, 백그라운드 서브에이전트를 중지하려면 `1`로 설정하세요. 기본적으로 이들은 세션의 다음 프로세스로 넘겨집니다(v2.1.198) |
| `CLAUDE_CODE_DISABLE_ADVISOR_TOOL` | advisor 도구와 `/advisor` 명령을 비활성화하려면 `1`로 설정하세요. advisor 사용을 생략하는 것의 환경 변수 등가물. advisor 구성을 위해 `advisorModel`과 함께 사용하세요(최소 v2.1.98) |
| `CLAUDE_CODE_DISABLE_AGENT_VIEW` | 백그라운드 에이전트와 agent view(`claude agents`, `--bg`, `/background`, 온디맨드 슈퍼바이저)를 끄려면 `1`로 설정하세요. `disableAgentView` 설정의 환경 변수 등가물 *(공식 설정 페이지에서 참조됨; env-vars 페이지에는 나열되지 않음)* |
| `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS` | 내장 Explore 및 Plan 서브에이전트를 비활성화하려면 `1`로 설정하세요. Claude가 대신 검색 도구 또는 general-purpose 서브에이전트로 탐색합니다. Plan 모드는 Explore 및 Plan 에이전트를 시작하는 대신 파일을 직접 읽습니다(v2.1.198) |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | 실험적 agent teams 기능 활성화(`1`로 활성화). 세션 내에서 조율된 서브에이전트 팀을 스폰할 수 있게 합니다. 시작 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) 참고 |
| `CLAUDE_CODE_DISABLE_WORKFLOWS` | [동적 워크플로](https://code.claude.com/docs/en/workflows)(`/workflows`)와 번들 워크플로 슬래시 명령을 비활성화하려면 `1`로 설정하세요. `disableWorkflows` 설정의 환경 변수 등가물 |
| `CLAUDE_CODE_ENABLE_AUTO_MODE` | **v2.1.207 기준으로 auto 모드가 Bedrock, Vertex AI, Foundry에서 기본적으로 사용 가능합니다 — 이 옵트인은 더 이상 필요하지 않습니다.** 이전(v2.1.158–v2.1.206): 해당 제공자에서 [auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode)를 사용 가능하게 하려면 `1`로 설정. auto 모드를 끄려면 설정에서 `disableAutoMode: "disable"`을 사용하세요 |
| `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` | Claude Code의 내장 기능(번들 스킬)을 모델로부터 숨기려면 `1`로 설정하세요. `disableBundledSkills` 설정의 환경 변수 등가물(v2.1.169) |
| `CLAUDE_CODE_DISABLE_ARTIFACT` | 세션 출력을 claude.ai의 비공개 웹 페이지로 게시하는 [Artifact](https://code.claude.com/docs/en/artifacts) 도구를 비활성화하려면 `1`로 설정하세요. `disableArtifact` 설정과 동일합니다 |
| `CLAUDE_CODE_ARTIFACT_AUTO_OPEN` | 새 아티팩트가 게시될 때 Claude Code가 브라우저를 자동으로 여는 것을 중단하려면 `0`으로 설정하세요. 기존 아티팩트를 다시 게시하는 것은 이 설정과 관계없이 브라우저를 열지 않습니다 |
| `ENABLE_TOOL_SEARCH` | MCP 도구 검색 임계값(예: `auto:5`) |
| `ENABLE_PROMPT_CACHING_1H` | 1시간 프롬프트 캐시 TTL 옵트인. 더 이상 사용되지 않는 `ENABLE_PROMPT_CACHING_1H_BEDROCK`을 대체 *(v2.1.108 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `FORCE_PROMPT_CACHING_5M` | 5분 프롬프트 캐시 TTL 강제 *(v2.1.108 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` | away summary / 유휴 세션 요약에서 옵트아웃. 비활성화하려면 `0`으로 설정. `awaySummaryEnabled` 설정과 함께 사용됩니다(v2.1.110) |
| `DISABLE_PROMPT_CACHING` | 모든 프롬프트 캐싱 비활성화(`1`로 비활성화) |
| `DISABLE_PROMPT_CACHING_HAIKU` | Haiku 프롬프트 캐싱 비활성화 |
| `DISABLE_PROMPT_CACHING_SONNET` | Sonnet 프롬프트 캐싱 비활성화 |
| `DISABLE_PROMPT_CACHING_OPUS` | Opus 프롬프트 캐싱 비활성화 |
| `ENABLE_PROMPT_CACHING_1H_BEDROCK` | Bedrock에서 1시간 캐시 TTL 요청(`1`로 활성화) *(공식 문서에 없음 — 미검증; v2.1.108 changelog는 더 이상 사용되지 않으며 `ENABLE_PROMPT_CACHING_1H`으로 대체되었다고 함)* |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | 실험적 beta 기능 비활성화(`1`로 비활성화) |
| `CLAUDE_CODE_SHELL` | 자동 셸 감지 재정의 |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | 기본 파일 읽기 토큰 한계 재정의 |
| `CLAUDE_CODE_GLOB_HIDDEN` | Claude가 Glob 도구를 호출할 때 결과에서 dotfile을 제외하려면 `false`로 설정하세요. 기본적으로 포함됩니다. `@` 파일 자동 완성, `ls`, Grep, Read에는 영향을 주지 않습니다 |
| `CLAUDE_CODE_GLOB_NO_IGNORE` | Glob 도구가 `.gitignore` 패턴을 존중하게 하려면 `false`로 설정하세요. 기본적으로 Glob은 gitignore된 것을 포함한 모든 매칭 파일을 반환합니다. 자체 `respectGitignore` 설정이 있는 `@` 파일 자동 완성에는 영향을 주지 않습니다 |
| `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Glob 파일 발견 타임아웃(초) |
| `CLAUDE_CODE_ENABLE_TASKS` | 세션이 구조화된 Task 도구(`TaskCreate`, `TaskUpdate`, `TaskGet`, `TaskList`)를 사용할지 레거시 `TodoWrite` 도구를 사용할지 제어합니다. v2.1.142 기준으로 Task 도구가 모든 모드에서 기본값입니다. `TodoWrite`로 되돌리려면 `0`으로 설정하세요 |
| `CLAUDE_CODE_SIMPLE` | 최소 시스템 프롬프트와 Bash, 파일 읽기, 파일 편집 도구만으로 실행하려면 `1`로 설정하세요. 시작 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) 참고 |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | 유휴 기간(ms) 후 SDK 모드 자동 종료 |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | 적응형 사고 비활성화(`1`로 비활성화) |
| `CLAUDE_CODE_DISABLE_THINKING` | 확장 사고 강제 비활성화(`1`로 비활성화) |
| `DISABLE_INTERLEAVED_THINKING` | interleaved-thinking beta 헤더가 전송되는 것을 방지(`1`로 비활성화) |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | 모든 1M 컨텍스트 윈도 모델을 자동 압축을 통해 200K 컨텍스트 크기로 유지합니다. v2.1.223 기준으로 이는 (하나뿐만이 아니라) 모든 1M 모델에 적용됩니다. 배포 전반에 걸쳐 1M 컨텍스트를 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_ACCOUNT_UUID` | 인증용 계정 UUID 재정의 |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | git 관련 시스템 프롬프트 지침 비활성화 |
| `CLAUDE_CODE_ATTRIBUTION_HEADER` | 시스템 프롬프트에서 Claude Code attribution 블록을 생략하려면 `0`으로 설정하세요 |
| `CLAUDE_CODE_NEW_INIT` | `/init`이 대화형 설정 흐름을 실행하게 하려면 `true`로 설정하세요. 코드베이스를 탐색하기 전에 어떤 파일(CLAUDE.md, 스킬, hook)을 생성할지 묻습니다. 이것이 없으면 `/init`은 CLAUDE.md를 자동으로 생성합니다 |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | 하나 이상의 읽기 전용 플러그인 시드 디렉터리 경로, Unix에서는 `:`, Windows에서는 `;`로 구분. 미리 채워진 플러그인을 컨테이너 이미지에 번들합니다. Claude Code는 시작 시 이 디렉터리에서 마켓플레이스를 등록하고 다시 클론하지 않고 미리 캐시된 플러그인을 사용합니다 |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | Claude.ai MCP 서버 활성화 |
| `CLAUDE_CODE_EFFORT_LEVEL` | effort 레벨 설정: `low`, `medium`, `high`, `xhigh`(Opus 4.7 및 4.8, v2.1.111), `max`(Opus 4.6 전용), 또는 `auto`(모델 기본값 사용). `/effort`와 `effortLevel` 설정보다 우선합니다. 시작 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) 참고 |
| `CLAUDE_EFFORT` | 읽기 전용. 셸 스크립트와 hook이 현재 계층에 적응할 수 있도록 활성 effort 레벨과 함께 Bash 도구 하위 프로세스와 hook 핸들러에 주입됩니다(`CLAUDE_CODE_EFFORT_LEVEL`의 동반자; v2.1.133). 스킬 파일 내에서는 `${CLAUDE_EFFORT}`를 사용하세요 *(changelog에 있음, 공식 env-vars 페이지에는 없음 — 읽기 전용, 사용자 구성 불가)* |
| `CLAUDE_CODE_ALWAYS_ENABLE_EFFORT` | effort 레벨 선택을 일반적으로 지원하지 않는 모델을 포함하여 모든 모델에서 effort 매개변수를 강제 활성화하려면 `1`로 설정하세요. 표준 effort 지원 집합 밖의 모델에서 `/effort`와 `effortLevel` 설정이 적용되도록 합니다(v2.1.154) |
| `CLAUDE_CODE_MAX_TURNS` | 중지 전 최대 에이전트 턴 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | `DISABLE_AUTOUPDATER`, `DISABLE_FEEDBACK_COMMAND`, `DISABLE_ERROR_REPORTING`, `DISABLE_TELEMETRY`를 설정하는 것과 동일 |
| `CLAUDE_CODE_SKIP_SETTINGS_SETUP` | 첫 실행 설정 설정 흐름 건너뛰기 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_PROMPT_CACHING_ENABLED` | 프롬프트 캐싱 동작 재정의 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_DISABLE_TOOLS` | 비활성화할 도구의 쉼표 구분 목록 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_DISABLE_MCP` | 모든 MCP 서버 비활성화(`1`로 비활성화) *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | 응답당 최대 출력 토큰. 기본값: 32,000(v2.1.77 기준 Opus 4.6은 64,000). 상한: 64,000(v2.1.77 기준 Opus 4.6 및 Sonnet 4.6은 128,000) |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | fast 모드 완전 비활성화(`1`로 비활성화) |
| `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` | **v2.1.160에서 제거됨** — 이제 환경 변수는 no-op입니다. fast 모드는 이 변수와 관계없이 기본 모델에서 실행됩니다. 이전에는 [fast mode](https://code.claude.com/docs/en/fast-mode)를 기본값 대신 Claude Opus 4.6에 고정했습니다(v2.1.142–v2.1.159) |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | 스트리밍 요청이 스트림 중간에 실패할 때 비스트리밍 폴백을 비활성화하려면 `1`로 설정하세요. 스트리밍 오류가 대신 재시도 계층으로 전파됩니다. 프록시나 게이트웨이가 폴백으로 중복 도구 실행을 유발할 때 유용합니다(v2.1.83) |
| `CLAUDE_ENABLE_STREAM_WATCHDOG` | 멈춘 스트림을 중단하는 스트림 유휴 watchdog. v2.1.163 기준으로 기본적으로 활성화됨; 비활성화하려면 `0`으로 설정 |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | Anthropic API에서 기본적으로 활성화됨(v2.1.139+); 옵트아웃하려면 `0`으로 설정 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | auto memory 비활성화(`1`로 비활성화) |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | `/rewind`용 파일 체크포인팅 비활성화(`1`로 비활성화) |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | 첨부 처리 비활성화(`1`로 비활성화) |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | CLAUDE.md 파일 로드 방지(`1`로 비활성화) |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | 시작 시 `--add-dir`로 지정된 추가 디렉터리에서 CLAUDE.md 메모리 파일 로드(`1`로 활성화). 시작 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) 참고 |
| `CLAUDE_CODE_DISABLE_POLICY_SKILLS` | 시스템 전역 managed 스킬 디렉터리에서 스킬 로드 건너뛰기(`1`로 비활성화) |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` | 이전 세션이 턴 중간에 끝났으면 자동 재개(`1`로 활성화) |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN_MAX_AGE_MS` | 재시작 시 자동 재개될 중단된 턴의 최대 나이(ms). 이보다 오래된 턴은 건너뛰어집니다. `CLAUDE_CODE_RESUME_INTERRUPTED_TURN=1`일 때만 적용됩니다 |
| `CLAUDE_CODE_RESUME_PROMPT` | 자동 재개된 중단 턴 시작 시 주입되는 커스텀 프롬프트. 기본값은 내장 연속 메시지 |
| `CLAUDE_CODE_SAFE_MODE` | safe 모드를 활성화하려면 `1`로 설정하세요. 잠재적으로 파괴적인 작업을 제한하고 더 엄격한 도구 가드레일을 강제합니다 |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | 프롬프트 기록과 세션 트랜스크립트를 디스크에 쓰는 것을 건너뛰려면 `1`로 설정하세요. 이 변수가 설정된 채 시작된 세션은 `--resume`, `--continue`, 위쪽 화살표 기록에 나타나지 않습니다. 임시 스크립트 세션에 유용합니다 |
| `CLAUDE_CODE_USER_EMAIL` | 인증을 위해 사용자 이메일을 동기적으로 제공 |
| `CLAUDE_CODE_ORGANIZATION_UUID` | 인증을 위해 조직 UUID를 동기적으로 제공 |
| `CLAUDE_CONFIG_DIR` | 커스텀 설정 디렉터리(기본 `~/.claude`를 재정의) |
| `CLAUDE_CODE_TMPDIR` | 내부 임시 파일에 사용되는 임시 디렉터리를 재정의합니다. Claude Code는 이 경로에 `/claude/`를 추가합니다. 기본값: Unix/macOS는 `/tmp`, Windows는 `os.tmpdir()` |
| `ANTHROPIC_CUSTOM_HEADERS` | API 요청용 커스텀 헤더(`Name: Value` 형식, 여러 헤더는 개행 구분) |
| `CLAUDE_CODE_EXTRA_BODY` | 모든 API 요청 본문의 최상위에 병합할 JSON 객체. 벤더별 필드(예: 커스텀 게이트웨이용 라우팅 힌트)를 주입하는 데 사용 |
| `CLAUDE_CODE_PROPAGATE_TRACEPARENT` | 커스텀 프록시를 통해 라우팅할 때 요청을 통해 W3C `traceparent` 헤더를 전파하여 Claude Code 트레이스를 업스트림 텔레메트리에 연결하려면 `1`로 설정하세요 |
| `ANTHROPIC_FOUNDRY_API_KEY` | Microsoft Foundry 인증용 API 키 |
| `ANTHROPIC_FOUNDRY_BASE_URL` | Foundry 리소스용 Base URL |
| `ANTHROPIC_FOUNDRY_RESOURCE` | Foundry 리소스 이름 |
| `ANTHROPIC_FOUNDRY_AUTH_TOKEN` | Foundry용 Azure 인증 토큰(서비스 주체 인증의 대안, v2.1.203+) |
| `AWS_BEARER_TOKEN_BEDROCK` | 인증용 Bedrock API 키 |
| `ANTHROPIC_SMALL_FAST_MODEL` | **DEPRECATED** — 대신 `ANTHROPIC_DEFAULT_HAIKU_MODEL`을 사용하세요 |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | 더 이상 사용되지 않는 Haiku 클래스 모델 재정의용 AWS 리전 |
| `CLAUDE_CODE_SHELL_PREFIX` | bash 명령 앞에 붙는 명령 접두사 |
| `BASH_DEFAULT_TIMEOUT_MS` | 기본 bash 명령 타임아웃(ms) |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | Bedrock용 AWS 인증 건너뛰기(`1`로 건너뛰기) |
| `CLAUDE_CODE_AWS_CHAIN_RESOLVE_TIMEOUT_MS` | Bedrock에서 AWS 자격 증명 체인 해석의 타임아웃(밀리초). 자격 증명 소스가 없는 환경에서 체인이 빠르게 실패하도록 더 낮게(예: `5000`) 설정하세요. 기본값: `60000`(60초)(v2.1.207) |
| `CLAUDE_CODE_DISABLE_BEDROCK_CONTENT_TYPE_GUARD` | 예상치 못한 MIME 유형의 응답을 거부하는 Bedrock content-type 가드를 비활성화하려면 `1`로 설정하세요. 게이트웨이가 비표준 content type을 반환하지만 응답 본문이 유효할 때 사용하세요(v2.1.208) |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | Foundry용 Azure 인증 건너뛰기(`1`로 건너뛰기) |
| `CLAUDE_CODE_SKIP_MANTLE_AUTH` | Bedrock Mantle용 AWS 인증 건너뛰기(예: LLM 게이트웨이 사용 시) |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | Vertex용 Google 인증 건너뛰기(`1`로 건너뛰기) |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | 프록시가 DNS 해석을 수행하도록 허용 |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | `apiKeyHelper`의 자격 증명 갱신 간격(ms) |
| `CLAUDE_CODE_CLIENT_CERT` | mTLS용 클라이언트 인증서 경로 |
| `CLAUDE_CODE_CLIENT_KEY` | mTLS용 클라이언트 개인 키 경로 |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | 암호화된 mTLS 키의 passphrase |
| `CLAUDE_CODE_CERT_STORE` | TLS 연결용 CA 인증서 소스의 쉼표 구분 목록: `bundled`(Claude Code와 함께 배송되는 Mozilla CA 세트) 및/또는 `system`(OS 신뢰 저장소). 기본값: `bundled,system`. system 저장소 통합에는 네이티브 바이너리 배포가 필요합니다; Node.js 런타임에서는 이 값과 관계없이 번들 세트만 사용됩니다(v2.1.101) |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | 플러그인 마켓플레이스 git 클론 타임아웃(ms)(기본값: 120000) |
| `CLAUDE_CODE_PLUGIN_PREFER_HTTPS` | GitHub `owner/repo` 축약 플러그인 소스를 SSH 대신 HTTPS로 클론하려면 `1`로 설정하세요. 플러그인 설치/업데이트와 `/plugin marketplace add`/`update`에 적용됩니다. `github.com`용 SSH 키가 구성되지 않은 CI 러너나 컨테이너에 유용합니다(v2.1.141) |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | 플러그인 루트 디렉터리 재정의 |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | 공식 마켓플레이스 자동 추가 건너뛰기(`1`로 비활성화) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` | 첫 쿼리 전에 플러그인 설치 완료 대기(`1`로 활성화) |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL_TIMEOUT_MS` | 동기 플러그인 설치 타임아웃(ms) |
| `CLAUDE_CODE_SYNC_SKILLS` | 세션 시작 전에 스킬이 최신인지 보장하도록 첫 쿼리 전에 스킬 동기화 완료를 대기하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_SYNC_SKILLS_INSTALL_TIMEOUT_MS` | 동기 스킬 동기화의 스킬 설치 단계 타임아웃(ms) |
| `CLAUDE_CODE_SYNC_SKILLS_WAIT_TIMEOUT_MS` | `CLAUDE_CODE_SYNC_SKILLS=1`일 때 모든 스킬 동기화가 완료되기를 기다리는 타임아웃(ms) |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | `git pull`이 실패할 때 마켓플레이스 캐시를 지우고 다시 클론하는 대신 기존 캐시를 유지하려면 `1`로 설정하세요. 다시 클론해도 같은 방식으로 실패할 오프라인 또는 에어갭 환경에 유용합니다 |
| `CLAUDE_CODE_ENABLE_BACKGROUND_PLUGIN_REFRESH` | 백그라운드 설치가 완료된 후 세션 턴 경계에서 플러그인 상태를 새로 고침(`1`로 활성화). 이것이 없으면 새로 설치된 플러그인은 다음 세션에 적용됩니다 |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO` | UI에서 이메일/조직 정보 숨기기 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_DISABLE_CRON` | 예약/cron 작업 비활성화(`1`로 비활성화) |
| `DISABLE_INSTALLATION_CHECKS` | 설치 경고 비활성화 |
| `DISABLE_FEEDBACK_COMMAND` | `/feedback` 명령 비활성화. 이전 이름 `DISABLE_BUG_COMMAND`도 허용됩니다 |
| `DISABLE_DOCTOR_COMMAND` | `/doctor` 명령 숨기기(`1`로 비활성화) |
| `DISABLE_LOGIN_COMMAND` | `/login` 명령 숨기기(`1`로 비활성화) |
| `DISABLE_LOGOUT_COMMAND` | `/logout` 명령 숨기기(`1`로 비활성화) |
| `DISABLE_UPGRADE_COMMAND` | `/upgrade` 명령 숨기기(`1`로 비활성화) |
| `DISABLE_EXTRA_USAGE_COMMAND` | `/extra-usage` 명령 숨기기 — v2.1.144에서 `/usage-credits`로 이름 변경되었지만 이 환경 변수 이름은 변경되지 않음(`1`로 비활성화) |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | `/install-github-app` 명령 숨기기(`1`로 비활성화) |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS` | 플레이버 텍스트 및 비필수 모델 호출 비활성화 *(공식 문서에 없음 — 미검증)* |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | 디버그 로그 파일 경로 재정의. 이름과 달리 이것은 디렉터리가 아니라 파일 경로입니다. `--debug`, `/debug`, 또는 `DEBUG` 환경 변수를 통해 별도로 디버그 모드가 활성화되어야 합니다; 이 변수만 설정해도 로깅이 활성화되지 않습니다. 디버그 모드를 활성화하고 경로를 한 번에 설정하려면 `--debug-file`을 사용하세요. 기본값: `~/.claude/debug/<session-id>.txt` |
| `CLAUDE_CODE_DEBUG_LOG_LEVEL` | 최소 디버그 로그 레벨 |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | 긴 작업의 자동 백그라운드화 강제(`1`로 활성화) |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | Opus 4.0/4.1을 새 모델로 재매핑하는 것을 방지(`1`로 비활성화) |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | 기본값뿐만 아니라 모든 주 모델에 대해 폴백 모델 트리거(`1`로 활성화) |
| `CCR_FORCE_BUNDLE` | GitHub 접근이 가능하더라도 `claude --remote`가 로컬 저장소를 번들하고 업로드하도록 강제하려면 `1`로 설정하세요. 시작 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) 참고 |
| `CLAUDE_CODE_GIT_BASH_PATH` | Windows 전용: Git Bash 실행 파일(`bash.exe`)의 경로. Git Bash가 설치되었지만 PATH에 없을 때 사용하세요 |
| `DISABLE_COST_WARNINGS` | 비용 경고 메시지 비활성화 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 서브에이전트용 모델 재정의(예: `haiku`, `sonnet`) |
| `CLAUDE_CODE_FORWARD_SUBAGENT_TEXT` | 서브에이전트 스트리밍 텍스트 출력을 부모 세션으로 실시간 전달하려면 `1`로 설정하세요. 기본적으로 서브에이전트 출력은 서브에이전트가 완료될 때까지 버퍼링됩니다. v2.1.219 기준으로 전달은 depth 2+의 중첩 서브에이전트에도 작동합니다(v2.1.211) |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | 하위 프로세스 환경(Bash 도구, hook, MCP stdio 서버)에서 Anthropic 및 클라우드 제공자 자격 증명을 제거하려면 `1`로 설정하세요. 하위 프로세스가 API 키를 상속하지 않아야 하는 방어 심층화에 사용하세요(v2.1.83) |
| `CLAUDE_CODE_SCRIPT_CAPS` | `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB`이 설정될 때 특정 스크립트가 세션당 호출될 수 있는 횟수를 제한하는 JSON 객체. 키는 명령 텍스트와 매칭되는 부분 문자열이고; 값은 정수 호출 한계입니다. 예를 들어 `{"deploy.sh": 2}`는 `deploy.sh`를 최대 두 번 호출하도록 허용합니다. 매칭은 부분 문자열 기반이며; `xargs` 또는 `find -exec`를 통한 런타임 팬아웃은 감지되지 않습니다 — 이것은 방어 심층화 제어입니다 |
| `CLAUDE_CODE_PERFORCE_MODE` | Perforce 인식 쓰기 보호를 활성화하려면 `1`로 설정하세요. 설정되면 대상 파일에 owner-write 비트가 없을 때 Edit, Write, NotebookEdit가 `p4 edit <file>` 힌트와 함께 실패합니다. Perforce는 `p4 edit`가 파일을 열 때까지 동기화된 파일에서 이 비트를 지웁니다. Claude Code가 Perforce 변경 추적을 우회하는 것을 방지합니다(v2.1.98) |
| `CLAUDE_CODE_PROCESS_WRAPPER` | 시작 시 Claude Code 프로세스를 래핑합니다. 래퍼 실행 파일과 인수를 지정하세요; Claude Code가 마지막 인수로 추가됩니다. 프로파일링 도구, 샌드박스, 또는 추적(예: `CLAUDE_CODE_PROCESS_WRAPPER="strace -e trace=file"`)에 유용합니다. *(v2.1.208 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_MAX_RETRIES` | API 요청 재시도 횟수 재정의(기본값: 10) |
| `CLAUDE_CODE_RETRY_WATCHDOG` | 비용량 API 오류의 재시도 횟수를 300으로 올립니다. 용량 오류에는 표준 재시도 상한(`CLAUDE_CODE_MAX_RETRIES`, 기본값: 10)이 여전히 적용됩니다(v2.1.199) |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | 최대 병렬 읽기 전용 도구(기본값: 10) |
| `CLAUDE_CODE_MAX_WEB_SEARCHES_PER_SESSION` | 세션당 허용되는 최대 웹 검색 수(기본값: `200`). 긴 에이전트 세션에서 통제 불능 도구 사용을 방지 *(v2.1.212 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION` | 세션당 스폰될 수 있는 최대 서브에이전트 수(기본값: `200`). 깊이 중첩된 에이전트 워크플로에서 리소스 고갈을 방지 *(v2.1.212 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` | 동시에 실행되는 최대 서브에이전트 수(기본값: `20`). 병렬 에이전트 팬아웃을 제한합니다; 초과 에이전트는 대기열에 들어가 슬롯이 비면 실행됩니다(v2.1.217) |
| `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` | 서브에이전트의 최대 중첩 깊이(v2.1.219 기준 기본값: `3`). 깊이 한계의 서브에이전트는 더 이상 서브에이전트를 스폰할 수 없습니다(v2.1.217) |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | SDK 모드에서 내장 서브에이전트 유형 비활성화(`1`로 비활성화) |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | SDK 모드에서 MCP 도구의 `mcp__<server>__` 접두사 건너뛰기(`1`로 활성화) |
| `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS` | 백그라운드 서브에이전트의 멈춤 타임아웃(ms)(기본값: 600000 / 10분). 이 기간 동안 출력이 없으면 서브에이전트가 종료됩니다 |
| `MCP_CONNECTION_NONBLOCKING` | `-p` 모드에서 MCP 연결 대기를 완전히 건너뛰려면 `true`로 설정하세요. `--mcp-config` 서버 연결을 가장 느린 서버에서 차단하는 대신 5초로 제한합니다 *(v2.1.89 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS` | 느린 MCP 도구 호출이 자동으로 백그라운드화되는 기간(밀리초)(기본값: `120000` / 2분). 이 임계값을 초과하는 MCP 호출은 에이전트 턴을 차단하지 않고 백그라운드에서 계속 실행됩니다 *(v2.1.212 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` | MCP 도구 호출의 유휴 타임아웃(ms). 이 윈도 내에 스트리밍 응답을 생성하지 않으면 도구 호출이 취소됩니다. 게이트웨이나 MCP 서버가 조용히 멈출 때 사용하세요 — 총 실제 시간 한계인 `MCP_TOOL_TIMEOUT`과 구별됩니다(v2.1.187) |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd hook 타임아웃(ms)(하드 1.5초 한계를 대체) |
| `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` | Stop hook이 세션 종료를 차단할 수 있는 최대 횟수(기본값: `8`). 이 횟수만큼 차단한 후에는 hook 종료 코드와 관계없이 세션이 종료됩니다 |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | 피드백 설문 프롬프트 비활성화(`1`로 비활성화) |
| `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL` | Anthropic 대상 비필수 트래픽이 차단될 때 세션 품질 설문을 자체 OpenTelemetry 컬렉터로 라우팅하려면 `1`로 설정하세요. 설문 평가는 구성된 컬렉터로 OTEL 이벤트로만 방출됩니다 — 설문 데이터는 Anthropic으로 전송되지 않습니다. `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`, `DISABLE_TELEMETRY`, 또는 `DO_NOT_TRACK`이 설정될 때 적용됩니다; 그 외에는 효과가 없습니다. `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY`와 조직 제품 피드백 정책이 우선합니다(v2.1.136) |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | 터미널 제목 업데이트 비활성화(`1`로 비활성화) |
| `CLAUDE_CODE_TMUX_TRUECOLOR` | tmux 내부에서 24비트 truecolor 출력을 허용하려면 `1`로 설정하세요. 기본적으로 Claude Code는 `$TMUX`가 설정되면 256색으로 제한하는데, tmux가 구성되지 않는 한 truecolor 이스케이프 시퀀스를 통과시키지 않기 때문입니다. `~/.tmux.conf`에 `set -ga terminal-overrides ',*:Tc'`를 추가한 후 이를 설정하세요 |
| `CLAUDE_CODE_NO_FLICKER` | 깜빡임 없는 alt-screen 렌더링을 활성화하려면 `1`로 설정하세요. 전체 화면 재그리기 중 시각적 깜빡임을 제거합니다(v2.1.88) |
| `CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT` | 전체 화면 렌더링에서 매 프레임마다 전체 화면을 다시 그리려면 `1`로 설정하세요. 비정상적인 터미널 에뮬레이터에서 부분 재그리기가 시각적 아티팩트를 생성할 때 사용하세요 |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` | 전체 화면 렌더링을 비활성화하고 클래식 메인 화면 렌더러를 사용하려면 `1`로 설정하세요. 대화가 터미널의 네이티브 스크롤백에 유지되어 `Cmd+f`와 tmux copy 모드가 평소처럼 작동합니다. `CLAUDE_CODE_NO_FLICKER`와 `tui` 설정보다 우선합니다. `/tui default`로도 전환할 수 있습니다(v2.1.132) |
| `CLAUDE_CODE_FORCE_SYNC_OUTPUT` | 터미널이 지원하지만 자동 감지되지 않을 때 DEC private mode 2026 동기화 출력을 강제 활성화하려면 `1`로 설정하세요. BSU/ESU를 구현하지만 기능 프로브에 응답하지 않는 Emacs `eat` 같은 에뮬레이터에 유용합니다. tmux 아래에서는 효과가 없습니다(v2.1.129) |
| `CLAUDE_CODE_SCROLL_SPEED` | 전체 화면 렌더링용 마우스 휠 스크롤 배수. 더 빠른 스크롤을 위해 늘리고, 더 미세한 제어를 위해 줄이세요 |
| `CLAUDE_CODE_DISABLE_VIRTUAL_SCROLL` | 전체 화면 렌더링에서 가상 스크롤을 비활성화하고 트랜스크립트의 모든 메시지를 렌더링하려면 `1`로 설정하세요. 전체 화면 모드에서 스크롤 시 메시지가 나타나야 할 곳에 빈 영역이 보이면 사용하세요 |
| `CLAUDE_CODE_DISABLE_MOUSE` | 전체 화면 렌더링에서 마우스 추적을 비활성화하려면 `1`로 설정하세요. 마우스 이벤트가 터미널 멀티플렉서나 접근성 도구와 충돌할 때 유용합니다 |
| `CLAUDE_CODE_DISABLE_MOUSE_CLICKS` | 휠 스크롤은 보존하면서 전체 화면 렌더링에서 마우스 클릭 상호작용을 비활성화하려면 `1`로 설정하세요. 클릭 이벤트 억제를 전체 마우스 추적 비활성화(`CLAUDE_CODE_DISABLE_MOUSE`)와 구별합니다. 터미널 선택이나 외부 접근성 도구가 앱 내 마우스 클릭과 충돌할 때 유용합니다 *(v2.1.195 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_HIDE_CWD` | Claude Code 시작 로고 배너에서 현재 작업 디렉터리를 숨기려면 `1`로 설정하세요. CWD 경로가 호스트나 프로젝트 레이아웃에 대한 정보를 누출하는 화면 녹화, 데모, 또는 공유 세션에서 유용합니다(v2.1.119) |
| `CLAUDE_CODE_ACCESSIBILITY` | 스크린 리더 및 접근성 도구를 위해 네이티브 터미널 커서를 계속 표시하려면 `1`로 설정하세요 |
| `CLAUDE_AX_SCREEN_READER` | 스크린 리더 친화적 출력을 렌더링하려면 `1`로 설정하세요: 장식용 테두리나 애니메이션 없는 평문. `axScreenReader` 설정이 `true`일 때도 스크린 리더 모드를 강제로 끄려면 `0`으로 설정하세요. `--ax-screen-reader` CLI 플래그가 우선합니다(v2.1.181+) |
| `CLAUDE_CODE_NATIVE_CURSOR` | Claude Code의 커스텀 커서 문자 대신 터미널 자체 커서를 입력 캐럿 위치에 표시하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | diff 출력에서 구문 강조를 비활성화하려면 `0`으로 설정하세요 |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL` | 자동 IDE 확장 설치 건너뛰기(`1`로 건너뛰기) |
| `CLAUDE_CODE_AUTO_CONNECT_IDE` | 자동 IDE 연결 동작 재정의 |
| `CLAUDE_CODE_IDE_HOST_OVERRIDE` | 연결용 IDE 호스트 주소 재정의 |
| `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | IDE 잠금 파일 검증 건너뛰기(`1`로 건너뛰기) |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | OTel 헤더 헬퍼 스크립트의 디바운스 간격(ms) |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS` | OpenTelemetry 플러시 타임아웃(ms) |
| `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS` | OpenTelemetry 종료 타임아웃(ms) |
| `CLAUDE_ENABLE_BYTE_WATCHDOG` | 바이트 레벨 스트리밍 유휴 watchdog를 강제 활성화하려면 `1`로, 강제 비활성화하려면 `0`으로 설정하세요. 미설정 시 watchdog는 Anthropic API 연결에 대해 기본적으로 활성화됩니다. 바이트 watchdog는 이벤트 레벨 watchdog와 독립적으로, `CLAUDE_STREAM_IDLE_TIMEOUT_MS`로 설정된 기간(최소 5분) 동안 wire에 바이트가 도착하지 않으면 연결을 중단합니다 |
| `CLAUDE_ENABLE_BYTE_WATCHDOG_BEDROCK` | Bedrock 연결에 대해 특별히 바이트 레벨 스트리밍 유휴 watchdog를 활성화하려면 `1`로 설정하세요. 기본적으로 바이트 watchdog는 Bedrock에서 활성이 아닙니다. 타임아웃을 구성하려면 `CLAUDE_STREAM_IDLE_TIMEOUT_MS`와 함께 사용하세요 |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS` | 스트리밍 유휴 watchdog의 타임아웃(ms). 두 watchdog가 적용됩니다: **바이트 레벨**(기본값 및 최소 `300000` / 5분, wire에 바이트가 도착하지 않으면 중단)과 **이벤트 레벨**(기본값 `90000` / 90초, 최소 없음, SSE 이벤트가 도착하지 않으면 중단). 바이트 watchdog는 Anthropic API 연결에 대해 기본적으로 활성화됩니다; `CLAUDE_ENABLE_BYTE_WATCHDOG`로 제어하세요. 장기 실행 도구나 느린 네트워크가 조기 타임아웃 오류를 유발하면 이벤트 타임아웃을 늘리세요 |
| `OTEL_LOG_TOOL_DETAILS` | OpenTelemetry 이벤트에 `tool_parameters`를 포함하려면 `1`로 설정하세요. 프라이버시를 위해 기본적으로 생략됩니다(v2.1.85) |
| `CLAUDE_CODE_OTEL_CONTENT_MAX_LENGTH` | OpenTelemetry 이벤트 페이로드의 최대 콘텐츠 길이(바이트)(기본값: `61440` / 60 KB). 큰 도구 입력, 출력, 또는 메시지 본문을 OTel 이벤트로 방출하기 전에 잘라 과대 페이로드를 방지 *(v2.1.214 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `OTEL_LOG_RAW_API_BODIES` | 전체 API 요청 및 응답 본문을 OpenTelemetry 로그 이벤트로 방출하려면 `1`로 설정하세요. 프라이버시와 페이로드 크기를 위해 기본적으로 생략됩니다. 게이트웨이나 프록시에서 디버깅하는 데 유용합니다 *(v2.1.111 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `OTEL_RESOURCE_ATTRIBUTES` | Claude Code가 방출하는 모든 OpenTelemetry 메트릭 데이터 포인트에 리소스 속성으로 추가되는 쉼표 구분 `key=value` 쌍. 컬렉터에서 필터링하기 위해 모든 메트릭에 나타나는 환경 또는 배포 레이블(예: `environment=production,team=platform`)을 첨부하는 데 사용(v2.1.162) |
| `OTEL_LOG_USER_PROMPTS` | OpenTelemetry LLM 요청 스팬에 `user_system_prompt` 필드를 포함하려면 `1`로 설정하세요. 프라이버시를 위해 기본적으로 생략됩니다 — 사용자 프롬프트는 민감한 데이터를 포함할 수 있으므로, OTel 컬렉터를 제어하고 정책이 마련된 경우에만 옵트인하세요 *(v2.1.121 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `OTEL_LOG_ASSISTANT_RESPONSES` | 모델 응답 텍스트를 OpenTelemetry 로그 이벤트에 포함하려면 `1`로 설정하세요. 프라이버시와 페이로드 크기를 위해 기본적으로 생략됩니다. OTel 컬렉터를 제어하고 모델 출력 처리 정책이 있는 경우에만 사용하세요 *(v2.1.193 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | 메트릭 및 로그용 OpenTelemetry 컬렉터 엔드포인트 URL. [Monitoring](https://code.claude.com/docs/en/monitoring-usage) 참고 |
| `OTEL_EXPORTER_OTLP_HEADERS` | 컬렉터 인증용 OpenTelemetry 익스포터 헤더(`Name=Value` 형식, 쉼표 구분) |
| `OTEL_LOG_TOOL_CONTENT` | 전체 도구 입력 및 출력을 OpenTelemetry 로그 이벤트로 방출하려면 `1`로 설정하세요. 프라이버시를 위해 기본적으로 생략됩니다 |
| `OTEL_METRICS_EXPORTER` | OpenTelemetry 메트릭 익스포터 유형(예: `otlp`). [Monitoring](https://code.claude.com/docs/en/monitoring-usage) 참고 |
| `OTEL_TRACES_EXPORTER` | OpenTelemetry 트레이스 익스포터 유형(예: `otlp`). [Monitoring](https://code.claude.com/docs/en/monitoring-usage) 참고 |
| `OTEL_METRICS_INCLUDE_ENTRYPOINT` | 세션 진입점(예: 대화형 vs `-p` vs SDK)을 모든 OpenTelemetry 메트릭 데이터 포인트의 레이블로 포함하려면 `1`로 설정하세요. Claude Code가 어떻게 호출되었는지로 메트릭을 분류하는 데 유용합니다(v2.1.161) |
| `OTEL_ATTRIBUTE_VALUE_LENGTH_LIMIT` | OpenTelemetry 속성 값의 최대 길이(바이트). 이 한계를 초과하는 속성 값은 잘립니다. 큰 속성 값을 가진 스팬을 방출할 때 메모리와 페이로드 크기를 제한하는 데 사용 |
| `OTEL_METRICS_INCLUDE_ACCOUNT_UUID` | 인증된 계정 UUID를 모든 OpenTelemetry 메트릭 데이터 포인트의 레이블로 포함하려면 `1`로 설정하세요 |
| `OTEL_METRICS_INCLUDE_SESSION_ID` | 세션 ID를 모든 OpenTelemetry 메트릭 데이터 포인트의 레이블로 포함하려면 `1`로 설정하세요 |
| `OTEL_METRICS_INCLUDE_VERSION` | Claude Code 버전을 모든 OpenTelemetry 메트릭 데이터 포인트의 레이블로 포함하려면 `1`로 설정하세요 |
| `OTEL_METRICS_INCLUDE_RESOURCE_ATTRIBUTES` | `OTEL_RESOURCE_ATTRIBUTES` 키/값 쌍을 모든 메트릭 데이터 포인트에 차원으로 승격하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_OTEL_DIAG_STDERR` | OpenTelemetry SDK 진단 출력을 stderr로 방출하려면 `1`로 설정하세요. 컬렉터 연결성이나 익스포터 구성 문제를 해결하는 데 사용하세요 |
| `CLAUDE_CODE_FORK_SUBAGENT` | 외부 빌드(비-Anthropic 서명 배포)에서 forked 서브에이전트를 활성화하려면 `1`로 설정하세요. Forked 서브에이전트는 메인 에이전트의 컨텍스트를 공유하는 대신 격리된 하위 프로세스에서 실행됩니다 *(v2.1.117 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_MCP_SERVER_NAME` | MCP 서버의 이름, `headersHelper` 스크립트가 서버별 인증 헤더를 생성할 수 있도록 환경 변수로 전달됨 *(v2.1.85 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
| `CLAUDE_CODE_MCP_SERVER_URL` | MCP 서버의 URL, `CLAUDE_CODE_MCP_SERVER_NAME`과 함께 `headersHelper` 스크립트에 환경 변수로 전달됨 *(v2.1.85 changelog에 있음, 공식 env-vars 페이지에는 아직 없음)* |
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
| `MAX_THINKING_TOKENS` | 응답당 최대 확장 사고 토큰. Anthropic API에서 확장 사고를 완전히 비활성화하려면 `0`으로 설정(`--thinking disabled`와 동일). 고정 사고 예산을 사용할 때만 적용됩니다 — 적응형 사고 모델(Opus 4.7+)에서는 effort 레벨이 대신 사고 깊이를 제어합니다 |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | 자동 압축 계산에 사용되는 컨텍스트 용량(토큰 단위) 설정. 기본값은 모델의 컨텍스트 윈도(표준 200K, 확장 컨텍스트 모델은 1M). 1M 모델을 압축에 대해 500K로 취급하려면 더 낮은 값(예: `500000`)을 사용하세요. 실제 컨텍스트 윈도로 제한됩니다. `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`가 이 값의 비율로 적용됩니다. 이를 설정하면 압축 임계값이 상태 표시줄의 `used_percentage`에서 분리됩니다 |
| `DISABLE_AUTO_COMPACT` | 자동 컨텍스트 압축 비활성화(`1`로 비활성화). 수동 `/compact`는 여전히 작동합니다 *(공식 문서에 없음 — 미검증)* |
| `DISABLE_COMPACT` | 모든 압축 비활성화 — 자동 및 수동 모두(`1`로 비활성화) |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | 프롬프트 제안 활성화 |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED` | 세션에 plan 모드 요구 |
| `CLAUDE_CODE_TEAM_NAME` | agent teams용 팀 이름 |
| `CLAUDE_CODE_TASK_LIST_ID` | 작업 통합용 작업 목록 ID |
| `CLAUDE_ENV_FILE` | 커스텀 환경 파일 경로 |
| `FORCE_AUTOUPDATE_PLUGINS` | 플러그인 자동 업데이트 강제(`1`로 활성화) |
| `HTTP_PROXY` | 네트워크 요청용 HTTP 프록시 URL |
| `HTTPS_PROXY` | 네트워크 요청용 HTTPS 프록시 URL |
| `NO_PROXY` | 프록시를 우회하는 호스트의 쉼표 구분 목록 |
| `MCP_TOOL_TIMEOUT` | MCP 도구 실행 타임아웃(ms) |
| `MCP_CLIENT_SECRET` | MCP OAuth 클라이언트 시크릿 |
| `MCP_OAUTH_CALLBACK_PORT` | MCP OAuth 콜백 포트 |
| `IS_DEMO` | 데모 모드 활성화 |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | 슬래시 명령 도구 출력의 문자 예산 |
| `TASK_MAX_OUTPUT_LENGTH` | 백그라운드 작업 결과의 최대 출력 길이(바이트). 이 한계를 초과하는 결과는 잘립니다 |
| `MAX_STRUCTURED_OUTPUT_RETRIES` | 구조화된 출력(스키마 제약 도구 호출)이 검증에 실패할 때의 최대 재시도 횟수. 기본값: `3` |
| `DEBUG` | 상세 디버그 로깅을 활성화하려면 `claude:*` 또는 다른 디버그 네임스페이스로 설정하세요. 대부분의 목적에서 `--debug` 플래그와 동일합니다 |
| `FORCE_HYPERLINK` | 명시적으로 지원을 광고하지 않는 터미널에서도 OSC 8 하이퍼링크 렌더링을 강제하려면 `1`로 설정하세요. 터미널이 하이퍼링크를 지원하지만 자동 감지되지 않을 때 사용하세요 |
| `DISABLE_GROWTHBOOK` | GrowthBook 기능 플래그 클라이언트를 비활성화하여 기능 플래그 CDN으로의 아웃바운드 호출을 방지하려면 `1`로 설정하세요. 에어갭 또는 방화벽 제한 환경에서 유용합니다 |
| `DISABLE_PROMPT_CACHING_FABLE` | Fable 모델 요청에 대해 특별히 프롬프트 캐싱을 비활성화하려면 `1`로 설정하세요 |
| `CLAUDE_CODE_FORCE_STRIKETHROUGH` | 지원을 확인하지 않은 터미널에서도 diff 출력의 삭제된 라인에 대한 취소선 텍스트 렌더링을 강제 활성화하려면 `1`로 설정하세요 |
| `CLAUDE_DISABLE_ADOPT` | Claude Code가 같은 터미널에서 실행 중일 수 있는 기존 `claude` 프로세스를 채택하는 것을 방지하려면 `1`로 설정하세요. 프로세스 채택이 예기치 않은 동작을 유발하는 환경에서 사용됩니다 |
| `CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS` | 대기 표시기를 출력하기 전 백그라운드 작업이 출력을 생성하기를 기다리는 최대 시간(ms). 백그라운드 작업이 여전히 실행 중임을 사용자에게 알리기 전에 Claude Code가 조용히 대기하는 시간을 제어합니다 |
| `CLAUDE_CODE_TEAM_TEARDOWN_PARK_TIMEOUT_MS` | 팀메이트를 파킹할 때 agent team 해체의 타임아웃(ms). 리드 에이전트가 진행하기 전에 팀원의 해체 완료를 기다리는 시간을 제어합니다 |
| `CLAUDE_CODE_SIMPLE_SYSTEM_PROMPT` | `--simple` / `CLAUDE_CODE_SIMPLE=1` 모드에서 실행할 때 주입할 커스텀 시스템 프롬프트 |
| `CLAUDE_CODE_ENABLE_APPEND_SUBAGENT_PROMPT` | 각 서브에이전트의 시스템 프롬프트에 부모의 시스템 프롬프트를 추가하려면 `1`로 설정하세요. 미설정 시 서브에이전트는 자체 프롬프트만 받습니다 |
| `CLAUDE_CODE_SKIP_FAST_MODE_ORG_CHECK` | fast 모드 사용 가능 여부를 결정하는 조직 레벨 검사를 건너뛰려면 `1`로 설정하세요. 조직 상태 엔드포인트에 도달할 수 없고 fast 모드가 항상 사용 가능한 것으로 취급되어야 하는 환경에서 사용하세요 |
| `CLAUDE_CODE_USE_ANTHROPIC_AWS` | API 요청에 AWS의 Claude Platform을 사용하려면 `1`로 설정하세요. `ANTHROPIC_AWS_API_KEY`와 `ANTHROPIC_AWS_WORKSPACE_ID`가 필요합니다 |
| `CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH` | AWS의 Claude Platform을 사용할 때 인증을 건너뛰려면 `1`로 설정하세요. 자격 증명이 외부에서 주입되는 환경에서 사용하세요 |
| `CLAUDE_CODE_SKIP_AWS_CRED_CACHE` | AWS 자격 증명 캐싱을 비활성화하려면 `1`로 설정하세요. 모든 요청에서 새로운 자격 증명 해석을 강제합니다. 자격 증명이 자주 순환할 때 유용합니다 |
| `CLAUDE_CODE_USE_NATIVE_FILE_SEARCH` | 내장 검색 대신 macOS(Spotlight/mdfind)와 Windows(Windows Search)에서 Glob 도구에 네이티브 OS 파일 검색을 사용하려면 `1`로 설정하세요. 네이티브 인덱싱이 구성된 대규모 저장소에서 더 빠를 수 있습니다 |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU` | Claude 3.5 Haiku용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_3_5_SONNET` | Claude 3.5 Sonnet용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_3_7_SONNET` | Claude 3.7 Sonnet용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_0_OPUS` | Claude 4.0 Opus용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_0_SONNET` | Claude 4.0 Sonnet용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_1_OPUS` | Claude 4.1 Opus용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_5_OPUS` | Claude 4.5 Opus용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_5_SONNET` | Claude 4.5 Sonnet용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_6_OPUS` | Claude 4.6 Opus용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_6_SONNET` | Claude 4.6 Sonnet용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_7_OPUS` | Claude 4.7 Opus용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_4_8_OPUS` | Claude 4.8 Opus용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_5_OPUS` | Claude 5 Opus용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_5_SONNET` | Claude 5 Sonnet용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_FABLE_5` | Claude Fable 5용 Vertex AI 리전 재정의 |
| `VERTEX_REGION_CLAUDE_HAIKU_4_5` | Claude Haiku 4.5용 Vertex AI 리전 재정의 |
| `CLAUDE_BYTE_STREAM_IDLE_TIMEOUT_MS` | 멈춘 연결을 중단하기 전 바이트 레벨 스트리밍 유휴 watchdog의 타임아웃(밀리초). `CLAUDE_ENABLE_BYTE_WATCHDOG`이 활성일 때 적용됩니다. 최소: 300,000(5분) |
| `CLAUDE_CODE_DISABLE_ADMIN_ENV_UNION` | 여러 admin 제어 managed 소스에 걸친 `env`의 키별 union 동작을 비활성화하려면 `1`로 설정하세요. 설정되면 우선하는 managed 소스의 `env` 블록만 사용됩니다(v2.1.223) |
| `CLAUDE_CODE_ENABLE_OPUS_4_7_FAST_MODE` | Opus 4.7에서 fast 모드를 활성화하려면 `1`로 설정하세요. 기본적으로 Opus 4.7의 기본 effort(`xhigh`)가 fast-mode 모델 계층과 다르기 때문에 fast 모드는 Opus 4.7에서 사용할 수 없습니다 |
| `MCP_DISCOVERY_CACHE` | MCP 서버 발견 결과를 위한 로컬 캐시 파일 경로. 설정되면 Claude Code가 시작 시 발견 엔드포인트를 쿼리하는 대신 이 파일에서 발견 결과를 읽어, MCP 서버가 많은 환경에서 지연을 줄입니다 |
| `USE_BUILTIN_RIPGREP` | Grep 도구에 `PATH`에서 찾은 `rg` 대신 Claude Code의 번들 ripgrep 바이너리를 사용하려면 `1`로 설정하세요. 시스템 `rg` 버전이 호환되지 않거나 설치되지 않았을 때 유용합니다. 시작 전용 변수로도 구성 가능 — [CLI Startup Flags](./claude-cli-startup-flags.md#environment-variables) 참고 |
| `ANTHROPIC_BEDROCK_REGION_PREFIX` | Bedrock 교차 리전 inference profile ID용 리전 접두사. 설정되면 Claude Code가 교차 리전 inference profile ARN을 자동으로 구성하기 위해 이 값을 Bedrock 모델 ID 앞에 붙입니다(v2.1.224) *(v2.1.224 changelog에 있음; 공식 env-vars 페이지에는 아직 없음)* |

---

## Useful Commands

| Command | Description |
|---------|-------------|
| `/model` | 모델 전환 및 effort 레벨 조정(Opus 4.7 및 4.8) |
| `/effort` | effort 레벨 직접 설정: `low`, `medium`, `high`, `xhigh`(Fable 5, Opus 5, Sonnet 5, Opus 4.7, 4.8), `max`(세션 전용; 모든 effort 지원 모델에서 사용 가능), 또는 `ultracode`(세션 전용; ultracode 워크플로 활성화, v2.1.203+)(v2.1.76+) |
| `/config` | 대화형 구성 UI; 프롬프트 기반 설정을 위한 `key=value` 구문도 허용: `/config model=sonnet`(v2.1.181) |
| `/autocompact` | 자동 압축 윈도 크기를 토큰 단위로 설정. `autoCompactWindow`를 사용자 설정에 기록합니다. 세션별로 동일한 효과를 위해 `--autocompact <tokens>` CLI 플래그를 사용하세요(v2.1.221+) |
| `/memory` | 모든 메모리 파일 보기/편집 |
| `/agents` | 서브에이전트 관리 |
| `/mcp` | MCP 서버 관리 |
| `/hooks` | 구성된 hook 보기 |
| `/plugin` | 플러그인 관리 |
| `claude plugin tag` | 배포를 위해 마켓플레이스에서 플러그인 버전을 태그합니다. 플러그인 이름과 버전과 함께 마켓플레이스 저장소에서 실행하세요(v2.1.118) |
| `claude plugin prune` | 마켓플레이스 소스가 더 이상 존재하지 않는 플러그인 제거(예: 마켓플레이스 삭제 또는 `extraKnownMarketplaces` 항목 제거). 로컬 캐시를 정리하고 고아 플러그인을 비활성화합니다(v2.1.121) |
| `claude plugin details <plugin>` | 플러그인의 컴포넌트 인벤토리(명령, 에이전트, 스킬, hook)와 추가하는 세션당 컨텍스트 토큰 비용을 표시합니다. managed 환경에서 플러그인을 활성화하기 전에 토큰 지출을 감사하는 데 유용합니다(v2.1.139) |
| `/keybindings` | 커스텀 키보드 단축키 구성 |
| `/skills` | 스킬 보기 및 관리 |
| `/permissions` | 권한 규칙 보기 및 관리 |
| `/usage-credits` | 남은 사용 크레딧 및 한도 보기. v2.1.144에서 `/extra-usage`에서 이름 변경됨(이전 이름도 여전히 작동) |
| `claude gateway` | 조직 관리형 배포를 위한 Claude Gateway 연결 관리. managed 설정에 `forceLoginMethod: "gateway"`가 필요합니다(v2.1.195) |
| `claude auto-mode reset` | 현재 세션의 auto-mode 분류를 리셋합니다. 확인을 프롬프트합니다; 프롬프트를 건너뛰려면 `--yes`를 전달하세요(v2.1.212) |
| `/fork` | 현재 세션 컨텍스트를 새 격리된 서브에이전트 세션으로 분기(v2.1.212) |
| `/subtask` | 별도 컨텍스트에서 격리된 하위 작업 시작. 하위 작업은 독립적으로 실행되고 완료되면 결과가 반환됩니다(v2.1.212) |
| `--doctor` | 구성 문제 진단 |
| `--debug` | hook 실행 세부 정보가 있는 디버그 모드 |

---

## Quick Reference: Complete Example

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "sonnet",
  "advisorModel": "opus",
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
  "autoCompactWindow": 800000,
  "skillListingMaxDescChars": 1536,
  "skillListingBudgetFraction": 0.01,
  "disableAgentView": false,
  "disableWorkflows": false,
  "workflowKeywordTriggerEnabled": true,
  "workflowSizeGuideline": "medium",
  "syntaxHighlightingDisabled": false,
  "emojiCompletionEnabled": true,

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
      "Bash(npm run *)",
      "Bash(git *)",
      "WebFetch(domain:*)",
      "mcp__memory__*",
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
- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [Claude Code Model Configuration](https://code.claude.com/docs/en/model-config)
- [Claude Code Status Line Reference](https://code.claude.com/docs/en/statusline)
- [Claude Code Environment Variables](https://code.claude.com/docs/en/env-vars)
- [Claude Code Settings JSON Schema](https://json.schemastore.org/claude-code-settings.json)
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code GitHub Settings Examples](https://github.com/feiskyer/claude-code-settings)
- [Claude Code Permissions Reference](https://code.claude.com/docs/en/permissions)
- [Claude Code Sandbox Reference](https://code.claude.com/docs/en/sandboxing)
