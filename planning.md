# TakeMeter Planning — r/LetsTalkMusic Discourse Classifier

## Community

**Chosen community:** r/LetsTalkMusic

r/LetsTalkMusic is a music discussion subreddit that explicitly positions itself as a space for substantive engagement — not just "what are you listening to" posts, but actual discourse about albums, artists, genres, and the craft of music. This makes it a strong fit for a classification task: the community has implicit norms about what constitutes a good post, discourse quality varies meaningfully across threads, and there's enough text density per post to give a classifier real signal to work with.

The discourse is varied in a useful way: some posts engage seriously with production choices or historical context; others are purely expressive ("this album destroys me every time"); others are more like open-ended prompts designed to generate discussion rather than make a claim. These aren't just quality differences — they're structurally different kinds of posts, which is exactly what makes the engagement-type split viable here.

## Labels

The taxonomy is organized around what kind of thing the post is doing — not how well-reasoned it is, but its communicative function.

### critique

**Definition:** The post engages with how the music works — discussing production, composition, lyrics, influences, structure, or historical/cultural context — in a way that goes beyond stating a preference.

**Example 1:**

> "What makes In the Aeroplane Over the Sea hold up is how Mangum layers acoustic and lo-fi textures against genuinely dissonant horn arrangements. The production sounds accidental but it's very deliberate — the rawness is doing emotional work that a cleaner recording would undermine."

**Example 2:**

> "Kendrick's use of the unreliable narrator on good kid, m.A.A.d city is criminally underrated. The whole album reframes itself on a second listen once you realize he's not a neutral witness to his own story — the sequencing is doing a lot of that work."

**Uncertain case:**

> "Radiohead's guitar tones on The Bends are so underrated — everything after that got more electronic but those riffs are genuinely some of the best in rock."

This mentions a specific element (guitar tones) but doesn't explain why they work or what they're doing. It could be critique (engages with craft) or taste (preference stated without analysis). **Decision rule:** If the post names a specific musical element and makes a claim about its function or effect, label it `critique`. If it names an element only to say it likes it, label it `taste`.

### taste

**Definition:** The post expresses a preference, opinion, or emotional response to music without engaging with how or why the music works the way it does.

**Example 1:**

> "I've been on a massive Joni Mitchell kick lately. Blue is just one of those records that feels like it was made specifically for your worst moments."

**Example 2:**

> "Hot take but Yeezus is Kanye's best album. It's the only one where it feels like he genuinely didn't care what anyone thought."

**Uncertain case:**

> "The production on DAMN. feels so claustrophobic and I think that's what makes it his most intense project."

This names a production quality (claustrophobic) and connects it to an effect (intensity), which edges toward critique. **Decision rule:** If the observation about the music is attached to a preference or feeling ("makes it his most intense") without unpacking how that effect is achieved, label it `taste`. Critique requires engaging with mechanism, not just outcome.

### discussion_prompt

**Definition:** The post is primarily oriented toward inviting conversation rather than making or defending a claim — it poses a question, proposes a hypothetical, or opens a topic without taking a clear position.

**Example 1:**

> "What's an album you think is genuinely misunderstood — not underrated, but actively misread by most people who talk about it?"

**Example 2:**

> "Does anyone else feel like music criticism has gotten worse at talking about pop? I feel like there's still a residual bias against taking it seriously, but I'm not sure if that's changed in the past decade."

**Uncertain case:**

> "I've been thinking about whether artists owe anything to their early fanbase when they change sound. Feels like the answer is obviously no, but I want to hear what people think."

This posts a mild opinion ("obviously no") but immediately defers to the community, making it more prompt than assertion. **Decision rule:** If the post's primary function is to generate responses from others rather than to defend a position, label it `discussion_prompt` — even if it contains a soft opinion.

## Hard Edge Cases

The hardest boundary is **critique vs. taste**.

