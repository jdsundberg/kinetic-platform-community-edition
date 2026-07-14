# Kinetic Platform - AI functionality

Build Kinetic applications with Claude by connecting two pieces:

- **MCP server** — gives Claude tools to read and write your Kinetic Platform (spaces, kapps, forms, submissions, users, teams).
- **Skills** — teaches Claude how the platform works (form engine, workflow, KQL, portals, APIs).

## Prerequisites

- Kinetic Platform v7 or higher, already installed (see [README.md](README.md))
- A space to build in — the README creates one called `first`
- [Claude Code](https://claude.com/claude-code) installed
- Node.js 18+ and Git

The examples below use the README's values: domain `example.com`, space `first`, space user `joe` / `joe1`, system user `admin`. Substitute your own.

---

## 1. Install the MCP server

Clone and build it anywhere on your workstation:

```bash
git clone https://github.com/kineticdata/kinetic-platform-mgnt-mcp-server.git
cd kinetic-platform-mgnt-mcp-server
npm install
npm run build
```

Register it with Claude Code, pointing at the built `dist/index.js`:

```bash
claude mcp add kinetic-platform \
  -e KINETIC_SERVER_URL=https://first.example.com \
  -e KINETIC_USERNAME=joe \
  -e KINETIC_PASSWORD=joe1 \
  -e KINETIC_SYSTEM_URL=https://example.com \
  -e KINETIC_SYSTEM_USERNAME=admin \
  -e KINETIC_SYSTEM_PASSWORD=adminpassword \
  -e KINETIC_ALLOW_SELF_SIGNED=true \
  -- node /ABSOLUTE/PATH/TO/kinetic-platform-mgnt-mcp-server/dist/index.js --stdio
```

Notes:
- `KINETIC_SERVER_URL` is the **space** URL; `KINETIC_SYSTEM_URL` is the **platform** URL. The system variables are only needed for platform-level work (creating spaces, etc.).
- `KINETIC_ALLOW_SELF_SIGNED=true` is required if you installed with the default self-signed certificate. Drop it once you have a real certificate.
- Add `-s user` to make the server available in every project instead of just the current directory.

Verify the connection:

```bash
claude mcp list
```

### Claude Desktop

Use `config/claude-desktop.example.json` in the MCP server repo as a starting point for `claude_desktop_config.json`.

---

## 2. Load the Skills

Clone the skills library:

```bash
git clone https://github.com/kineticdata/kinetic-platform-ai-skills.git ~/kinetic-skills
```

Reference it from your project's `CLAUDE.md` (or `~/.claude/CLAUDE.md` to load it for every project):

```markdown
@~/kinetic-skills/CLAUDE.md
```

Claude Code follows `@`-imports transitively, so that one line pulls in the whole library — recipes, platform concepts, front-end patterns, and API reference.

For teams, add it as a submodule so everyone gets the same version:

```bash
git submodule add https://github.com/kineticdata/kinetic-platform-ai-skills.git ai-skills
```

```markdown
@ai-skills/CLAUDE.md
```

Other tools (Cursor, GitHub Copilot) are covered in the [skills repo README](https://github.com/kineticdata/kinetic-platform-ai-skills).

---

## 3. Test that it works

Start Claude Code and ask it something read-only:

```
Using the Kinetic MCP server, list the kapps in my space and show me the forms in each one.
```

You should see Claude call the Kinetic tools and return your actual kapps and forms. If it does, both the connection and your credentials are good.

To confirm the skills loaded, ask:

```
What is the deferral pattern in Kinetic, and when should I use it?
```

A grounded answer about Kinetic's approval workflow pattern means the skills are in context. A vague or generic answer means the `@`-import isn't resolving — check the path in your `CLAUDE.md`.

### Build something

```
Build me a simple helpdesk application on the Kinetic Platform
```

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `self signed certificate in certificate chain` | Set `KINETIC_ALLOW_SELF_SIGNED=true` |
| Server shows as failed in `claude mcp list` | Run `npm run build` — `dist/index.js` must exist, and the path must be absolute |
| `401` / auth errors | Check `KINETIC_SERVER_URL` is the space URL (`https://first.example.com`), not the platform URL |
| Claude can't reach the server | Confirm `/etc/hosts` maps your domain, per the README |
| Skills don't seem loaded | Verify the `@`-import path in `CLAUDE.md` resolves to a real file |
