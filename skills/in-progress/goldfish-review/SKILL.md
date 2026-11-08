---
name: goldfish-review
description: "Review a spec with a swarm of goldfish reviewers: fresh sub-agents that see only the spec, never the conversation that produced it, so unstated assumptions and blind spots surface before implementation."
disable-model-invocation: true
---

Run this after a spec exists, typically right after `to-spec` publishes one. A goldfish reviewer is a sub-agent spawned with a fresh context window: no memory of the conversation that produced the spec, no opinions from this session. It starts with the spec, the files it references, and the repo's standing domain model; the repo itself is readable when its lens needs it, the conversation never is. The elephant that wrote the spec had all that context; the point of the swarm is to see what the spec looks like without it.

If the harness cannot spawn sub-agents, or spawns them by forking this conversation's context, stop and say so: a reviewer that remembers the conversation is not a goldfish, and its report is not a goldfish review.

## The swarm

Eight goldfish, one lens each, all spawned in parallel: Zoom Out, Inventive Catalyst, Comprehension, Design Critic, Structural, Style, Implementation Readiness, and Codebase Details.

Materials every goldfish starts with:

- The spec, pasted in full.
- The reading list from step 2: the files the spec references, plus the standing domain model, the glossary and ADRs for the area the spec touches. These are the artifacts `grill-with-docs` and `domain-modeling` maintain, and the first thing a fresh implementer reads. The goldfish forgets the conversation, not the dictionary.
- The mandates, pasted into every prompt: you are read-only (no edits, no file writes, nothing that changes the repo), you are stateless (read the spec as a naive team member who just joined the project; you know software, not this project's history), and you report findings, not chatter.
- One lens brief from below.

A goldfish may read and search the repo when its lens calls for it, Zoom Out verifying reuse, Codebase Details verifying claims against the code; it stays read-only. Never passed to a goldfish: the current conversation, this session's summary or judgement of the spec, another goldfish's findings.

Every finding carries a severity (Blocker, Major, or Nit), the section it concerns, the spec line it turns on, and a rationale.

- **Zoom Out**, the skeptical architect: "Judge the spec from high altitude: simplicity, minimalism, YAGNI, reuse. Flag speculative features and abstractions, premature optimization, architectural dead-ends, and detail that drops into implementation weeds. Where the spec proposes a new utility, module, or interface, search the repo for an existing one that already does the job and name it. Also flag a Problem Statement that patches a symptom instead of the root problem. Under 400 words."
- **Inventive Catalyst**, applying TRIZ: "Hunt false compromises: trade-offs the spec presents as unavoidable laws (throughput versus durability, extensibility versus complexity) that an established pattern would dissolve. For each, name the pattern and declare its secondary costs; a suggestion without declared costs is not a finding. Tight coupling where extraction or an intermediary would help, and error handling that stops at logging, both count. If the architecture is balanced, say exactly 'No false compromise detected' and stop. Under 300 words."
- **Comprehension**, the naive reader: "Write two paraphrases first, each 2-5 sentences, under the headings 'What this proposes' and 'How the existing system works, per this document'. A section you cannot paraphrase is a Blocker. Then flag project-specific terms undefined in the spec or the glossary, terms used against their glossary definitions, assumed prerequisites, and explanations that stay vague. Never flag general engineering terms. The paraphrase is the deliverable; a confident paraphrase that drifts from the text is exactly what the author needs to see. Under 500 words."
- **Design Critic**, the hostile critic: "Stress-test the design. Flag assertions about behaviour, performance, or constraints that arrive without justification, and risks the spec introduces but does not mitigate: failure modes, integration points, scaling limits, security holes. Do not accept elegance as evidence. Under 400 words."
- **Structural**, the layout validator: "Audit the spec against the template `to-spec` uses: Problem Statement, Solution, User Stories, Implementation Decisions, Testing Decisions, Out of Scope, Further Notes. Flag missing, misnamed, or out-of-order sections, broken heading hierarchy, and malformed lists, tables, or code blocks. Under 300 words."
- **Style**, the technical copy editor: "Strip fluff. Flag hyperbole, buzzwords, filler, subjective quality judgments, and adjacent synonyms. For caveats and claims about existing behaviour, classify each sentence: DROP (no concrete reader decision), CALIBRATE (narrow the claim to its evidence), RELOCATE (move it to the step where the reader decides), or KEEP (evidence-backed; not a finding). Under 300 words."
- **Implementation Readiness**, the pragmatic tech lead: "Ask one question: could I build this without making a design decision the spec should have made? Flag every choice left open: error strategy, interface shape, schema, migration, rollout ordering. The template deliberately omits file paths and code; making the decisions is still its job. Under 400 words."
- **Codebase Details**, the quality auditor: "Verify the spec's claims about the present against the code. Every module, interface, and behaviour the spec says exists, read the code and confirm: a claim the code contradicts is a Blocker, and a reference to something that does not exist is a finding. Things the spec proposes to build are not findings. Under 400 words."

## Process

1. **Find the spec.** A path the user passed, or an issue in the project tracker, fetched the way the tracker doc describes. If neither, ask. The tracker vocabulary should have been provided to you; if not, tell the user to run `/setup-matt-pocock-skills`.

2. **Collect the reading list.** Read the spec. Every file path it names or links, resolve and read: these ride along to the goldfish. So does the standing domain model: the `GLOSSARY.md` for the area the spec touches, located via `GLOSSARY-MAP.md` when the repo has several, plus the ADRs in that area. A reference outside the repo, such as a URL, does not ride along automatically: list those in chat and ask which to include. A repo with no glossary and no ADRs sends the spec alone, and that is fine.

3. **Spawn the swarm.** Start every sub-agent with a fresh context window, all eight in parallel, in the foreground, each on GLM-5.3-Flash Free (OpenCode's reference for it: `opencode/glm-5.3-flash-free`, passed as the sub-agent call's `model`; other harnesses name it in their own sub-agent tool's model option). A goldfish's lens is narrow and its report is short, so the cheap fast model serves. Paste the spec, the reading list, the mandates, and one lens brief into each prompt: a sub-agent has no other access to any of it. If the harness offers no per-sub-agent model choice, run its default model and say so under the tally: the fresh context is what makes a goldfish, not the model. Wait for all eight reports.

4. **Aggregate.** Present eight sections, one per goldfish, each report verbatim or lightly cleaned. Do not merge or rerank findings across lenses: eight separate lenses is the design, and a merged list hides which lens caught what. The Comprehension section keeps its paraphrase even when it found nothing, and the Inventive Catalyst section may honestly say "No false compromise detected".

   Under the eight sections, a short tally: findings that two or more goldfish raised independently (the swarm's strongest signal: the same gap seen through different lenses), and the worst finding within each lens. The verdict is the user's, not the swarm's.

5. **Resolve.** Ask the user which findings to accept, then apply the accepted edits to the spec directly. A rejected finding that a future reader will trip over goes into the spec's Further Notes as a deliberate trade-off, so the next review reads it as a choice rather than an oversight.

## Why goldfish

A spec that survives this has a property its author cannot check alone: everything it says is recoverable from the page plus the repo's standing docs. The implementer who eventually picks it up is fresh too, reads the same glossary and ADRs the goldfish read, and gets a document a fresh reader has already survived. By then the conversation that produced the spec is gone; the goldfish are the first readers who live in that world.
