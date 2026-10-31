---
name: writing-for-humans
description: Write prose that reads human-written. Use when drafting or editing text for human readers (docs, PRs, tickets, emails, posts, comments) to strip AI tells without losing facts or voice.
---

## Scope

Governs text a human reads. Agent-facing documents (skills, `AGENTS.md`) follow `writing-for-agents` instead. A compression mode (e.g. `caveman`) governs the agent's own chat replies; this skill governs the text artifacts it produces. Code, commands, paths, error strings, YAML, data, and link targets stay verbatim.

## Why tells exist

A model picks the choice that fits the widest range of readers; a person chooses for one reader and one subject, so human choices are uneven and specific. Every tell below is a default choice. A tell counts in proportion to how rarely a careful writer would make it on purpose: the strongest justify an edit on one sighting, a *weak* tell needs company from other tells in the same passage.

## Method

1. **Set the voice.** A writing sample from the user overrides every rule below. Match its sentence length, punctuation, word choice, and quirks, including its dash rate. No sample → take voice from the text type: personal writing keeps opinion, humor, and asides; reference and factual text stays neutral and plain.
2. **Write or rewrite.** Every sentence must give the reader something they did not already have: a fact, a decision, an instruction. Drafting → apply the tells as you write. Editing → mark each tell, then rewrite the sentence around its point rather than patching the flagged phrase.
3. **Audit.** Re-read asking "what here reads machine-written?" Staging, dashes, triads, and bold labels most often survive a first pass. Then verify the draft added or dropped no fact, name, number, date, quote, citation, or claim: never invent one, and a lost supported claim is an error unless a tell calls for the cut. Missing detail → ask or write the plainer sentence.
4. **Keep the person.** Cutting tells is half the job; voiceless clean prose is its own tell. Keep or restore the writer's opinions, uncertainty, odd specific details, era-bound references, and genuine asides where the voice allows. Vary sentence length; real writing alternates short and long.

## Tells

### Staging: signaling importance instead of stating

- **Not X but Y.** "not just X but Y", "it's not X, it's Y", split across sentences ("This does not mean X. It means Y."), clipped tails ("no guessing", "no wasted motion"). State the point. Keep a contrast only when the negative half corrects a belief the reader holds.
- **Dramatic closers and fragments.** One-line paragraphs that restate the point ("Let that sink in.", "That distinction matters.", "Read that again."), sentences naming what an example showed ("It was a lesson in patience."), fragment runs ("No prior. No nostalgia."), ALL-CAPS or every.single.word emphasis. Keep a short line only when it adds a fact; merge fragments into a sentence with a claim.
- **Deep-sounding sayings.** "at its core", "the real question is", "what really matters", "fundamentally", "X is the language/currency of Y", "X becomes a trap", "X is not a tool but a mirror". Replace the aphorism with the specific claim.
- **Run-ups.** "Let's dive in", "Here's what you need to know", "Here's the thing", "Honestly?", "Real talk", "Quick note", "Without further ado", "In this article we'll explore". Delete the announcement; make the point.
- **Arguing with no one.** "I'm not saying X, but", "Don't get me wrong", "To be clear", "Some might say… but". Delete the rebuttal; state the position.

### Inflation: dressing facts as more than they are

- **Stock AI words.** additionally, bolster, crucial, delve, elevate, empower, enduring, enhance, garner, intricate, interplay, landscape (abstract), meticulous, pivotal, showcase, tapestry (abstract), testament, underscore, vibrant; marketing set: unlock, seamless, cutting-edge, game-changer, next-level, revolutionary, journey. Use the plain word.
- **Inflated significance.** "marking a pivotal moment", "setting the stage for", "a testament to", "plays a key role", "reflects a broader", "indelible mark"; stock "challenges and outlook" sections; send-offs like "the future looks bright" or "exciting times ahead". Keep the fact, drop the significance; end on the last concrete fact or a real stated plan.
- **Sales language.** nestled, in the heart of, breathtaking, stunning, renowned, rich (figurative), must-visit, profound, groundbreaking (figurative), diverse array, commitment to. State what the thing is.
- **Empty proof.** "trusted by thousands", "industry-leading", "world-class", prestige outlet lists, "active social media presence with N followers". Keep only named, verifiable sources; cut the rest.
- **Unnamed authority.** "experts say", "industry reports suggest", "some critics argue", "observers note". Name the source or cut the claim.
- **-ing riders.** "highlighting", "underscoring", "ensuring", "reflecting", "symbolizing", "fostering", "showcasing" bolted onto a fact. Keep the fact; keep the rider only if the source supports it.
- **Vague links.** "associated with", "connected to", "linked to", "tied to". Name the actual relationship; if the source does not say, keep the vague wording rather than inventing one.
- **Fancy copulas.** "serves as", "stands as", "functions as", "boasts", "features", "represents". Write is/are/has.
- **False agency.** Abstractions with mind-verbs: "the data tells us", "the dashboard understands", "the market rewards". Ordinary product verbs are fine ("the filter narrows the list"); the tell is a verb needing a mind: understands, decides, wants, believes. Name the actor or state what it does.
- **Invented specifics.** Realistic but unsourced numbers, quotes, testimonials, dates. Flag or cut; this stays a defect even when it sounds human.

