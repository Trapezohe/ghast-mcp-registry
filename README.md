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

## Requesting a new server be listed

Open an issue in this repo with the **[Registry]** title prefix —
the in-app "提交收录请求 / Submit listing request" button on the
marketplace toolbar pre-fills one for you.

## Schema

User-facing prose fields (`description`, `installation.description`,
`parameter.description`, and individual `prerequisites[]` entries)
may be either:

- **a plain string** — applies to every locale, or
- **a `{ locale: text, ... }` object** keyed by BCP-47 codes
  (`en`, `zh-CN`, `zh-TW`, `ja`, `ko`, `fr`, etc.).

The client resolves the best available string for the user's current
language with a fallback chain (exact match → base language match →
`en` → first non-empty entry). English (`en`) is the registry's
lingua franca — please always include an `en` translation.

Identifiers and technical fields (`name`, `author`, `id`, `category`,
`tags`, `installation.name`, `parameter.name`, `parameter.key`,
`parameter.placeholder`, `config.command`, etc.) are kept in their
original form across all locales.

```jsonc
{
  "generatedAt": "ISO-8601 timestamp — only displayed as 'last
                  updated' in the UI, not used for cache control",
  "servers": [
    {
      "id":          "unique slug, used as the marketplace install
                      key (stored as registry_id on disk)",
      "name":        "display name (kept in original form)",
      "description": "string | { en, zh-CN, zh-TW, ja, … }",
      "author":      "vendor or person",
      "url":         "(optional) homepage / repo link",
      "license":     "(optional) SPDX id e.g. MIT, Apache-2.0",
      "category":    "(optional) bucket the marketplace UI groups by",
      "tags":        ["(optional)", "string", "array"],
      "featured":    "(optional) true → pinned in marketplace",
      "verified":    "(optional) true → verified badge in UI",
      "installations": [
        {
          "name":          "install variant label (e.g. 'npx',
                            'docker', 'macOS via brew')",
          "description":   "(optional) string | { en, zh-CN, … }",
          "config":        "MCP server config — same shape as the
                            Settings UI manual-entry form",
          "prerequisites": "(optional) array; each entry is either a
                            string or a { en, zh-CN, … } object",
          "parameters":    "(optional) array; descriptions inside
                            each parameter can be localised — see
                            below",
          "transports":    "(optional) array — 'stdio' / 'http' /
                            'sse'. Informational only for now."
        }
      ]
    }
  ]
}
```

### Localised description example

```json
{
  "description": {
    "en": "Read and write the local filesystem.",
    "zh-CN": "读写本地文件系统。",
    "ja": "ローカルファイルシステムを読み書き。"
  }
}
```

A plain-string `description` is still valid; it just shows the same
text to every locale.

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
  "description": {
    "en": "GitHub PAT — repo scope minimum.",
    "zh-CN": "GitHub PAT —— 至少需要 repo 权限。"
  },
  "placeholder": "ghp_xxxxxxxxxxxxxxxxxxxx",
  "required": true
}
```

Variable references in `config` use `${VARIABLE_NAME}` and match the
`name` field. The install dialog prompts the user for each declared
parameter and does textual substitution before persisting the server
config to the Ghast DB.

## Contributing

Open a PR adding or editing server entries. Please keep the
catalogue tight — this is a curated marketplace, not an exhaustive
index. Servers should have:

- a real homepage / repo URL,
- a sensible default install (the user should be able to install
  with at most one or two parameter fills),
- a license (no proprietary-only servers without explicit author
  consent),
- a working npm / pypi / cargo / brew install path (no source-from-
  scratch builds),
- at minimum English (`en`) prose, ideally with `zh-CN` /
  `zh-TW` / `ja` translations for the description, installation
  notes, and parameter descriptions.
