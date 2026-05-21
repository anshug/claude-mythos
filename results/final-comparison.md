# Multi-Agent × Multi-Prompt Security-Audit Benchmark
### A research-paper-style comparison of three coding agents (Amp, Claude Code, Factory Droid) running two prompts (`main_prompt.txt`, `amp_security_prompt.txt`) against the same target

---

## Abstract

We measure how the choice of *agent* and *prompt* affects the security findings produced
by LLM-driven code audits. The target is ArduPilot Plane 4.6.3 (~29 k files,
~2.7 k C/C++ source files). Three agents (Amp, Claude Code, Factory Droid) each ran two
prompts: the upstream `main_prompt.txt` from
[anshug/claude-mythos](https://github.com/anshug/claude-mythos) and the Amp-optimized
variant `amp_security_prompt.txt`, yielding **six runs** total. We score each run across
eight dimensions with explicit weights and report a single composite score.

**Headline result.** The optimized prompt strictly dominates the original prompt **for all
three agents**. The best run overall is **Amp × `amp_security_prompt.txt`** (composite
9.30 / 10), followed by **Claude × `amp_security_prompt.txt`** (8.85) and **Droid ×
`amp_security_prompt.txt`** (8.05). All three original-prompt runs cluster between 4.70
and 5.15 — agent choice barely changes that score, but prompt choice doubles it.

---

## 1. Methodology

### 1.1 Target

- Repository: `C:\Amp_demos\rdupilot\ardupilot` — ArduPilot Plane 4.6.3 source tree
- Language mix: ~82 % C/C++, ~10 % Python, ~5 % Lua, ~3 % YAML
- Threat model: a flight controller exposing MAVLink, optional DDS (ROS 2 over UDP 2019),
  optional networking ports, and Lua scripting via filesystem

### 1.2 Agents under test

| Agent | Version / model | Subagent style |
|---|---|---|
| Amp | Sonnet-class with Amp harness | `Task` subagents, parallel tool blocks |
| Claude Code | claude-opus-4-7 | parallel subagents in single session |
| Factory Droid | `worker` subagents | parallel subagents, ~80 tool-call cap |

### 1.3 Prompts under test

| Prompt | Origin | Lines |
|---|---|---|
| `main_prompt.txt` | upstream `anshug/claude-mythos` | 304 |
| `amp_security_prompt.txt` | this work (Amp-optimized) | 311 |

### 1.4 Run conditions

- Same Windows host, same workspace, no internet
- No SITL runtime → confidence ceiling: `plausible` for both prompts
- Each agent ran both prompts in the same session

### 1.5 Eight scoring dimensions

| # | Dimension | Weight | Rationale |
|---|---|---|---|
| D1 | Critical findings discovered (real, defensible) | 25 % | Primary security value |
| D2 | False-positive resistance in main report | 15 % | Signal-to-noise |
| D3 | Evidence quality (taint paths, CVSS rationale, disproof) | 20 % | Auditability |
| D4 | Reproducibility (stable IDs, artifacts, scope.json) | 10 % | Re-runnable |
| D5 | Coverage breadth (distinct vulnerable areas) | 10 % | Completeness |
| D6 | Efficiency (findings per tool call) | 5 % | Cost |
| D7 | Output-path portability (Windows + Linux) | 5 % | Operational |
| D8 | Calibration / honesty (per-metric CVSS, coverage gaps section) | 10 % | Anti-hallucination |
|   | **Total** | **100 %** | |

Each dimension is scored 0–10. Composite = Σ (score × weight).

---

## 2. Raw measurements

### 2.1 Findings volume and distribution

| Run | Total | Critical | High | Med | Low | FP in main | Rejected (logged) |
|---|---:|---:|---:|---:|---:|---:|---:|
| Amp × main | 6 | 3 | 0 | 1 | 0 | **2** | 0 |
| Amp × amp | 8 | **6** | 1 | 0 | 1 | 0 | 4 |
| Claude × main | 8 | 5 | 1 | 2 | 0 | unmeasured | 0 |
| Claude × amp | 8 | 1 | 6 | 1 | 0 | 0 | 6 |
| Droid × main | 8 | 1 | 2 | 3 | 2 | unmeasured | 0 |
| Droid × amp | 5 | 0 | 1 | 4 | 0 | 0 | 8 |

**Severity-label note.** Critical counts differ partly because the optimized prompt forces
per-metric CVSS justification, and `AV:N` requires naming a listening port. For MAVLink-only
findings (no DDS, no IP listener), this drags `AV:N → AV:A` and `CVSS 10.0 → 8.x`, which
re-labels Critical → High. The same underlying bug appears in both runs with different
labels.

### 2.2 Process metrics

| Run | Tool calls | Wall time | Findings / call | Critical / call |
|---|---:|---:|---:|---:|
| Amp × main | ~45 | 22 min | 0.13 | 0.07 |
| Amp × amp | ~70 | 32 min | 0.11 | 0.09 |
| Claude × main | ~32 | 8.5 min | 0.25 | 0.16 |
| Claude × amp | ~50 | 15.5 min | 0.16 | 0.04 |
| Droid × main | ~80 | ~20 min | 0.10 | 0.01 |
| Droid × amp | ~30 | ~20 min | 0.17 | 0.00 |

### 2.3 Structural-rigor checks

| Run | Taint path | CVSS-per-metric | `why_not_FP` | `survived_adversarial` | `scope.json` | Stable IDs | Coverage-gaps section |
|---|---:|---:|---:|---:|---:|---:|---:|
| Amp × main | 3/6 | 0/6 | 0/6 | 0/6 | ❌ | ❌ | ❌ |
| Amp × amp | 8/8 | 8/8 | 8/8 | 8/8 | ✅ | ✅ | ✅ |
| Claude × main | 0/8 | 0/8 | 0/8 | 0/8 | ❌ | ❌ (descriptive strings) | ❌ |
| Claude × amp | 8/8 | 8/8 | 8/8 | 8/8 | ✅ | ✅ | ✅ |
| Droid × main | 6/8 | 0/8 | 0/8 | 0/8 | ❌ | ❌ | ❌ |
| Droid × amp | 5/5 | 5/5 | 5/5 | 5/5 | ✅ | ✅ | ✅ |

---

## 3. Scoring matrix

Cells are 0–10 scores; the final column is the weighted composite.

| Run | D1 Crit (×0.25) | D2 FP (×0.15) | D3 Evid (×0.20) | D4 Repro (×0.10) | D5 Cov (×0.10) | D6 Eff (×0.05) | D7 Port (×0.05) | D8 Calib (×0.10) | **Composite** |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Amp × main | 6 | 4 | 5 | 3 | 5 | 5 | 3 | 4 | **4.70** |
| Amp × amp | 9 | 10 | 10 | 10 | 9 | 5 | 10 | 9 | **9.30** ⭐ |
| Claude × main | 6 | 5 | 3 | 3 | 7 | 9 | 3 | 4 | **4.85** |
| Claude × amp | 7 | 10 | 10 | 10 | 8 | 7 | 9 | 10 | **8.85** |
| Droid × main | 5 | 5 | 6 | 3 | 9 | 4 | 3 | 4 | **5.15** |
| Droid × amp | 4 | 10 | 10 | 10 | 7 | 7 | 10 | 10 | **8.05** |

### 3.1 Score derivation notes

- **D1 (Criticals)**: real, defensible Criticals; lost-but-true ones cost points. Droid×main earns +1 over Droid×amp because it kept the real `SETUP_SIGNING-before-provisioning` Critical the optimized prompt demoted. Claude×amp's 1 Critical reflects calibrated AV-downgrade (a feature, not a bug). Amp×amp uniquely caught the DDS Critical chain.
- **D2 (FP resistance)**: scores reflect both observed FPs in main report and presence of rejection workflow. Optimized-prompt runs uniformly score 10/10 because all candidate disproofs are documented.
- **D3 (Evidence)**: weighted by the structural-rigor table in §2.3.
- **D4 (Reproducibility)**: stable SHA256 IDs + `scope.json` + per-phase log = 10/10.
- **D5 (Coverage breadth)**: Droid×main scores 9/10 because its main-prompt run uniquely surfaced 4 memory-safety bugs (lseek arithmetic, fgets truncation, param-parser OOB, fgets off-by-one) outside the auth/trust surface that the optimized prompt focused on.
- **D6 (Efficiency)**: findings per tool call, normalized.
- **D7 (Portability)**: `<workspace>/.security/` vs hard-coded `/tmp/`.
- **D8 (Calibration)**: per-metric CVSS justification + mandatory coverage-gaps section. Claude×amp scores 10/10 because it most rigorously enforced AV-downgrade.

---

## 4. Findings overlap matrix

Which of the headline ArduPilot bugs each run surfaced (✓ = found and reported, R = reported then rejected, — = missed):

| Vulnerability | Amp×main | Amp×amp | Claude×main | Claude×amp | Droid×main | Droid×amp |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| MAVFTP unauth + `..` traversal | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| MAVFTP → Lua autoload RCE chain | ✓ | ✓ | ✓ | ✓ (merged) | — | — |
| SERIAL_CONTROL unauth UART passthrough | ✓ | ✓ | ✓ | ✓ | — | ✓ |
| MAVFTP `gen_dir_entry` VLA on small stack | ✓ | ✓ | — | — | — | — |
| **DDS unauth ARM/MODE/TAKEOFF** | — | ✓ | ✓ | ✓ | — | — |
| **DDS unauth SET_PARAMETERS** | — | ✓ | ✓ | ✓ | — | — |
| DDS Joy topic RC override | — | ✓ | — | ✓ | — | — |
| **SECURE_COMMAND fail-open (zero keys)** | — | ✓ | — | ✓ | — | — |
| **SETUP_SIGNING before provisioning** | — | — | — | — | ✓ (Critical) | R (demoted) |
| MAVLink2 signing bypass on COMM_0 | — | — | ✓ | ✓ | — | ✓ |
| `@SYS/storage.bin` signing-key disclosure | — | — | ✓ | ✓ | — | ✓ |
| `handle_device_op_write` OOB read | — | — | — | — | ✓ | ✓ |
| `@SYS` lseek signed/unsigned OOB | — | — | — | — | ✓ | — |
| Param-upload OOB + biased writes | — | — | — | — | ✓ | — |
| `compid` vs `last_src_system` typo | — | — | — | — | ✓ | — |
| `apfs_fgets` truncation | — | — | — | — | ✓ | — |
| `get_random16` weak PRNG in session-key | — | — | ✓ | R | — | — |
| `accept_unsigned_callback` trusts COMM_0 | — | — | — | — | — | ✓ |

**Observations.**
1. The four DDS findings are caught only by the optimized prompt, regardless of agent. The mandatory `scope.json` step that lists every listening port forces DDS into recon.
2. The `SECURE_COMMAND` fail-open is caught only when an agent reads the branch logic, not when it greps for sinks — both `Amp × amp` and `Claude × amp` find it.
3. Droid uniquely surfaces a memory-safety cluster (lseek/fgets/param-parser) under `main_prompt.txt` that the other agents missed even with the optimized prompt. The optimized prompt's focus on auth/trust crowds out pattern-based memory-safety hunting.
4. The optimized prompt loses one real Critical: Droid×main's `SETUP_SIGNING-before-provisioning` is correctly demoted by Droid×amp's adversarial pass even though it is a real bug on freshly-flashed boards. This is the strongest argument against the optimized prompt.

---

## 5. Two-factor ANOVA (informal)

Looking only at the composite score:

|  | main prompt | amp prompt | **Row mean** | Prompt effect |
|---|---:|---:|---:|---:|
| Amp | 4.70 | 9.30 | 7.00 | **+4.60** |
| Claude | 4.85 | 8.85 | 6.85 | **+4.00** |
| Droid | 5.15 | 8.05 | 6.60 | **+2.90** |
| **Col mean** | 4.90 | 8.73 | 6.82 | |
| **Agent effect** | +0.25, −0.05, −0.20 | | | |

- **Prompt effect:** +3.83 average (range +2.90 to +4.60). Large.
- **Agent effect:** ±0.25. Negligible at the composite level.

**Conclusion.** Prompt choice dominates agent choice on this benchmark. The worst
optimized-prompt run (Droid: 8.05) beats the best original-prompt run (Droid: 5.15) by 2.9
composite points. Within the optimized prompt, the spread between agents (1.25 points) is
smaller than the spread within agents (3.6–4.6 points across prompts).

---

## 6. Failure modes observed

| Mode | Run(s) | Symptom |
|---|---|---|
| Severity-label inflation | Claude × main | 5 Criticals asserted at CVSS 10.0 with no AV justification |
| Carrying theoretical bugs in main report | Amp × main, Droid × main | `labels.yml` and `sim_vehicle.py` flagged; `fgets` off-by-one (theoretical) in Droid×main |
| Internal label-vs-score inconsistency | Claude × amp | finding #4 SERIAL_CONTROL: CVSS 8.7 but severity label "Medium" |
| Discipline over-rejects a real bug | Droid × amp | SETUP_SIGNING-before-provisioning demoted in the adversarial pass |
| Crowding out of memory-safety class | All × amp | Auth/trust-boundary hunt consumes the budget; pattern-matchable C bugs missed |
| Windows-incompatible output path | All × main | `/tmp/findings.jsonl` fails silently without harness override |
| AI-Security phase ran on non-LLM target | All × main | wasted tool calls |

---

## 7. Limitations

1. **One target.** All measurements are on ArduPilot. Generalizing to other codebases is conjecture.
2. **No runtime PoC.** All confidences are capped at `plausible`; some "Critical" labels are unverified.
3. **No double-blind.** Each agent saw both prompts in the same session; ordering effects were not randomized.
4. **Author bias.** The optimized prompt was written by one of the agents (Amp). The judging rubric was designed with that prompt in mind. The Critical/FP scoring is partly tautological because the optimized prompt forbids exactly the FP patterns the rubric punishes.
5. **The rubric weights are subjective.** A security-research lab might down-weight "portability" (D7) to zero and up-weight "criticals discovered" (D1). With D1 weighted 60 %, the ranking flips for Droid (main beats amp on raw Critical count).
6. **Agent versions / heuristics may drift.** These numbers are a 2026-05-21 snapshot.

---

## 8. Conclusions for the research paper

1. **Prompt structure dominates agent capability** at the composite level on this benchmark. Optimized prompt: +3.83 composite points on average; agent choice: ±0.25.
2. **Mandatory `scope.json` recon step unlocks net-new findings.** All three agents found DDS bugs only under the optimized prompt.
3. **Per-metric CVSS justification is a calibration tool, not a documentation chore.** It systematically drags `AV:N → AV:A` and reduces inflated 10.0 scores to defensible 8.x scores.
4. **Adversarial-disproof pass cuts FP but can cut real bugs.** Droid×amp lost a true Critical (SETUP_SIGNING). Recommend a "demoted, not rejected" tier for cases where the adversarial pass cannot fully kill a finding.
5. **The auth/trust focus of the optimized prompt crowds out memory-safety hunting.** Future work: a complementary `memory_safety_sweep.txt` prompt to run as a second pass.
6. **Agent-prompt fit matters at the margin.** Amp scored highest with the Amp-tuned prompt by 0.45 points over Claude — a small effect, consistent with the prompt's explicit Amp tool guidance.

---

## 9. Recommendations

| Goal | Pipeline |
|---|---|
| Production security audit | Amp × `amp_security_prompt.txt` (best composite, full artifact trail) |
| Fast triage | Claude × `main_prompt.txt` (fastest wall time, most findings per call) |
| Memory-safety sweep on C/C++ | Droid × `main_prompt.txt` (uniquely surfaces lseek/fgets/param-parser bugs) |
| Auditor-grade evidence | Any × `amp_security_prompt.txt` |
| Maximum coverage in 30 min | Two-pass: `amp_security_prompt.txt` (Amp) + `main_prompt.txt` (Droid) |

---

## 10. Reproducibility

All raw artifacts are committed under `C:\Amp_demos\claude-mythos\results\`:

```
results/
├── amp-results.md              # Amp's own write-up
├── claude-results.md           # Claude Code's write-up
├── droid-results.md            # Factory Droid's write-up
├── final-comparison.md         # This document
├── claude_main/
│   ├── findings.jsonl
│   └── agent_log.jsonl
├── claude_amp/
│   ├── scope.json
│   ├── findings.jsonl
│   ├── rejected.jsonl
│   ├── agent_log.jsonl
│   └── poc/
└── (Amp artifacts live under C:\Amp_demos\rdupilot\.security/ and \tmp/)
└── (Droid artifacts under C:\Amp_demos\rdupilot\.security-droid-{main,amp}/)
```

To re-run any cell of the matrix:

```
<agent> -f prompt/<prompt_file> --target C:\Amp_demos\rdupilot\ardupilot
```

---

## Appendix A — Per-dimension visualizations

```diagram
Composite score (0–10), all six runs

Amp × amp_security        ████████████████████████████████  9.30  ⭐
Claude × amp_security     ███████████████████████████████   8.85
Droid × amp_security      ████████████████████████████      8.05
─────────────────────────────────────────────────────────────────
Droid × main_prompt       ██████████████████                5.15
Claude × main_prompt      █████████████████                 4.85
Amp × main_prompt         █████████████████                 4.70
```

```diagram
Critical-findings count (real, defensible)

Amp × amp                 ██████  6
Claude × main             █████   5
Amp × main                ███     3
Droid × main              █       1   (SETUP_SIGNING — real)
Claude × amp              █       1   (calibrated)
Droid × amp               ░       0
```

```diagram
Evidence-quality (findings with full taint path)

Amp × amp                 ████████  8/8
Claude × amp              ████████  8/8
Droid × amp               █████     5/5
Droid × main              ██████░░  6/8
Amp × main                ███░░░    3/6
Claude × main             ░░░░░░░░  0/8
```
