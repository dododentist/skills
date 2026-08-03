# DodoDentist Agent Skills

[Agent Skills](https://agentskills.io) for
[DodoDentist](https://www.dododentist.com/) — dental practice management:
clinics, patients, appointments, treatments, invoices and payments.

## Install

```bash
npx skills add dododentist/skills
```

Works with any agent that supports the Skills standard — Claude Code, Cursor,
Codex, Gemini CLI, GitHub Copilot, OpenCode, and
[others](https://agentskills.io/clients). Update later with
`npx skills update`.

To install a single skill:

```bash
npx skills add dododentist/skills@dododentist
npx skills add dododentist/skills@clinic-administration
```

## Skills

| Skill                                                                | Backed by                                                          | Use it for                                                                                                        |
| -------------------------------------------------------------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| [`dododentist`](skills/dododentist/SKILL.md)                         | the [`dododentist` CLI](https://github.com/dododentist/cli) (npm)  | Practice records from the shell — patients, appointments, treatments, invoices, payments — with `npx dododentist`  |
| [`clinic-administration`](skills/clinic-administration/SKILL.md)     | the DodoDentist MCP connector (`https://mcp.dododentist.com/mcp`)  | Read-only clinic administration through MCP tools in hosts like Claude, ChatGPT, Cursor and Codex                  |

The two surfaces are deliberately not equivalent. The CLI drives the
key-authenticated REST API and reaches real patient records, so it is for the
practice's own tooling. The MCP connector is administrative-only: clinic
metadata, bookable availability, provider display names, the procedure
catalog, non-identifying calendar blocks and aggregate schedule counts — never
patient records, individual appointments or clinical data, and never a write.

Both need a DodoDentist account with at least one clinic —
[sign up](https://app.dododentist.com), then `dododentist login` (CLI) or
connect the MCP server (OAuth).

## Other entrypoints

- **CLI** — `npm install -g dododentist` or `npx dododentist`, source at
  [dododentist/cli](https://github.com/dododentist/cli).
- **MCP connector** — `https://mcp.dododentist.com/mcp` (Streamable HTTP,
  OAuth); setup for every MCP client at
  [dododentist.com/developers](https://www.dododentist.com/developers/#mcp).
- **Claude Code plugin** — the connector bundled with the
  `clinic-administration` skill:
  [dododentist/claude-plugin](https://github.com/dododentist/claude-plugin).
- **REST API** — API keys and reference at
  [dododentist.com/developers](https://www.dododentist.com/developers/).

## License

[Apache-2.0](LICENSE)
