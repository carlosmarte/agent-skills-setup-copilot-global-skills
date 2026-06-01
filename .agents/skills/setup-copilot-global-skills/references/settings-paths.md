# VS Code settings & skill path matrix

## Global (Personal) skill directories

| Platform     | Primary path                          | Alternative                 |
|--------------|---------------------------------------|-----------------------------|
| macOS/Linux  | `~/.copilot/skills/<skill-name>/`     | `~/.agents/skills/<name>/`  |
| Windows      | `%USERPROFILE%\.copilot\skills\<name>\` | —                         |

Personal skills in these locations are auto-discovered — no settings change needed.

## VS Code `settings.json` location

| Platform | Path                                                      |
|----------|-----------------------------------------------------------|
| macOS    | `~/Library/Application Support/Code/User/settings.json`   |
| Linux    | `~/.config/Code/User/settings.json`                       |
| Windows  | `%APPDATA%\Code\User\settings.json`                       |

## Linking a centralized registry

Add the registry's `skills` directory to `chat.agentSkillsLocations`. Read the file,
merge into any existing array, de-duplicate, and write back — never replace the whole file.

Before:
```json
{
  "editor.fontSize": 13
}
```

After (registry path `~/org-skills`):
```json
{
  "editor.fontSize": 13,
  "chat.agentSkillsLocations": [
    "~/org-skills/skills"
  ]
}
```

If `chat.agentSkillsLocations` already exists, append the new entry rather than replacing:
```json
"chat.agentSkillsLocations": [
  "~/existing-registry/skills",
  "~/org-skills/skills"
]
```

## Preconditions checklist

- [ ] Registry path exists on disk and contains `.git` (clone first if not).
- [ ] `settings.json` read before edit; existing keys preserved.
- [ ] New path de-duplicated against existing entries.
- [ ] `gh auth status` authenticated before any `gh pr create` / `gh issue create`.
