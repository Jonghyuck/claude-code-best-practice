<!--
  이 문서는 best-practice/claude-power-ups.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Power-ups Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Apr%2002%2C%202026-white?style=flat&labelColor=555)

애니메이션 데모와 함께 Claude Code 기능을 알려 주는 인터랙티브 레슨입니다. 각 power-up은 대부분의 사람이 놓치는 Claude Code의 기능 하나를 가르쳐 줍니다. v2.1.90에서 도입되었습니다.

<table width="100%">
<tr>
<td><a href="../">← Claude Code Best Practice로 돌아가기</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Usage

```bash
claude
/powerup
```

---

## Power-ups (10)

<p align="center">
  <img src="assets/claude-power-ups/powerup-menu.png" alt="Power-ups menu showing 10 lessons" width="700">
</p>

| # | Power-up | 주제 |
|---|----------|--------|
| 1 | Talk to your codebase | `@` 파일, 라인 참조 |
| 2 | Steer with modes | `shift+tab`, plan, auto |
| 3 | Undo anything | `/rewind`, `Esc-Esc` |
| 4 | Run in the background | 태스크, `/tasks` |
| 5 | Teach Claude your rules | `CLAUDE.md`, `/memory` |
| 6 | Extend with tools | MCP, `/mcp` |
| 7 | Automate your workflow | 스킬, 훅 |
| 8 | Multiply yourself | 서브에이전트, `/agents` |
| 9 | Code from anywhere | `/remote-control`, `/teleport` |
| 10 | Dial the model | `/model`, `/effort` |

---

## Example: Dial the model

마지막 power-up은 애니메이션 데모로 모델 전환과 effort 제어를 가르쳐 줍니다.

<p align="center">
  <img src="assets/claude-power-ups/dial-the-model-1.png" alt="Dial the model — demo thinking deeply" width="700">
</p>

<p align="center">
  <img src="assets/claude-power-ups/dial-the-model-2.png" alt="Dial the model — demo showing hypotheses" width="700">
</p>

<p align="center">
  <img src="assets/claude-power-ups/dial-the-model-3.png" alt="Dial the model — demo setting effort to high" width="700">
</p>

---

## Sources

- [Changelog — v2.1.90](https://code.claude.com/docs/en/changelog)
