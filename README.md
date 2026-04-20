[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

# 🔴 Claude Mythos Red Teaming Framework

> Turn LLMs into **multi-agent offensive security systems** capable of discovering real, exploitable vulnerabilities.

---

## ⚔️ What is this?

**Claude Mythos Red Teaming Framework** is a production-grade prompt + methodology that transforms LLMs into a **coordinated red team**.

Unlike traditional prompts that generate shallow findings, this framework enables:

- Multi-agent vulnerability discovery  
- Adversarial exploit chaining  
- Runtime exploit validation  
- AI/LLM-specific attack detection  

---

## 🚨 Why this matters

Most teams are doing this:

> “Find vulnerabilities in this code.”

That’s not how attackers operate.

Real attackers:
- Iterate  
- Chain bugs  
- Bypass defenses  
- Validate exploits  

This framework makes LLMs do the same.

---

## 🧠 Architecture

```
RECON → HUNTER → ADVERSARIAL → EXPLOIT → TRIAGE → AI SECURITY
```

Each agent has a specialized role:

| Agent | Role |
|------|------|
| Recon | Maps attack surface |
| Hunter | Finds vulnerabilities |
| Adversarial | Chains exploits |
| Exploit | Validates with PoCs |
| Triage | Scores severity (CVSS) |
| AI Security | Detects LLM-specific risks |

---

## 🔥 Key Capabilities

### 🔍 Deep Vulnerability Discovery
- Injection (SQL, command, template)
- Auth/AuthZ bypass
- Logic flaws & race conditions
- Deserialization & memory issues
- Path traversal & file abuse
- Crypto misuse

### ⚡ Exploit Validation
- Generates real PoCs
- Runtime execution
- Multi-tier validation:
  - **Confirmed**
  - **Plausible**
  - **Theoretical**

### 🧠 Adversarial Mode
- Multi-step exploit chains
- Privilege escalation
- Defense bypass techniques
- Encoding tricks & edge cases

### 🤖 AI Security
- Prompt injection
- Context poisoning (RAG)
- Tool misuse
- Data exfiltration
- Unsafe agent chaining

### 🔐 Supply Chain & Secrets
- Hardcoded credential detection
- Dependency risk analysis
- CI/CD attack vectors

---

## ⚡ Quick Start

### 1. Clone the repo

```bash
git clone https://github.com/anshug/claude-mythos.git
cd claude-mythos
```

---

### 2. Use the prompt

Copy the prompt from:

```
/prompt/main_prompt.txt
```

Paste it into:
- Claude  
- ChatGPT  
- Any local LLM runtime  

---

### 3. Provide your target

- Full codebase
- Runtime environment (optional but recommended)

---

### 4. Run analysis

```
Analyze this codebase for high-impact, exploitable vulnerabilities.
```

---

## 🧾 Output Example

```json
{
  "agent": "EXPLOIT",
  "phase": 4,
  "file_path": "/src/auth/login.js",
  "vuln_class": "SQL Injection",
  "confidence": "confirmed",
  "cvss_score": 9.8,
  "summary": "Authentication bypass via unsanitized SQL query"
}
```

---

## 🎯 Use Cases

- Red Teaming / Offensive Security  
- AppSec Reviews  
- AI Agent Security Testing  
- Startup Security Readiness  
- Bug Bounty Augmentation  

---

## ⚠️ Constraints

- Run ONLY in isolated environments  
- No data exfiltration  
- No destructive actions outside scope  
- Authorized testing only  

---

## 🏆 What makes this different

Most tools:
- Static scanning  
- High noise  
- No validation  

This framework:
- Thinks like an attacker  
- Validates exploits  
- Chains vulnerabilities  
- Focuses on **real-world impact**

---

## 🧪 Validation Model

| Tier | Meaning |
|------|--------|
| Tier 1 | Confirmed (runtime exploit) |
| Tier 2 | Plausible (validated path) |
| Tier 3 | Theoretical (pattern only) |

> High/Critical issues require Tier 1 or Tier 2.

---

## 🔥 Roadmap

- [ ] Autonomous multi-agent execution (CrewAI / LangGraph)
- [ ] Fuzzer + tool integrations
- [ ] CI/CD pipeline integration
- [ ] Benchmark datasets
- [ ] AI red teaming test suite

---

## 🤝 Contributing

Contributions welcome from.
- Security researchers  
- Red teamers  
- AI security engineers  

Open an issue or PR.

See CONTRIBUTING.md for the bar, ethics, and license.

---

## 👤 Author

**Anshu Gupta**  
- CISO / Security Leader  
- Founder, Tejas Cyber Network  
- AI Security & Offensive Security Practitioner  

---

## ⭐ Support

If this helped you:
- Star the repo ⭐  
- Share with your team  
- Use it in your security workflows  

---

## ⚖️ License

This work is licensed under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). See [`LICENSE`](./LICENSE) for the full text.

You are free to use, share, and adapt this framework for any purpose, including commercial use, provided you give appropriate credit.

**Suggested attribution:**

> Adapted from [claude-mythos](https://github.com/anshug/claude-mythos) by Anshu Gupta, licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

For inline use in a prompt file, a single-line credit is enough:

    # Adapted from claude-mythos (https://github.com/anshug/claude-mythos) by Anshu Gupta, CC BY 4.0

No endorsement implied. Use of this framework does not constitute endorsement by Anshu Gupta, Fixin Security, or Tejas Cyber Network of any product, service, or implementation.

---

## ⚖️ Disclaimer

This project is for:
- Authorized security testing  
- Research and education  

Do NOT use on systems without permission.
