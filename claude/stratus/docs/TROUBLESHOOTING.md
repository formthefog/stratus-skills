# Stratus Troubleshooting Guide

Common issues and solutions.

## Connection Issues

### API Connection Refused

**Symptom:**
```
ConnectionError: Connection refused to http://212.115.124.137:8000
```

**Causes:**
- Stratus server is down
- Network connectivity issue
- Firewall blocking connection

**Solutions:**
1. Check server status:
   ```bash
   curl http://212.115.124.137:8000/health
   ```

2. Verify network:
   ```bash
   ping 212.115.124.137
   ```

3. Check firewall rules

4. Use `/stratus status` for diagnostic

---

### Request Timeout

**Symptom:**
```
TimeoutError: Request timed out after 30s
```

**Causes:**
- Complex query taking too long
- Server overloaded
- Network latency

**Solutions:**
1. Simplify query
2. Increase timeout:
   ```bash
   export STRATUS_TIMEOUT=60
   ```
3. Retry request
4. Check server load with `/stratus status`

---

## Authentication Issues

### Invalid API Key

**Symptom:**
```
401 Unauthorized: Invalid API key format
```

**Solution:**
Verify API key format:
```bash
echo $STRATUS_API_KEY | grep "stratus_sk_live_"
```

Should start with `stratus_sk_live_` or `stratus_sk_test_`

---

## Response Issues

### `return_intermediate: false` Crashes

**Symptom:**
Rollout endpoint crashes with `return_intermediate: false`

**Solution:**
**Always use `return_intermediate: true`** in rollout requests

```json
{
  "goal": "...",
  "max_steps": 5,
  "return_intermediate": true  // ← Required
}
```

This is a known issue (see docs/API.md).

---

### Incorrect Results

**Symptom:**
Stratus returns wrong answer

**Debugging:**
1. Run with verbose output:
   ```bash
   /stratus debug [query]
   ```

2. Compare with other models:
   ```bash
   /stratus compare [query]
   ```

3. Try smaller query:
   ```bash
   /stratus ask "simplified version"
   ```

4. Check model size - try medium if using small

---

## Performance Issues

### Slow Response Times

**Check latency:**
```bash
/stratus stats
```

**Expected:**
- <10ms per prediction (small)
- <15ms per prediction (medium)

**If slower:**
1. Check server load: `/stratus status`
2. Try batching requests
3. Use caching for repeated queries
4. Check network latency: `ping 212.115.124.137`

---

### High Token Usage (No Reduction)

**Check token reduction:**
Look for `stratus` metadata in response:
```json
{
  "stratus": {
    "token_reduction": 24.3
  }
}
```

**If missing or low:**
1. Verify using correct model name: `stratus-x1ac-{size}-{llm}`
2. Check query is appropriate for Stratus (agent task, not chat)
3. Try `/stratus test` to verify integration

---

## Diagnostic Commands

### Quick Health Check

```bash
/stratus test
```

Verifies:
- API connectivity
- Authentication
- Model response
- Basic functionality

### Full Diagnostic

```bash
/stratus diagnose
```

Checks:
- All endpoints
- Model health
- Performance metrics
- Known issues

### Server Status

```bash
/stratus status
```

Shows:
- Connection status
- Endpoint health
- GPU memory
- Queue depth

---

## Error Messages

### "Missing required field: goal"

**Fix:** Include `goal` field in rollout request:
```json
{
  "goal": "Your task description",
  "max_steps": 5
}
```

### "Model prediction failed"

**Causes:**
- Malformed input
- Server error
- Model crash

**Solutions:**
1. Check request format
2. Simplify query
3. Check server logs: `/stratus logs`
4. Retry request

### "Invalid model name"

**Fix:** Use correct format: `stratus-x1ac-{size}-{llm}`

Valid examples:
- `stratus-x1ac-small-claude-sonnet-4-5`
- `stratus-x1ac-medium-gpt-4o`

---

## Getting Help

1. Run diagnostics: `/stratus diagnose`
2. Check documentation: docs/
3. Review known issues: docs/API.md#known-issues
4. Contact Formation support

---

**For more:**
- API reference: `API.md`
- Integration guide: `INTEGRATION.md`
- Main skill: `../SKILL.md`
