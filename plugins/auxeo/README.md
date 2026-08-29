# Auxeo Agent Plugin

Auxeo is a read-only advertising data plugin for Codex. It connects Codex to the
Auxeo Remote MCP for governed Meta, Google Ads, TikTok Ads, and AppsFlyer queries,
analysis, monitoring, and reporting.

## Install

Ask your Auxeo workspace administrator to add the Auxeo marketplace once. Then
open the Codex/ChatGPT Desktop Plugins Directory, install **Auxeo**, and complete
the host-provided **Authenticate** flow. The host opens the Auxeo sign-in page and
returns to the plugin automatically; users do not run an MCP login command, copy
an authorization URL, or configure a localhost callback.

Start a new Codex task after the Desktop app reports that authentication succeeded.

This release requires Codex CLI/Desktop 0.144.5 or newer. Plugins are not
supported in the Codex IDE extension. The unique `auxeo_ads_read` name avoids
overwriting an existing local `ads_read` MCP server.

If a Host can read MCP Resources but does not expose Auxeo custom Tools in the
active task, read `ads-contract://host-tool-fallback-v1` and use the governed
`ads-query://execute/{tool_name}{?arguments}` Resource Template. This recovery
path uses the same OAuth, membership, resource scope, validation, deadline, and
artifact ownership as the normal Tool path.

The marketplace is public, but data is not. Reading Auxeo data requires Auth0
sign-in, active organization membership, and an assigned resource scope. This
package contains no backend source code, advertising credentials, tokens, or data.
It provides no advertising-platform write operations.
