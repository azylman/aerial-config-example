# aerial-config-example

Starter template and example configuration repository for the [Aerial AI Assistant](https://github.com/azylman/aerial).

This repository demonstrates how to structure your private configuration, modular persona rules, channel policies, microservice configs, Homepage dashboard extensions, and Kubernetes GitOps overrides for Aerial.

---

## Architecture & Quickstart

Aerial is designed with a **Decoupled Multi-Repository Architecture**:
- **Core Engine Monorepo** ([`azylman/aerial`](https://github.com/azylman/aerial)): Monorepo containing core Go microservices (`eri`, `haro`, `coven`, `vanadis`, `synapse`), MCP servers, and turnkey base Kubernetes manifests (`k8s/`).
- **Private Configuration Repository** (this repository): Your private configuration containing `config.yaml`, personal rules, per-channel prompt directives, and environment-specific cluster overrides.

### Single-Node k3s Default Behavior
Out of the box, Aerial's base Kubernetes manifests deploy turnkey onto a **single-node vanilla k3s cluster**:
- Storage provisions automatically via k3s's built-in `local-path` storage provisioner.
- Single-instance CloudNativePG PostgreSQL (`Cluster/postgres` with 1 instance and preferred anti-affinity).
- Core services run unconstrained across available CPU and memory resources.
- Flux CD reconciles both `aerial-core` from upstream `azylman/aerial` and `aerial-config` from your private configuration repository.

---

## Repository Structure

```text
aerial-config-example/
├── config.yaml                   # Core runtime options, models, channels, microservices, and MCP servers
├── rules/                        # Modular instruction rules loaded into context
│   ├── common/
│   │   └── 01-identity.md        # User identity, household addresses, and aliases
│   ├── persona/
│   │   └── 01-persona.md         # Persona guidelines, tone of voice, and conversational style
│   └── discord/
│       └── 01-formatting.md      # Discord formatting constraints and code block hygiene
├── channels/                     # Per-channel instructions auto-discovered by channel name
│   ├── general.md
│   └── lounge.md
├── services/                     # Microservice configuration files mounted into cluster workloads
│   ├── eri/eri.yaml              # Eri execution engine configuration
│   ├── haro/haro.yaml            # Haro scheduler and heartbeat service configuration
│   ├── coven/coven.yaml          # Coven semantic and procedural memory configuration
│   ├── synapse/synapse.yaml      # Synapse Discord ingress and webhook router configuration
│   ├── homepage/homepage.yaml    # Homepage service configuration
│   └── victoriametrics/scrape.yml# Modular Prometheus metrics scrape configurations
├── homepage/                     # Homepage dashboard HUD configurations
│   ├── bookmarks.yaml            # Custom bookmark categories and quick links
│   ├── services.yaml             # External service cards and application links
│   ├── widgets.yaml              # Top-level search, weather, and system resource widgets
│   └── settings.yaml             # Layout column grids, theme, and color styling
├── k8s/                          # Kubernetes manifests and Flux GitOps overrides
│   ├── kustomization.yaml        # Kustomization referencing cluster-overrides.yaml and homepage ConfigMaps
│   └── cluster-overrides.yaml    # Flux GitRepositories and Kustomizations for aerial and aerial-config
├── custom-skills/                # Drop-in custom skill runbooks automatically loaded into context
│   └── weather-query/
│       └── SKILL.md
├── .github/                      # Continuous integration and automation
│   ├── workflows/
│   │   └── validate.yml          # Pre-merge YAML, Markdown, and Kubernetes validation
│   └── dependabot.yml            # Dependabot configuration for GitHub Actions
└── .gitignore                    # Local ignore rules
```

---

## Multi-Node HA & Cluster Overrides

When scaling beyond a single node into a multi-node homelab or high-availability environment, you do not need to fork or modify upstream manifests in `azylman/aerial`.

Instead, use `k8s/cluster-overrides.yaml`. Flux CD evaluates `Kustomization/aerial-core` with `spec.patches` to customize the base manifests during reconciliation.

### Available Overrides in `k8s/cluster-overrides.yaml`:
1. **High-Availability PostgreSQL (CloudNativePG)**:
   Uncomment the `Cluster/postgres` patch in `k8s/cluster-overrides.yaml` to scale PostgreSQL to 2 instances with strict pod anti-affinity across worker nodes:
   ```yaml
   - patch: |
       - op: replace
         path: /spec/instances
         value: 2
       - op: replace
         path: /spec/affinity/podAntiAffinityType
         value: required
     target:
       group: postgresql.cnpg.io
       version: v1
       kind: Cluster
       name: postgres
       namespace: default
   ```
2. **Workload Node Pinning (`nodeSelector`)**:
   Uncomment the deployment patches to pin AI inference or heavy execution daemons (e.g. `eri`, `coven`, `vanadis`) to dedicated compute or GPU nodes:
   ```yaml
   - patch: |
       apiVersion: apps/v1
       kind: Deployment
       metadata:
         name: eri
         namespace: default
       spec:
         template:
           spec:
             nodeSelector:
               kubernetes.io/hostname: compute-node-1
     target:
       group: apps
       version: v1
       kind: Deployment
       name: eri
       namespace: default
   ```
3. **Storage & Secret Mounts (NAS / SMB / NFS / Custom TLS)**:
   Mount external persistent volumes or custom certificates into workloads by declaring extra volumes and volumeMounts in `spec.patches`.

---

## Configuration Reference

### 1. Core Configuration (`config.yaml`)
- **`model`**: Primary high-effort LLM model for multi-step reasoning, coding, and complex tasks (e.g. `Gemini 3.8 Flash (High)`).
- **`low_effort_model`**: Fast, efficient model for ambient triage, classification, and background routines (e.g. `Gemini 3.8 Flash (Low)`).
- **`timezone`**: Timezone for scheduled cron routines and timestamp rendering (e.g. `America/Los_Angeles`).
- **`system_channel`**: Discord channel name or Snowflake ID for startup announcements and diagnostic alerts.
- **`admin_users`**: List of Discord user Snowflake IDs or Discord usernames authorized to perform system-level operations.
- **`vanadis_url`**, **`haro_url`**, **`coven_url`**, **`docket_url`**: Internal microservice endpoints resolved via cluster CoreDNS (`*.aerial`).
- **`channels`**: Interaction policies per Discord channel:
  - **`mode`**: `threads` (messages spawn/route to threads), `channel` (in-channel direct interaction), or `ignore` (completely ignored).
  - **`wake_mode`**: `classifier` (mentions, direct replies, and ambient relevance scoring), `mention` (strictly explicit @mentions or direct replies), or `all` (responds to every message).
  - **`ambient_wake_threshold`**: Confidence threshold (0.0 to 1.0) for ambient messages to wake Aerial (default: 0.80).
- **`mcp_servers`**: Remote Model Context Protocol (MCP) server endpoints reachable over the internal cluster mesh.

### 2. Persona & Modular Rules (`rules/`)
Rules are organized into modular categories (`rules/common/`, `rules/persona/`, `rules/discord/`) and automatically concatenated and synchronized into context. Keep each file under 22.5 KB (23,040 bytes).

### 3. Per-Channel Guidelines (`channels/<channel-name>.md`)
Place Markdown files named after your Discord channels (e.g., `channels/general.md` or `channels/lounge.md`). Aerial auto-discovers and injects them dynamically into turns for that channel. Threads automatically inherit instructions from their parent channel.

### 4. Homepage Dashboard Extensions (`homepage/`)
Aerial includes an integrated [Homepage](https://gethomepage.dev/) dashboard HUD. Files in `homepage/` cleanly extend the core dashboard:
- **`bookmarks.yaml`**: Add personal bookmark categories and links.
- **`services.yaml`**: Add external service links and custom monitoring widgets.
- **`widgets.yaml`**: Configure header widgets like OpenMeteo weather coordinates or resource monitors.
- **`settings.yaml`**: Customize dashboard column layouts and themes.

### 5. Modular Telemetry Scrapes (`services/victoriametrics/scrape.yml`)
Define custom Prometheus scrape targets in `services/victoriametrics/scrape.yml`. VictoriaMetrics dynamically discovers and scrapes these targets.

### 6. Custom Skills (`custom-skills/`)
Add specialized runbooks in `custom-skills/<skill-name>/SKILL.md`. Aerial discovers them automatically and exposes them via progressive disclosure.

---

## How to Use This Template

1. **Create your private repository**:
   - Create a **private** repository on GitHub (e.g. `your-username/my-aerial-config`).
   - Copy or fork the files from this repository into your private repo.

2. **Configure your options**:
   - Update `config.yaml` with your preferred models, admin users, and channel policies.
   - Adjust `rules/` with your preferred persona, communication style, and identity guidelines.
   - Add any desired channel instructions in `channels/`.
   - Customize `homepage/` dashboard services, bookmarks, and widgets.

3. **Deploy with Flux CD**:
   - In `k8s/cluster-overrides.yaml`, point `GitRepository/aerial-config` and `AERIAL_CONFIG_REPO_URL` to your private repository URL.
   - Apply `k8s/` or let Flux reconcile `cluster-overrides.yaml`.
   - When updates are pushed to your repository, Kubernetes `git-sync` sidecars and Flux CD automatically pull and hot-reload changes seamlessly!
