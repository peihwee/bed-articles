# Setup Environment (Pre-step)

Complete this guide before [express-server.md](./express-server.md).

Run all commands below from your project root folder (the folder that contains `package.json`) unless stated otherwise.

## 1) Install Visual Studio Code

1. Go to https://code.visualstudio.com/
2. Download VS Code for Windows.
3. Install with default options.
4. Open VS Code once installation is complete.

Why: VS Code gives you an editor, terminal, and extensions in one place.

## 1.1) Set VS Code integrated terminal to PowerShell

1. Open VS Code.
2. Press `Ctrl + Shift + P`.
3. Run `Terminal: Select Default Profile`.
4. Choose `PowerShell`.
5. Open a new terminal (`Terminal` -> `New Terminal`).

Quick check in the terminal:

```powershell
$PSVersionTable.PSVersion
```

Why: this guide uses PowerShell commands, so matching terminal profile avoids command syntax issues.

## 1.2) Open terminal directly at the project root

When you need to run project commands, open terminal in the root folder:

1. In VS Code Explorer, right-click the root project folder.
2. Click `Open in Integrated Terminal`.

If you cannot find the Explorer icon, open it with:

- `Ctrl + Shift + E` (Explorer shortcut)
- or `Ctrl + Shift + P`, then run `View: Show Explorer`

Useful integrated terminal shortcuts:

- ``Ctrl + ` ``: toggle terminal panel
- ``Ctrl + Shift + ` ``: create new terminal

If you cannot find the backtick key (`), use this keyboard reference image:

![Backtick key reference](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ_7V3Rlmv1yph87WNJkKBVmFT-ukD9E_s5Sg&s)

You can confirm you are at the root with:

```powershell
Get-Location
```

Why root folder is important:

1. `npm install`, `npm run dev`, and `npm uninstall` use the nearest `package.json`.
2. Running npm commands in the wrong folder can install packages in the wrong place.
3. Git commands should run in the repository root so staging/commit/sync targets the correct project.
4. Tooling files like `.gitignore`, `.env`, and `package-lock.json` are managed from root.

## 2) Install Node.js

1. Go to https://nodejs.org/
2. Install the LTS version.
3. Open PowerShell and verify:

```powershell
node -v
npm -v
```

Why: Node.js runs your JavaScript server and npm manages packages.

## 3) Install Git SCM

1. Go to https://git-scm.com/downloads
2. Install Git for Windows.
3. Verify:

```powershell
git --version
```

Why: Git tracks code history and lets you collaborate.

## 4) Install GitHub CLI

1. Go to https://cli.github.com/
2. Install GitHub CLI (`gh`).
3. Verify:

```powershell
gh --version
```

Why: `gh` makes GitHub tasks faster from terminal (auth, clone, PRs).

## 5) Set up Git identity and credentials

Set your commit identity:

```powershell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Set credential helper (Windows):

```powershell
git config --global credential.helper manager-core
```

Log in to GitHub CLI:

```powershell
gh auth login
```

Why: this avoids typing username/password repeatedly and ensures commits show the correct author.

## 6) Clone the repository you are given

If your trainer gives you a GitHub URL, clone with either command:

```powershell
git clone https://github.com/ORG/REPO.git
```

or

```powershell
gh repo clone ORG/REPO
```

Then enter the project folder:

```powershell
cd REPO
```

Tip: replace `REPO` with the actual folder name created by clone.

## 7) Create `.gitignore` and understand why

Create `.gitignore` in the project root:

```gitignore
node_modules/
.env
```

Why `.gitignore` matters:

1. Prevents committing files that are local-only or sensitive.
2. Keeps repository clean and small.
3. Avoids environment-specific conflicts between teammates.

Important: `.gitignore` only affects untracked files. If `.env` or `node_modules` was already committed before, remove it from tracking once:

```powershell
git rm -r --cached node_modules .env
```

## 8) What is `.env` and why ignore it

`.env` stores environment variables such as:

```env
PORT=3000
DB_PASSWORD=super_secret
API_KEY=abcd1234
```

Why ignore `.env`:

1. It often contains secrets.
2. Secrets should never be committed to Git history.
3. Each developer/environment can have different values.

## 9) What is `node_modules` and why ignore it

`node_modules` is the folder where npm installs dependencies.

Why ignore `node_modules`:

1. It is large and can contain thousands of files.
2. It can be rebuilt from `package.json` + `package-lock.json`.
3. Committing it makes the repository slow and noisy.

## 10) Practice dependency workflow (install, reinstall, uninstall)

### A) Install one simple module

`dayjs` is a small JavaScript date/time library. It helps you format and calculate dates easily.

For this beginner practice, use a module that shows color in terminal output:

Example module: `chalk`

```powershell
npm install chalk
```

This updates:

1. `package.json`
2. `package-lock.json`
3. `node_modules/`

Try it quickly:

1. Create `test-color.js`
2. Make sure your project is in ES Module mode:

```powershell
npm pkg set type="module"
```

```json
{
	"type": "module"
}
```

Optional check:

```powershell
npm pkg get type
```

This is required because `import chalk from 'chalk';` uses ESM `import` syntax.

If you do not set `type` to `module`, use one of these alternatives:

1. Rename file to `test-color.mjs`, or
2. Use CommonJS syntax (`const chalk = require('chalk')`) in a CommonJS project.

3. Paste:

```js
import chalk from 'chalk';

console.log(chalk.green('Success message in green'));
console.log(chalk.yellow('Warning message in yellow'));
console.log(chalk.red('Error message in red'));
```

4. Run:

```powershell
node test-color.js
```

### B) Delete `node_modules` and reinstall dependencies

Delete folder:

```powershell
Remove-Item -Recurse -Force node_modules
```

Install back using npm install:

```powershell
npm i
```

This recreates `node_modules` from your project dependency files.

### C) Uninstall module

```powershell
npm uninstall chalk
```

This removes it from `package.json`, lockfile, and `node_modules`.

## 11) Faster clone and sync using Visual Studio Code

Official reference (recommended):

https://code.visualstudio.com/docs/sourcecontrol/quickstart

### A) Fast clone in VS Code UI

1. Open VS Code.

> **Important:** Open a new/empty VS Code window first (`File` -> `New Window`).
> The clone option is easiest to see in an empty window (Welcome page or Command Palette).

2. Open Source Control (`Ctrl + Shift + G`).
3. Click `Clone Repository`.
4. Paste the repository URL given to you.
5. Choose local folder.
6. Click `Open` when clone finishes.

If you do not see `Clone Repository`, use Command Palette:

- Press `Ctrl + Shift + P`
- Run `Git: Clone`

### B) Fast sync in Source Control panel

1. Open Source Control (`Ctrl + Shift + G`).
2. Review changed files.
3. Enter commit message.
4. Click `Commit`.
5. Click `Sync Changes` (or `Push` / `Pull` buttons).

### C) Recommended quick workflow

1. Pull latest changes before you start.
2. Make edits.
3. Commit small logical changes.
4. Sync immediately after each commit.
5. Resolve conflicts early if prompted.

---

## Next step

Continue with the server setup guide:

[express-server.md](./express-server.md)
