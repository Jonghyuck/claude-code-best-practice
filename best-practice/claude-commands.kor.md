<!--
  이 문서는 best-practice/claude-commands.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Commands Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Aug%2009%2C%202026%2011%3A17%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.226-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-commands-implementation.md)

Claude Code 커맨드 — frontmatter 필드와 공식 내장 슬래시 커맨드.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter Fields (20)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | 표시 이름 및 `/slash-command` 식별자. 생략하면 디렉터리 이름을 기본값으로 사용 |
| `description` | string | Recommended | 커맨드가 하는 일. 자동완성에 표시되며 Claude가 자동 발견에 사용 |
| `when_to_use` | string | No | Claude가 스킬을 언제 호출해야 하는지에 대한 추가 맥락 — 트리거 문구나 예시 요청. 목록에서 `description` 뒤에 덧붙여지며 1,536자 상한에 포함됨 |
| `argument-hint` | string | No | 자동완성 중 표시되는 힌트 (예: `[issue-number]`, `[filename]`) |
| `arguments` | string/list | No | 커맨드 내용에서 `$name` 치환에 사용할 이름 있는 위치 인자. 공백으로 구분된 문자열 또는 YAML 리스트를 허용 — 이름이 순서대로 인자 위치에 매핑됨 |
| `disable-model-invocation` | boolean | No | `true`로 설정하면 Claude가 이 커맨드를 자동으로 호출하지 못하게 함 |
| `user-invocable` | boolean | No | `false`로 설정하면 `/` 메뉴에서 숨김 — 커맨드가 배경 지식 전용이 됨 |
| `paths` | string/list | No | 이 스킬이 활성화되는 시점을 제한하는 glob 패턴. 쉼표로 구분된 문자열 또는 YAML 리스트를 허용. 설정하면 Claude가 해당 패턴에 매칭되는 파일로 작업할 때만 스킬을 자동으로 로드 |
| `allowed-tools` | string | No | 이 커맨드가 활성일 때 권한 프롬프트 없이 허용되는 도구 |
| `disallowed-tools` | string/list | No | 이 커맨드가 활성인 동안 Claude의 사용 가능 도구 풀에서 제거되는 도구. 다음 메시지를 보내면 해제됨. `allowed-tools`의 반대 |
| `model` | string | No | 이 커맨드가 실행될 때 사용할 모델 (예: `haiku`, `sonnet`, `opus`) |
| `effort` | string | No | 호출 시 모델 effort 수준을 재정의 (`low`, `medium`, `high`, `xhigh`, `max`) |
| `context` | string | No | `fork`로 설정하면 격리된 서브에이전트 컨텍스트에서 커맨드를 실행 |
| `agent` | string | No | `context: fork`가 설정된 경우의 서브에이전트 유형 (기본값: `general-purpose`) |
| `background` | boolean | No | `context: fork`가 있을 때만 적용. `false`로 설정하면 forked 서브에이전트를 백그라운드로 실행하는 대신, 스킬을 호출한 턴에서 결과를 대기함. 기본값: `true`. v2.1.218+ 필요 |
| `shell` | string | No | `` !`command` `` 블록에 사용할 셸 — `bash`(기본값) 또는 `powershell`을 허용. `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` 필요 |
| `metadata` | object | No | 자체 키-값 데이터를 위한 자유 형식 YAML 맵. Claude Code는 그 내용(반드시 맵이어야 함)을 무시함. 자체 툴링에서 읽는 카탈로그나 엔타이틀먼트 필드에 유용. `paths` 같은 예약 필드 이름을 키로 재사용하지 말 것 |
| `license` | string | No | [Agent Skills](https://agentskills.io) 명세에 따라 스킬을 다루는 라이선스. Claude Code는 이 필드를 받아들이지만 이에 따라 동작하지는 않음 |
| `compatibility` | string | No | [Agent Skills](https://agentskills.io) 명세에 따른 스킬의 환경 요구사항 (예: 의도된 제품이나 시스템 사전 요건). 최대 500자 허용. Claude Code는 이 필드를 받아들이지만 이에 따라 동작하지는 않음 |
| `hooks` | object | No | 이 커맨드로 범위가 한정된 라이프사이클 훅 |

---

## ![Official](../!/tags/official.svg) **(89)**

| # | Command | Tag | Description |
|---|---------|-----|-------------|
| 1 | `/design-login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | claude.ai 계정으로 `/design-sync`의 디자인 시스템 접근을 인가 |
| 2 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Anthropic 계정에 로그인 |
| 3 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Anthropic 계정에서 로그아웃 |
| 4 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 대화형 마법사를 통해 Amazon Bedrock 인증, 리전, 모델 핀을 구성. `CLAUDE_CODE_USE_BEDROCK=1`이 설정된 경우에만 표시됨. 처음 Bedrock을 사용하는 경우 로그인 화면에서도 이 마법사에 접근 가능 |
| 5 | `/setup-vertex` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 대화형 마법사를 통해 Google Cloud의 Agent Platform 인증, 프로젝트, 리전, 모델 핀을 구성. `CLAUDE_CODE_USE_VERTEX=1`이 설정된 경우에만 표시됨. 처음 사용하는 경우 로그인 화면에서도 이 마법사에 접근 가능 |
| 6 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 더 높은 요금제 등급으로 전환하기 위한 업그레이드 페이지 열기 |
| 7 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 현재 세션의 프롬프트 바 색상 설정. 사용 가능한 색상: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan`. `default`로 초기화. 인자 없이 실행하면 무작위 색상 선택 |
| 8 | `/config [key=value ...]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 테마, 모델, 출력 스타일 및 기타 환경설정을 조정하는 Settings 인터페이스 열기. v2.1.181부터 하나 이상의 `key=value` 쌍을 전달해 인터페이스를 열지 않고 설정을 직접 지정 가능. 예: `/config thinking=false`. `key=value` 형식은 비대화형(`-p`) 및 Remote Control 모드에서도 동작. `/config help`를 실행하면 설정 가능한 키 목록을 표시. 별칭: `/settings` |
| 9 | `/focus` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 마지막 프롬프트, 편집 diffstat이 포함된 한 줄 도구 호출 요약, 최종 응답만 보여주는 포커스 뷰 토글. 선택은 세션 간 유지됨. 재정의하려면 설정에서 `viewMode`를 지정. 전체 화면 렌더링에서만 사용 가능 |
| 10 | `/import [codex\|gemini] [--dry-run] [--yes]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 사용자 머신의 다른 코딩 에이전트(현재 OpenAI Codex 및 Google Gemini CLI)의 구성을 Claude Code로 가져오기. 인스트럭션 파일, MCP 서버, 커맨드, 서브에이전트, 스킬 포함. 비대화형 모드(`-p`)에서는 발견한 내용을 나열하고 가져오기를 확정하는 커맨드를 제공. `--dry-run`은 쓰기 없이 미리보기하고, `--yes`는 대화형 선택기를 건너뜀 |
| 11 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 키바인딩 구성 파일을 열거나 생성 |
| 12 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 도구 권한의 allow, ask, deny 규칙 관리. 범위별로 규칙을 보고, 규칙을 추가하거나 제거하고, 작업 디렉터리를 관리하고, 최근 auto 모드 거부를 검토할 수 있는 대화형 대화상자 열기. 별칭: `/allowed-tools` |
| 13 | `/powerup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 애니메이션 데모가 포함된 빠른 대화형 레슨으로 Claude Code 기능 알아보기 |
| 14 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 개인정보 설정 보기 및 업데이트. Pro 및 Max 요금제 구독자만 사용 가능 |
| 15 | `/radio` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 브라우저에서 Claude FM lo-fi 라디오 열기. 브라우저가 없으면 스트림 URL을 출력. Bedrock, Vertex, Foundry에서는 사용 불가 |
| 16 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 샌드박스 모드 토글. 지원되는 플랫폼에서만 사용 가능 |
| 17 | `/scroll-speed` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 마우스 휠 스크롤 속도를 대화형으로 조정 |
| 18 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Claude Code의 상태 표시줄 구성. 원하는 것을 설명하거나, 인자 없이 실행해 셸 프롬프트에서 자동 구성 |
| 19 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Claude Code 스티커 주문 |
| 20 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Shift+Enter 및 기타 단축키를 위한 터미널 키바인딩 구성. VS Code, Cursor, Devin Desktop, Alacritty, Zed처럼 필요한 터미널에서만 표시됨 |
| 21 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 색상 테마 변경. 라이트 및 다크 변형, 색맹 접근성(daltonized) 테마, 터미널의 색상 팔레트를 사용하는 ANSI 테마, 터미널의 라이트/다크 모드를 따르는 "Auto (match terminal)" 옵션, 그리고 `~/.claude/themes/`나 플러그인에서 로드한 커스텀 테마 포함. "New custom theme…"를 선택해 직접 만들 수 있음 |
| 22 | `/tui [default\|fullscreen]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 터미널 UI 렌더러를 설정하고 대화를 그대로 유지한 채 재실행. `fullscreen`은 깜빡임 없는 alt-screen 렌더러를 활성화. 인자 없이 실행하면 활성 렌더러를 출력 |
| 23 | `/voice [hold\|tap\|off]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 음성 받아쓰기 토글, 또는 특정 모드로 활성화. Claude.ai 계정 필요 |
| 24 | `/autocompact [auto\|<tokens>]` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 자동 컴팩트 창 설정: Claude Code가 자동으로 컴팩트하기 전에 컨텍스트 창이 얼마나 차는지. `500k` 같은 토큰 수를 전달하거나, `auto`로 모델에 맞춰 튜닝된 기본값으로 복귀. Claude Code가 값을 설정에 저장하고 즉시 적용. 인자 없이 실행하면 현재 창을 보여주는 대화상자를 엶. v2.1.221+ 필요 |
| 25 | `/context [all]` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 현재 컨텍스트 사용량을 색상 그리드로 시각화. 컨텍스트를 많이 쓰는 도구, 메모리 팽창, 용량 경고에 대한 최적화 제안을 표시. `all`을 전달하면 전체 분석을 펼침 |
| 26 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | `/usage`의 별칭 |
| 27 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 프로젝트 영역, 상호작용 패턴, 마찰 지점을 포함해 Claude Code 세션을 분석하는 리포트 생성 |
| 28 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | `/usage`의 별칭. Stats 탭에서 열림 |
| 29 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 버전, 모델, 계정, 연결 상태를 보여주는 Settings 인터페이스(Status 탭) 열기. 세션이 백그라운드 작업(attached 또는 unattended)으로 실행 중인지 대화형으로 실행 중인지 보여주는 Session kind 행 포함. Claude가 응답하는 동안에도 현재 응답이 끝나기를 기다리지 않고 동작 |
| 30 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 세션 비용, 요금제 사용 한도, 활동 통계 표시. Pro, Max, Team, Enterprise 요금제에서는 스킬, 서브에이전트, 플러그인, MCP 서버별 사용량 분석 포함. `/cost`와 `/stats`가 별칭 |
| 31 | `/usage-credits` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 한도에 도달해도 계속 작업할 수 있도록 사용 크레딧 구성. 이전 `/extra-usage` |
| 32 | `/bug [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 버그 신고 또는 대화 공유. 포함할 세션 기록의 양을 선택하고 전송 전 동의 화면에서 확인. 별칭: `/share` |
| 33 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Claude Code에 대한 제품 피드백 전송. `/bug`와 동일한 대화상자를 엶 |
| 34 | `/heapdump` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 높은 메모리 사용량 진단을 위해 JavaScript 힙 스냅샷과 메모리 분석을 `~/Desktop`에 기록. 메모리 증가에 대한 버그 신고 시 유용 |
| 35 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 도움말과 사용 가능한 커맨드 표시 |
| 36 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 대화형 버전 선택기로 변경 로그 보기. 특정 버전을 선택하면 릴리스 노트를 보거나 모든 버전을 표시하도록 선택 |
| 37 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 백그라운드에서 실행 중인 모든 것을 보고 관리. `/bashes`로도 사용 가능 |
| 38 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 마지막 어시스턴트 응답을 클립보드에 복사. 숫자 `N`을 전달하면 N번째 최신 응답을 복사: `/copy 2`는 끝에서 두 번째를 복사. 코드 블록이 있으면 개별 블록이나 전체 응답을 선택하는 대화형 선택기를 표시. 선택기에서 `w`를 누르면 클립보드 대신 파일에 선택 내용을 기록하며, SSH에서 유용 |
| 39 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 현재 대화를 일반 텍스트로 내보내기. 파일명을 지정하면 해당 파일에 직접 기록. 지정하지 않으면 클립보드 복사나 파일 저장 대화상자를 엶 |
| 40 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 에이전트 구성 관리 지침 출력 — Claude에게 서브에이전트를 만들거나 관리하도록 요청하거나, `.claude/agents/` 또는 `~/.claude/agents/`를 직접 편집 |
| 41 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Claude in Chrome 설정 구성 |
| 42 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 도구 이벤트에 대한 훅 구성 보기 |
| 43 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | IDE 통합 관리 및 상태 표시 |
| 44 | `/mcp [reconnect <server>\|enable\|disable [<server>\|all]]` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | MCP 서버 연결과 OAuth 인증 관리. 인자 없이 실행하면 대화형 목록을 열고, `reconnect <server>`를 전달하면 연결이 끊긴 서버 하나를 재연결하고, `enable`/`disable`을 서버 이름이나 `all`과 함께 전달하면 대화상자를 열지 않고 연결 상태를 변경 |
| 45 | `/plugin [subcommand]` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Claude Code 플러그인 관리. 인자 없이 실행하면 플러그인 메뉴를 열고, `list`, `install`, `enable`, `disable` 같은 하위 커맨드를 전달하면 직접 동작 |
| 46 | `/reload-plugins [--force]` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 재시작 없이 대기 중인 변경을 적용하기 위해 모든 활성 플러그인 재로드. 재로드된 각 구성 요소의 개수를 보고하고 로드 오류를 표시. 재로드가 로드되는 MCP 도구를 변경하고 프롬프트 캐시를 무효화하는 경우, 커맨드가 경고하며 `--force`를 전달하지 않으면 건너뜀 |
| 47 | `/reload-skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 세션 중 디스크에서 추가되거나 변경된 스킬을 재시작 없이 사용할 수 있도록 스킬 및 커맨드 디렉터리를 재스캔. 사용 가능한 스킬 수와 추가/제거된 수를 보고 |
| 48 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 사용 가능한 스킬 목록. 입력해 필터링. `t`를 눌러 토큰 수로 정렬. `Space`를 눌러 스킬의 가시성을 순환(스킬 재정의를 통한 네 가지 상태)한 뒤 `Enter`로 저장 |
| 49 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | `CLAUDE.md` 메모리 파일 편집, 자동 메모리 활성화/비활성화, 자동 메모리 항목 보기 |
| 50 | `/advisor [model\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 작업 중 핵심 순간에 두 번째 모델에게 조언을 구하는 advisor 도구 활성화/비활성화. 모델 이름(`opus`, `sonnet`)이나 전체 모델 ID를 허용. 인자 없이 실행하면 선택기를 엶. `off`로 비활성화 |
| 51 | `/effort [low\|medium\|high\|xhigh\|max\|ultracode]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 모델 effort 수준 설정. 사용 가능한 수준은 모델에 따라 다르며 `low`, `medium`, `high`, `xhigh`, `max`(세션 한정), `ultracode`(`xhigh` 추론과 자동 워크플로 오케스트레이션 결합; 세션 한정)를 포함. 인자 없이 실행하면 수준을 선택하는 대화형 슬라이더를 엶. `auto`는 모델 기본값으로 초기화. 현재 응답이 끝나기를 기다리지 않고 즉시 적용됨 |
| 52 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 빠른 모드 켜기/끄기 토글 |
| 53 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | AI 모델을 전환하고 새 세션의 기본값으로 저장. 행에서 `s`를 누르면 현재 세션에만 전환. 지원하는 모델은 좌우 화살표로 effort 수준을 조정. 이전 출력이 있는 대화 도중 전환하는 경우, Claude가 변경을 적용하기 전에 경고 |
| 54 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 친구에게 Claude Code 무료 1주일 공유. 계정이 자격을 갖춘 경우에만 표시됨 |
| 55 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 프롬프트에서 바로 plan 모드 진입. 선택적 설명을 전달하면 plan 모드에 진입하고 즉시 해당 작업으로 시작. 예: `/plan fix the auth bug` |
| 56 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 현재 세션 동안 파일 접근을 위한 작업 디렉터리 추가. 대부분의 `.claude/` 구성은 추가된 디렉터리에서 발견되지 않음 |
| 57 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 커밋되지 않은 변경과 턴별 diff를 보여주는 대화형 diff 뷰어 열기. 좌우 화살표로 현재 git diff와 개별 Claude 턴을 전환하고, 상하로 파일을 탐색 |
| 58 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 프로젝트를 `CLAUDE.md` 가이드로 초기화. `CLAUDE_CODE_NEW_INIT=1`을 설정하면 스킬, 훅, 개인 메모리 파일까지 안내하는 대화형 흐름 진행 |
| 59 | `/review [PR]` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | `/code-review`의 별칭. 현재 diff, 또는 전달한 PR 번호, 브랜치, 경로를 리뷰. 동일한 effort 수준과 플래그를 허용. 심층 클라우드 리뷰에는 `/code-review ultra` 사용 |
| 60 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 현재 브랜치의 대기 중인 변경을 보안 취약점에 대해 분석. 브랜치와 origin의 기본 브랜치 간 diff를 리뷰하고 인젝션, 인증 문제, 데이터 노출 같은 위험을 식별. `origin` 리모트 필요 |
| 61 | `/team-onboarding` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Claude Code 사용 기록으로부터 팀 온보딩 가이드 생성. Claude가 지난 30일간의 세션, 커맨드, MCP 서버 사용을 분석해 마크다운 가이드를 생성. Pro, Max, Team, Enterprise 요금제의 claude.ai 구독자에게는 팀원이 Claude Code에서 바로 열 수 있는 공유 링크도 반환 |
| 62 | `/ultrareview [PR or branch]` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 클라우드 샌드박스에서 심층 멀티 에이전트 코드 리뷰 실행. 선호되는 호출 방식은 `/code-review ultra`이며, `/ultrareview`는 별칭으로 유지됨. Pro와 Max에서 무료 3회 실행 포함, 이후 사용 크레딧 필요 |
| 63 | `/autofix-pr [prompt]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 현재 브랜치의 PR을 감시하며 CI가 실패하거나 리뷰어가 코멘트를 남기면 수정을 푸시하는 Claude Code on the web 세션 생성. 체크아웃된 브랜치에서 `gh pr view`로 열린 PR을 감지. 다른 PR을 감시하려면 먼저 해당 브랜치를 체크아웃. `gh` CLI와 Claude Code on the web 접근 필요 |
| 64 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Claude Code Desktop 앱에서 현재 세션 계속하기. macOS와 Windows 전용. 별칭: `/app` |
| 65 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 저장소에 Claude GitHub App 설치, GitHub Actions 워크플로와 시크릿을 설정하는 선택 단계 포함 |
| 66 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Claude Slack 앱 설치. OAuth 흐름을 완료하기 위해 브라우저를 엶 |
| 67 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Claude 모바일 앱을 다운로드할 QR 코드 표시. 별칭: `/ios`, `/android` |
| 68 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 이 세션을 claude.ai에서 원격 제어할 수 있게 만들기. 별칭: `/rc` |
| 69 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 클라우드 에이전트의 기본 환경 선택 |
| 70 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 루틴 생성, 업데이트, 목록 조회, 실행. Claude가 대화형으로 설정을 안내. 별칭: `/routines` |
| 71 | `/teleport` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Claude Code on the web 세션을 이 터미널로 가져오기: 선택기를 열고 브랜치와 대화를 가져옴. `/tp`로도 사용 가능. claude.ai 구독 필요 |
| 72 | `/web-setup` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 로컬 `gh` CLI 자격 증명을 사용해 GitHub 계정을 Claude Code on the web에 연결. GitHub가 연결되지 않은 경우 `/schedule`이 자동으로 이를 요청 |
| 73 | `/background [prompt]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 현재 세션을 백그라운드 에이전트로 실행하도록 분리하고 이 터미널을 해제. 프롬프트를 전달하면 분리 전에 지시를 하나 더 보냄. `claude agents`로 세션을 모니터링. 별칭: `/bg` |
| 74 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 이 지점에서 현재 대화의 브랜치 생성 |
| 75 | `/btw [question]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 대화에 추가하지 않고 빠른 곁가지 질문하기. 인자 없이 실행하면 가장 최근 곁가지 질문의 오버레이를 다시 엶 |
| 76 | `/cd <path>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 프롬프트 캐시를 깨지 않고 세션을 새 작업 디렉터리로 이동 |
| 77 | `/clear [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 빈 컨텍스트로 새 대화 시작. 선택적 `name`을 전달하면 `/resume`으로 쉽게 검색하도록 이전 대화에 레이블을 지정. 같은 대화를 계속하면서 컨텍스트를 확보하려면 `/compact`를 대신 사용. 별칭: `/reset`, `/new` |
| 78 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 선택적 포커스 지시와 함께 대화 컴팩트 |
| 79 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | CLI 종료. attached 백그라운드 세션에서는 분리되며 세션은 계속 실행됨. 별칭: `/quit` |
| 80 | `/fork [prompt]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 현재 대화를 새 백그라운드 세션으로 복사하고 여기서 계속 작업 |
| 81 | `/goal [condition\|clear]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 목표 설정 — Claude가 조건이 충족될 때까지 여러 턴에 걸쳐 계속 작업. 인자 없이 실행하면 현재 또는 가장 최근에 달성한 목표를 표시. `clear`, `stop`, `off`, `reset`, `none`, `cancel`로 활성 목표를 조기에 제거 |
| 82 | `/list-agents` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | Claude가 메시지를 보낼 수 있는 서브에이전트와 다른 Claude Code 세션을 각각에 사용할 이름과 함께 나열. agent-team 팀원은 나열되지 않음. `/peers`로도 사용 가능. 교차 세션 메시징이 활성화된 경우에만 사용 가능 |
| 83 | `/recap` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 진행 중인 대화에 영향을 주지 않고 요청 시 현재 세션의 한 줄 요약 생성 |
| 84 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 현재 세션 이름을 바꾸고 프롬프트 바에 이름 표시. 이름 없이 실행하면 대화 기록에서 자동 생성 |
| 85 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | ID나 이름으로 대화를 재개하거나 세션 선택기를 엶. v2.1.144부터 백그라운드 세션이 선택기에 `bg`로 표시됨. 별칭: `/continue` |
| 86 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 대화 및/또는 코드를 이전 지점으로 되돌리거나, 선택한 메시지부터 요약. 체크포인팅 참고. 별칭: `/checkpoint`, `/undo` |
| 87 | `/stop` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 현재 백그라운드 세션 중지. 백그라운드 세션에 attached된 동안에만 사용 가능. 트랜스크립트와 워크트리는 유지됨. 중지 없이 분리하려면 `/exit`를 사용하거나 `←`를 누름 |
| 88 | `/subtask <task>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | forked 서브에이전트 생성: 전체 대화를 상속받아 사용자가 계속 작업하는 동안 작업을 수행하는 백그라운드 서브에이전트. 완료되면 결과가 이 대화로 반환됨 |
| 89 | `/workflows` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 실행 중이거나 완료된 워크플로를 보고, 일시정지하고, 재개하고, 저장하는 워크플로 진행 뷰 열기 |

`/debug` 같은 번들 스킬도 슬래시 커맨드 메뉴에 나타날 수 있지만, 내장 커맨드는 아닙니다.

---

## Sources

- [Claude Code Commands](https://code.claude.com/docs/en/commands)
- [Claude Code Interactive Mode](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
