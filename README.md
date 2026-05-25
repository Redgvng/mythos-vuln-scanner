# mythos-vuln-scanner

An 8-phase vulnerability research pipeline inspired by [Cloudflare's Mythos / Project Glasswing](https://blog.cloudflare.com/fr-fr/cyber-frontier-models/).

## What it does

Instead of producing a flat list of bugs, this skill orchestrates parallel agents to **build exploit chains** — combining multiple vulnerability primitives into confirmed, PoC-backed findings with reachability analysis.

## Pipeline

| Phase | Description |
|-------|-------------|
| 1 — Reconnaissance | Survey repo: languages, trust boundaries, entry points, data flows → initial task queue |
| 2 — Parallel Hunt | One agent per `(attack_class × code_zone)` pair (up to 32 concurrent) |
| 3 — PoC Validation | Generate + compile + execute minimal harness, loop ≤5× on failure |
| 4 — Adversarial Refutation | Fresh agent with zero prior context tries to disprove each finding |
| 5 — Gap Coverage | Re-queue unanalyzed zones with a different analysis angle |
| 6 — Deduplication | Group by root cause → assign `MYTHOS-NNN` IDs |
| 7 — Reachability Tracing | Can attacker-controlled input reach this site? |
| 8 — Structured Report | Machine-readable JSON at `./reports/mythos_report_<timestamp>.json` |

## Attack class defaults by language

| Language | Classes |
|----------|---------|
| C/C++ | `buffer_overflow`, `uaf`, `oob_read`, `oob_write`, `integer_overflow`, `format_string` |
| Rust | `unsafe_block_misuse`, `integer_overflow`, `panic_reachability` |
| Go | `race_condition`, `integer_overflow`, `path_traversal` |
| JS/TS | `prototype_pollution`, `injection`, `xss`, `deserialization` |
| Python | `injection`, `deserialization`, `ssrf`, `path_traversal` |
| Java | `injection`, `deserialization`, `xxe`, `idor` |

## Installation

```bash
cp -r . ~/.skills/mythos-vuln-scanner
```

## Usage

```
/mythos-vuln-scanner ~/path/to/repo
```

Or naturally in conversation:
> "Run a mythos scan on ~/myproject"
> "Find CVE-grade bugs in this codebase"
> "Build exploit chains for ~/myapp"

## Output format

```json
{
  "scan_metadata": { "repo_path": "...", "timestamp": "...", "phases_completed": 8 },
  "findings": [{
    "vuln_id": "MYTHOS-001",
    "severity": "critical|high|medium|low",
    "attack_class": "buffer_overflow",
    "file": "src/parser.c",
    "line_range": [142, 158],
    "root_cause": "...",
    "exploit_chain": ["oob_read_primitive", "control_flow_hijack"],
    "poc_status": "confirmed|plausible_unconfirmed|not_attempted|rejected",
    "reachable_from_untrusted_input": true,
    "reproduction_steps": []
  }],
  "gaps": [],
  "rejected": []
}
```

## Authorization

**Authorized use only.** PoC execution (Phase 3) requires:
- Sandboxed / isolated environment
- Explicit authorization (pentest engagement, CTF, security research, defensive audit)

If not authorized to execute code, Phase 3 is automatically skipped and all findings are marked `not_attempted`.

## Inspiration

- [Cloudflare Mythos / Project Glasswing](https://blog.cloudflare.com/fr-fr/cyber-frontier-models/)
- [Exploit chain construction methodology](https://botcrawl.com/cloudflare-mythos-exploit-chains/)

## License

MIT
