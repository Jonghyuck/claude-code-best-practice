<!--
  이 문서는 tutorial/day0/README.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Day 0 — Claude Code Setup

이 가이드는 사용자의 머신에 Claude Code를 설치하고 인증하여 바로 사용을 시작할 수 있도록 안내합니다.

## Step 1: Install Claude Code

사용하는 운영체제를 선택하세요:

| OS | Guide |
|----|-------|
| Windows | [windows.md](windows.md) |
| Linux | [linux.md](linux.md) |
| macOS | [mac.md](mac.md) |

사용하는 운영체제에 맞는 가이드를 따른 다음, 인증을 위해 다시 이곳으로 돌아오세요.

---

## Step 2: Verify Installation

운영체제별 가이드를 따른 후, 모든 것이 정상적으로 작동하는지 확인하세요:

```bash
node --version    # Should show v18.x or higher
claude --version  # Should show the installed Claude Code version
```

---

## Step 3: Login

<img src="assets/login.png" alt="Claude Code login screen" width="50%">

터미널에서 `claude`를 실행하세요. 처음 실행하면 로그인 방법을 선택하라는 안내가 표시됩니다.

### Method 1: Subscription (Claude Pro / Max)

- **Claude.ai account**를 선택합니다
- 브라우저가 열립니다 — 로그인하고 권한을 승인하세요
- 터미널로 돌아오면 로그인이 완료됩니다

### Method 2a: API Key (Team Invite)

팀 관리자가 Anthropic 대시보드에서 사용자를 초대합니다.

- **초대 이메일**을 받게 됩니다 — 이를 수락하고 Anthropic 계정을 생성하세요
- 터미널에서 `claude`를 실행합니다
- **Anthropic API Key**를 선택합니다
- 대시보드에서 키가 **자동 생성**됩니다 — 별도의 수동 설정이 필요 없습니다
- Claude Code가 즉시 작동하기 시작합니다

### Method 2b: API Key (You have the key)

누군가 (Slack, 이메일 등을 통해) 키를 공유해 주었거나 직접 키를 생성한 경우:

- 터미널에서 `claude`를 실행합니다
- **Anthropic API Key**를 선택합니다
- 키를 붙여넣습니다 (`sk-ant-`로 시작)
- 키는 **영구적으로 저장**됩니다 — 다시 묻지 않습니다

---
