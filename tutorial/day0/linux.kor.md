<!--
  이 문서는 tutorial/day0/linux.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Linux Setup

[Back to Day 0](README.md)

## Prerequisites

**Node.js v18 이상**과 **npm**이 필요합니다.

## Step 1: Install Node.js

### Option A: Via nodejs.org Download Page with fnm (Recommended)

**fnm**(Fast Node Manager)은 Node.js가 공식적으로 권장하는 도구입니다. 빠르고 가벼우며, 나중에 필요할 때 Node 버전을 손쉽게 전환할 수 있습니다.

1. 브라우저를 열고 [nodejs.org/en/download](https://nodejs.org/en/download)로 이동합니다.

2. **"Get Node.js® vXX.XX.X (LTS) for __ using __ with __"**라고 표시된 드롭다운 줄이 보일 것입니다. 드롭다운을 다음과 같이 설정합니다.

   | Dropdown | Select |
   |----------|--------|
   | Version | **vXX.XX.X (LTS)** — 기본 LTS 버전을 그대로 두고 변경하지 마세요 |
   | OS | **Linux** |
   | Package Manager | **fnm** ("Recommended (Official)" 항목 아래) |
   | Package Format | **npm** — 기본값을 그대로 두세요 |

3. 페이지에 실행해야 할 정확한 명령어가 표시됩니다. 터미널을 열고 그 명령어를 복사해 붙여넣으세요. 대략 다음과 같은 형태일 것입니다.

   ```bash
   # Step 1 — Install fnm
   curl -fsSL https://fnm.vercel.app/install | bash

   # Step 2 — Restart your terminal or reload your shell profile
   source ~/.bashrc   # or: source ~/.zshrc (if you use zsh)

   # Step 3 — Install Node.js
   fnm install 24   # The page will show the exact version number
   ```

   > 버전 번호는 위와 다를 수 있습니다 — 항상 웹사이트에 표시된 값을 사용하세요.

4. **터미널을 닫았다가 다시 여세요**(또는 위의 `source` 명령을 실행하세요). 그래야 `fnm`, `node`, `npm`을 사용할 수 있습니다.

> **왜 fnm인가요?** Node.js 다운로드 페이지에서 "Recommended (Official)" 범주에 속해 있기 때문입니다. nvm과 마찬가지로 Node를 홈 디렉터리에 설치하므로 npm 전역 설치 시 `sudo`가 전혀 필요 없습니다 — 게다가 fnm은 (Rust로 작성되어) 훨씬 빠르며 Windows, macOS, Linux에서 동일하게 동작합니다.

### Option B: Using your distro's package manager

이 방법이 더 빠르지만 오래된 버전의 Node.js가 설치될 수 있습니다. **설치 후 버전을 확인하세요** — v18 미만이라면 Option A를 대신 사용하세요.

**Ubuntu / Debian:**

```bash
sudo apt update
sudo apt install -y nodejs npm

# Check the version
node --version   # Must be v18 or higher
```

**Fedora:**

```bash
sudo dnf install -y nodejs npm
```

**Arch Linux:**

```bash
sudo pacman -S nodejs npm
```

### Option C: NodeSource (Latest LTS via apt, no nvm)

nvm을 사용하지 않고 최신 LTS를 원하는 Ubuntu/Debian 사용자를 위한 방법입니다.

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
```

## Step 2: Verify Node.js

```bash
node --version
npm --version
```

둘 다 버전 번호를 출력해야 합니다. `node --version`은 반드시 v18.x 이상을 표시해야 합니다.

## Step 3: Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

> **권한 오류가 나나요?**
> - **fnm**이나 **nvm**을 사용한 경우: 이 오류는 발생하지 않아야 합니다. 활성화되어 있는지 확인하세요(`which node`가 `/usr/...`이 아니라 홈 디렉터리 내부 경로를 가리켜야 합니다).
> - 시스템 설치를 사용한 경우: `sudo npm install -g @anthropic-ai/claude-code`를 사용하거나, npm의 전역 디렉터리 권한을 수정하세요.
>   ```bash
>   mkdir -p ~/.npm-global
>   npm config set prefix '~/.npm-global'
>   echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
>   source ~/.bashrc
>   ```

## Step 4: Verify Claude Code

```bash
claude --version
```

Claude Code 버전이 출력되는 것을 확인할 수 있습니다. 이제 인증 설정을 위해 [README.md](README.md)로 돌아가세요.

---

## Notes

- **WSL (Windows Subsystem for Linux):** 이 가이드는 WSL 안에서도 동작합니다. WSL 터미널에서 이 단계들을 그대로 따라 하면 됩니다.
- **PATH 문제:** 설치 후 `claude`를 찾을 수 없다면, npm의 전역 bin이 PATH에 포함되어 있는지 확인하세요. `npm config get prefix`를 실행하고, 그 경로의 `bin/` 하위 디렉터리가 PATH에 포함되어 있어야 합니다.
