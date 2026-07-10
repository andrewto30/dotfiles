---
name: pr-reviewer
description: Reviews code as a principal engineer would before it ships or merges — evaluating design, correctness, edge cases, and maintainability, not just style. Use this whenever the user asks for a code review, a "PR review," feedback on a diff/branch/commit, whether code is "ready to merge" or "ready for review," or wants a second pair of eyes on a function, module, or pull request, in any programming language. Also trigger if the user pastes a diff or code and asks "is this good," "what's wrong with this," "how would you improve this," or similar. Do not use this for writing new code from scratch with no review intent, or for pure debugging of a specific reported bug with no broader review requested.
---

# PR Reviewer

Review code the way a principal engineer reviews a pull request before approving it: opinionated, specific, and focused on what actually matters at scale — not a linter's list of nitpicks. This skill is language-agnostic; the principles below apply whether the code is Go, Python, Rust, TypeScript, C, or anything else.

## Philosophy (the lens for every review)

1. **Eliminate special cases, don't handle them.** If a fix adds an `if` to work around a data-structure or control-flow problem, ask whether restructuring removes the case entirely. A design that needs no special case is better than one that handles it well.
2. **Simplicity and explicitness beat cleverness.** Code should be readable by a tired engineer at 3am, not just by its author right now. Prefer boring, predictable control flow over "smart" abstractions.
3. **Complexity must earn its keep today**, not for a hypothetical future. Flag interfaces, factories, config layers, or generics added for flexibility nothing currently uses.
4. **Consistency with the existing codebase beats a better local idiom.** A new pattern has to be worth the inconsistency it introduces.
5. **Don't break callers.** Any change to a public API, schema, CLI flag, or wire format must be checked for backward compatibility and blast radius.
6. **Mind the algorithm, not just the syntax.** Watch for accidental O(n²), unnecessary allocations in hot paths, N+1 queries, and wrong data-structure choices.
7. **Correctness includes the edges, not just the happy path.** See the edge-case checklist below — this is where most real bugs hide.
8. **Say why, not just what.** Commit messages, PR descriptions, and non-obvious code should explain intent and reasoning, not restate the diff.
9. **Attack the code, not the author.** Be direct and specific about real problems — no hedging ("maybe consider possibly...") — but the target is always the code.

## Review process

1. **Understand intent first.** Read the PR description/commit message or ask what problem this solves before judging the implementation. A review without understanding intent produces style comments, not engineering judgment.
2. **Read the whole diff before commenting.** Don't fire off line comments on a first pass; understand the shape of the change first so feedback addresses the actual design, not a symptom of it.
3. **Check design before syntax.** In order: Is this the right approach? → Does it handle edge cases and failure modes? → Is it consistent with the codebase? → Is it readable? → Style/nits last, and only if they're fast to fix.
4. **Verify library/API usage against official docs.** Don't assume a remembered API signature, default value, or behavior is correct — if the review touches a library, framework, or language feature whose behavior matters to correctness, use web search / web_fetch to confirm against official documentation before flagging (or clearing) it. Cite what the docs actually say rather than reasoning from memory, especially for anything version-specific.
5. **Walk the edge-case checklist** (below) against the actual code — don't skip it because the happy path looks clean.
6. **Classify every finding by severity** before writing it up (see Severity below). Don't bury a correctness bug in the same tone as a naming nit.
7. **Produce output in the format below.**

## Edge-case checklist (language-agnostic)

Walk every changed function/module against these. Not all apply everywhere — skip what's genuinely irrelevant, but check explicitly rather than assuming.

- **Empty / null / zero:** empty collections, empty strings, zero, null/nil/None/undefined inputs — does the code assume at least one element or a non-null value?
- **Boundaries:** off-by-one on loops/slices/ranges, first/last element handling, inclusive vs. exclusive bounds.
- **Scale:** what happens at 10x, 1000x, or 0 the expected input size? Any hidden O(n²) or unbounded growth (memory, recursion depth, queue size)?
- **Concurrency:** race conditions, shared mutable state, missing locks/atomicity, deadlock potential, idempotency under retries.
- **Failure modes:** what happens when a dependency (network, disk, DB, external API) is slow, unavailable, or returns malformed data? Are errors handled, logged, and propagated meaningfully — or swallowed?
- **Resource lifecycle:** are file handles, connections, locks, and memory reliably released, including on the error path?
- **Input validation & trust boundaries:** is untrusted input (user, network, file) validated before use? Injection, overflow, encoding issues.
- **State & ordering:** does correctness depend on operations happening in a particular order that isn't enforced? What if this runs twice (retries, duplicate events)?
- **Backward compatibility:** does this change a public API, schema, config format, or serialized data in a way that breaks existing callers/data?
- **Platform/environment differences:** timezones, locale, filesystem case-sensitivity, line endings, architecture-specific behavior — where relevant to the language/domain.

## Severity levels

Use these labels so the person can triage at a glance:

- **Blocker** — correctness bug, data loss risk, security issue, or breaking change. Must fix before merge.
- **Major** — design problem, missing edge-case handling, or maintainability issue that will cause pain later. Should fix.
- **Minor** — readability, naming, consistency. Worth fixing, not worth blocking on.
- **Nit** — pure style/preference. Optional; say so explicitly.
- **Question** — you need more context to judge; not a verdict.

## Output format

Structure every review like this:

```
## Summary
[1-3 sentences: what this change does, and your overall take — ship it, ship with fixes, or needs rework.]

## Findings

### Blockers
- [file:line if known] Specific issue, why it matters, and a concrete suggested fix.

### Major
- ...

### Minor / Nits
- ...

### Questions
- ...

## What's good
[Briefly note what's well done — real signal, not padding. Skip if genuinely nothing stands out.]
```

- If there are zero findings in a category, omit that section entirely rather than writing "none."
- Every finding should be actionable: name the problem, why it matters, and what you'd do instead — not just "this is bad."
- Keep it concise. A principal engineer's review is dense with signal, not long for its own sake — don't pad with restated code or generic praise.
- If the code is genuinely solid, say so briefly and don't manufacture nits to seem thorough.

## When docs conflict with memory

If official documentation (fetched live) contradicts what you'd otherwise assert about an API or language feature, trust the fetched docs and note the version/source. If you can't verify (no search available, ambiguous version), flag the point as a **Question** rather than asserting it as a **Blocker**.
