# skill-installer

A Claude skill that installs other skills from GitHub repositories. Say "install skill from [repo]" and get one-click install buttons — no terminal needed.

## What it does

1. Clones the GitHub repo
2. Finds all `SKILL.md` files
3. Lists available skills (name + description)
4. Packages selected skills as `.skill` archives
5. Presents "Save skill" buttons for one-click installation

## Install

### Option 1: Download .skill file

Download `skill-installer.skill` from [Releases](../../releases) and upload it in Claude (Customize > Skills > Upload).

### Option 2: Manual

Copy the `skill-installer/` folder to `~/.claude/skills/`:

```bash
git clone https://github.com/zhuoma250/skill-installer.git /tmp/si
cp -r /tmp/si/skill-installer ~/.claude/skills/
rm -rf /tmp/si
```

## Usage

Once installed, just tell Claude what you want:

```
帮我从 anthropics/skills 安装 canvas-design
```

```
Install the frontend-design skill from https://github.com/anthropics/skills
```

```
有没有做 GIF 的 skill？帮我找一个装上
```

The skill triggers automatically when you mention installing, downloading, or adding skills from GitHub.

## How it works

The skill teaches Claude a standardized workflow for packaging skills:

- Uses `git clone --depth 1` for fast downloads
- Reads YAML frontmatter to extract skill metadata
- Packages with correct `.skill` ZIP structure (skill folder at root)
- Uses `present_files` API to render install buttons in Cowork/Claude.ai
- Handles edge cases: missing SKILL.md, large repos, naming conflicts

## Compatibility

- Claude Code
- Claude Cowork
- Claude.ai (with skills support)

## Eval Results

Tested against 3 scenarios (URL install, multi-select, fuzzy search):

| Config | Pass Rate | Avg Time | Avg Tokens |
|--------|-----------|----------|------------|
| With skill | 89% | 51s | 38K |
| Without skill | 17% | 74s | 44K |

+72pp improvement in task completion quality.

## License

MIT
