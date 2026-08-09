<!--
  이 문서는 best-practice/claude-skills.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Skills Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Jul%2011%2C%202026%2010%3A02%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.207-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-skills-implementation.md)

Claude Code 스킬 — 프론트매터 필드와 공식 번들 스킬.

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
| `name` | string | No | 표시 이름이자 `/slash-command` 식별자. 생략하면 디렉터리 이름을 기본값으로 사용 |
| `description` | string | Recommended | 스킬이 하는 일. 자동완성에 표시되며 Claude의 자동 발견에 사용됨 |
| `when_to_use` | string | No | Claude가 스킬을 언제 호출해야 하는지에 대한 추가 컨텍스트 — 트리거 문구와 예시 요청. 스킬 목록에서 `description`에 이어 붙으며, 1,536자 제한에 포함됨 |
| `argument-hint` | string | No | 자동완성 중 표시되는 힌트(예: `[issue-number]`, `[filename]`) |
| `arguments` | string/list | No | 스킬 내용에서 `$name` 치환에 쓰이는 이름 붙은 위치 인자. 공백으로 구분된 문자열 또는 YAML 리스트를 허용 — 이름은 순서대로 인자 위치에 매핑됨 |
| `disable-model-invocation` | boolean | No | Claude가 이 스킬을 자동으로 호출하지 못하게 하려면 `true`로 설정 |
| `user-invocable` | boolean | No | `false`로 설정하면 `/` 메뉴에서 숨김 — 스킬이 배경 지식 전용이 되며, 에이전트 프리로딩을 위한 용도 |
| `allowed-tools` | string | No | 이 스킬이 활성일 때 권한 프롬프트 없이 허용되는 도구 |
| `disallowed-tools` | string/list | No | 스킬이 활성인 동안 Claude가 사용할 수 있는 도구 풀에서 제거되는 도구(예: 백그라운드 루프에서 `AskUserQuestion` 차단). 공백/쉼표로 구분된 문자열 또는 YAML 리스트를 허용 — 제한은 다음 메시지에서 해제됨 |
| `model` | string | No | 이 스킬이 실행될 때 사용할 모델(예: `haiku`, `sonnet`, `opus`) |
| `effort` | string | No | 호출 시 모델 노력 수준을 재정의(`low`, `medium`, `high`, `xhigh`, `max`) |
| `context` | string | No | `fork`로 설정하면 스킬을 격리된 서브에이전트 컨텍스트에서 실행 |
| `agent` | string | No | `context: fork`가 설정된 경우의 서브에이전트 유형(기본값: `general-purpose`) |
| `hooks` | object | No | 이 스킬로 범위가 한정된 라이프사이클 훅 |
| `paths` | string/list | No | 스킬이 자동 활성화되는 시점을 제한하는 glob 패턴. 쉼표로 구분된 문자열 또는 YAML 리스트를 허용 — Claude는 매칭되는 파일을 다룰 때만 스킬을 로드함 |
| `shell` | string | No | `` !`command` `` 블록에 쓰이는 셸 — `bash`(기본값) 또는 `powershell`. `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`이 필요함 |

---

## ![Official](../!/tags/official.svg) **(13)**

| # | Skill | Description |
|---|-------|-------------|
| 1 | `code-review` | 선택한 노력 수준에서 현재 diff의 정확성 버그를 검토(low/medium: 확신도 높은 소수의 발견, high→max: 더 넓은 범위) — `--comment`는 발견 내용을 인라인 PR 코멘트로 게시 |
| 2 | `batch` | 여러 파일에 걸쳐 명령을 일괄 실행 |
| 3 | `debug` | 실패하는 명령이나 코드 문제를 디버깅 |
| 4 | `loop` | 프롬프트나 슬래시 명령을 반복 주기로 실행(최대 3일) |
| 5 | `claude-api` | Claude API 또는 Anthropic SDK로 앱을 구축 — `anthropic` / `@anthropic-ai/sdk` import에서 트리거됨 |
| 6 | `fewer-permission-prompts` | 트랜스크립트에서 자주 쓰이는 읽기 전용 Bash/MCP 호출을 스캔해 `.claude/settings.json`에 우선순위가 매겨진 허용 목록을 추가하여 권한 프롬프트를 줄임 |
| 7 | `run` | 프로젝트의 앱을 실행·구동해 변경 사항이 실제 앱에서 동작하는지 확인(테스트만이 아님). v2.1.145 필요 |
| 8 | `verify` | 앱을 빌드·실행하여 코드 변경이 의도대로 동작하는지, 테스트나 타입 체크로 대체하지 않고 확인. v2.1.145 필요 |
| 9 | `run-skill-generator` | `/run`과 `/verify`에 프로젝트를 빌드·구동하는 방법을 가르침 — 프로젝트별 실행 레시피를 `.claude/skills/run-<name>/`에 기록. v2.1.145 필요 |
| 10 | `simplify` | 변경된 코드에서 정리 기회(재사용, 단순화, 효율성, 추상화 수준)를 검토하며, 4개의 검토 에이전트를 병렬로 실행. v2.1.154부터는 정확성 버그를 찾지 **않음** — 그것은 `/code-review`를 사용 |
| 11 | `design-sync` | 리포지토리의 React 디자인 시스템을 변환해 Claude Design에 업로드 — 디자인 시스템 이름을 선택적으로 지정 가능(예: `/design-sync Acme DS`). 최초 동기화는 모든 컴포넌트를 검증하며 대규모 리포에서는 수 시간이 걸릴 수 있음. Anthropic API에서만 사용 가능(Bedrock, Google Cloud Agent Platform, Microsoft Foundry에서는 불가) |
| 12 | `dataviz` | 접근성 있고 일관된 시각화를 위한 색상 팔레트 검증기와 함께 차트, 그래프, 대시보드를 디자인 — 어떤 출력 매체든 차트, 그래프, 플롯, 데이터 시각화 요청에서 트리거됨. v2.1.198에 도입 |
| 13 | `doctor` | Claude Code 구성에 대한 설정/상태 점검. `disableBundledSkills`에서 면제되는 유일한 번들 스킬 — 해당 설정이 켜져 있어도 계속 입력 가능. v2.1.205에서 내장 명령에서 번들 스킬로 재분류됨 |

See also: [Official Skills Repository](https://github.com/anthropics/skills/tree/main/skills) for community-maintained installable skills.

---

## Sources

- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Skills Discovery in Monorepos](../reports/claude-skills-for-larger-mono-repos.md)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
