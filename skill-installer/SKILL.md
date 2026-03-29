---
name: skill-installer
description: "Installs Claude skills from GitHub repositories. Clones repos, finds SKILL.md files, packages them as .skill archives, and presents one-click install buttons. Use when the user wants to install, download, or add a skill from GitHub — triggers include 'install skill', 'add this skill', 'download from github', '帮我装skill', '安装skill', or any GitHub URL containing skills. Also triggers when browsing or searching for skills to install."
---

# Skill Installer

Automates finding, downloading, and installing Claude skills from GitHub. Turns a manual multi-step process (clone → find SKILL.md → copy to directory) into a one-click experience.

## Workflow

### 1. Parse the request

The user might provide:
- A full URL: `https://github.com/anthropics/skills`
- A shorthand: `anthropics/skills`
- A specific skill: "install frontend-design from anthropics/skills"
- A vague request: "find me a skill for testing web apps"

For vague requests, search GitHub for repos containing SKILL.md files first, then confirm with the user.

### 2. Clone the repository

```bash
git clone --depth 1 https://github.com/<owner>/<repo>.git /tmp/skill-install-<repo>
```

Always use `--depth 1` for speed. If clone fails (private repo, network issue), tell the user and suggest alternatives.

### 3. Discover available skills

```bash
find /tmp/skill-install-<repo> -name "SKILL.md" -type f
```

For each SKILL.md, read only the YAML frontmatter (name + description). Do not dump full SKILL.md contents into conversation — just show a summary:

> Found 3 skills:
> - **frontend-design**: Create distinctive, production-grade frontend interfaces...
> - **webapp-testing**: Toolkit for testing web applications using Playwright...
>
> Which ones do you want? (say "all" for everything)

If the user already specified which skill(s), skip selection.

### 4. Check for conflicts

Before packaging, check if the user already has a skill with the same name installed. Look at the available skills list in the current session. If a conflict exists, warn: "You already have **X** installed. This will replace it. Continue?"

### 5. Package as .skill files

This is the critical step. The `.skill` file is a zip archive with a specific structure.

**Required structure** — the zip must contain a folder named after the skill, with SKILL.md inside:

```
skill-name.skill (zip archive)
└── skill-name/
    ├── SKILL.md
    ├── scripts/     (if present)
    ├── examples/    (if present)
    ├── references/  (if present)
    └── assets/      (if present)
```

**Packaging commands:**

```bash
SKILL_NAME="frontend-design"
SKILL_SRC="/tmp/skill-install-<repo>/path/to/skill-dir"

# Create package with correct folder structure
PKG_DIR="/tmp/skill-pkg"
mkdir -p "$PKG_DIR/$SKILL_NAME"
cp -r "$SKILL_SRC"/* "$PKG_DIR/$SKILL_NAME"/

# Create .skill archive
cd "$PKG_DIR"
zip -r "/sessions/brave-focused-lamport/${SKILL_NAME}.skill" "$SKILL_NAME"/

# Clean up package dir
rm -rf "$PKG_DIR"
```

The folder name inside the zip should match the `name` field in the SKILL.md frontmatter. If they don't match, use the frontmatter name.

### 6. Present for one-click install

Use `present_files` to show the `.skill` file(s). This renders a card with a "Save skill" button.

```python
present_files([
    {"file_path": "/sessions/brave-focused-lamport/frontend-design.skill"},
    {"file_path": "/sessions/brave-focused-lamport/webapp-testing.skill"}
])
```

Present all selected skills in a single `present_files` call when installing multiple.

### 7. Clean up

```bash
rm -rf /tmp/skill-install-<repo>
```

Tell the user what was installed and briefly describe each skill's trigger (so they know how to use it).

## Edge Cases

**No SKILL.md found** — Repo might not be a skill repo. Check for AGENTS.md or README for clues, then tell the user.

**Deeply nested skills** — Some repos nest skills in subdirectories (e.g., `skills/frontend-design/SKILL.md`). The `find` command handles any depth.

**Very large repos (100+ skills)** — Show the first 20 with a note that more exist. Ask the user to narrow down by category or keyword.

**Non-standard SKILL.md** — If frontmatter is missing or malformed, skip that skill and note it in the summary.

## What NOT to do

- Never write directly to `~/.claude/skills/` — always use `.skill` + `present_files`
- Never install without showing the user what they're getting (name + description at minimum)
- Never dump full SKILL.md content into the conversation — keep it to frontmatter summaries
- Never clone without `--depth 1` — full history is never needed
