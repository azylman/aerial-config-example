# aerial-config-example

Starter template and example configuration repository for the [Aerial AI Assistant](https://github.com/azylman/aerial).

This repository demonstrates how to structure your private configuration, persona rules, channel policies, custom skills, Homepage dashboard extensions, and optional sidecar services for Aerial.

---

## Repository Structure

```text
aerial-config-example/
├── config.yaml                   # Agent models, channel policies, git sync, and MCP servers
├── AGENTS.md                     # Persona instructions, communication style, and user preferences
├── channels/                     # Convention auto-discovered per-channel Markdown guidelines
│   ├── general.md
│   └── lounge.md
├── homepage/                     # Homepage dashboard customizations (two-repo extension model)
│   ├── bookmarks.yaml            # Custom bookmark categories and quick links
│   ├── services.yaml             # External services and application cards
│   ├── widgets.yaml              # Top-level search, weather, and system resource widgets
│   └── settings.yaml             # Layout column grids, theme, and color styling
├── victoriametrics/              # Modular Prometheus metrics scrape configurations
│   └── scrape-example.yml        # Auto-discovered and live-reloaded by VictoriaMetrics TSDB
├── custom-skills/                # Drop-in custom skill runbooks automatically loaded into context
│   └── weather-query/
│       └── SKILL.md
├── docker-compose.override.yml   # Optional sidecar containers (e.g. Brave Search MCP server)
├── .github/                      # Continuous integration workflows
│   └── workflows/
│       └── validate.yml          # Pre-merge YAML syntax, schema, and persona validation
├── .env.example                  # Template for environment variables (do not commit .env)
└── .gitignore                    # Protects secrets and local environment files
```

---

## Configuration Layers & Options

### 1. Core Configuration (`config.yaml`)
- **`model`**: Primary high-effort LLM model for multi-step reasoning, coding, and complex tasks (e.g. `Gemini 3.8 Flash (High)`).
- **`low_effort_model`**: Fast, efficient model for ambient triage, classification, and background routines (e.g. `Gemini 3.8 Flash (Low)`).
- **`timezone`**: Timezone for scheduled cron routines and timestamp rendering (e.g. `America/Los_Angeles`).
- **`system_channel`**: Discord channel name or Snowflake ID for startup announcements and diagnostic alerts.
- **`admin_users`**: List of Discord user Snowflake IDs or Discord usernames authorized to perform system-level operations.
- **`channels`**: Interaction policies per Discord channel:
  - **`mode`**: `threads` (messages spawn/route to threads), `channel` (in-channel direct interaction), or `ignore` (completely ignored).
  - **`wake_mode`**: `classifier` (direct mentions, replies, keywords, and ambient scoring), `mention` (strictly explicit @mentions or direct replies), or `all` (responds to every message).
  - **`ambient_wake_threshold`**: Confidence threshold (0.0 to 1.0) for ambient messages to wake Aerial (default: 0.80).
  - **`ambient_wake_prompt`**: Custom prompt criteria for the ambient classifier in that channel.
  - **`hooks`**: Optional channel lifecycle webhooks (`on_wake`, `pre_turn`, `post_turn`) to integrate external sidecars or APIs.
- **`git_sync`**: Multi-repository synchronization settings.
- **`mcp_servers`**: Additional remote Model Context Protocol (MCP) server endpoints over Streamable HTTP / SSE.

### 2. Persona & Style Overrides (`AGENTS.md`)
Customize Aerial's personality, tone of voice, operational boundaries, and response style. Instructions here take precedence over base engine rules.

### 3. Per-Channel Guidelines (`channels/<channel-name>.md`)
Place Markdown files named after your Discord channels (e.g., `channels/general.md` or `channels/lounge.md`). Aerial auto-discovers and injects them dynamically into turns for that channel. Threads automatically inherit instructions from their parent channel.

### 4. Homepage Dashboard Extensions (`homepage/`)
Aerial includes an integrated [Homepage](https://gethomepage.dev/) dashboard HUD. Files in `homepage/` cleanly extend the core dashboard:
- **`bookmarks.yaml`**: Add personal bookmark categories and links.
- **`services.yaml`**: Add external service links and custom monitoring widgets.
- **`widgets.yaml`**: Configure header widgets like OpenMeteo weather coordinates or resource monitors.
- **`settings.yaml`**: Customize dashboard column layouts and themes.

### 5. Modular Telemetry Scrapes (`victoriametrics/`)
Define custom Prometheus scrape targets in `victoriametrics/*.yml`. VictoriaMetrics dynamically discovers and live-reloads these configs every 15 seconds without container restarts.

### 6. Custom Skills (`custom-skills/`)
Add specialized runbooks in `custom-skills/<skill-name>/SKILL.md`. Aerial discovers them automatically and exposes them via progressive disclosure.

### 7. Custom Sidecars (`docker-compose.override.yml`)
Define extra containers, local MCP servers, or background services connected to the `aerial-net` bridge network. Docker Compose natively includes and merges this file on the host.

---

## How to Use This Template

1. **Create your private repository**:
   - Create a **private** repository on GitHub (e.g. `your-username/my-aerial-config`).
   - Copy or fork the files from this repository into your private repo.

2. **Configure your options**:
   - Update `config.yaml` with your preferred models, admin users, and channel policies.
   - Adjust `AGENTS.md` with your preferred persona and instructions.
   - Add any desired channel instructions in `channels/`.
   - Customize `homepage/` dashboard services, bookmarks, and widgets.
   - Use `.env.example` as a reference for your host `.env` file.

3. **Connect to Aerial**:
   - In Aerial's `.env` file on your host machine, configure your repository URL and GitHub token:
     ```ini
     AERIAL_CONFIG_REPO_URL=https://github.com/your-username/my-aerial-config.git
     GITHUB_PAT=your_github_personal_access_token
     ```
   - When Aerial starts up, `aerial-brain` and `aerial-hangar` automatically clone and synchronize your configuration into `/share/aerial-config` and hot-reload changes seamlessly!
