<!--
  이 문서는 reports/claude-skills-for-larger-mono-repos.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Understanding Claude Skills Discovery in Large Monorepos

모노레포에서 Claude Code로 작업할 때, 스킬이 어떻게 발견되어 컨텍스트에 로드되는지 이해하는 것은 프로젝트별 역량을 효과적으로 구성하는 데 매우 중요합니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## Important Difference from CLAUDE.md

**스킬은 CLAUDE.md 파일과 동일한 로딩 동작을 갖지 않습니다.** CLAUDE.md 파일은 디렉터리 트리를 위로(UP) 거슬러 올라가며(상위 로딩) 로드되지만, 스킬은 프로젝트 내부의 중첩된 디렉터리에 초점을 맞춘 다른 발견 메커니즘을 사용합니다.

## How Skills Are Discovered

### 1. Standard Skill Locations

스킬은 범위(scope)에 따라 다음의 고정된 위치에서 로드됩니다.

| Location | Path | Applies to |
|----------|------|------------|
| Enterprise | Managed settings | 조직 내 모든 사용자 |
| Personal | `~/.claude/skills/<skill-name>/SKILL.md` | 사용자의 모든 프로젝트 |
| Project | `.claude/skills/<skill-name>/SKILL.md` | 이 프로젝트만 |
| Plugin | `<plugin>/skills/<skill-name>/SKILL.md` | 플러그인이 활성화된 곳 |

### 2. Automatic Discovery from Nested Directories

하위 디렉터리의 파일로 작업할 때, Claude Code는 중첩된 `.claude/skills/` 디렉터리에서 스킬을 자동으로 발견합니다. 예를 들어 `packages/frontend/`에 있는 파일을 편집하고 있다면, Claude Code는 `packages/frontend/.claude/skills/`에서도 스킬을 찾습니다.

이는 패키지마다 고유한 스킬을 갖는 모노레포 구성을 지원합니다.

## Example Monorepo Structure

여러 패키지로 분리된 전형적인 모노레포를 살펴봅시다.

```
/mymonorepo/
├── .claude/
│   └── skills/
│       └── shared-conventions/SKILL.md    # Project-level skill
├── packages/
│   ├── frontend/
│   │   ├── .claude/
│   │   │   └── skills/
│   │   │       └── react-patterns/SKILL.md  # Frontend-specific skill
│   │   └── src/
│   │       └── App.tsx
│   ├── backend/
│   │   ├── .claude/
│   │   │   └── skills/
│   │   │       └── api-design/SKILL.md      # Backend-specific skill
│   │   └── src/
│   └── shared/
│       ├── .claude/
│       │   └── skills/
│       │       └── utils-patterns/SKILL.md  # Shared utilities skill
│       └── src/
```

## Scenario 1: Just Started Claude at Root (No Files Edited Yet)

`/mymonorepo/`에서 Claude Code를 실행했고 아직 아무 파일도 편집하지 않은 경우:

```bash
cd /mymonorepo
claude
# Just started - no files edited yet
```

| Skill | In Context? | Reason |
|-------|-------------|--------|
| `shared-conventions` | **Yes** | 루트 `.claude/skills/`에 있는 프로젝트 수준 스킬 |
| `react-patterns` | **No** | 발견되지 않음 — `packages/frontend/`의 파일로 아직 작업하지 않음 |
| `api-design` | **No** | 발견되지 않음 — `packages/backend/`의 파일로 아직 작업하지 않음 |
| `utils-patterns` | **No** | 발견되지 않음 — `packages/shared/`의 파일로 아직 작업하지 않음 |

## Scenario 2: After Editing Files in a Package

Claude에게 `packages/frontend/src/App.tsx`를 편집하도록 요청한 후:

| Skill | In Context? | Reason |
|-------|-------------|--------|
| `shared-conventions` | **Yes** | 루트 `.claude/skills/`에 있는 프로젝트 수준 스킬 |
| `react-patterns` | **Yes** | `packages/frontend/`의 파일을 편집할 때 발견됨 |
| `api-design` | **No** | 여전히 발견되지 않음 — `packages/backend/`의 파일로 아직 작업하지 않음 |
| `utils-patterns` | **No** | 여전히 발견되지 않음 — `packages/shared/`의 파일로 아직 작업하지 않음 |

**핵심 인사이트**: 중첩된 스킬은 해당 디렉터리의 파일로 작업할 때 **필요 시점에(on-demand)** 발견됩니다. 세션 시작 시 미리 로드되지 않습니다.

