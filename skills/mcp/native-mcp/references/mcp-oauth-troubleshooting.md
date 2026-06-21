# MCP OAuth Troubleshooting — Port Mismatch & Auth Failures

## Common Failure: `invalid_grant: Invalid redirect URI`

### Root Cause

Hermes picks a **new random port** for the OAuth callback listener on every process restart. If the OAuth app registration (stored in `~/.hermes/mcp_oauth/<server>/`) was created with the old port, the provider rejects the token exchange because the redirect URI doesn't match.

This was a known bug (GitHub issue #5344), resolved via PR #11383. Older Hermes versions are affected.

### Fix 1: Use `hermes mcp login` from CLI (Recommended)

The dashboard's 30-second auth timeout often races with interactive OAuth. Use the CLI instead:

```bash
hermes mcp login <server_name>
```

This waits up to 5 minutes and handles port assignment cleanly.

### Fix 2: Paste-the-URL Method (Works on Remote Hosts)

When the browser redirect to `127.0.0.1:<port>/callback` fails (remote server, port mismatch, etc.):

1. Run `hermes mcp login <server_name>`
2. Open the printed authorize URL in your browser
3. Approve the OAuth
4. The redirect to `127.0.0.1` will fail — **copy the full URL from the address bar**
5. Paste it at the terminal prompt (`>`)
6. Hermes extracts the `?code=...&state=...` and completes the exchange

Also works with a bare `?code=...&state=...` string.

### Fix 3: Force Re-Registration

Delete the cached client registration to force a fresh OAuth app registration with the current port:

```bash
rm -rf ~/.hermes/mcp_oauth/<server_name>/
hermes mcp login <server_name>
```

### Fix 4: Update Hermes

If on a version before the #11383 fix:

```bash
hermes update
```

## Common Pitfalls

- **Dashboard shows "fail to trust" loop**: 30-second timeout too short. Use `hermes mcp login` from CLI.
- **Port collision with multiple OAuth MCP servers**: Each server needs its own port. Fixed in newer Hermes (per-server port tracking instead of shared global).
- **SSH/remote host auth**: Use paste-the-URL or SSH port forwarding (`ssh -N -L <port>:127.0.0.1:<port> user@host`).
- **Auth works once, fails on restart**: Port changed. Either paste-the-URL or delete client registration and re-register.

## Reference

Full OAuth over SSH guide: https://hermes-agent.nousresearch.com/docs/guides/oauth-over-ssh
