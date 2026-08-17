Install the official Auxeo Codex plugin from its public marketplace.

Run these commands:

```bash
codex plugin marketplace add https://github.com/Hatcherthekid/auxeo-plugin-marketplace.git --ref main
codex plugin add auxeo@auxeo
codex mcp login ads_read
```

After authentication succeeds, start a new Codex task so the Auxeo skills and MCP tools are loaded.
The marketplace is public, but Auxeo data access still requires an invited organization member and assigned resource scope.
