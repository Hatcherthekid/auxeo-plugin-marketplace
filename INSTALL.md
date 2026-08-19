# Install Auxeo for Codex

Your Auxeo workspace administrator adds this marketplace once. Users then open
the Codex/ChatGPT Desktop **Plugins Directory**, install **Auxeo**, and complete
the host-provided **Authenticate** flow. Codex opens the Auxeo sign-in page and
returns automatically. Users do not run an MCP login command, copy an OAuth URL,
or configure a localhost callback.

After Desktop reports that authentication succeeded, start a new Codex task so
the Auxeo skills and MCP tools are loaded. Requires Codex CLI/Desktop 0.144.5 or
newer. The Codex IDE extension does not support plugins.
The unique `auxeo_ads_read` name preserves any existing local `ads_read` MCP server.
The marketplace is public, but Auxeo data access still requires an invited organization member and assigned resource scope.
