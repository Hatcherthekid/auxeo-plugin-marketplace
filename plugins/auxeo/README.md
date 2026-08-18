# Auxeo Agent Plugin

Auxeo is a read-only advertising data plugin for Codex. It connects Codex to the
Auxeo Remote MCP for governed Meta, Google Ads, TikTok Ads, and AppsFlyer queries,
analysis, monitoring, and reporting.

## Install

```bash
codex plugin marketplace add https://github.com/Hatcherthekid/auxeo-plugin-marketplace.git --ref main
codex plugin add auxeo@auxeo
codex mcp login auxeo_ads_read --scopes ads.read,reports.generate
```

Start a new Codex task after authentication so the plugin tools and skills load.

This release requires Codex CLI/Desktop 0.144.5 or newer. Plugins are not
supported in the Codex IDE extension. The unique `auxeo_ads_read` name avoids
overwriting an existing local `ads_read` MCP server.

The marketplace is public, but data is not. Reading Auxeo data requires Auth0
sign-in, active organization membership, and an assigned resource scope. This
package contains no backend source code, advertising credentials, tokens, or data.
It provides no advertising-platform write operations.
