# Writing an Intent Specification

This guide is for the **Extractor**: the agent that reads the original document and writes the intent specification. The spec you produce is the *only* bridge to a blind author who will recreate the document without ever seeing it. Everything the reconstruction must get right has to be in the spec; everything you leave out is lost.

The core discipline: **describe, don't quote.** Re-express every behavior, rule, and idea in your own words, at one level of abstraction above the original's prose. The blind author must re-derive all sentences themselves. Only explicitly licensed material (see "Intent-bearing tokens" and "Licensed verbatim blocks") may be carried over exactly.

Be aware that any phrase you quote in the spec becomes de facto licensed: the blind author will often reuse it word-for-word, and the Verifier will trace that overlap back to you ("extractor echo") rather than to leakage. So quote a phrase only when its exact wording is genuinely intent-bearing; otherwise paraphrase, even in passing illustrations.

## The preamble

Open the spec with a short preamble that:

- Addresses the blind author directly and states they will never see the original.
- States that all wording here is deliberately re-described and that the author must re-derive every sentence, roster, and example on their own.
- States that only the explicitly called-out intent-bearing tokens and preserved quirks must be reproduced exactly.

Title the spec by *describing* the document ("Intent Specification — Customer-Feedback Triage Prompt"), never by naming it. **Nowhere in the spec may the original's filename, path, directory, or repository appear.** The blind author cannot navigate to what it cannot name — keeping identifiers out of the spec is a load-bearing part of the blindness guarantee, not politeness.

## Required section: fidelity criteria for this document type

Every spec must contain a section titled **"Fidelity criteria for this document type"**. First characterize what kind of document this is, then state, as a checklist, what "faithful reconstruction" means for it. This section becomes the Verifier's rubric — write each criterion so a third party could score it pass/fail.

What intent *means* varies by type:

| Document type | Intent is primarily… | Surface wording is… |
|---|---|---|
| Prompt / system instruction | Behavioral rules, constraints, carve-outs, persona, output discipline, trigger phrases | Re-derived; only trigger phrases and format strings preserved |
| Code | Public interface, contracts, algorithms, error behavior, invariants, side effects | Re-derived; only identifiers that form the public API preserved |
| Procedural instructions | Step sequence, safety gates, preconditions, invariants (flags, orderings) | Re-derived; only exact commands/flags that are load-bearing preserved |
| Poetry / creative writing | Meaning, emotional effect, form, meter, rhyme scheme, imagery, tone, voice | Expected to diverge — score form and effect, not word overlap |
| Documentation / essay | Claims, arguments, structure, audience, level of detail | Re-derived; only proper nouns and technical terms preserved |
| Configuration | The semantics of each key: what it controls and why it has the value it has | Keys and values are usually intent-bearing; commentary re-derived |

For hybrid documents, state criteria per part. For types not listed, derive analogous criteria — the question is always "what would have to survive for a reasonable observer to say the rebuilt document *does the same job*?"

## Anatomy of the spec body

Cover the document in sections, each following these rules:

- **Attach rationale to constraints.** "Handle the word *so* conservatively — it often carries causal weight, so blanket removal would change meaning" reconstructs better than a bare rule, because the blind author can reason about edge cases instead of pattern-matching.
- **State rules atomically**, one per bullet, so each can be individually verified later.
- **Describe catalogs by function, and instruct the author to rebuild them.** For a table of commands, symbols, options, or entries: name the categories that must be covered and the distinctive entries that are intent-bearing, then tell the author to build the roster themselves. Do not copy row layouts or entry prose. A roster that differs in non-distinctive entries is a *success* of blind reconstruction, not a failure.
- **Give examples as coverage requirements, not transcriptions.** If the original contains a worked example, require the author to compose a brand-new one and list the features it must visibly demonstrate. If the example must state a fact, require a short, low-stakes, uncontroversial one.
- **Specify structure and ordering** at the level of section flow ("intro, then rules, then catalog, then example"), plus any self-consistency requirements (e.g. a document that mandates curly quotes must itself use curly quotes).

## Intent-bearing tokens

List, explicitly and exhaustively, the tokens that must survive **verbatim** because they *are* the interface or the intent:

- Exact glyphs (a bullet marker that must be an en dash `–`, not a hyphen).
- Format strings (`YYYY-MM-DD`).
- Trigger phrases, command names, flags, public API identifiers.
- Any literal string another system matches on.

For each, say *why* it is intent-bearing. Everything not on this list is presumed re-derivable, and overlap on it will be scored as suspected leakage.

## Preserved quirks

If the document contains oddities that are (or must be treated as) intentional — a dangling unmatched closing tag on the last line, an internal labeling inconsistency where the intro calls a section by one name and the example labels it another — call each out under a **"Preserved quirks"** heading and require exact reproduction. Distinguish these from defects you are choosing not to carry forward; if you judge something a plain error not worth preserving, say so explicitly so the Verifier doesn't score its absence as a miss. When in doubt, preserve: the skill's job is reconstruction, not improvement.

## Personal and identifying data

By default, do **not** carry real personal data (names of people, private addresses, personal email addresses, company-internal names) into the spec. Instead:

- Describe the *categories* and their purpose ("a personal-vocabulary list of names the speech engine mishears, grouped into people, places, and companies including legal-suffix variants").
- Instruct the author to invent clearly fictional placeholder entries and to add a note telling the reader to swap in real values.
- State in the spec that this de-identification is intentional, so the Verifier scores placeholder divergence as expected, not as a gap.

The user may override this default and ask for real data to be preserved; only then may it appear.

## Licensed verbatim blocks

Some documents' intent genuinely requires exact wording at scale — legal boilerplate, a quotation that must be exact, a checksum-matched string. Declare each such span in a **"Licensed verbatim blocks"** section and include the text. The Verifier will license that overlap and note that the blindness audit is correspondingly limited for those spans. Use this sparingly: every licensed block shrinks the meaningful part of the reconstruction.

## Sizing

Do not fear a spec longer than the original — abstract re-description plus rationale often is. Completeness beats brevity: the blind author has nothing else.
