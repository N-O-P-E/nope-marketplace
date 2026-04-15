# Studio N.O.P.E. Claude Marketplace
*We are Not Of Planet Earth*

Claude Code plugins by [Studio N.O.P.E.](https://github.com/N-O-P-E) — we automate the browser clicks, configuration loops, and setup handholding so you stay in flow and ship what actually matters. Every plugin here is something we use ourselves.

## Plugins

| Plugin | Description |
|--------|-------------|
| [**gcp-setup**](plugins/gcp-setup) | Drives Chrome through Google Cloud Console — project creation, APIs, service accounts, IAM, billing, OAuth, Cloud Run. Auto-detects what your project needs from the codebase. |
| [**search-console**](plugins/search-console) | Audits Google Search Console via browser. Surfaces indexing issues, sitemap gaps, Core Web Vitals, and structured data warnings — then implements the fixes. |

## Install

Add the marketplace to Claude Code:

```
/plugin marketplace add n-o-p-e/claude-marketplace
```

Install a plugin:

```
/plugin install gcp-setup@nope-marketplace
/plugin install search-console@nope-marketplace
```

## Chrome DevTools MCP

Both plugins drive Chrome via [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp). Set it up once:

**Add to `~/.claude/settings.json`:**
```json
{
  "mcpServers": {
    "chrome-devtools-mcp": {
      "command": "npx",
      "args": ["chrome-devtools-mcp@latest"]
    }
  }
}
```

Or via Claude Code:
```
/mcp add chrome-devtools-mcp -- npx chrome-devtools-mcp@latest
```

Chrome launches automatically on first use. If something goes wrong, see the [troubleshooting guide](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md).

## About

We're [@tijsluitse](https://github.com/tijsluitse) and [@basfijneman](https://github.com/basfijneman) — creative solution engineers using AI to help humans realise their dreams.

Want to work with us? We help teams build smarter workflows with AI-powered tooling, Shopify development, and creative engineering. Reach out at [info@studionope.nl](mailto:info@studionope.nl) or visit [studionope.nl](https://studionope.nl).

## License

[MIT](LICENSE)
