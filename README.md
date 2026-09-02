# aerial-config-example

Starter template and example configuration repository for the [Aerial AI Assistant](https://github.com/azylman/aerial).

This repository demonstrates how to structure your private configuration, persona rules, custom skills, and optional sidecar services for Aerial.

---

## Repository Structure

```text
aerial-config-example/
├── config.yaml                   # Agent runtime options, git synchronization, and MCP servers
├── AGENTS.md                     # Persona instructions, communication style, and custom preferences
├── custom-skills/                # Drop-in custom skill runbooks automatically loaded into context
│   └── weather-query/
│       └── SKILL.md
├── docker-compose.override.yml   # Optional sidecar containers (e.g. Brave Search MCP server)
├── .env.example                  # Template for environment variables (do not commit .env)
└── .gitignore                    # Protects secrets and local files
```

---

## How to Use This Template

1. **Create your private repository**:
   - Create a **private** repository on GitHub (e.g. `your-username/my-aerial-config`).
   - Copy or fork the files from this repository into your private repo.

2. **Configure your options**:
   - **`config.yaml`**: Configure agent model, timeout, timezone, Discord system alert channel, admin allowlist (`admin_users`), channel interaction policies (`channels:` with `threads`, `channel`, or `ignore` modes), git sync repositories, and custom MCP server endpoints.
   - **`AGENTS.md`**: Customize Aerial's persona, communication tone, and operational guidelines.
   - **`custom-skills/`**: Add custom operational runbooks in `custom-skills/<skill-name>/SKILL.md`.
   - **`docker-compose.override.yml`**: Define additional sidecar containers or local MCP server instances (e.g., Brave Search MCP) connected to `aerial-net`.
   - **`.env.example`**: Use this template to create your `.env` file on the host machine.

3. **Connect to Aerial**:
   - In Aerial's `.env` file on your host machine, configure your repository URL and GitHub token:
     ```ini
     AERIAL_CONFIG_REPO_URL=https://github.com/your-username/my-aerial-config.git
     GITHUB_PAT=your_github_personal_access_token
     ```
   - When Aerial starts up, it automatically clones and synchronizes your configuration into `/share/aerial-config` and hot-reloads changes seamlessly!
