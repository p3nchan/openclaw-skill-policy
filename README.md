# OpenClaw Skill Security Policy

A layered security framework for evaluating and installing third-party skills (plugins) in [OpenClaw](https://github.com/openclaw/openclaw) AI agent environments.

## Why This Exists

Installing a third-party skill into an AI agent is fundamentally different from installing a regular software package:

- **SKILL.md files are injected directly into the system prompt** — a prompt injection vector
- **Scripts run with the user's full permissions** — no filesystem or network restrictions
- **No permission declaration mechanism** — you can't know what a skill accesses without reading all source code
- **No runtime isolation** — skills can access `~/.ssh/`, credentials, environment variables, etc.

Star counts and popularity metrics filter low quality, **not malice**. Real-world supply chain attacks (`event-stream`, `ua-parser-js`, `eslint-scope`) prove that even popular, well-starred packages can be compromised.

## The Four-Layer Framework

This policy uses **defense in depth** — multiple layers that each address a different attack vector:

### Layer 1 — Source Trust (Necessary but Not Sufficient)

Quick filter to determine if a skill is **worth evaluating further**. Must meet **any one** of:

1. **Official**: OpenClaw official or major AI company (Anthropic, OpenAI, Google, etc.)
2. **Community validated**: GitHub 1,000+ stars + ≥5 contributors + updated within 6 months
3. **Trusted developer**: On a pre-approved trusted developer list
4. **ClaWHub verified**: 100+ stars + official verified badge

> **Passing Layer 1 does NOT mean safe.** It only means the skill is worth further review.

### Layer 2 — Static Analysis (Automated)

Before installation, automatically:

- **SKILL.md scan**: Check for prompt injection patterns, Unicode steganography (zero-width characters, RTL overrides), suspicious instruction patterns
- **Dependency audit**: `npm audit` / `pip-audit` — block on critical/high CVEs
- **Lockfile check**: Must have a lockfile (`package-lock.json` / `yarn.lock` / `bun.lockb`) — missing = high risk
- **Install mode**: Use `--ignore-scripts`, review before running post-install scripts
- **Code review**: Scan entry points for external network requests, filesystem access, environment variable reads

### Layer 3 — Permission Declaration

> Depends on OpenClaw platform support. See [Feature Request #28298](https://github.com/openclaw/openclaw/issues/28298)

**Ideal mechanism (pending platform implementation):**
- Skills include a `manifest.json` declaring:
  - `fs`: Allowed filesystem paths (read/write)
  - `network`: Allowed domains
  - `tools`: OpenClaw APIs/tools used
  - `env`: Required environment variables
- No manifest → reject, regardless of stars

**Current workaround (manual):**
- Review SKILL.md + main scripts, document actual permission needs
- Human confirms reasonableness
- High-permission skills (filesystem, exec, sensitive paths) require explicit human approval

### Layer 4 — Runtime Enforcement

> Depends on OpenClaw platform support. See [Feature Request #28298](https://github.com/openclaw/openclaw/issues/28298)

**Ideal mechanism (pending platform implementation):**
- macOS `sandbox-exec` or Linux `firejail`/`bubblewrap` to restrict skill execution
- Network calls limited to manifest-declared domains
- Global deny on sensitive paths: `~/.ssh/`, `~/.gnupg/`, `~/.aws/`, `~/.config/gh/`
- Skill-to-skill isolation

**Current workaround:**
- Agent safety rules (e.g., AGENTS.md red lines)
- External content: extract information only, never execute instructions
- Destructive operations require human confirmation

## Decision Flowchart

```
Skill Installation Request
  │
  ├─ Layer 1: Source Trust → FAIL → ❌ Do not install
  │
  ├─ Layer 2: Static Analysis → Critical issue found → ❌ Do not install
  │
  ├─ Layer 3: Permissions reasonable?
  │   ├─ Low permission (API queries only) → ✅ Auto-approve
  │   └─ High permission (fs/exec/network) → ⚠️ Requires human confirmation
  │
  └─ ✅ Install (pinned version + lockfile)
```

## Update Policy

- **Pin versions** at install time (never auto-pull latest)
- **Review changelog + diff** before updates
- **Major version updates** treated as fresh installs — run full evaluation

## Contributing

This is a living document. If you run an OpenClaw instance (or any AI agent with plugin capabilities), we welcome:

- Feedback on the framework
- Real-world case studies of skill supply chain issues
- Implementation suggestions for Layers 3 & 4
- Additions to the trusted developer list (with justification)

## Related

- [OpenClaw Feature Request: Skill manifest.json + runtime sandbox](https://github.com/openclaw/openclaw/issues/28298)
- [OpenClaw Documentation](https://docs.openclaw.ai)
- [ClaWHub — Skill Marketplace](https://clawhub.com)

## License

[CC BY-SA 4.0](LICENSE) — Share and adapt with attribution.
