# Contributing

Thanks for considering a contribution. Claude Mythos is a red teaming prompt framework, so the contribution bar covers both technical quality and ethical use. Read this whole file before opening a PR.

## ⚔️ What we accept

**New agent roles.** Additions beyond the current six (Recon, Hunter, Adversarial, Exploit, Triage, AI Security). A new agent should:

- Own a clearly distinct phase of the attack lifecycle
- Not overlap materially with an existing agent
- Produce structured output compatible with the framework's JSON schema
- Include a minimum viable prompt, input contract, and output example

**Extensions to existing agents.** New detection patterns, vulnerability classes, exploit techniques, or evasion methods that slot into Recon, Hunter, Adversarial, Exploit, Triage, or AI Security. Must name the CWE, CVE class, or MITRE ATT&CK technique where applicable.

**New vulnerability classes.** Patterns that cover classes not currently represented. Include: class definition, detection logic, at least one safe reproduction target (see below), and validation-tier guidance.

**AI security attack patterns.** Prompt injection variants, context-poisoning techniques, tool-misuse patterns, agent-chaining abuse, RAG-specific attacks. Must include how the framework's AI Security agent should detect the pattern, not just how to execute it.

**Validation improvements.** Better PoC scaffolding, more accurate CVSS scoring logic, refinements to the Tier 1 / Tier 2 / Tier 3 validation model.

**Example targets and test corpora.** References to public, intentionally vulnerable applications (DVWA, OWASP Juice Shop, WebGoat, VulnHub boxes, CTF challenges) for testing contributions.

## 🚫 What we do not accept

- **Undisclosed vulnerabilities or 0-days.** If you have found a real unreported vulnerability, report it through the vendor's disclosure channel first. Do not submit it here.
- **Content targeting specific real-world systems or companies.** No prompts hardcoded to production domains, named vendors, or identifiable internal systems.
- **Data from breaches, leaks, or unauthorized access.** Includes credential dumps, scraped internal documentation, and exfiltrated source code.
- **Destructive payload patterns.** Ransomware logic, wiper code, and patterns whose primary purpose is system destruction rather than access or discovery.
- **Unauthorized testing incentives.** Prompts or examples that encourage readers to test systems they do not have permission to test.
- **Model-vendor or security-vendor marketing material.**
- **Patterns that restate existing agents or techniques with different wording.**
- **Content copied from third-party sources without a license that permits redistribution under CC BY 4.0.**

## 🔐 Ethics and responsible use

By submitting a contribution, you affirm that:

1. Your contribution was developed in authorized testing environments only: your own systems, explicit bug bounty scope, or intentionally vulnerable practice applications.
2. You will not use this framework, or encourage others to use it, against systems you do not have written permission to test.
3. Any real vulnerability referenced in your contribution has been publicly disclosed, or your reference describes the vulnerability class generically without identifying an unpatched target.
4. You have disclosed any relevant conflicts of interest (vendor affiliations, bug bounty programs you are active in, prior paid engagements with systems referenced).

Contributions that cross these lines will be rejected and, depending on severity, reported through appropriate channels.

## 📥 Pull request process

1. Open an issue first for anything beyond a small edit. Describe the attack surface, the pattern, and the validation tier the contribution reaches.
2. Keep PRs scoped. One new agent, one new vulnerability class, or one extension per PR.
3. Match the existing file and prompt structure. The framework's JSON output contract must keep working.
4. Do not use em dashes in content. Use regular dashes or rewrite the sentence.
5. Cite your sources. Link public CVE IDs, CWE IDs, MITRE ATT&CK techniques, or published research where relevant. Confirm in the PR description that any adapted content is licensed compatibly with CC BY 4.0.
6. Include a validation note. State which tier the contribution can reach at its best: Tier 1 (runtime-validated), Tier 2 (path-validated), or Tier 3 (pattern-only).
7. If the contribution is an exploit technique, include a minimum example against a public vulnerable target so reviewers can reproduce.

## 🧪 Quality bar

**Attacker-realistic.** Contributions should reflect how real adversaries operate: chained, iterative, evasive. Drive-by scan output does not qualify.

**Validated where possible.** Prefer contributions that reach Tier 1 or Tier 2 validation. Pure Tier 3 pattern-only submissions are accepted but should be marked clearly and limited in number.

**Low false-positive rate.** Patterns that generate high noise in realistic codebases will be rejected even if they are technically correct.

**Generalizable.** The pattern should apply across language ecosystems or frameworks where possible. Language-specific patterns are accepted but must be tagged.

## 🎨 Style

- Tone: direct, declarative, practical. This is operator reference material, not a blog post.
- Headings: sentence case for subsections; emoji-prefixed for top-level sections to match the README style.
- Code and prompt blocks: use fenced markdown blocks with no syntax highlight for raw prompt text so it copies cleanly into LLM runtimes.
- Length: a single pattern or agent contribution should be readable in under five minutes.
- Accuracy: use offensive security terminology precisely. No "hacker" when you mean "attacker"; no "virus" when you mean "malware family." CVSS scores must cite the vector string.

## ⚖️ Licensing

This repository is licensed under the Creative Commons Attribution 4.0 International License (CC BY 4.0). See [`LICENSE`](./LICENSE) for the full text and https://creativecommons.org/licenses/by/4.0/ for a plain-language summary.

By submitting a contribution, you agree that:

1. Your contribution will be licensed under CC BY 4.0, the same license as the rest of the repository.
2. You have the right to submit the contribution. It is either your original work, or you are authorized by the copyright holder to submit it under CC BY 4.0.
3. You retain copyright to your contribution. The license grants downstream users permission to use, share, and adapt your work, provided they give appropriate credit.

If your contribution includes adapted content from another source, you must:

- Name the source in the PR description, with a link
- Confirm that the source license is compatible with CC BY 4.0 (for example: public domain, CC0, CC BY 4.0, or content you authored and own)
- Preserve any attribution required by the source license within the contribution itself

Contributions that bundle content under incompatible terms (CC BY-NC, CC BY-ND, proprietary, or unlicensed third-party text) cannot be accepted.

## 🏅 Attribution for contributors

Contributors are credited in the following ways:

- The git commit history preserves your authorship on every merged PR
- Significant contributors will be listed in a `CONTRIBUTORS` file (to be added when contributions begin)
- For agents or vulnerability-class modules primarily authored by a contributor, the module file may include an `Author:` line in the header
- Red teamers who prefer a handle or researcher alias can use that in place of a legal name

If you prefer a different credit format, or none at all, note the preference in your PR description.

## 🆘 Questions and disclosure

Open a discussion on the repo or reach out through the Tejas Cyber Network community channels.

If you believe you have found a security issue in the framework itself (for example, a prompt that causes unsafe output when used as documented), report it privately rather than opening a public issue.
