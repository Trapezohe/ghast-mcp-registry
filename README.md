# Ghast MCP Registry

This repository hosts the curated catalogue of Model Context Protocol
(MCP) servers exposed by **[Ghast](https://ghast.trapezohe.ai)**'s in-
app marketplace (Settings → MCP 服务器 → 市场).

The Ghast desktop client fetches `registry.json` from this repo's
`main` branch on every visit to the marketplace tab, so editing this
file is how new servers show up in the UI — no client release
required.

## Endpoint

```
https://raw.githubusercontent.com/Trapezohe/ghast-mcp-registry/main/registry.json
```

## Schema

```jsonc
{
  "generatedAt": "ISO-8601 timestamp — only displayed as 'last
                  updated' in the UI, not used for cache control",
  "servers": [
    {
      "id":          "unique slug, used as the marketplace install
                      key (stored as registry_id on disk)",
      "name":        "display name",
      "description": "one or two sentences",
      "author":      "vendor or person",
      "url":         "(optional) homepage / repo link",
      "license":     "(optional) SPDX id e.g. MIT, Apache-2.0",
      "category":    "(optional) bucket the marketplace UI groups by
                      — files / web / dev / search / database /
                      browser / docs / thinking / memory /
                      communication",
      "tags":        ["(optional)", "string", "array"],
      "featured":    "(optional) true → marketplace UI pins the
                      server near the top",
      "verified":    "(optional) true → marketplace UI renders the
                      verified badge",
      "installations": [
        {
          "name":          "display name of this install variant
                            (e.g. 'npx', 'docker', 'macOS via brew')",
          "description":   "(optional) extra context for this
                            install path specifically",
          "config":        "MCP server config — same shape as the
                            Settings UI manual-entry form",
          "prerequisites": "(optional) array of plain-text strings,
                            shown as bullet points before install",
          "parameters":    "(optional) array of fill-in fields; the
                            install dialog substitutes `${VAR}`
                            references inside `config` with the
                            user-entered values",
          "transports":    "(optional) array — 'stdio' / 'http' /
                            'sse'. Informational only for now."
        }
      ]
    }
  ]
}
```

### `config` shape (stdio command)

```json
{
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "${ALLOWED_PATH}"],
  "env": { "OPTIONAL_VAR": "value" }
}
```

### `config` shape (HTTP / SSE)

```json
{
  "url": "https://example.com/mcp",
  "headers": { "Authorization": "Bearer ${API_KEY}" },
  "transport": "http"
}
```

### `parameters` example

```json
{
  "name": "GITHUB_PERSONAL_ACCESS_TOKEN",
  "description": "GitHub PAT — repo scope minimum",
  "placeholder": "ghp_xxxxxxxxxxxxxxxxxxxx",
  "required": true
}
```

Variable references in `config` use `${VARIABLE_NAME}` and match the
`name` field. The install dialog prompts the user for each declared
parameter and does textual substitution before persisting the server
config to the Ghast DB.

## Contributing

Open a PR adding a new server entry. Please keep the catalogue tight
— this is a curated marketplace, not an exhaustive index. Servers
should have:

- a real homepage / repo URL,
- a sensible default install (the user should be able to install
  with at most one or two parameter fills),
- a license (no proprietary-only servers without explicit author
  consent),
- a working npm / pypi / cargo / brew install path (no source-from-
  scratch builds).
