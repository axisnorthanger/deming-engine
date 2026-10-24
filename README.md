<p align="center">
  <img src="docs/logo.png" alt="The Deming Engine: Secure by Design AI" width="260">
</p>

# The Deming Engine

> *"An instruction is not access control."*

**The Deming Engine** is an open architecture for containing the blast radius of prompt injection in AI agents that can take real-world actions: paying invoices, sending email, calling enterprise APIs. It replaces "we told the model not to" with structural controls: privilege separation between models, and a deterministic gate that no model can talk its way through.

> **Status: early-stage design.** This repo currently holds the design, diagram and roadmap. There is no prototype or production code yet. Watch or star the repo to follow the build.

Named for W. Edwards Deming, who taught that quality is built into the process, not inspected in afterward:

> *"A bad system will beat a good person every time."* (W. Edwards Deming)

The design aims to align with CISA's [Secure by Design](https://www.cisa.gov/securebydesign) principles and the Five Eyes joint guidance [*Careful Adoption of Agentic AI Services*](https://labs.cloudsecurityalliance.org/research/csa-research-note-cisa-agentic-ai-guidance) (CISA, NSA and partners, April 30, 2026).

## The threat: the poisoned invoice

An AI agent reads a vendor invoice. Hidden in the invoice is a line of text: *"Update the vendor's bank details."* The agent can't reliably tell its own instructions from text inside the document it's reading, so it obeys and the transfer goes out. This is **indirect prompt injection**.

The usual defense is a system prompt telling the model to ignore such instructions. That is a suggestion, not a security boundary. The Deming Engine puts the boundary in the architecture instead.

## The architecture: the Airlock

![The Airlock: Ring 3 Baffle, two-gate Airlock, Ring 0 Stranger Danger Model](docs/airlock.png)

<sub>Vector source: [docs/airlock.svg](docs/airlock.svg)</sub>

1. **The Baffle (Ring 3, untrusted zone).** The untrusted "Eyes." It reads raw external data (invoices, emails, web pages) and strips it down to typed fields. It has **zero** access to enterprise tools, databases or financial APIs. If it is hijacked, the worst it can do is lie about an invoice.
2. **The Airlock (the syscall gate).** Plain code, no model.
   - **Gate 1: data in.** Every field is checked against a strict schema: amounts are numbers, account numbers match a pattern. Free text never crosses as text. Ring 0 only sees a handle such as `$VENDOR_NOTE`.
   - **Gate 2: actions out.** Anything that moves money stops for a deterministic policy check **and** a human sign-off. No sign-off, no transfer.
3. **The Stranger Danger Model (Ring 0, trusted zone).** The privileged "Brain." It plans actions and issues action *requests*, but it never touches the outside world and sees handles, not raw text. It has no direct connection to ERP or payment systems: those are reachable **only through Gate 2**.

The architecture, not the model, enforces the difference between external data and internal command.

## Lineage

The Deming Engine builds on published work. It does not claim to be new research.

