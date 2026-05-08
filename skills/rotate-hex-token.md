Help the user rotate an API token stored in macOS Keychain.

Ask which token to rotate. Known tokens:
- **hex** — Keychain service: `hex-api`, account: `HEX_API_TOKEN`. Used by customer-research Hex MCP server. Also in `~/Projects/customer-research/.env` as `HEX_API_TOKEN`.

Then ask the user to paste the new token value.

Once they provide it, run:
```
security add-generic-password -s <service> -a <account> -w "<new-token>" -U
```

Also update any .env files that reference the token.

After updating, remind the user to restart Claude Code (or `source ~/.zshrc`) so MCP servers pick up the new value.
