[![PyPI](https://img.shields.io/pypi/v/agentverif-sign.svg?style=flat-square)](https://pypi.org/project/agentverif-sign/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)

# agentverif GitHub Action

Certify your AI agent packages automatically.
Sign on every push. Block tampered agents in CI.

Scan, sign, and certify AI agent packages automatically in CI.
Every signature is preceded by a full OWASP LLM Top 10 scan —
packages scoring below 70 are refused before any signing occurs.

```
SCAN → SIGN → VERIFY
```

| Step | What happens |
|------|-------------|
| **SCAN** | OWASP LLM Top 10 scan runs automatically. Score < 70 = refused. |
| **SIGN** | `SIGNATURE.json` injected. License ID issued and registered. |
| **VERIFY** | Buyers verify at [verify.agentverif.com](https://verify.agentverif.com) or via CLI. |

## Usage

### Sign your agent (vendors)
```yaml
- uses: trusthandoff/agentverif@v1
  with:
    mode: sign
    agent_zip: ./my-agent.zip
    tier: indie
```

> **Note:** `mode: sign` always runs the full OWASP LLM Top 10 scan
> before issuing any signature. There is no way to sign without scanning.
> Use `mode: verify` to check an already-signed package.

### Verify before deploy (buyers/CI gate)
```yaml
- uses: trusthandoff/agentverif@v1
  with:
    mode: verify
    agent_zip: ./my-agent.zip
    fail_on_unsigned: "true"
```

### Full workflow example
```yaml
name: agentverif
on: [push, pull_request]
jobs:
  certify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Sign agent
        uses: trusthandoff/agentverif@v1
        id: sign
        with:
          mode: sign
          agent_zip: ./agent.zip
      - name: Show license
        run: echo "License ${{ steps.sign.outputs.license_id }}"
```

### Full SCAN → SIGN → VERIFY pipeline

```yaml
name: agentverif certification
on: [push, pull_request]

jobs:
  certify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Scan + Sign agent
        uses: trusthandoff/agentverif@v1
        id: sign
        with:
          mode: sign              # scan runs automatically first
          agent_zip: ./agent.zip
          tier: indie             # free forever

      - name: Show certificate
        run: |
          echo "License: ${{ steps.sign.outputs.license_id }}"
          echo "Status:  ${{ steps.sign.outputs.status }}"
          echo "Hash:    ${{ steps.sign.outputs.zip_hash }}"

      - name: Verify before deploy
        uses: trusthandoff/agentverif@v1
        with:
          mode: verify
          agent_zip: ./agent.zip
          fail_on_unsigned: "true"
```

## Inputs
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| mode | yes | verify | sign or verify |
| agent_zip | yes | — | Path to ZIP file |
| tier | no | indie | indie / pro / enterprise |
| api_key | no | — | Pro/Enterprise API key |
| fail_on_unsigned | no | true | Fail build if unsigned |

## Outputs
| Output | Description |
|--------|-------------|
| license_id | Generated license ID |
| status | VERIFIED / UNREGISTERED / UNSIGNED / MODIFIED / REVOKED |
| zip_hash | SHA256 hash of package |

## Free tier

Indie plan is free forever — no API key required for local signing.

**First 100 Founding Vendors** get lifetime Pro free (worth €100/year).
[Claim your spot → agentverif.com/first100](https://agentverif.com/first100)
23 spots remaining. After 100, this closes permanently.

---

## Web UI

No CLI or Action required to get started.
Upload, scan, and sign at [sign.agentverif.com](https://sign.agentverif.com).
Buyers verify at [verify.agentverif.com](https://verify.agentverif.com).

---

## Full docs

[agentverif.com/docs](https://agentverif.com/docs) — CLI reference,
SIGNATURE.json format, environment variables, FAQ.
