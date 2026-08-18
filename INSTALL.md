# Install Auxeo for Codex

Install the official Auxeo Codex plugin from its public marketplace.

Run these commands:

```bash
codex plugin marketplace add https://github.com/Hatcherthekid/auxeo-plugin-marketplace.git --ref main
codex plugin add auxeo@auxeo
codex mcp login auxeo_ads_read --scopes ads.read
```

After authentication succeeds, start a new Codex task so the Auxeo skills and MCP tools are loaded.
Requires Codex CLI/Desktop 0.144.5 or newer. The Codex IDE extension does not support plugins.
The unique `auxeo_ads_read` name preserves any existing local `ads_read` MCP server.
The marketplace is public, but Auxeo data access still requires an invited organization member and assigned resource scope.
