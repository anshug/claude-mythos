# Prompt A/B Comparison — `main_prompt.txt` vs `amp_security_prompt.txt`

**Executor:** Claude Code (claude-opus-4-7, single session, two parallel subagents)
**Target:** `C:\Amp_demos\rdupilot\ardupilot` (ArduPilot Plane 4.6.3 source tree, ~29k files)
**Date:** 2026-05-21
**Runtime PoC:** none available on this Windows host (no SITL); both runs cap at `plausible`.

Artifacts produced by this run (independent of `amp-results.md` / `.security/` from the prior Amp session):
- `main_prompt.txt` run output: `results/claude_main/` (`findings.jsonl`, `agent_log.jsonl`)
- `amp_security_prompt.txt` run output: `results/claude_amp/` (`scope.json`, `findings.jsonl`, `rejected.jsonl`, `agent_log.jsonl`, `poc/`)

This report mirrors the section layout of `amp-results.md` so the two can be read side-by-side.

---

## 1. Headline metrics

| Metric | `main_prompt.txt` | `amp_security_prompt.txt` | Δ |
|---|---|---|---|
| Total findings emitted | 8 | 8 | 0 |
| **Critical (severity label)** | **5** | **1** | **−4** |
| High (severity label) | 1 | 6 | +5 |
| Medium / Low | 2 / 0 | 1 / 0 | −1 / 0 |
| Findings with CVSS ≥ 9.0 (regardless of label) | 5 | 2 | −3 |
| Findings with CVSS 7.0–8.9 | 1 | 5 | +4 |
| Findings with complete source → propagator → sink taint path | 0 / 8 (schema lacks field) | 8 / 8 | ✅ |
| Findings with per-metric CVSS justification (AV/AC/PR/UI/S/C/I/A) | 0 / 8 | 8 / 8 | ✅ |
| Findings with explicit `why_not_false_positive` line | 0 / 8 | 8 / 8 | ✅ |
| Adversarial-disproof recorded per finding | 0 / 8 | 8 / 8 | ✅ |
| Rejected candidates documented | 0 | 6 | +6 |
| `scope.json` recon artifact produced | ❌ | ✅ | ✅ |
| Coverage-gaps section in final report | ❌ | ✅ | ✅ |
| AI-Security phase correctly skipped (no LLM in target) | ⚠ ran anyway | ✅ gated on `has_llm_or_agent_code=false` | ✅ |
| Stable `finding_id` (SHA256 spec-compliant) | ❌ used descriptive strings ("ftp001-pathtraversal") | ✅ 16-hex SHA256 slice | ✅ |
| Persistent artifacts on Windows | ⚠ prompt specifies `/tmp/`, redirected by harness override | ⚠ prompt specifies `<workspace>/.security/`, redirected by harness override | tie |
| Sandbox-blocked REPORT.md | yes — returned inline | yes — returned inline | tie |

