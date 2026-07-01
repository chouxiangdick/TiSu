# Pre-Publish Privacy Scan

> **When to use**: Before pushing a coach skill design or any Hermes-related project to a public GitHub repo.
>
> **Why**: Real usernames, machine names, IPs, API tokens, chat IDs, and profile names can leak into README examples, reference files, and skill metadata. One missed occurrence is a privacy incident.

## Automated Scan Script

Run this from the repo root before every `git push` to a public remote:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== Privacy Scan ==="
fail=0
check() {
  local pattern="$1" label="$2" include="$3"
  local result
  result=$(grep -rnE "$pattern" . --include="$include" 2>/dev/null | grep -v "Binary file" || true)
  if [ -z "$result" ]; then
    echo "  ✓ $label"
  else
    fail=$((fail + 1))
    echo "  ✗ $label"
    echo "$result" | head -10
  fi
}

# Names — real person identifiers
check "你的真名\|你姓.*名\|你本人的名字" "Real name (Chinese)" "*.md"
check "firstName\|lastName.*用户" "Real name placeholder" "*.md"

# Machine hostnames
check "(WIN-|DESKTOP-|MacBook-Pro|iv-|ip-)" "Machine hostname" "*.md"

# Internal IPs (10.x, 192.168.x, 172.16-31.x, 100.x Tailscale)
check "\b(10\.\d{1,3}\.\d{1,3}\.\d{1,3}|192\.168\.\d{1,3}\.\d{1,3}|100\.\d{1,3}\.\d{1,3}\.\d{1,3})\b" "Internal IP address" "*.md"

# API tokens
check "(ghp_|sk-|t_[a-zA-Z0-9]|cli_|oc_)[a-zA-Z0-9]{20,}" "API token" "*.md *.yaml *.json"

# Real emails (not example.com)
check "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\b" "Email address" "*.md *.yaml *.json"
result=$(grep -rnE "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\b" . --include="*.md" --include="*.yaml" --include="*.json" 2>/dev/null | grep -v "@example.com\|@users.noreply.github.com" || true)
if [ -z "$result" ]; then echo "  ✓ Email (no real ones)"; else echo "  ✗ Email"; echo "$result" | head -5; fail=$((fail+1)); fi

# Profile-specific names
check "jiawen\|feishu\|weixin\|wechat" "Profile name" "*.md"

echo ""
echo "=== Result: $fail issue(s) found ==="
exit $fail
```

Save as `scripts/privacy-scan.sh` and run before each push.

## What to Sanitize

| Category | Search Pattern | Replace With |
|----------|---------------|--------------|
| Real usernames | 玄哥, Xuan, 绍玄, user's real name | `你`, `you`, `User` |
| Machine names | `WIN-*`, `DESKTOP-*`, `iv-*`, `ip-*` | `<hostname>` |
| Private IPs | `100.64-127.x`, `10.x.x.x`, `192.168.x.x` | `<ip>` |
| API tokens | `ghp_*`, `sk-*`, `cli_*`, `t_*`, `oc_*` | `<token>` |
| Chat/User IDs | `oc_*`, `ou_*`, `im.bot.*` | `<chat_id>` |
| Profile names | `feishu`, `jiawen`, `weixin` | `<profile>` |

## Integration with Git

Add a pre-push hook to automate the scan:

```bash
cp scripts/privacy-scan.sh .git/hooks/pre-push
chmod +x .git/hooks/pre-push
```

This rejects pushes that contain un-sanitized personal data.