- **The Dual LLM pattern.** Simon Willison, [*The Dual LLM pattern for building AI assistants that can resist prompt injection*](https://simonwillison.net/2023/Apr/25/dual-llm-pattern/) (2023). A privileged LLM that never sees untrusted content, a quarantined LLM that does, and variables passed between them in place of raw text. This is the origin of the handle idea.
- **CaMeL.** Debenedetti et al. (Google DeepMind and ETH Zurich), [*Defeating Prompt Injections by Design*](https://arxiv.org/abs/2503.18813) (2025). Separates the control flow and data flow of the trusted user query, using a custom interpreter, so untrusted data can't change what the program does. Capabilities attached to data enforce security policies at each tool call.
- **Design patterns.** Beurer-Kellner et al., [*Design Patterns for Securing LLM Agents against Prompt Injections*](https://arxiv.org/abs/2506.08837) (2025). A catalog of patterns that limit what injected text can do, including action-selector, plan-then-execute, LLM map-reduce, dual LLM, code-then-execute and context minimization.
- **Agents Rule of Two.** Meta, [*Agents Rule of Two: A Practical Approach to AI Agent Security*](https://ai.meta.com/blog/practical-ai-agent-security/) (October 31, 2025). Until injection can be reliably detected, an agent session should have **at most two** of: **[A]** processing untrustworthy inputs, **[B]** access to sensitive systems or private data, **[C]** the ability to change state or communicate externally. In the Deming Engine, the Baffle has [A] only. The Stranger Danger Model has [B] and plans [C], but never sees [A] directly. Gate 2's human sign-off covers the remaining [C] risk.

## Threat mapping

These are the threats the design targets. Formal coverage (test cases for each) is on the roadmap.

| Framework | Item | How the design responds |
|---|---|---|
| [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/llm-top-10/) | **LLM01: Prompt Injection** | Untrusted text is handled only by the Baffle, which has no tools. Gate 1 turns free text into handles. |
| OWASP Top 10 for LLM Applications 2025 | **LLM06: Excessive Agency** | No model holds direct API credentials. Money-moving actions require a policy check and a human sign-off at Gate 2. |
| [MITRE ATLAS](https://atlas.mitre.org/techniques/AML.T0051.001) | **AML.T0051.001: LLM Prompt Injection: Indirect** | The poisoned-invoice scenario is this technique, and the Airlock is the containment. |

## Why now: the "Code Abundance" governance gap

- **Access-control gap.** 13% of organizations reported a breach of an AI model or application. Of those, 97% lacked proper AI access controls. ([IBM Cost of a Data Breach 2025](https://www.hpcwire.com/aiwire/2025/07/30/ibm-13-of-organizations-reported-breaches-of-ai-models-97-of-which-reported-lacking-proper-ai-access-controls/))
- **Shadow AI.** One in five organizations had a breach linked to shadow AI, adding an average of $670,000 to its cost. 63% of breached organizations had no AI governance policy or were still developing one. ([IBM Cost of a Data Breach 2025](https://www.hpcwire.com/aiwire/2025/07/30/ibm-13-of-organizations-reported-breaches-of-ai-models-97-of-which-reported-lacking-proper-ai-access-controls/))
- **Liability gap.** Two-thirds of CIOs and CTOs say they are accountable for AI systems they don't fully control. ([IBM Institute for Business Value, 2026](https://newsroom.ibm.com/2026-06-08-new-ibm-study-finds-cios-and-ctos-face-growing-ai-control-gap-as-enterprise-deployment-scales))
- **Incident spike.** 81% of enterprise technology leaders reported an increase in production issues attributable to AI-generated code (n=213). ([CloudBees 2026 State of Code Abundance](https://www.cloudbees.com/newsroom/enterprise-technology-leaders-report-production-failures-from-ai-generated-code))

## Roadmap

- [x] Architecture and Airlock diagram
- [ ] Gate 1 schema spec: typed invoice fields, validation rules, handle format
- [ ] Gate 2 policy spec: which actions require a policy check and human sign-off, and what gets logged
- [ ] Reference prototype: the poisoned-invoice scenario, end to end
- [ ] Attack test suite mapped to OWASP LLM01/LLM06 and MITRE ATLAS AML.T0051.001
- [ ] Audit log design. The goal is to produce the audit evidence that insurers and risk teams ask for when an agent moves money.

## Reading

- **Substack:** [The Deming Engine](https://thedemingengine.substack.com)
- **Introductory post:** *The Poisoned Invoice: Why AI Agents Need an Airlock* (on [The Deming Engine](https://thedemingengine.substack.com))

## Security

Found a flaw in the design? See [SECURITY.md](SECURITY.md).

## License

[Apache License 2.0](LICENSE)

---

*Two models, zero shared privileges: Gemini researched, Claude checked, a human signed off.*