## Key Behavior: Description vs Full Content

스킬 설명(description)은 Claude가 무엇이 사용 가능한지 알 수 있도록 컨텍스트에 로드되지만, **전체 스킬 내용은 호출될 때만 로드됩니다.** 이는 중요한 최적화입니다.

- **Descriptions**: 항상 컨텍스트에 존재(문자 예산 범위 내)
- **Full content**: 스킬이 호출될 때 필요 시점에 로드됨

> Note: 스킬을 미리 로드한 서브에이전트는 다르게 동작합니다 — 전체 스킬 내용이 시작 시점에 주입됩니다.

## Priority Order (When Skills Share Names)

여러 수준에서 스킬 이름이 겹칠 경우, 우선순위가 높은 위치가 우선합니다.

| Priority | Location | Scope |
|----------|----------|-------|
| 1 (highest) | Enterprise | 조직 전체 |
| 2 | Personal (`~/.claude/skills/`) | 사용자의 모든 프로젝트 |
| 3 (lowest) | Project (`.claude/skills/`) | 이 프로젝트만 |

플러그인 스킬은 `plugin-name:skill-name` 네임스페이스를 사용하므로 다른 수준과 충돌하지 않습니다.

## Why This Design Works for Monorepos

- **패키지별 스킬이 격리된 상태로 유지됩니다** — `packages/frontend/`에서 작업하는 프런트엔드 개발자는 백엔드 스킬로 컨텍스트가 어수선해지지 않으면서 프런트엔드 전용 스킬을 얻습니다.

- **자동 발견으로 설정이 줄어듭니다** — 패키지 수준 스킬을 명시적으로 등록할 필요가 없습니다. 해당 디렉터리에서 작업할 때 발견됩니다.

- **컨텍스트가 최적화됩니다** — 처음에는 스킬 설명만 로드되고, 중첩된 스킬은 필요 시점에 발견됩니다.

- **팀이 각자의 스킬을 유지 관리할 수 있습니다** — 각 패키지 팀은 다른 팀과 조율하지 않고도 자신의 도메인에 특화된 스킬을 정의할 수 있습니다.

## Character Budget Considerations

스킬 설명은 문자 예산(기본값 15,000자)까지 컨텍스트에 로드됩니다. 많은 패키지와 스킬을 가진 대규모 모노레포에서는 이 한도에 도달할 수 있습니다.

- `/context`를 실행해 제외된 스킬에 대한 경고를 확인하세요
- `SLASH_COMMAND_TOOL_CHAR_BUDGET` 환경 변수를 설정해 한도를 늘리세요

## Best Practices

1. **공유 워크플로는 루트 `.claude/skills/`에 두세요** — 저장소 전반의 관례, 커밋 워크플로, 공유 패턴.

2. **패키지별 스킬은 패키지의 `.claude/skills/`에 두세요** — 해당 패키지 고유의 프레임워크별 패턴, 컴포넌트 관례, 테스트 유틸리티.

3. **위험한 스킬에는 `disable-model-invocation: true`를 사용하세요** — 배포나 파괴적인 스킬은 사용자의 명시적 호출을 요구해야 합니다.

4. **스킬 설명은 간결하게 유지하세요** — 설명은 항상 컨텍스트에 존재하므로(문자 예산까지), 장황한 설명은 컨텍스트 공간을 낭비합니다.

5. **스킬 이름에 네임스페이스를 사용하세요** — 혼동을 피하기 위해 패키지 이름을 접두사로 붙이는 것을 고려하세요(예: `frontend-review`, `backend-deploy`).

## Comparison: Skills vs CLAUDE.md Loading

| Behavior | CLAUDE.md | Skills |
|----------|-----------|--------|
| 상위 로딩 (디렉터리 트리를 위로) | Yes | No |
| 중첩/하위 발견 (디렉터리 트리를 아래로) | Yes (지연) | Yes (자동 발견) |
| 전역 위치 | `~/.claude/CLAUDE.md` | `~/.claude/skills/` |
| 프로젝트 위치 | `.claude/` 또는 저장소 루트 | `.claude/skills/` |
| 내용 로딩 | 전체 내용 | 설명만 (호출 시 전체) |

---

## Sources

- [Claude Code Documentation - Extend Claude with Skills](https://code.claude.com/docs/en/skills)
- [Claude Code Documentation - Automatic Discovery from Nested Directories](https://code.claude.com/docs/en/skills#automatic-discovery-from-nested-directories)
