---
name: goldfish-review
description: "Review a spec with a swarm of goldfish reviewers: fresh sub-agents that see only the spec, never the conversation that produced it, so unstated assumptions and blind spots surface before implementation."
disable-model-invocation: true
---

Run this after a spec exists, typically right after `to-spec` publishes one. A goldfish reviewer is a sub-agent spawned with a fresh context window: no memory of the conversation that produced the spec, no opinions from this session, nothing but the spec, the files it references, and the repo's standing domain model. The elephant that wrote the spec had all that context; the point of the swarm is to see what the spec looks like without it.

The metaphor and the review discipline come from Google's [elephant-goldfish-model](https://github.com/google/elephant-goldfish-model): elephants design with context, goldfish review without it.

If the harness cannot spawn sub-agents, or spawns them by forking this conversation's context, stop and say so: a reviewer that remembers the conversation is not a goldfish, and its report is not a goldfish review.

## The swarm

Five goldfish, one lens each, all spawned in parallel. Each gets the same materials and a different brief.

Materials every goldfish gets, and nothing else:

- The spec, pasted in full.
- The files the spec itself references, resolved and pasted (see step 2).
- The repo's standing domain model: the glossary and ADRs for the area the spec touches, collected in step 2. These are the artifacts `grill-with-docs` and `domain-modeling` maintain, and the first thing a fresh implementer reads. The goldfish forgets the conversation, not the dictionary.
- One lens brief from below.

Never passed to a goldfish: the current conversation, this session's summary or judgement of the spec, another goldfish's findings.

- **Comprehension**: "Restate the spec section by section in your own words, plainly. Then list every place you could not: a term undefined in the spec or the glossary, a term used against its glossary definition, a referent with no antecedent, two sections that say different things. The paraphrase is the deliverable; a confident paraphrase that drifts from the text is exactly what the author needs to see. Under 500 words."
- **Assumptions**: "List every unstated assumption the spec depends on: things the author clearly knows but never wrote. For each, say what breaks if the assumption is wrong, and quote the spec line that leans on it. Under 400 words."
- **Counter-argument**: "Make the strongest case against this spec. What is the author not seeing? Where does it break on an edge case, a scale change, or a hostile input? What would have to be true for the opposite design to be the right one? No praise, no hedging. Under 400 words."
- **Readiness**: "A fresh implementer gets this spec and the repo it lives in, but none of the conversation behind it. List every decision they would have to guess: ambiguities, missing interfaces, contradictions between Implementation Decisions, Testing Decisions that name seams the rest of the spec never establishes. Quote the spec line for each finding. Under 400 words."
- **Scope**: "Cross-check User Stories against Implementation Decisions and Out of Scope. List: stories with no decision backing them, decisions no story asked for, and Out of Scope items the stories quietly depend on. Quote the spec line for each finding. Under 400 words."

The lens briefs name the sections of the spec template `to-spec` produces. A missing section is itself a finding: report it under the lens that misses it most.

## Process

1. **Find the spec.** A path the user passed, or an issue in the project tracker, fetched the way the tracker doc describes. If neither, ask. The tracker vocabulary should have been provided to you; if not, tell the user to run `/setup-matt-pocock-skills`.

2. **Collect the reading list.** Read the spec. Every file path it names or links, resolve and read: these ride along to the goldfish. So does the standing domain model: the `GLOSSARY.md` for the area the spec touches, located via `GLOSSARY-MAP.md` when the repo has several, plus the ADRs in that area. A reference outside the repo, such as a URL, does not ride along automatically: list those in chat and ask which to include. A repo with no glossary and no ADRs sends the spec alone, and that is fine.

3. **Spawn the swarm.** Start every sub-agent with a fresh context window, all five in parallel, in the foreground. Paste the spec, the reading list, and one lens brief into each prompt: a sub-agent has no other access to any of it. Wait for all five reports.

4. **Aggregate.** Present five sections, one per goldfish, each report verbatim or lightly cleaned. Do not merge or rerank findings across lenses: five separate lenses is the design, and a merged list hides which lens caught what. The Comprehension section keeps its paraphrase even when it found nothing.

   Under the five sections, a short tally: findings that two or more goldfish raised independently (the swarm's strongest signal: the same gap seen through different lenses), and the worst finding within each lens. The verdict is the user's, not the swarm's.

5. **Resolve.** Ask the user which findings to accept, then apply the accepted edits to the spec directly. A rejected finding that a future reader will trip over goes into the spec's Further Notes as a deliberate trade-off, so the next review reads it as a choice rather than an oversight.

## Why goldfish

A spec that survives this has a property its author cannot check alone: everything it says is recoverable from the page plus the repo's standing docs. The implementer who eventually picks it up is fresh too, reads the same glossary and ADRs the goldfish read, and gets a document a fresh reader has already survived. By then the conversation that produced the spec is gone; the goldfish are the first readers who live in that world.
