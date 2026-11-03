# quickfix-skill

A manual-only agent skill that makes the **smallest possible fix** (1 to a few lines) for a build/compile error. It checks the fix with a rebuild and a simple smoke run only. If a real fix needs bigger changes, it rolls back its edits and reports the options to you instead of doing a large refactor.

Works with **Claude Code** (`/quickfix`) and **Codex** (`$quickfix`).

## Install (user scope)

**Any agent, any OS (needs Node.js):**

```sh
npx skills add vb2007/quickfix-skill -g -a claude-code -a codex
```

**Claude Code: Linux / macOS / WSL / Git Bash**

```sh
curl -fsSL --create-dirs -o ~/.claude/skills/quickfix/SKILL.md https://raw.githubusercontent.com/vb2007/quickfix-skill/main/skills/quickfix/SKILL.md
```

**Claude Code: Windows PowerShell**

```powershell
$d="$HOME\.claude\skills\quickfix"; New-Item -ItemType Directory -Force $d | Out-Null
iwr https://raw.githubusercontent.com/vb2007/quickfix-skill/main/skills/quickfix/SKILL.md -OutFile "$d\SKILL.md"
```

**Codex: Linux / macOS / WSL / Git Bash**

```sh
u=https://raw.githubusercontent.com/vb2007/quickfix-skill/main/skills/quickfix; d=~/.agents/skills/quickfix
curl -fsSL --create-dirs -o $d/SKILL.md $u/SKILL.md -o $d/agents/openai.yaml $u/agents/openai.yaml
```

**Codex: Windows PowerShell**

```powershell
$u="https://raw.githubusercontent.com/vb2007/quickfix-skill/main/skills/quickfix"; $d="$HOME\.agents\skills\quickfix"
New-Item -ItemType Directory -Force "$d\agents" | Out-Null
iwr "$u/SKILL.md" -OutFile "$d\SKILL.md"; iwr "$u/agents/openai.yaml" -OutFile "$d\agents\openai.yaml"
```

**Keep it updated with git (Linux / macOS):** clone the repo once and symlink it, then run `git pull` to update.

```sh
git clone https://github.com/vb2007/quickfix-skill ~/.local/share/quickfix-skill
ln -s ~/.local/share/quickfix-skill/skills/quickfix ~/.claude/skills/quickfix   # Claude Code
ln -s ~/.local/share/quickfix-skill/skills/quickfix ~/.agents/skills/quickfix   # Codex
```

Restart the agent after you install. To uninstall, delete the `quickfix` folder.

## Usage

```text
/quickfix                          # Claude Code: finds the build command, runs it, fixes the first error
/quickfix npm run build            # use this build command
/quickfix <paste error output>     # start from this error

$quickfix                          # Codex: same, or pick it from /skills
$quickfix dotnet build
```

It replies in one of two ways:
- **`Fixed:`** the file and line it changed, the diff, and how it checked the fix.
- **`Not a quick fix:`** the cause and why the fix isn't small, plus 1–2 options with rough sizes. Its edits are rolled back, and it waits for you to choose.

The skill never hides errors to get a green build (`@ts-ignore`, `any`, disabling warnings, deleting code, and so on). It doesn't install tools or run the full test suite.

## License

[Unlicense](LICENSE) (public domain).
