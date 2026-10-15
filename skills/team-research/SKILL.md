---
name: team-research
description: Exhaustive, adversarially-tested, fully-cited research into any topic using a team of subagents (scouts, relentless researchers, an arbiter, an anti-slop editor). Produces a verbose markdown reference document named after the topic slug (SLUG.md) for use as future context. Use when the user says "team-research", "research X exhaustively", "deep research on X", "build a research doc on X", or wants rigorous, source-cited, cross-examined research rather than a quick answer. Not for quick lookups.
---

# team-research

Four phases. Output: `SLUG.md` in the current directory, where SLUG is the kebab-case slug of the topic (`"Raft consensus in practice"` -> `raft-consensus-in-practice.md`).

**Iron rules (apply to every agent, state them in every prompt you write):**
- No statement without a citation. Citation = URL (or DOI / book + page) + a short **verbatim quote** from the source that supports it + access date.
- Fetch the source (WebFetch / read it). A search snippet is not a source.
- Prefer primary sources. Record source type: `primary | peer-reviewed | official-docs | reputable-secondary | practitioner | community | unknown`.
- Record negative results ("searched X, found nothing") and contradictions. Never smooth them over.
- Do not infer beyond what the quote says. Label inference as `INFERENCE` and keep it out of the claim ledger.

## Setup

1. Derive SLUG. Create scratch dir `.team-research/SLUG/` (all agent output goes to files here, so your context stays small; agents return only a short summary + file path). Add `.team-research/` to `.gitignore` if the repo has one.
2. If the topic is ambiguous, ask the user one clarifying question (scope, time horizon, audience). Otherwise proceed.

## Phase 1: Expedition (breadth)

Spawn 4-6 `general-purpose` scouts **in parallel in one message**, each with a different lens, e.g.: overview and terminology; history and origins; current state, key players and numbers; academic/primary literature; practitioner and community experience; controversies, failures and open questions.

Each scout writes `.team-research/SLUG/scout-<lens>.md`: a sketch map of the landscape as a list of candidate **areas** (name, 1-2 sentence description, why it matters, 2+ cited sources, difficulty/uncertainty). Same citation rule applies. Shallow is fine; wide is the point.

You then merge and dedupe into one numbered area list and **present it to the user** (name, one line, and what scouts disagreed on). Ask which areas to pursue (multi-select via AskUserQuestion, or a numbered list if more than 4; allow "all", additions, and exclusions). **Stop and wait.** Do not start Phase 2 without the selection.

## Phase 2: Heavy research (depth, overlap, no assumptions)

Spawn 5-7 `general-purpose` researchers **in parallel**. Every researcher investigates **every** selected area. Overlap is deliberate: independent corroboration is evidence, and divergence is a signal. Do not tell researchers what others are doing or finding.

Each gets a distinct methodology and must use as many distinct sources as possible (target 15+ per area; keep going until new searches stop surfacing new sources):

| Role | Method |
|---|---|
| Primary-source hunter | Original documents, specs, standards, datasets, source code, filings, first-hand accounts. Trace every secondary claim back to its origin. |
| Literature reviewer | Peer-reviewed papers, surveys, meta-analyses, citation chains (forward and backward). |
| Practitioner / empirical | Postmortems, benchmarks, case studies, issue trackers, forums, talks; what actually happens vs. what docs say. |
| Historian / chronologist | Timeline, version/era changes, what was once true and no longer is. Dates every claim. |
| Quantitative analyst | Numbers, measurements, sizes, rates; recompute or sanity-check figures; note methodology and sample limits. |
| **Adversarial (mandatory, may use 2)** | Assume the prevailing view is wrong. Hunt for counter-evidence, retractions, failed replications, conflicts of interest, outdated claims, marketing disguised as fact, circular citation. Steelman the strongest opposing position per area. |

Each researcher writes `.team-research/SLUG/researcher-<role>.md` as a **claim ledger**. One entry per claim:

