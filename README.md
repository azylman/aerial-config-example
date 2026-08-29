# aerial-config-example

Starter template and example configuration repository for the [Aerial AI Assistant](https://github.com/azylman/aerial).

This repository demonstrates how to structure your private configuration, persona rules, and custom skills for Aerial.

---

## Repository Structure

`	ext
aerial-config-example/
├── config.yaml               # Non-secret agent runtime options & sync settings
├── AGENTS.md                 # User persona rules, tone guidelines & instructions
├── custom-skills/            # Drop-in custom skills automatically loaded into context
│   └── smart-home/
│       └── SKILL.md
├── .env.example              # Template for private credentials (do not commit .env)
└── .gitignore                # Protects secrets and temp files
`

---

## How to Use This Template

1. **Create your private repository**:
   - Create a **private** repository on GitHub (e.g. your-username/my-aerial-config).
   - Copy or fork the files from this repository into your private repo.

2. **Configure your options**:
   - Edit config.yaml to set your desired model, timezone, and system alert channel.
   - Edit AGENTS.md to customize Aerial's personality, tone, and private operational guidelines.
   - Add any custom runbooks into custom-skills/<skill-name>/SKILL.md.

3. **Connect to Aerial**:
   - In Aerial's .env file on your host machine, set:
     `ini
     AERIAL_CONFIG_REPO_URL=https://github.com/your-username/my-aerial-config.git
     GITHUB_PAT=your_github_personal_access_token
     `
   - When Aerial boots up, it will automatically clone and sync your private repository into /share/aerial-config and hot-reload changes on the fly!
