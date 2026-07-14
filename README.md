## Register

Register is a Chrome extension that compares a draft with writing you choose.

Save articles, essays, or your own past work into a local library. Register keeps every text inside your browser. When you paste in a new draft, it measures both the draft and the library across 27 metrics.

These include sentence-length patterns, vocabulary range, concrete word use, passive voice, nominalizations, modal verbs, and Biber’s six register dimensions.

The result looks like this:

> This draft uses half as many short sentences as your usual writing and scores three standard deviations higher on abstraction.

That gives you a specific edit to make. Add shorter sentences. Replace abstract nouns with concrete actions. Run the draft again and see whether the score moves closer to your corpus.

Your corpus can contain writers whose style you admire or your own older work when you want the draft to sound like you.

## Who it is for

Register helps when a draft needs to match an existing voice.

Writers using LLMs can compare generated text with their own past work and catch flat sentence rhythm, repetitive vocabulary, and abstract phrasing.

Ghostwriters and agency copywriters can build a corpus from a client’s published work and measure how closely each draft matches it.

Instead of asking, “Does this sound right?”, Register shows exactly where the draft differs.

---

## Install

1. `chrome://extensions` → enable **Developer mode**
2. **Load unpacked** → select the `dist/` folder

Loading unpacked sidesteps Chrome Web Store review entirely, which matters: extensions that ship ONNX/WASM have been rejected for "including remotely hosted code in a Manifest V3 item." Nothing here needs the store.

## Rebuild

```
npm install
node build.mjs
```

---

## The design rule

**The corpus stores text, never metric vectors.** Metrics churn. If you save the vector and later invent a better measure, the corpus is dead. Analysis is a *view*, recomputed on demand and cached against an engine version. Bump `ENGINE` in `src/lib/db.js` and every reading is recomputed from source.

---

## What it measures

### Register (Biber 1988)

Six dimensions, computed from 67 lexico-grammatical features. Feature definitions, word lists, LOB/London-Lund means and SDs, and the dimension formulas are ported from Andrea Nini's Multidimensional Analysis Tagger, so scores are comparable to the published register space.

| | Negative pole | Positive pole |
|---|---|---|
| D1 | Informational production | Involved production |
| D2 | Non-narrative | Narrative concerns |
| D3 | Situation-dependent reference | Explicit reference |
| D4 | Neutral exposition | Overt persuasion |
| D5 | Non-abstract | Abstract information |
| D6 | Edited | On-line informational elaboration |

Each text is also snapped to its nearest of Biber's eight text types (Learned exposition, Imaginative narrative, Intimate interpersonal interaction, and so on).

### Rhythm
Sentence length mean, SD, skew, **lag-1 autocorrelation** (does a short sentence deliberately follow a long one, or does length wander at random?), longest consecutive run of sentences under eight words, percentage over 35, commas per sentence.

### Vocabulary
**MTLD** (bidirectional, TTR floor 0.72) and **MATTR** (window 50). Both chosen because they are the only diversity indices that hold steady across text length; plain TTR is worthless and is not computed. Plus mean Zipf frequency, percentage of content words below Zipf 3, percentage absent from the norms entirely, and average word length.

### Concreteness
Mean and SD of Brysbaert/Warriner/Kuperman concreteness over content words; percentage of content words below 2.5 on the 1–5 scale.

### Grammar
Nominalisation rate, agentless passive rate, *be* as main verb, modal rate.

### Taste
k-means over all 27 standardised metrics, with k chosen by silhouette. If the silhouette is under 0.25 the tool says so and refuses to split, rather than inventing groups. A PCA scatter plots the corpus with your draft projected into the same space.

---

## What it deliberately does not measure

**Cohesion.** Connective density, lexical overlap, LSA sentence-to-sentence similarity. Crossley and McNamara found that cohesion is *not* the feature that discriminates high-proficiency from low-proficiency text, and that expert raters weight coherence over cohesion, with high-knowledge readers apparently benefiting from *lower* cohesion. Tracking it as a quality target would optimise for the wrong thing. Word-level properties (frequency, concreteness, diversity) are what actually predicted expert essay grades, and those are what this tool tracks.

---

## Honest limits

- **POS tagging is Universal, not Penn.** wink-nlp gives UPOS plus lemma; tense, participles and contractions are reconstructed in `tagset.js` from morphology plus a ~140-verb irregular table. Around 45 of the 67 Biber features are exact; the clause-level ones (WHIZ deletions, pied-piping, that-relatives, subordinator-that deletion) are pattern approximations. Directionally right, not publication-grade.
- **No dependency parser.** Mean dependency distance would be the best single proxy for syntactic memory load, and there is no credible JS dependency parser. It is not implemented.
- **No surprisal tier yet.** Uniform Information Density (variance of per-token surprisal) is the sharpest available measure of where a reader stutters. It needs distilgpt2 through transformers.js in an offscreen document. The manifest already carries `wasm-unsafe-eval` for it. Not wired.
- **Below 300 words the reading is noisy.** The UI says so when it happens. Biber's centroids live in a five-dimensional space and short texts bounce around in it.
- **Norms cover 37,058 lemmas.** Words outside that list are reported as `offNorms` rather than silently dropped, because a high off-norms rate is itself a signal.

---

## Provenance

- Biber feature set, word lists, means/SDs, dimension formulas, text-type centroids: Nini's Multidimensional Analysis Tagger (GPL). Biber, D. (1988), *Variation across Speech and Writing*.
- Concreteness and SUBTLEX-US frequency counts: Brysbaert, Warriner & Kuperman (2014), 37,058 lemmas. Zipf computed as log10(count per billion) against SUBTLEX-US's 51M-word corpus.
- MTLD: McCarthy & Jarvis (2010). MATTR: Covington & McFall (2010).
- Article extraction: `@mozilla/readability`. NLP: `wink-nlp` + `wink-eng-lite-web-model` (MIT).

---