```
### C<role>-<n>  [area: <area>]
Claim: <one atomic, falsifiable statement>
Source: <URL / DOI> | type: <source type> | author/org: <..> | date: <..> | accessed: <YYYY-MM-DD>
Quote: "<verbatim supporting text>"
Confidence: high|medium|low  (why, one line)
Contradicted-by / corroborated-by: <other sources found, if any>
```

Plus a closing section: gaps, dead ends, and unresolved contradictions. Return only the path and a 5-line summary.

## Phase 3: Arbitration

Spawn one `general-purpose` **arbiter** (fresh context, no stake in any claim). It reads all ledgers and decides inclusion **on merit alone**: the claim <-> quote <-> source triple. Rubric, per claim:

- **Support**: does the quote actually entail the claim (not merely relate to it)?
- **Source weight**: primary/peer-reviewed/official > reputable secondary > practitioner > community > unknown. Check authority, independence, conflicts of interest, recency.
- **Corroboration**: independent confirmation across researchers/sources raises weight; shared upstream origin (circular citation) does not count as independent.
- **Contradiction**: unresolved conflicts are not deleted; they are carried as `DISPUTED` with both sides.
- **Verification**: the arbiter re-fetches a sample of citations (all of the high-impact and all of the borderline ones) and rejects any whose quote does not appear in the source.
- **Value**: does it matter for the topic and the selected areas?

Verdicts: `ACCEPT | ACCEPT-DISPUTED | REJECT | BORDERLINE`. Merge duplicates, keeping the best citation and listing the rest as corroboration.

**Right of reply.** For every `BORDERLINE` claim, the arbiter writes a short challenge (what is weak: thin source, quote mismatch, uncorroborated, stale) to `.team-research/SLUG/challenges.md`. You route each challenge back to the **contributing researcher** (continue the same agent with SendMessage if available, otherwise spawn a fresh one given its ledger and the challenge). The researcher does more research and replies with strengthened evidence in the same claim format, or concedes. Max 2 rounds per claim. The arbiter then rules finally: `ACCEPT`, `ACCEPT-DISPUTED`, or `REJECT`. Rejection happens only after the reply window. The arbiter may also ask any researcher to revisit an area that is thinly covered.

Arbiter writes `.team-research/SLUG/arbiter-draft.md`: the full research document (below) built only from accepted claims, plus a `rulings.md` table (claim id, verdict, one-line reason).

### Document structure (verbose and detailed: this is context for future work, not a summary)

1. Title, date, scope, selected areas, method (roles used, counts of sources/claims accepted/rejected).
2. Executive summary (only claims that appear, cited, later in the doc).
3. One section per area: detailed findings, each statement carrying an inline citation `[n]`; include numbers, dates, names, versions, definitions, and direct short quotes where useful.
4. Contested and disputed points: each side, its evidence, the arbiter's assessment.
5. Adversarial findings: strongest case against the prevailing view, and how it held up.
6. Gaps and unknowns: what could not be established and what was tried.
7. Rejected-claims digest (brief, with reason) so future readers do not re-litigate.
8. Sources: numbered list, with type, date, accessed date.

## Phase 4: Anti-slop pass

Spawn one editor agent (or, if the `stop-slop` skill is installed, have it invoke that skill) over `arbiter-draft.md`. Remove AI tells: filler openers, hedging stacks, "it's important to note", rule-of-three padding, empty intensifiers, uniform sentence rhythm, vague attributions ("experts say"), summary sentences that restate the paragraph, bullet-ification of prose that should be prose, and em-dash overuse. Keep it dense and concrete.

The editor **may not** change any fact, number, name, date, quote, claim scope, or citation marker. After the pass, verify mechanically: the set of `[n]` markers and the Sources list are identical to the draft (e.g. `grep -o '\[[0-9]*\]' | sort | uniq -c` before and after), and quoted strings are unchanged. If not, reject the edit and redo.

Write the result to `./SLUG.md`. Tell the user the path, the counts (sources, claims accepted / disputed / rejected), the biggest gaps, and that `.team-research/SLUG/` holds the raw ledgers and rulings for audit.
