---
name: setup-copilot-global-skills
description: Configure global (personal) or custom centralized directory structures for GitHub Copilot Agent Skills, register them so VS Code discovers them, and submit new skills to a shared registry via the gh CLI. Use when a developer asks to set up a personal Copilot skills folder (~/.copilot/skills or ~/.agents/skills), wire VS Code's chat.agentSkillsLocations to an org's central skill registry, onboard a machine to a shared skills repo, or open a PR/issue contributing a skill upstream. Covers macOS, Linux, and Windows path differences.
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
argument-hint: "[global|custom] [skill-name] [registry-path]"
---

# Setup Copilot Global Skills

Configure where GitHub Copilot Agent Skills live and ensure VS Code can discover them. Two scopes are supported: **Global (Personal)** folders on the local machine, and **Custom (Centralized Registry)** repositories shared across a team and linked through VS Code settings.

## Storage models

Copilot Agent Skills are discovered from three kinds of location:

1. **Project-Level** — `.github/skills/`, `.claude/skills/`, or `.agents/skills/` at a repository root. Scoped to one repo. (Not configured by this skill; created per-project.)
2. **Global (Personal)** — `~/.copilot/skills/` (macOS/Linux) or `%USERPROFILE%\.copilot\skills\` (Windows). Available to the user across all repos. `~/.agents/skills/` is an accepted alternative on macOS/Linux.
3. **Custom (Centralized Registry)** — a shared, version-controlled repo cloned locally and linked via VS Code's `chat.agentSkillsLocations` array.

## Step 1 — Determine the scope

Ask the user (unless they already said): **Global/Personal** folder, or **Custom/Centralized Registry**?

- If they passed an argument (`global` or `custom`), use it.
- If ambiguous, ask before touching the filesystem or settings.

## Step 2 — Setup Global (Personal) skills

1. Detect the platform before running any command (macOS/Linux vs Windows) — paths differ.
2. Create the skill directory:
   - **macOS/Linux:** `mkdir -p ~/.copilot/skills/<skill-name>/` (or `~/.agents/skills/<skill-name>/` if the user prefers the `.agents` convention).
   - **Windows:** `mkdir %USERPROFILE%\.copilot\skills\<skill-name>\`
3. Scaffold a minimal `SKILL.md` inside the new directory (see `assets/SKILL.template.md`). Fill in `name` (must equal the folder name) and a real `description`.
4. Confirm the path back to the user. Global personal skills need no VS Code settings change — `~/.copilot/skills/` is auto-discovered.

## Step 3 — Setup Custom Location (Centralized Registry)

1. **Ask for the local path** to the central registry (e.g. `~/org-skills` or `my-org-skills-repo`).
2. **Verify the clone exists.** Before editing any settings, confirm the path is present and is a git repo:
   - `test -d "<path>/.git"` — if missing, offer to `git clone <remote> <path>` first. **Never** append a non-existent path to settings.
3. **Locate VS Code `settings.json`:**
   - macOS: `~/Library/Application Support/Code/User/settings.json`
   - Linux: `~/.config/Code/User/settings.json`
   - Windows: `%APPDATA%\Code\User\settings.json`
4. **Append the registry's skills directory** to the `chat.agentSkillsLocations` array (create the array if absent). Use the path the user gave, suffixed with `/skills`:
   ```json
   "chat.agentSkillsLocations": [
       "~/<user-provided-registry>/skills"
   ]
   ```
   - Read the file first, preserve existing entries, and de-duplicate — do not clobber other settings or repeat an existing path.

See `references/settings-paths.md` for the full path matrix and a worked settings.json edit.

## Step 4 — Enforce version control (Centralized only)

When the user is submitting a **new** skill to the centralized registry:

1. Create a branch, add the skill files, and commit.
2. Submit the contribution with the **`gh` CLI only**:
   - `gh pr create` to open a pull request, or
   - `gh issue create` to file a contribution request.

**Strict rule:** Never use the `github-create_pull_request` MCP tool (or any GitHub MCP tool) for this. Only the `gh` CLI is permitted. Confirm `gh auth status` is authenticated before pushing.

## Guardrails

- **Detect platform first.** Never run a macOS/Linux `mkdir -p` form on Windows or vice-versa.
- **Verify before linking.** A path must exist on disk before it is added to `chat.agentSkillsLocations`.
- **Preserve existing settings.** Read → merge → write; never overwrite `settings.json` wholesale.
- **gh CLI only** for any registry PR/issue — never a GitHub MCP tool.
- **No host-specific absolute paths** baked into scaffolded skill files; use `~/`, `%USERPROFILE%`, or relative references.

## Example

> User: "Set up my custom centralized registry."
> Agent: "What's the local path to the registry?" → User: "org-skills"
> Agent verifies `~/org-skills/.git` exists, then appends `"~/org-skills/skills"` to `chat.agentSkillsLocations`, preserving prior entries. Done.
