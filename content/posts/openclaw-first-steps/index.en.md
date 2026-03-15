---
title: "OpenClaw first steps: installation, basic usage, and security"
date: 2026-03-15
draft: false
tags: ["openclaw", "ai", "automation", "security"]
categories: ["AI", "Development", "Productivity"]
showTableOfContents: true
---
If you want to get started with OpenClaw without getting lost in setup details, this guide takes you from zero to productive: installation, daily basics, and practical security recommendations.

<!--more-->

## What is OpenClaw?

OpenClaw is a platform for running AI agents connected to real tools (files, terminal, web, messaging, and more), with persistent memory and configurable safety controls.

Think of OpenClaw as an operational assistant, not only a chat interface.

## Prerequisites

Before installation:

- Linux, macOS, or Windows (WSL recommended for development workflows)
- Node.js 22+
- npm or pnpm
- Access to a model provider account (for example OpenAI)

## Quick installation

Install OpenClaw globally:

```bash
npm install -g openclaw
```

Verify installation:

```bash
openclaw --version
```

Run initial onboarding:

```bash
openclaw onboard
```

This setup flow configures core elements such as authentication, workspace, and baseline runtime options.

## Recommended initial checks

### 1) Verify overall status

```bash
openclaw status --deep
```

This quickly shows:

- gateway health
- channel status
- active sessions
- security warnings

### 2) Check for updates

```bash
openclaw update status
```

### 3) Run a security audit

```bash
openclaw security audit --deep
```

## Basic day-to-day usage

### Start/restart gateway

```bash
openclaw gateway start
openclaw gateway restart
```

### Follow logs in real time

```bash
openclaw logs --follow
```

### Send a test message from CLI

```bash
openclaw message send --channel telegram --target <chat_id> --message "Hello from OpenClaw"
```

### Work with sessions

```bash
openclaw sessions list
```

## Workspace baseline structure

OpenClaw uses a per-agent workspace to preserve continuity.

Common files:

- `SOUL.md`: agent style/persona
- `USER.md`: user context
- `MEMORY.md`: long-term memory
- `memory/YYYY-MM-DD.md`: daily notes
- `HEARTBEAT.md`: proactive periodic tasks

Practical recommendation: keep `USER.md` and `MEMORY.md` up to date for more useful and consistent outcomes.

## Security recommendations (important)

### 1) Keep local bind by default

Use `loopback` unless you explicitly need remote access.

```bash
openclaw config get gateway.bind
```

If remote access is required, prefer an SSH tunnel over direct port exposure.

### 2) Avoid elevated execution unless necessary

Use elevated mode only when required and with clear intent.

### 3) Audit regularly

Run periodically:

```bash
openclaw security audit --deep
```

### 4) Keep OpenClaw and OS updated

- `openclaw update status`
- system updates (`apt`, `dnf`, etc.)

### 5) Define strict channel access policies

Set `dmPolicy`, `groupPolicy`, allowlists, and pairing according to your risk profile.

### 6) Do not expose secrets in plain text

Store tokens and credentials safely and never commit them to repositories.

## Suggested personal production flow

1. Install OpenClaw
2. Run `onboard`
3. Validate with `status --deep`
4. Run `security audit --deep`
5. Configure channels and minimum-access policies
6. Test in a controlled environment
7. Enable automations (cron/heartbeats)

## Common early issues

- **"access not configured" in Telegram**: pairing approval missing or policy too restrictive.
- **Cannot use host tools**: likely running inside sandbox (check `agents.defaults.sandbox.mode`).
- **Command not found**: isolated environment or incomplete PATH setup.

## Conclusion

OpenClaw has a short onboarding path if you focus on essentials first: clean install, status validation, and security checks from day one.

With a well-defined workspace and access policies, you get a powerful and reliable assistant for real-world automation without losing control.

## Resources

- Documentation: https://docs.openclaw.ai
- Repository: https://github.com/openclaw/openclaw
- Community: https://discord.com/invite/clawd
