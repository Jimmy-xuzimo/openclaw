## Summary

This PR addresses two security vulnerabilities identified during an Aisle Security review of PR #48296. The fixes prevent potential resource exhaustion and information disclosure.

## Vulnerabilities Fixed

### 1. Unbounded Limit in ACP listSessions (CWE-400) - `src/acp/translator.ts`
**Problem:** The `limit` parameter from `params._meta` is forwarded to the gateway without any upper bound, allowing a malicious client to request excessive resources.

**Fix:** Added `DEFAULT_LIMIT = 100` and `MAX_LIMIT = 200` constants. The limit is now clamped: `Math.min(MAX_LIMIT, Math.max(1, Math.floor(rawLimit ?? DEFAULT_LIMIT)))`.

### 2. Sensitive Data in Telegram Error Messages (CWE-532) - `extensions/telegram/src/send.ts`
**Problem:** Error messages include raw `params.input` containing sensitive recipient identifiers (chat IDs, usernames, t.me links), which get logged and persisted.

**Fix:** Created `safePreview()` helper that:
- Truncates input to 128 characters max
- Escapes control characters via JSON.stringify
- Passes through `redactSensitiveText()` to redact sensitive patterns (API keys, tokens, etc.)

## Security Impact

| Vulnerability | Severity | CWE | Fix Impact |
|--------------|----------|-----|------------|
| Unbounded session limit | Low | CWE-400 | Caps resource usage |
| Sensitive data leakage | Low | CWE-532 | Redacts PII from logs |

## Files Changed

- `src/acp/translator.ts` - Limit enforcement fix
- `extensions/telegram/src/send.ts` - Info leakage fix

---

*This is an AI-assisted PR addressing security findings from Aisle Security review of PR #48296.*