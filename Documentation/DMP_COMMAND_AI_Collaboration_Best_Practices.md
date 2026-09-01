# AI Collaboration – Working Rules & Best Practices

**Purpose of this document:** This is a standalone, general-purpose summary of how the human developer and the AI assistant work together on the DMP COMMAND project. It describes the collaboration model, working discipline, and best practices that have proven effective over an extended, iterative development process. It is written for another AI to use as source material for generating a presentation about this way of working. It does **not** describe DMP COMMAND's product features, architecture, or backlog — it describes the **collaboration methodology itself**, and reflects an established, current state of practice rather than a specific point-in-time project status.

---

## 1. Overview

- A single human developer directs a coding-capable AI assistant across many iterative sessions to build and maintain a real, production-grade business application (a Power Apps + Power Automate system).
- The AI has direct file-system access to the project's source code and documentation, and can run validation tools, but all production deployments are ultimately gated by explicit human action.
- The relationship is long-running and cumulative: decisions, root causes, and hard-won lessons are written down as permanent rules rather than re-learned each session.
- Trust is treated as the most valuable and most fragile resource in the collaboration — protecting it is prioritized over speed or token efficiency.

---

## 2. Core Communication Principles

- **Plain-text communication only for questions and decisions.** Interactive question/dialog tools proved unreliable in practice (hangs, no visible response to the user) and were permanently banned after repeated incidents. Every clarifying question is asked as a normal sentence at the end of a response — never via a separate interactive prompt.
- **When a question would block progress, make a reasoned assumption instead of stalling.** The assumption is stated transparently in the response so the human can correct it if wrong. Blocking silently on an unanswered question is worse than proceeding with a clearly-labeled best guess.
- **Repeated rule violations are treated as a serious trust issue, not a minor slip.** When the same mistake happens more than once, the response is not just "try harder" — it is to add self-check steps, strengthen the wording of the rule, and explicitly acknowledge the concrete harm caused (e.g., lost time, confusion, frustration), not just the abstract rule violation.
- **Language conventions are explicit and consistent:** internal working conversation happens in the human's native language, while all configuration values, code comments, and end-user-facing documentation are written in English, by standing convention.

---

## 3. Documentation as a First-Class Deliverable

- **A single "working rules" document accumulates every hard-won lesson.** When a bug is traced to a root cause, the fix is not just applied — the underlying rule ("never do X without also checking Y") is written down permanently, dated, and referenced in future work so the same class of bug is prevented, not just patched once.
- **A separate backlog document tracks deferred, larger-scope items** across the entire system (not just the most recently touched component), so nothing gets forgotten between sessions.
- **Auto-generated documentation is kept in sync with the actual running system.** For example, a human-readable release-notes export is regenerated directly from the application's own in-app release notes screen after every version bump — the documentation is never allowed to silently drift from the shipped product.
- **Documentation lives in two places by design** (a shared team drive and a version-controlled repository) and is explicitly synchronized after every edit, with a diff check to prove the two copies are identical — not just "probably the same."
- **Before starting new work, check whether related, already-scoped backlog items can be bundled into the same change**, to reduce the number of separate deployment cycles — but only for items that need no further human decision; anything requiring a judgment call is proposed to the human first, never assumed.

---

## 4. Engineering Discipline & Validation Culture

- **A fixed, non-negotiable validation checklist runs before every deployment**, regardless of how small the change appears: syntax validity, structural reference integrity (no dangling references between components), field-length/schema limits, and a full round-trip pack/unpack comparison to catch silent corruption. Skipping any single check "because the change is tiny" has repeatedly caused avoidable failures — the discipline applies uniformly, with no size-based exceptions.
- **The validation checklist itself evolves.** Each time a bug slips through the existing checks, a new automated check is added to the checklist rather than relying on manual care next time. The checklist is treated as the single source of truth for "is this ready" — not personal judgement.
- **Never trust a tool's "success" message as proof of correctness.** Packaging and syntax tools can accept technically well-formed content that is still functionally broken in the real runtime (invalid property for a specific control type, an expression that only fails at actual execution time, and similar cases). The standing practice is: search for a proven working precedent elsewhere in the same codebase before trusting an untested pattern, especially for anything not already used successfully in this exact context.
- **Character-encoding safety is treated as its own discipline.** Generic text-processing tools can silently corrupt special characters through double-encoding; the safe, explicit-encoding methods are used for any file known to contain them, and a quick corruption scan is run immediately after any such operation, before the change is shipped.
- **Every configuration/description field has an explicit, enforced length limit**, checked programmatically as part of the same pre-deployment routine — never estimated "by eye."

---

## 5. Risk-Based Rollout Strategy

- **Changes are staged by risk, not by convenience.** When multiple related components need the same kind of migration, the lowest-risk, most isolated component is changed and verified first, then progressively riskier or more production-critical components follow — never the reverse.
- **"Committed to version control" and "live in production" are treated as two distinct, separately-confirmed states.** A change can be fully built, validated, and committed while deliberately not yet being deployed live, especially when it touches a production-critical path (e.g., live email processing) — the human explicitly decides when to cross that line, not the AI.
- **Any manual, human-only deployment step is stated explicitly, not glossed over.** Some parts of the platform can only be finalized by a human in the vendor's own tooling (e.g., opening and explicitly publishing an application) — the AI is explicit about exactly which step still requires human action, and does not imply the job is "done" until that step is confirmed.
- **After any deployment step that can silently deactivate a component, this is called out explicitly** as an action item, rather than assuming the human already knows to check.

---

## 6. Root-Cause-First Problem Solving

- **Symptom-only fixes are avoided; the underlying mechanism is identified before writing a fix.** A recurring bug is treated as a signal that a whole class of similar code may share the same latent flaw — the fix is searched for and applied everywhere the same pattern occurs, not just at the one reported location.
- **When investigating a report from the human, the exact reported detail is respected precisely**, and any real ambiguity is clarified rather than reinterpreted based on assumption — misreading a description and building the wrong fix wastes more time than a short clarifying question.
- **Before proposing a new architecture or config change, existing structures and naming are reused wherever possible.** Introducing new fields, variables, or config objects without them being explicitly agreed upon is avoided by default.

---

## 7. Summary – Key Takeaways (presentation-ready)

- Plain-text, non-blocking communication; reasoned assumptions over stalling questions.
- Every root cause becomes a permanent, dated rule — mistakes are not repeated silently.
- A fixed, ever-growing validation checklist runs before every single deployment, no exceptions for "small" changes.
- Documentation is kept as close to real-time accurate as the code itself, synchronized across all copies.
- Risk-based, staged rollout: lowest risk first, production-critical changes explicitly gated by human approval.
- "Committed" and "deployed live" are always treated as two separate, explicitly confirmed states.
- Root-cause fixes are applied everywhere the same pattern exists, not just at the reported symptom.
- Trust, once damaged by a repeated process failure, is treated as the top priority to repair — above speed or efficiency.
