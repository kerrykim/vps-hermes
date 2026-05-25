# vps-hermes

A Claude Code plugin that automates the full setup of [Hermes Agent](https://github.com/NousResearch/hermes-agent) on a VPS — from first SSH login to a running Hermes instance.

## What it does

Run `/vps-hermes-setup` in Claude Code and it will walk you through:

1. **Tailscale on local PC** — installs and connects your local machine to a tailnet
2. **VPS user creation** — SSHes in as root and creates a non-root user with sudo access
3. **Tailscale on VPS** — installs Tailscale on the VPS and joins the same tailnet
4. **SSH key setup** — generates a key pair and configures `~/.ssh/config` so you can connect with `ssh <alias>`
5. **Hermes install** — runs the official one-liner installer on the VPS

## Requirements

- A fresh VPS with root SSH access (IP address ready)
- Claude Code running on your local machine
- A Tailscale account (free tier works)

## Installation

```bash
# Add the local marketplace (one-time setup)
claude plugin marketplace add https://github.com/kerrykim/vps-hermes

# Install the plugin
claude plugin install vps-hermes
```

## Usage

Open Claude Code and type:

```
/vps-hermes-setup
```

Claude will ask for:
- VPS IP address
- Username to create
- Password
- SSH alias (e.g. `henry` → `ssh henry`)

Then it handles the rest, pausing at Tailscale auth steps so you can approve devices in your browser.

## Plugin structure

```
plugins/vps-hermes/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── vps-hermes-setup/
        └── SKILL.md
```

## License

MIT
