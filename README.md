# Cauldron skills

Plugins and skills that connect AI agents to [Cauldron](https://cauldron.studio):
generate images, search your Library, read brand kits, and upscale, extend,
resize, or cut out images without leaving your editor.

## Claude Code

```
/plugin marketplace add cauldronstudio/skills
/plugin install cauldron@cauldron
```

Then run `/mcp`, pick **cauldron**, and choose **Authenticate**. Cauldron opens
in your browser: sign in, pick the studio, click **Allow**. No key to paste. The
plugin bundles the Cauldron MCP server and three skills: `cauldron`, `migrate` (move a Dropbox library into Cauldron) and `cauldron-docs` (read the user guides at docs.cauldron.studio).

## Other agents (Cursor, Codex, and the rest)

```bash
npx skills add cauldronstudio/skills
```

This installs the skill only. Connect the MCP server yourself: it's a streamable
HTTP server at `https://app.cauldron.studio/api/mcp`. Clients that support MCP
sign-in (OAuth) need nothing else.

## Clients without MCP sign-in

Use a personal key: in Cauldron, open **Profile → Connected AI Tools → Create
key**. It starts with `oc_`, is shown once, and is pinned to one studio. Send it
as an `Authorization: Bearer oc_…` header.

The full guide covers what each tool does, what spends credits, and
troubleshooting: https://docs.cauldron.studio/guides/connected-ai-tools/

## License

MIT
