# infra-skills

Agent skills for infrastructure, mesh networking, device control, and OS-level automation — Tailscale, mesh fleet ops, hardware bridging, and terminal runtime management.

---

## Installation

```bash
npx skills add https://github.com/theleostark/infra-skills
```

---

## Skills by Category

### Mesh & Network (5)

| Skill | Purpose |
|-------|---------|
| **mesh-ops** | OpenClaw mesh fleet — dispatch, scan, audit, topology |
| **mesh-audit** | Disk usage, projects, services, health across mesh |
| **mesh-scanner** | Scan mesh for resources, APIs, configs |
| **mesh-security** | Live security monitoring, hardening automation |
| **mesh-context** | Gather project context from across mesh devices |
| **tailscale-mesh** | Tailscale ACL policy, SSH modes, Serve/Funnel |

### Cloud & Storage (4)

| Skill | Purpose |
|-------|---------|
| **oracle-cloud** | OCI — Always Free ARM instances, networking |
| **cloud-drive-routing** | Map cloud, local, subscribed storage surfaces |
| **storage-ops** | Audit roots, tier hot/warm/cold, offload |
| **here-now** | Instant static web hosting |

### Device & Hardware (7)

| Skill | Purpose |
|-------|---------|
| **cua-driver** | Drive native macOS apps via accessibility tree |
| **shadow-adb** | Android screen reading via ADB |
| **shadow-airplay-skill** | Apple TV and AirPlay control from terminal |
| **shadow-clone** | Clone root-owned/signed apps safely |
| **echo-loop** | ECHO device monitoring loop |
| **xteink-firmware** | ESP32-C3 e-paper firmware management |
| **trmnl-plugin-dev** | TRMNL e-paper private plugins |

### Terminal & UI (6)

| Skill | Purpose |
|-------|---------|
| **shadow-tui** | Terminal UI runtime launcher and config |
| **shadow-cast-cli** | CLI surface adapter — ANSI panels, tables, badges |
| **iterm-mesh** | iTerm2 profiles, panes, split layouts, grids |
| **shadow-term** | iTerm2 profiles as structured system prompts |
| **friday-hud** | F.R.I.D.A.Y. HUD-style dashboards |
| **friday-unified** | Unified Friday personal assistant + HUD |
| **nnn-cli** | Neural Network Node CLI — WebNN ML in terminal |

### Security & Auth (3)

| Skill | Purpose |
|-------|---------|
| **bitwarden-browser** | Bitwarden vault access via `bw` CLI |
| **ai-fi-manager** | API key and secret management |
| **shadow-connect** | Codex ↔ Shadow Lab mesh connecting layer |

### System Ops (6)

| Skill | Purpose |
|-------|---------|
| **disk-guard** | Disk space emergency response |
| **mole** | Disk and RAM pressure response playbook |
| **launchd-mise-service** | mise-managed runtimes as macOS launchd services |
| **shadow-os-skills** | Symlink graph, filesystem topology, launchd |
| **async-boundary-queue** | Delegate work across VPN/device boundaries |
| **clawhip** | Notification gateway runtime for OpenClaw |

---

## License

MIT License

---

**Last Updated**: 2026-05-13 · **GitHub**: [@theleostark](https://github.com/theleostark)