### Rhythm by rule

- **Forced triads.** Ideas padded into groups of three. List the items the content actually has.
- **Synonym cycling.** protagonist / main character / central figure in one paragraph. Pick the clearest word and repeat it.
- **False ranges.** "from X to Y, and everything in between" where X and Y are not a scale. List the concrete topics.
- **Repeated openings.** The same subject starting every sentence. Merge, change the subject, or lead with the action; deliberate anaphora can stay.
- **Em dashes.** None in output. Replace each with a period, comma, colon, or parentheses; this covers ` — ` and ` -- ` used as dashes. Skip inside code, commands, paths, URLs. One dash in a source text is *weak*; a text full of them is not. A user's sample sets the rate.
- **Connector colons.** Fine before a list or example, not mid-sentence to prop up a clause. Let the point stand alone.
- **Stacked qualifiers.** "could potentially possibly", "it's also possible that… might". Keep one qualifier. Scope statements, legal and safety notices, and real corrections stay; ordinary hedges like "perhaps" and "tends to" are human habits.
- **Uniform sentences.** Dense sentences the reader must re-parse → split, one idea per sentence. Over-compressed fragments dropping articles and verbs → write the whole sentence. Vary the lengths.

### Plain speech

- **Filler and fancy words.** "in order to" → "to"; "due to the fact that" → "because"; "in the event that" → "if"; "at this point in time" → "now"; utilize/leverage → use; facilitate → help; numerous → many; "it is important to note that" → delete.
- **Metaphor jargon.** substrate, wedge, vector, locus, nexus, paradigm, modality, north star, flywheel, scaffolding, bedrock, gold-plating, ratchet, endgame, harness/surface as metaphors. Use the concrete word.
- **Feelings over mechanisms.** "the database stays close at hand", "SQL you can read" name a feeling. Name the mechanism or the number instead. If the sentence could run unchanged in another project's docs, it says nothing. Cut it.
- **Mannered prose.** Rhetorical fragments, personified code ("the plan holds it", "rides along", "stands on"), stock framings. Say it literally.
- **Adverbs propping weak verbs.** "runs quickly" → "is fast" or the number; "significantly improves" → the measured delta.
- **Passive default.** "queries are validated" → "the compiler validates queries". Passive stays only when the actor is unknown or irrelevant ("the server was restarted at 03:00"); a page of actorless passives is the tell.

### Formatting by rule

- **Bold decoration.** Bolded key terms everywhere; list items that are a bold label plus colon restating the line ("- **Performance:** Performance improved…"). Remove the bold; turn label-restating lists into prose. A bold lead naming the item followed by new detail is fine.
- **Decorated headings.** Title Case → sentence case; remove emojis and arrows in headings and bullets, horizontal rules between every section, an H1 repeating the document title. A heading names what the section holds.
- **Heading echo.** A first sentence under a heading that restates it. Delete.
- **Quote marks.** Curly quotes → straight where the writer or format uses them (*weak*). Clusters of scare quotes around ordinary terms → write the emphasis into the sentence; keep quotes for dialogue, titles of works, real quotations.

### Leftovers and meta

- **Chat residue.** "Great question!", "I hope this helps!", "Let me know if", "Would you like me to", "You're absolutely right", "Of course!", "Here is a…". Remove the wrapper, keep the content. Letter salutations and sign-offs predate chatbots; they stay.
- **Knowledge-limit disclaimers.** "as of [date]", "while specific details are limited", "based on available information", followed by a guess ("likely grew up"). State the gap in one sentence or cut it; never fill it with a plausible invention.
- **Self-description.** Text about how the text was made ("was added to replace", "generated from", "the table below compares", "this section is organized by"). Keep only in changelogs, release notes, and migration guides, or when the reader cannot infer the convention.
- **Re-explaining the reader's context.** In replies: restating the problem and diagnosis so the decision lands last. Lead with the decision; keep only the reasoning that could change agreement.

## When to leave it

- Quoted text, titles, proper names, and passages that discuss a phrase rather than use it.
- Text written before ~November 2022 is not AI-written.
- A lone *weak* tell can be a human habit; act on clusters.
- The user's deliberate style (dashes in a sample, a shouted line in a voice that shouts): match it; surface the conflict rather than sanitizing silently.
- The quirks that carry voice: odd specific details, mixed feelings, era-bound slang, first-person choices, genuine asides and self-corrections.

## Output

- **Pasted text (default):** return the rewritten text; add a short tell list only when a fix needs input the source lacks.
- **Detect (user asks to check without rewriting):** list each tell found: name, the line, the fix in a few words. Name patterns, never a verdict on whether AI wrote the text.
- **File mode:** write only the final text to the file; preserve code blocks, inline code, commands, paths, frontmatter, data, and link targets. Then a one-line summary.
- **Embedded (another task invokes this mid-flight):** return only the final text.