The risk is that critique becomes a catch-all for any post that sounds smart, and taste catches everything casual. The actual distinction is about mechanism: critique engages with *how* something works; taste engages with *that* it works (or doesn't).

The canonical hard case is a post that names specific elements — a production technique, an influence, a structural choice — but only in service of expressing a preference, not explaining the element's function. These posts sound like critique but are operating as taste.

**Handling rule:** Ask "could you remove the opinion framing and still have a claim about how the music works?" If yes → `critique`. If the named element is just decoration on a preference statement → `taste`.

The taste / discussion_prompt boundary is easier: discussion prompts are addressed to the community ("what do you think," "does anyone else feel like"), while taste posts are self-contained expressions that don't require a response to be complete.

## Data Collection Plan

**Source:** r/LetsTalkMusic — top posts and comments from the past year, collected via the Reddit API (PRAW) or manual scraping of post threads.

**Target distribution:** At least 200 examples, aiming for roughly equal distribution:

- `critique`: ~70 examples
- `taste`: ~80 examples
- `discussion_prompt`: ~60 examples

The slight taste lean reflects what I expect to be more common in practice. If any label falls below 20% of the total after collection, I'll specifically seek out more examples of that type before annotating — either by searching for posts with characteristic phrasing or by pulling from older threads.

**Collection strategy:**

- Pull top-level posts first (these tend to be longer and more clearly one type)
- Supplement with comments from high-engagement threads if posts alone don't reach 200
- Collect more than 200 raw examples before labeling — aim for 250–270 to allow for discards (posts too short to label, spam, non-English)

**Split:** 70% train / 15% validation / 15% test (roughly 140 / 30 / 30 examples)

## Evaluation Metrics

**Primary metric:** per-class F1 score (reported for all three labels)

Accuracy alone is insufficient here because the label distribution may not be perfectly balanced, and because the cost of errors differs by class. A model that learns to predict taste constantly could achieve reasonable accuracy given its frequency — F1 forces us to evaluate whether the model is actually distinguishing all three classes.

**Secondary metrics:**

- **Macro F1** (average across classes, unweighted) — tells us overall whether the model handles all three labels, not just the common one
- **Confusion matrix** — shows which pairs of labels the model conflates, which is the most interpretable output for understanding failure modes
- **Per-class precision and recall** — lets us see whether errors are false positives (over-predicting a label) or false negatives (missing examples of a label)

For the baseline comparison: same metrics applied to the Groq zero-shot classifier, so the comparison is apples-to-apples.

**Definition of success:** A macro F1 above 0.70 on the test set would indicate the classifier is doing something genuinely useful — meaningfully better than chance (0.33 for three classes) and consistent enough across classes to be usable. Below 0.55, the model isn't reliably distinguishing all three types. I'd also want the critique / taste confusion rate to be under 30% — that boundary is the hardest and most important one.

## AI Tool Plan

### Label Stress-Testing

Before annotating any examples, I'll give Claude my three label definitions and the critique vs. taste decision rule, and ask it to generate 10 posts that sit at the boundary between those two labels. The prompt will be something like:

> "Here are my label definitions for a music discourse classifier. Generate 10 short Reddit-style posts that would be genuinely difficult to classify as either critique or taste using these definitions. The posts should look like real music discussion."

If Claude produces posts I can't cleanly assign using my current decision rule, that's a signal to tighten the definition before I start annotating 200 examples. I'll document any definition changes that result from this exercise in this file.

I'll also generate 5 boundary cases for taste vs. discussion_prompt using the same approach.

### Annotation Assistance

I'll use Claude to pre-label a batch of ~60 examples (roughly 30% of the dataset) before reviewing them myself. Workflow:

1. Feed Claude my label definitions and 5 labeled examples per class as a few-shot prompt
2. Ask it to predict the label for each of the 60 examples and give a one-sentence rationale
3. Review each prediction myself — accept, reject, or relabel
4. Track which examples were pre-labeled in a `prelabeled` column in the CSV (value: yes / no)

This speeds up annotation without removing my judgment. I'll note in the README what percentage of the final dataset was pre-labeled and how often I disagreed with the pre-label (that disagreement rate is itself informative about label difficulty).

### Failure Analysis

After generating test set predictions from both the fine-tuned model and the Groq baseline, I'll compile all wrong predictions into a list with: the post text, the true label, and the predicted label. I'll then give this list to Claude with the prompt:

> "Here are examples a music discourse classifier got wrong. Each entry includes the post text, correct label, and predicted label. Identify any systematic patterns in the errors — not just individual mistakes, but recurring reasons a certain type of post gets mislabeled."

I'll look specifically for:

- Whether critique / taste confusions cluster around a particular type of post (e.g., short posts, posts that name specific albums)
- Whether the model systematically over- or under-predicts any one label
- Whether errors correlate with post length or the presence of musician names

I'll verify any pattern Claude identifies by manually checking whether the examples it cites actually share the feature it's pointing to — AI-identified patterns are hypotheses, not findings, until I've confirmed them myself.

## AI Usage Disclosure

- **Label stress-testing:** Claude (claude.ai) used to generate boundary-case examples before annotation
- **Annotation assistance:** Claude used to pre-label ~30% of examples; all pre-labels reviewed and accepted/rejected by me
- **Failure analysis:** Claude used to identify error patterns from wrong predictions list; patterns verified manually
- **No AI assistance used in:** final label assignment decisions, model training, metric calculation, or written evaluation