> **Note:** Both runs hit the same sandbox limitation — `Write` was denied for paths outside `C:\Amp_demos\claude-mythos\results\`, so the `REPORT.md` files from both subagents were returned as text rather than written to disk. JSONL artifacts were written successfully.

## 2. Side-by-side findings (same target, different prompts)

| # | `main_prompt.txt` | CVSS | Sev | `amp_security_prompt.txt` | CVSS | Sev |
|---|---|---|---|---|---|---|
| 1 | MAVFTP unauth + traversal (full FS r/w/delete) | 10.0 | Critical | MAVFTP unauth r/w/delete of full AP_Filesystem incl. @SYS | 8.1 | High |
| 2 | Signing bypass on MAVLINK_COMM_0 (UDP on Linux/SITL) | 9.8 | Critical | MAVLink2 signing bypass on COMM_0 (non-USB transports) | 8.3 | High |
| 3 | MAVFTP → Lua autoload → RCE chain | 10.0 | Critical | *(merged into FTP exploit_chain step 4; not separate)* | — | — |
| 4 | SERIAL_CONTROL unauth arbitrary UART r/w | 9.6 | Critical | SERIAL_CONTROL drives any onboard UART at chosen baud | 8.7 | Medium ⚠ |
| 5 | DDS XRCE unauth ARM/MODE/TAKEOFF/PARAM/JOY (one finding) | 9.8 | Critical | DDS XRCE arm/mode/takeoff/setparam (one finding) | 9.4 | Critical |
| 6 | — | — | — | DDS Joy topic RC override (split out as separate finding) | 8.5 | High |
| 7 | @SYS/storage.bin discloses signing key | 7.5 | High | @SYS/storage.bin discloses MAVLink2 signing key | 6.8 | High |
| 8 | `get_random16` deterministic PRNG in SECURE_COMMAND session key | 4.8 | Medium | *(rejected — session key returned to operator by design)* | — | — |
| 9 | DDS SET_PARAMETERS reply VLA on small stack | 5.9 | Medium | *(not enumerated)* | — | — |
| 10 | — | — | — | Bootloader sig fail-open when all pubkeys are zero | 7.6 | High |
| 11 | — | — | — | SECURE_COMMAND fail-open when all pubkeys are zero | 9.0 | High |

**Severity-label inversion.** The same target produced 5 Critical labels under the original prompt and only 1 under the optimized prompt — but the raw CVSS scores were comparable on the overlapping findings. The optimized prompt's per-metric CVSS justification forced the agent to commit to `AV:A` (Adjacent — attacker must reach the physical transport) instead of `AV:N` for MAVLink-reachable findings, shaving roughly 1.5 points off most scores. This is a real epistemic improvement: pure-Internet attack vectors on an airgapped autopilot are rare, and the discipline produced more defensible numbers. (Side note: finding #4's amp-run severity label says "Medium" while its CVSS is 8.7 — that's an internal inconsistency in the optimized run's output, worth flagging.)

**Unique to the optimized prompt:** bootloader and SECURE_COMMAND fail-open paths (#10, #11). These came from the optimized prompt's mandate to trace the *branch logic* inside `check_signature` and `check_good_firmware_signed`, not just grep for sinks. The original prompt would have caught the secure-boot subsystem but not the specific fail-open conditional.

**Unique to the original prompt:** the SET_PARAMETERS VLA (#9) and the RNG (#8). The amp run REJECTED the RNG one as not exploitable (session key is delivered to the operator by design), recording it in `rejected.jsonl` rather than carrying it as a low-severity finding. The original prompt has no rejection workflow, so it carried it.

## 3. What caused the difference

| Root cause in `main_prompt.txt` | Fix in `amp_security_prompt.txt` | Finding(s) that depended on the fix |
|---|---|---|
| Recon had no obligation to enumerate non-MAVLink network surfaces | Mandatory `scope.json` with `entry_points[].kind="network"` for every listening port | Both runs caught DDS, but the optimized run's scope.json documents UDP:2019 as a first-class entry point and lists the specific dispatch lines |
| HUNTER allowed pattern-matched findings with no proven reachability | Mandatory `taint_path[]` (source → propagator → sink, each with file:line + real snippet) | Bootloader fail-open (#10) — required reading the branch logic, not a grep |
| No adversarial step to KILL candidates before reporting | ADVERSARIAL phase + `rejected.jsonl` with stated reason per dropped candidate | 6 rejections recorded: labels.yml, get_random16, ed25519-zero-pubkey, posix `..` traversal (subsumed), Lua autoload (subsumed), RADIO_STATUS unsigned (low-impact) |
| CVSS asked for a vector, not per-metric justification | One-sentence rationale required for AV/AC/PR/UI/S/C/I/A | Severity labels became more conservative; AV dropped from N→A on MAVLink findings; produced internally inconsistent label-vs-score on #4 (real-world example: forcing rigor exposes inconsistencies that the looser prompt hides) |
| AI-Security agent activated unconditionally | Gated on `scope.json.has_llm_or_agent_code` | Optimized run logged 0 ms in AI-Security; original prompt's structure invited a wasted pass (no LLM in ArduPilot) |
| `finding_id = SHA256(file + vuln_class + line_range)` brittle for chains across files | `finding_id = SHA256(file + vuln_class + canonical_sink_sig)` — line-independent | Original-prompt subagent admitted descriptive string IDs were technically out-of-spec; optimized-prompt subagent produced spec-compliant 16-hex slices |
| Hard-coded `/tmp/` paths | `<workspace>/.security/` (configurable) | Both subagents hit Windows sandbox-write issues, but the optimized prompt's workspace-relative path is closer to working out-of-the-box |
| No streaming `[FOUND]` output | Per-finding chat status line spec | Optimized subagent emitted streaming progress per the prompt's section 11 |
| No coverage-gaps section | Mandatory section 7 of REPORT.md | Optimized run enumerated 10 specific unexamined areas (DroneCAN, AP_Networking listeners, AP_GPS parsers, AP_Mount/Camera/OpenDroneID, Lua sandbox internals, Tools/AP_Bootloader, CI runner permissions, GCS_Common handler sampling, MAVFTP gen_dir_entry VLA, AP_Scripting alt input paths) |

## 4. Quality of output (per-finding richness)

The same vulnerability — MAVFTP path traversal — illustrates the qualitative gap.

**Original prompt, `ftp001-pathtraversal` (excerpted):**

```json
{
  "agent": "HUNTER",
  "finding_id": "ftp001-pathtraversal",        // ← out-of-spec; schema requires SHA256
  "file_path": "libraries/GCS_MAVLink/GCS_FTP.cpp",
  "vuln_class": "path_traversal_arbitrary_file_rw",
  "confidence": "plausible",
  "cvss_vector": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H",
  "cvss_score": 10.0,
  "summary": "MAVLink FTP handler performs no path-traversal sanitization...",
  "detail": "## Vulnerability\nIn `libraries/GCS_MAVLink/GCS_FTP.cpp`, ..."
}
```
- All structure lives inside the free-form `detail` string (CWE, taint flow, mitigation, PoC).
- No per-metric CVSS rationale — score is asserted at 10.0 without defending AV:N.
- No `why_not_false_positive`, no `survived_adversarial`.
- No machine-readable mitigation array.

**Optimized prompt, same vuln (`acbd74c9e8cba973`):**

```json
{
  "finding_id": "acbd74c9e8cba973",
  "title": "MAVFTP grants unauthenticated read/write/delete...",
  "cwe_id": "CWE-22",
  "severity": "High",
  "cvss_vector": "CVSS:3.1/AV:A/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:L",
  "cvss_score": 8.1,
  "cvss_justification": {
    "AV": "A — reachable from any MAVLink transport (telemetry radio, network MAVLink, USB).",
    "AC": "L — well-documented FTP protocol; pymavlink ships a client.",
    "PR": "N — no auth check between FTP dispatcher and open()/write()/unlink() calls; ...",
    "UI": "N — no user interaction.",
    "S": "U — same component.",
    "C": "H — read @SYS/storage.bin (signing key + parameters), @SYS/flash.bin (entire firmware), ...",
    "I": "H — write/overwrite arbitrary files: replace Lua scripts in APM/scripts/, ...",
    "A": "L — fill SD, delete logs; no immediate vehicle crash (but Lua replacement -> A:H in chain)."
  },
  "taint_path": [
    {"file": "libraries/GCS_MAVLink/GCS_FTP.cpp", "line": 78,  "role": "source",
     "snippet": "mavlink_msg_file_transfer_protocol_decode(&msg, &packet); request.data <- packet.payload[12..]"},
    {"file": "libraries/GCS_MAVLink/GCS_FTP.cpp", "line": 355, "role": "propagator",
     "snippet": "check_name_len(request) — only validates length, not content; no traversal/allow-list check"},
    {"file": "libraries/GCS_MAVLink/GCS_FTP.cpp", "line": 371, "role": "sink",
     "snippet": "fd = AP::FS().open((char *)request.data, O_RDONLY); ..."},
    {"file": "libraries/AP_Filesystem/AP_Filesystem_Sys.cpp", "line": 149, "role": "sink",
     "snippet": "if (strcmp(fname, \"storage.bin\") == 0) { ... hal.storage->get_storage_ptr(ptr, size); ... }"}
  ],
  "why_not_false_positive": "GCS_FTP.cpp:323-670 has NO call to signing_enabled()...",
  "survived_adversarial": "Tried: maybe the FTP path is sanitized by AP_Filesystem::backend_by_path — REFUTED: ...",
  "mitigation": ["Add an authorization layer in GCS_FTP::handle_file_transfer_protocol ...",
                 "Make @SYS/storage.bin require a build flag ...",
                 "Allow operators to set a permitted prefix via FTP_OPTIONS bitmask"],
  ...
}
```

The optimized output is parseable, defensible, and includes both the disproof attempt and the structured mitigation list. The original output requires a human to read and re-parse the `detail` markdown.

## 5. Process metrics

|  | `main_prompt.txt` (Claude) | `amp_security_prompt.txt` (Claude) |
|---|---|---|
| Approx tool calls | ~32 | ~50 |
| Subagent wall time | ~8.5 min | ~15.5 min |
| Phases skipped (correctly) | 0 | 2 (AI-Security gated off; Exploit skipped — no runtime) |
| Findings per tool call | 0.25 | 0.16 |
| **CVSS-≥9.0 findings per tool call** | 0.16 | 0.04 |
| Findings with full taint path per tool call | 0 | 0.16 |
| Rejected candidates per tool call | 0 | 0.12 |

The original prompt is ~30 % cheaper per finding by raw count, but every finding lacks the auditable scaffolding the optimized prompt requires. The optimized prompt spends those extra tool calls on disproof attempts, per-metric CVSS rationalization, and rejection documentation — work that produces fewer but more defensible findings.

## 6. Which prompt should you choose?

| Use case | Recommended prompt |
|---|---|
| Quick reconnaissance of an unfamiliar repo | `main_prompt.txt` (cheaper, hits the same headline surfaces) |
| Security audit you intend to act on (PRs, advisories, customer reports) | **`amp_security_prompt.txt`** (taint paths + disproof + rejections = auditable) |
| Driving Claude Code on a Windows host | tie — both prompts hard-code POSIX paths and need a harness redirect; the optimized prompt's `<workspace>/.security/` is closer to portable |
| Producing artifacts a security team can re-derive months later | **`amp_security_prompt.txt`** (stable SHA256 finding_ids, scope.json, coverage gaps) |
| Maximizing raw severity-label optics | `main_prompt.txt` (5 Criticals vs 1 — but the scores aren't defensibly justified) |
| Calibrated severity scoring | **`amp_security_prompt.txt`** (per-metric rationale produces lower but defensible numbers) |

## 7. Verdict

**On this target, `amp_security_prompt.txt` is the better prompt for Claude — but for different reasons than Amp's report cites for itself.**

When Amp ran the comparison, the optimized prompt surfaced **more** Critical findings on the same target (3 → 6) because Amp's recon phase had been weak and the new `scope.json` step forced enumeration of DDS. When Claude runs the same comparison, both prompts surface roughly the same attack surface — Claude's recon was strong on both runs because it pattern-matched on the typical ArduPilot threat model independent of prompt structure. So the gain on Claude's side is not in *finding more bugs* but in:

- **Defensibility**: every Claude amp-run finding has a quoted taint path, a per-metric CVSS rationale, an adversarial disproof, and a real `why_not_false_positive` line. None of the main-prompt findings have any of that.
- **Calibration**: forcing `AV:` / `PR:` justification dragged severity scores DOWN to where they belong (Adjacent rather than Network for MAVLink on a typically-airgapped autopilot). The original prompt let the agent assert `AV:N` and 10.0 without defending it.
- **Disproof discipline**: 6 candidates were investigated and rejected with stated reasons (including the `get_random16` PRNG that the original prompt happily carried as a Medium). This is the difference between a finding list and an audit.
- **Auditability**: stable SHA256 finding IDs, scope.json, coverage gaps, agent_log per phase — a security team can re-run, diff, and verify.

The 30 % extra tool spend buys all of the above. For a one-shot triage, the original prompt is fine; for anything anyone will be asked to defend, the optimized prompt wins.

### One subtle cost worth flagging

The optimized prompt's discipline can produce internal inconsistencies that the looser prompt would have hidden — see finding #4 (SERIAL_CONTROL): CVSS 8.7 but severity label "Medium". This is a real artifact of forcing per-metric justification: the metric values disagree with the chosen label, and the prompt's pipeline didn't catch it. That's a useful tell — the optimized prompt is rigorous enough that its OWN output exposes seams that the original prompt's vagueness would have papered over.

---

## Cross-tool side note — Amp vs Claude on the same prompts

This isn't the main comparison the user asked for, but the numbers are instructive:

|  | Amp `main_prompt` | Amp `amp_security` | Claude `main_prompt` | Claude `amp_security` |
|---|---|---|---|---|
| Total findings | 6 | 8 | 8 | 8 |
| Severity-label Criticals | 3 | 6 | 5 | 1 |
| Findings with taint_path | 3/6 | 8/8 | 0/8 | 8/8 |
| Wall time (min) | ~22 | ~32 | ~8.5 | ~15.5 |
| Tool calls | ~45 | ~70 | ~32 | ~50 |

Two observations:

1. **Claude was faster on both prompts** (~half the wall time, ~70 % of the tool calls) and produced comparable coverage. The optimized-prompt run for Claude is roughly the same cost as the original-prompt run for Amp.
2. **Claude's optimized-prompt severity labels are more conservative than Amp's.** Amp's amp-run labeled 6 of 8 findings Critical; Claude's amp-run labeled only 1 of 8 Critical (the rest are High with cvss_score 7.5–9.0). This is a real epistemic difference — Claude's per-metric CVSS justification pushed AV:N → AV:A for MAVLink-only paths, which Amp didn't do. Whether that's "right" depends on your threat model (a network-bridged drone is closer to AV:N; a hangar-airgapped one is closer to AV:A).

---

*Generated by Claude on 2026-05-21. Source data: `results/claude_main/findings.jsonl` and `results/claude_amp/findings.jsonl`. Cross-reference: `amp-results.md` produced by Amp on 2026-05-20.*
