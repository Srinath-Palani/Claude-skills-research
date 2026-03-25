# MCP Error Patterns — Quick Reference

This file is read at the start of every error-handling session. Match the user's error
against the signal column — a match skips Phase 2/3 and goes straight to Phase 4.

---

## Signal → Category Map

| Signal (error text / symptom) | Category | Go to fix |
|-------------------------------|----------|-----------|
| `ModuleNotFoundError: No module named 'mcp'` | Dependency | Fix: Dependency |
| `ImportError` | Dependency | Fix: Dependency |
| `Cannot find module '@modelcontextprotocol` | Dependency | Fix: Dependency |
| `npm ERR! code ENOENT` | Dependency | Fix: Dependency |
| `command not found: uv` | Environment/Path | Fix: Environment |
| `command not found: node` | Environment/Path | Fix: Environment |
| `command not found: python3` | Environment/Path | Fix: Environment |
| `No such file or directory` (in args path) | Configuration | Fix: Configuration |
| `JSON parse error` in settings | Configuration | Fix: Configuration |
| `spawn <cmd> ENOENT` | Configuration | Fix: Configuration |
| `<KEY>` still in settings.json | Authentication | Fix: Authentication |
| `401 Unauthorized` | Authentication | Fix: Authentication |
| `403 Forbidden` | Authentication | Fix: Authentication |
| `Invalid API key` | Authentication | Fix: Authentication |
| `token expired` | Authentication | Fix: Authentication |
| `KeyError` on env var | Authentication | Fix: Authentication |
| `Connection refused` | Transport | Fix: Transport |
| `ECONNREFUSED` | Transport | Fix: Transport |
| `STDIO process exited` | Transport | Fix: Transport |
| `EPIPE` / `broken pipe` | Transport | Fix: Transport |
| `spawn error` | Transport | Fix: Transport |
| `Unsupported protocol version` | Protocol Version | Fix: Protocol Version |
| `capability negotiation failed` | Protocol Version | Fix: Protocol Version |
| `method not found` (-32601) | Protocol Version | Fix: Protocol Version |
| `RuntimeError` in server code | Runtime/Crash | Fix: Runtime |
| `AttributeError` in server code | Runtime/Crash | Fix: Runtime |
| `TypeError` in tool function | Runtime/Crash | Fix: Runtime |
| `python3 --version` mismatch | Environment/Path | Fix: Environment |
| `requires-python` version conflict | Environment/Path | Fix: Environment |
| `permission denied` | Environment/Path | Fix: Environment |

---

## Common Multi-Layer Error Combinations

**"Server won't start + missing module"**
→ Dependency (venv not created) + Configuration (command points to system Python, not venv)
→ Fix order: 1. Dependency, 2. Configuration

**"401 after fresh install"**
→ Authentication — `<KEY>` placeholder still in settings.json, or wrong env var name
→ Fix order: 1. Authentication

**"Connection refused on port"**
→ Transport — usually a crash on startup due to missing env var
→ Fix order: 1. Run server directly and capture full output, 2. Authentication or Dependency

**"Unknown method / method not found"**
→ Protocol Version — client and server using different MCP spec versions
→ Fix order: 1. Upgrade mcp SDK, 2. Rebuild if TypeScript
