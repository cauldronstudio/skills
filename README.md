# Cauldron skills

Plugins and skills that connect AI agents to [Cauldron](https://opencauldron.ai):
generate images, search your Library, read brand kits, and upscale, extend,
resize, or cut out images without leaving your editor.

## Claude Code

```
/plugin marketplace add cauldronstudio/skills
/plugin install cauldron@cauldron
```

The install asks for your API key and stores it in your OS keychain. The plugin
bundles the Cauldron MCP server and the `cauldron` skill.

## Other agents (Cursor, Codex, and the rest)

```bash
npx skills add cauldronstudio/skills
```

This installs the skill only. Connect the MCP server yourself: it's a streamable
HTTP server at `https://studio.opencauldron.ai/api/mcp`, authenticated with an
`Authorization: Bearer oc_…` header.

## Getting a key

In Cauldron, open **Profile → Connected AI Tools → Create key**. The key starts
with `oc_` and is shown once. Each key is pinned to one studio.

The full guide covers what each tool does, what spends credits, and
troubleshooting: https://docs.opencauldron.ai/guides/connected-ai-tools/

## License

MIT
