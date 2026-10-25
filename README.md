# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

TakeMeter classifies posts from r/f1visa, a public subreddit where
international students ask about F-1 status, OPT, travel and school.
Almost every post is about those same topics, so the model does not sort
by subject. It sorts by what the poster is asking the community for:
`rule_question`, `experience_request`, `risk_check`, or `miscellaneous`
when the post is not asking a rule, experience or risk question about
F-1 status.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

Every label describes **what the post is asking the community for**, not its
topic. Almost every post in r/f1visa is about OPT, travel or status, so topic
can't tell posts apart. What they want back can.

> **How the fourth label came about:** I first tried a `share_vent` label for
> posts that give news, experience reports or feelings instead of asking for
> help. It was a real group but only about 22 of 575 posts (~4%), too few to
> train or measure on its own. I folded it into a broader `miscellaneous`
> label together with off-topic and speculative posts, which gives the model a
> "none of the above" option instead of forcing every post into a question
> label.

### `rule_question`

**Definition:** Asks what the rules or procedure are, in a way an official
source (USCIS, the regulations, a DSO) could answer completely.

**Example 1:**
> got rfe but recently changed address — hi everyone i just received an RFE for
> my post completion OPT but i recently moved and just changed my address
> yesterday. i highly doubt that the mail is gonna come into my new address. i
> was wondering if RFE notice will be uploaded to my portal in the document
> section?

**Example 2:**
> Paying PP with a foreign bank? — Can I use a foreign bank card (debit) to pay
> for Premium Processing for OPT or should it be a US debit/credit card?

### `experience_request`

**Definition:** Asks other people for their outcomes, timelines or data
points, without asking whether the poster's own plan is wise.

**Example 1:**
> OPT/STEM OPT RFE – how long after responding did you get approved? — If
> you've had an RFE on OPT or STEM OPT, could you please share: when USCIS
> received your response, when you got the approval, regular or premium
> processing. Any data points would really help.

**Example 2:**
> How long for F-1 OPT SSN card to arrive after in-person SSA visit? — For
> those on F-1 OPT who recently applied for an SSN in person at the SSA office
> in Ohio, how long did it take for the physical card to arrive in the mail?

### `risk_check`

**Definition:** Asks for a judgment on whether the poster's own situation or
plan is safe, risky, or likely to work out.

**Example 1:**
> Can I Visit Canada? — I'm currently on F1 OPT, working for a company in
> Boston. I'm thinking of taking a road trip to Canada to visit my bestfriend.
> I have a valid i20 with travel signature, a job to come back to, EAD card in
> hand. Is this safe to do? Any potential risks?

**Example 2:**
> Getting married while STEM OPT application pending — My initial
> post-completion OPT has ended, and I'm about two months into the 180-day
> extension. Would it be too risky to get married (to a US citizen) when my
> status isn't solid?

### `miscellaneous`

**Definition:** Does not ask the community a rule, experience or risk question
about F-1 status: it shares news, an experience report or feelings,
speculates, or asks about something outside F-1 rules. A non-immigration
event stays `miscellaneous` until they ask what to do on an F-1 form or
filing; then it is `rule_question`. Feelings or venting around the question
do not make it `miscellaneous` if they still ask whether a plan is safe,
reckless, or likely to work out.

**Example 1:**
> Recent STEM OPT POE - Seattle 09/02 — Here is my recent experience
> re-entering the US on F-1 STEM OPT. Hope this helps anyone traveling soon!
> Port of Entry: Seattle. Documents brought: travel I-20, employment letter,
> recent pay slips…

**Example 2:**
> OPT ending and very sad — I graduated in December 2025 after five years in
> the U.S. … Now it feels like all that hard work is now going to be placed in
> packing boxes.

### The hardest boundary

**Which two labels:** `experience_request` vs `risk_check`

**The decision rule I used every time:**
Label the post by the kind of answer that would settle it, not by how much of
their own story they tell. A long "my situation" section does not make it
`risk_check` on its own.

If the poster asks readers to judge their own plan — should I go ahead, is
this safe, will this work out for me, is this a good answer — it is
`risk_check`. That still holds when the sentence is "has anyone done this?"

It is `experience_request` only when other people's outcomes, timelines or
data points would settle the post on their own, and the poster does not put
their decision to the community.

Asking which next step to take on an already-filed case — upload this,
withdraw, wait, apply for a replacement — is `risk_check`, even when the
options are official procedures. It is `rule_question` only when they ask
what the procedure *is*, not which one to pick for themselves.

*Borderline post:* "Renewing F1 Visa Under STEM OPT — I am extremely nervous
about this decision … I am considering canceling the appointment and remain
here if it is safer this way. Has anybody been in these shoes lately?" →
`risk_check`. They ask for experiences, but what would settle it is whether
to go.

*The other side of the same line:* "Travel during pending Initial OPT — The
international office said that I can travel. Is there anyone who recently
traveled in this kind of situation?" → `experience_request`. Own
circumstances and "has anyone," but they do not ask whether the plan is wise.


---

## The Dataset

**Where the posts came from:**
Public posts from [r/f1visa](https://www.reddit.com/r/f1visa). Title plus full
body, not trimmed. One file, `labels.csv`, not pre-split.

**How I labelled them:**
I labelled 20 posts cold first — definitions only, no tools — and marked those
rows `cold` in the note column. An AI tool pre-labelled the rest from the same
definitions. I then read and corrected every pre-labelled row and marked them
`pre-labeled, checked`. Nothing in the file is still `needs review`.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `rule_question` | 214 | 37.3% |
| `risk_check` | 172 | 30.0% |
| `experience_request` | 126 | 22.0% |
| `miscellaneous` | 61 | 10.6% |
| **Total** | **573** | **100%** |

No label is above 70%. The largest is `rule_question` at 37.3%.

**Three hard cases**

**1.**
> *The post:* Employer transfer to Canada vs continuing in the US on F-1/CPT?
> STEM OPT ending; employer may transfer them to Canada, or they could enroll
> in a Day-1 CPT program. "I'm not looking for a debate on whether Day 1 CPT
> is good or bad. I'm mainly interested in hearing from anyone who faced a
> similar decision."
>
> *Could have been:* `risk_check` (own plan: Canada vs Day-1 CPT) or
> `experience_request` (asks only what others who chose each route later felt).
>
> *I chose, because:* `experience_request`. They put the decision in the post
> but did not ask readers to judge it. Other people's outcomes would settle
> what they asked.

**2.**
> *The post:* Travelling to hometown before starting job on F1-OPT. Job
> postponed so they can visit home; EAD, visa, passport, travel I-20 in hand.
> "Will that along with my offer letter be sufficient to go and come back?
> Has anyone done this and had any issues?"
>
> *Could have been:* `rule_question` (are these documents enough to re-enter)
> or `risk_check` (will *my* trip before the job starts go wrong).
>
> *I chose, because:* `risk_check`. They already have a plan and are asking
> whether it will cause issues, not only what the document list is.

**3.**
> *The post:* Regular OPT/Biom question. Filed April 21, biometrics June 8,
> still pending. "Does the 5 months processing time start when you complete
> biometrics or when you file? I have a job offer should I pay PP?"
>
> *Could have been:* `rule_question` (when the processing clock starts) or
> `risk_check` (should I pay premium on my case).
>
> *I chose, because:* `rule_question`. The first and main ask is a procedure
> fact a DSO could answer. "Should I pay PP?" is a second question, not the
> one that would settle the post.

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate `2e-5`, batch size 16, max length 128,
seed 42.

**Anything I changed from the defaults, and why:**
Nothing. The defaults are the run I am reporting.

**Device:** `mps` (Apple Silicon GPU), torch 2.14.1. This is part of the
result. The same data and seed on different hardware can move the numbers
slightly.

**Split sizes:** stratified 70/15/15.

| Label | train | val | test |
|---|---|---|---|
| `rule_question` | 150 | 32 | 32 |
| `risk_check` | 120 | 26 | 26 |
| `experience_request` | 88 | 19 | 19 |
| `miscellaneous` | 43 | 9 | 9 |
| **Total** | **401** | **86** | **86** |

`miscellaneous` has 9 examples in the test split, just above the notebook's
"fewer than 8" note. Its F1 will move a lot between seeds next unit.

### Seed 42 results (this unit's run)

Held-out accuracy was **0.419**. Macro F1 was **0.230**. Always predicting
`rule_question` on this split would have scored 32/86 ≈ 0.37, so the trained
model was only slightly above a majority guesser.

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `rule_question` | 0.42 | 0.88 | 0.57 | 32 |
| `experience_request` | 0.00 | 0.00 | 0.00 | 19 |
| `risk_check` | 0.40 | 0.31 | 0.35 | 26 |
| `miscellaneous` | 0.00 | 0.00 | 0.00 | 9 |

| true \ predicted | `rule_question` | `experience_request` | `risk_check` | `miscellaneous` |
|---|---|---|---|---|
| **`rule_question`** | 28 | 0 | 4 | 0 |
| **`experience_request`** | 14 | 0 | 5 | 0 |
| **`risk_check`** | 18 | 0 | 8 | 0 |
| **`miscellaneous`** | 6 | 0 | 3 | 0 |

The model predicted `rule_question` 66 times out of 86 (77%) and never
predicted `experience_request` or `miscellaneous`.

Confidence: most-sure third accuracy 0.500, least-sure third 0.321, gap
**0.179**.

### Against the five criteria (seed 42 only)

Criteria 1, 2, 3, and 5 have to hold on seeds 42, 7, and 2024. Seeds 7 and
2024 have not been run. This table is this seed only. Missing a target here
does not change the filed criteria.

| # | Criterion | Target | Seed 42 | Verdict |
|---|---|---|---|---|
| 1 | Held-out accuracy | ≥ 0.60 | 0.419 | Miss |
| 2 | Per-label F1 | RQ/ER/RC ≥ 0.50; misc ≥ 0.40 | 0.57 / 0.00 / 0.35 / 0.00 | Miss |
| 3 | Share of any predicted label | ≤ 0.50 | `rule_question` 66/86 = 0.77 | Miss |
| 4 | Second-reader agreement on the cold 20 | ≥ 15/20 | not measured this run | Not tested |
| 5 | Sure third − unsure third | ≥ 0.12 | 0.179 | Met |

### Why the misses

The taxonomy classifies **what the poster is asking for**, not the topic.
Almost every r/f1visa post uses the same words (OPT, I-20, USCIS, DSO).
DistilBERT, three epochs, 128 tokens, and 401 training rows is a weak setup
for that distinction. The model learned something closer to "this is an F-1
question" and mapped it onto `rule_question`, which is already 37% of the
file and 150 of 401 training rows.

That collapse is the same failure under criteria 1, 2, and 3. Accuracy sits
near the majority baseline. `experience_request` and `miscellaneous` have
F1 0.00 because they were never predicted. `risk_check` recovered only 8 of
26 true cases. Predicted `rule_question` is 77% of the test guesses, which
is the collapse criterion 3 was written to catch.

The other piece is the hardest boundary. `experience_request` and
`risk_check` share "has anyone" / "my situation" language. The model did
not learn the split: 14 of 19 true experience posts and 18 of 26 true risk
posts were called `rule_question`. `miscellaneous` has only 43 training
examples and 9 in test, and those posts still mention F-1 life, so they
look like questions.

Criterion 5 passed because the model is more confident when it repeats
`rule_question` on obvious rule posts (recall 0.875 on that label). That
gap ranks the majority habit. It does not mean the four-way distinction was
learned.

This is a finding about the task and the data, not a failed training
script. `results.json` and `test_split.csv` are the evidence. The next unit
is where three seeds, the baseline, and one improvement get tested. The
criteria stay as filed.

---

## How I Used AI

**Moment 1**

- *What I asked for:* A read of a large batch of r/f1visa posts: a summary
  of what people talk about most, whether they cite official sources, and
  whether the posts cluster as confusion, assertions, procedure questions,
  or something too mixed to classify.
- *What came back:* Topic did not separate the posts — almost everything
  was OPT, travel, or status. The useful split was the *kind of ask*: a
  rule an official source could answer, other people's outcomes, a judgment
  on the poster's own plan, or no real question.
- *What I changed:* I dropped topic-based labels and built the taxonomy
  around what the poster wants back. That is the four labels in this repo.
  It saved a full manual pass just to see the pattern, which I then checked
  against posts I actually read.

**Moment 2**

- *What I asked for:* After I labelled 20 posts cold, I asked the model to
  pre-label the rest from my definitions, and to leave genuinely borderline
  posts unlabelled so I would have to decide them myself.
- *What came back:* A labelled majority plus a held-out "dicey" set that
  sat on `experience_request` vs `risk_check`, or mixed a rule question
  with "should I."
- *What I changed:* I used those unlabelled clashes to write the decision
  rule (judge the post by the answer that would settle it; a next step on
  an already-filed case is `risk_check`). I then read and corrected every
  pre-labelled row. The dicey posts I labelled by hand are marked `hard
  case` in `labels.csv`.

**Moment 3 (unit 6)**

- *What I asked for:* A clear explanation of the parts of
  `data/staff_taxonomy.md` that confused me, and a copy of the 30 posts
  from `data/staff_posts.csv` set up as `my_staff_labels.csv` with the
  `text` column filled in and the `label` column empty.
- *What came back:* A clearer picture of the three staff labels and what
  kind of statement belongs to each, and a ready file with every post
  pasted exactly as written, so `agreement.py` could match them on text.
- *What I changed:* Nothing in the labels came from the AI. I labelled all
  30 posts by hand under the staff rules, and saved the file before I
  opened `data/staff_labels.csv`. The file saved me from building it and
  copying 30 posts by hand, which is where a retyped or trimmed post would
  have failed to match.

**Moment 4 (unit 6)**

- *What I asked for:* To read the results and data from `takemeter.ipynb`
  and the results files, and write them into the right README sections.
  I gave it the context of the task and the responses each section needed.
- *What came back:* The numbers placed into the run logs, verdicts and
  comparison tables, with my reasoning paraphrased into the written
  sections. It saved me copying and typing everything by hand.
- *What I changed:* I made the calls myself and corrected the write-up
  where it didn't match my reasoning. The clearest case was the staff
  disagreement. The first draft conceded to staff, and I replaced it with
  my actual reason for `hot_take`: the 2% has no stated source or method,
  so it isn't checkable. I also chose to report criterion 4 as not tested
  rather than substitute another measure, and chose the epochs change as
  my one improvement.

**Pre-labelling disclosure:**
20 rows were labelled cold, with no tools, and are marked `cold`. The other
540 were AI-pre-labelled from the written definitions, then read and
corrected by me, and are marked `pre-labeled, checked`. No row in the file
is still `needs review`.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

Both scored on the same 86 held-out posts in `test_split.csv` (seed 42). Baseline: `facebook/bart-large-mnli`, zero-shot, using my definitions from `label_definitions.txt`. Trained: `results.json` from unit 5.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.453 | 0.419 | -0.035 |
| Macro F1 | 0.274 | 0.230 | -0.044 |
| F1 — `experience_request` | 0.000 | 0.000 | +0.000 |
| F1 — `miscellaneous` | 0.000 | 0.000 | +0.000 |
| F1 — `risk_check` | 0.557 | 0.348 | -0.209 |
| F1 — `rule_question` | 0.540 | 0.571 | +0.032 |

**What I predicted before I looked:**
I predict the trained model will beat the baseline on `risk_check` and `experience_request`, because those two share F-1 vocabulary and differ only in intent ("should I…" vs. "has anyone…"). One-sentence definitions give a zero-shot model little to separate them, whereas the trained model saw 172 and 126 examples of the phrasing. I expect the baseline to match or beat it on `rule_question`, where the definition alone is enough. One caveat: my seed-42 `experience_request` F1 was 0.0, so this prediction could be wrong.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

The trained model did **not** beat the baseline. It lost on accuracy (0.419 vs. 0.453), on macro F1 (0.230 vs. 0.274) and on `risk_check` (0.348 vs. 0.557). It was only slightly ahead on `rule_question` (0.571 vs. 0.540). So on this split, my fine-tuning added nothing measurable over a general model reading my definitions. My prediction was wrong: I expected a win on `risk_check` and `experience_request`, and got a loss and a tie at zero.

The more important finding is that both models scored F1 0.0 on `experience_request` and `miscellaneous`. A trained model and an untrained one failing on the same two labels points at the task or the labels, not at the training. The baseline is also slower (a few seconds per post on `mps`), so on cost alone it is no better. This is one seed, so I'll revisit the comparison once I have three.

**Revisited after the improvement (10 epochs).** The baseline was only scored on the seed-42 split, so seed 42 is the only fair comparison. The before run's seed 42 reproduces `results.json` exactly, which confirms it is the same split. On those 86 posts, the 10-epoch model beats the baseline on accuracy (0.593 vs. 0.453) and on every label except `miscellaneous`: `rule_question` 0.657 vs. 0.540, `experience_request` 0.600 vs. 0.000, `risk_check` 0.607 vs. 0.557, and `miscellaneous` 0.000 for both. So my original prediction, a win on `experience_request` and `risk_check`, held once the model was trained long enough. The 3-epoch loss came from undertraining, not from fine-tuning adding nothing.

**Definitions vs. bare label names.** I also ran the baseline with no definitions, so the model saw only the label names. Same 86 held-out posts.

| Measure | With definitions | Bare names | Difference |
|---|---|---|---|
| Overall accuracy | 0.453 | 0.360 | +0.093 |
| Macro F1 | 0.274 | 0.306 | -0.032 |
| F1 — `experience_request` | 0.000 | 0.434 | -0.434 |
| F1 — `miscellaneous` | 0.000 | 0.182 | -0.182 |
| F1 — `risk_check` | 0.557 | 0.450 | +0.107 |
| F1 — `rule_question` | 0.540 | 0.158 | +0.382 |

My definitions were worth about 9 points of accuracy, but they did not help every label. They sharply improved `rule_question` and `risk_check`, and they made `experience_request` and `miscellaneous` *worse*: both dropped to F1 0.0. Bare names actually gave a higher macro F1. I report the with-definitions run as my baseline because that is the run `label_definitions.txt` was written for.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

Run on `mps` (Apple Silicon), fine-tuned `distilbert-base-uncased`, 3 epochs, lr 2e-5, batch 16, max length 128. Source: `results_three_seeds_before.json`. Accuracy moved only 0.023 across the three seeds (0.419 to 0.442), well under the ten-point warning line, so the poor result is stable and not a bad draw.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 on every seed | 0.419 | 0.442 | 0.419 | MISSED |
| 2. Per-label F1 (`rule_question` / `experience_request` / `risk_check` / `miscellaneous`) | first three ≥ 0.50, `miscellaneous` ≥ 0.40 | 0.571 / 0.000 / 0.348 / 0.000 | 0.539 / 0.000 / 0.509 / 0.000 | 0.537 / 0.190 / 0.294 / 0.000 | MISSED |
| 3. Balance: largest predicted-label share | ≤ 0.50 | 0.767 (`rule_question`) | 0.663 (`rule_question`) | 0.884 (`rule_question`) | MISSED |
| 4. Consistency: second reader agrees on the 20 `cold` rows | ≥ 15 / 20 | not run yet | not run yet | not run yet | NOT TESTED |
| 5. Confidence gap (most minus least confident third) | ≥ 0.12 | 0.179 | 0.536 | 0.464 | MET |

### Confusion matrix

Seed 42, the same split as `results.json`. Rows are true labels, columns are the model's guesses.

| true \ predicted | rule_question | experience_request | risk_check | miscellaneous |
|---|---|---|---|---|
| **rule_question** | 28 | 0 | 4 | 0 |
| **experience_request** | 14 | 0 | 5 | 0 |
| **risk_check** | 18 | 0 | 8 | 0 |
| **miscellaneous** | 6 | 0 | 3 | 0 |

**My biggest off-diagonal number, and what it means:**

The biggest off-diagonal cell is 18 real `risk_check` posts called `rule_question`, against only 4 real `rule_question` posts called `risk_check`. The model hasn't learned that boundary. It defaults to `rule_question` for anything that sounds procedural. The same pull shows in the other rows: 14 `experience_request` posts and 6 `miscellaneous` posts also went to `rule_question`, and neither of those labels was predicted at all on this seed.

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.60 on every seed | MISSED | Missed on all three seeds (0.419, 0.442, 0.419), each 0.16–0.18 short. The best seed is still nowhere near the target. |
| 2 | Per-label F1 | `rule_question`, `experience_request`, `risk_check` ≥ 0.50; `miscellaneous` ≥ 0.40 | MISSED | Only `rule_question` clears its floor on every seed (0.571, 0.539, 0.537). `risk_check` passes once (0.509 on seed 7) and fails twice (0.348, 0.294). `experience_request` never passes (0.000, 0.000, 0.190), and `miscellaneous` is 0.000 on every seed. One failing label on one seed is enough to miss. |
| 3 | Balance: largest predicted-label share | ≤ 0.50 on every seed | MISSED | `rule_question` is over half the guesses on every seed: 66/86 = 0.767, 57/86 = 0.663, 76/86 = 0.884 (column sums of each seed's confusion matrix). |
| 4 | Consistency: second reader on the 20 `cold` rows | ≥ 15 / 20 | NOT TESTED | No second reader labelled the 20 `cold` rows before this deadline, so there's no count to compare with 15. Untested isn't met. I didn't substitute the 29/30 staff agreement, because that's a different taxonomy on different posts. |
| 5 | Confidence gap (sure third − unsure third) | ≥ 0.12 on every seed | MET | Holds on all three seeds as raw differences: 0.179, 0.536, 0.464. Met as written. The diagnosis below explains why this pass is weaker than it looks. |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

**Criteria 1, 2 and 3 are one failure seen three ways: the model collapses onto `rule_question`.** On every seed, almost every row of the confusion matrix sends most of its posts to the `rule_question` column. 14, 15 and 17 of the 19 true `experience_request` posts went there, as did 18, 12 and 21 of the 26 true `risk_check` posts and 6, 6 and 9 of the 9 `miscellaneous` posts. `miscellaneous` was never predicted on any seed, and `experience_request` was predicted twice in 258 test predictions. That collapse is why criterion 3 fails, why accuracy sits barely above the 0.37 that always guessing `rule_question` would score (criterion 1), and why two labels have F1 0.000 (criterion 2). Accuracy moved only 0.023 across seeds, so this is a stable result, not an unlucky split.

**Is it my labelling or the model?** I can't test my own labels directly, because criterion 4 was not run. The evidence I do have points away from inconsistent labelling as the main cause:

- **Staff agreement (indirect):** I matched staff on 29 of 30 posts under a written taxonomy I didn't design. The one disagreement was a defensible reading of an undefined word ("checkable"), not a rule applied two ways (see the Agreement Report). That suggests I apply written decision rules consistently. It's a different taxonomy, so this is indirect.
- **The confusion goes the wrong direction for a labelling problem.** If my `experience_request` / `risk_check` rule were applied inconsistently, those two labels would trade posts with each other. They barely do: `experience_request` → `risk_check` is 5, 4 and 0, and `risk_check` → `experience_request` is 0 on every seed. Both drain into `rule_question` instead. Even the hardest boundary I named isn't where the model fails; it never got far enough to fail there.
- **The baseline fails on the same two labels.** The zero-shot model, which never saw my labels, also scored F1 0.000 on `experience_request` and `miscellaneous`. A model that never trained on my labels can't have learned my inconsistencies. So part of the difficulty is in the task and in how those two definitions are worded. With bare label names instead of my definitions, the baseline scored 0.434 and 0.182 on them.

**The training evidence points at undertraining.** Training loss started between 1.34 and 1.36 and ended between 1.18 and 1.21 on every seed. A model guessing uniformly over four labels scores ln 4 ≈ 1.386, so it barely moved off chance. Validation loss was still falling at epoch 3 (1.279 → 1.251 → 1.243 on seed 42), so training stopped while the model was still improving. The model's confidence says the same thing: its average top probability is about 0.40 whether it is right or wrong (0.41 vs. 0.39 on seed 42), only a little above the 0.25 of a pure guess. Three epochs over 401 rows is about 78 update steps. That's enough to learn "mostly say `rule_question`", the largest label (150 of 401 training rows). It isn't enough to learn an intent distinction in a community where every post uses the same F-1 vocabulary (OPT, I-20, USCIS, DSO).

**Per-label causes:**

- **`miscellaneous` (F1 0.000 × 3):** too few examples. It has 43 training rows and 9 test rows, the smallest label by far, and its posts still talk about F-1 life, so they look like questions. It was never predicted.
- **`experience_request` (F1 0.000, 0.000, 0.190):** a genuinely hard label for this setup. What separates it is intent ("has anyone…"), not vocabulary. Its posts go to `rule_question`, not to `risk_check`, so this is the collapse rather than the boundary I expected to be hardest.
- **`risk_check` (F1 0.348, 0.509, 0.294):** the jumpiest label. When the model happens to use `risk_check` more (seed 7, with 29 predictions), F1 clears 0.50. When it barely uses it (seed 2024, with 8 predictions), F1 drops to 0.294. Whether it passes depends on how far each seed collapsed, not on a learned boundary.

**Why criterion 5's pass is weaker than it looks.** The gap is real on all three seeds, but the sure third is mostly the model repeating `rule_question` on obvious rule posts (`rule_question` recall is 0.75–0.91). It ranks the majority habit, not a learned four-way distinction. The verdict stays MET because the criterion as written was met.

**Diagnosis in one line:** the main cause is a task the model can't reach in this training setup (an intent-not-topic distinction learned from too few update steps), plus too few `miscellaneous` examples. The evidence I have doesn't point at inconsistent labelling, but criterion 4 is the test that would confirm it, and it wasn't run.



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 29 / 30 = 96.7%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:* "Carbon plates aren't the whole story. My times dropped 2% in the shoes and another 2% the following year in the same pair, when the only thing that changed was going from 35 to 50 miles a week."
>
> *Staff said / I said:* Staff `analysis` / I said `hot_take`
>
> *My call, and why:* I still think `hot_take` is defensible, though staff's reading has a real case. To me the post reads as a confident claim ("carbon plates aren't the whole story") rather than an argument. The `analysis` definition asks for a *specific, checkable* fact, and the 2% isn't checkable: nothing says what it is 2% of, which races or distances, or how it was worked out. With no reference behind it, it reads like a bluff number, the "statistic in it" that Rule 1 warns about. The case for staff is that Rule 1 only asks whether the fact carries the argument, and the structure is an argument: the second 2% drop came with the shoes held constant and only the mileage changed, and that comparison is the whole reason given. But Rule 1 only matters once there is a checkable fact, and that is the step I don't think this post clears. It also separates this post from the other two runner's-own-numbers posts I labelled `analysis`. The sleep post says how its number was collected (eleven weeks of tracking, intervals 6–8 seconds per mile slower), and so does the heat post (the same loop, the same heart-rate effort, 90 seconds over six miles). This one gives a bare percentage with no method.
>
> *Which pile:* (c). The rule is ambiguous here and my reading is defensible. The taxonomy never says what makes a fact "checkable", and in particular whether a self-reported, unsourced figure counts.

**2.**
> *The post:* None. Staff and I matched on the other 29 posts.
>
> *Staff said / I said:* n/a
>
> *My call, and why:* n/a
>
> *Which pile:* n/a

**What the pattern in my disagreements tells me:**

With one disagreement out of 30, there isn't much of a pattern, but the single miss is informative. I agreed with staff on every `reaction` post and every bare-assertion `hot_take`. The one miss came on the `analysis` / `hot_take` boundary, and it wasn't me applying a rule loosely. Staff and I read an undefined word in the definition, "checkable", two different ways. I held a number to a higher standard (it has to say where it came from) than staff did (any specific number that does work in the argument). I applied my standard the same way across all of the runner's-own-numbers posts. For my own 200 labels, this is only indirect evidence, because it's a different taxonomy on different posts. It suggests I apply a written rule consistently. The weak point is a definition that leans on a word nobody has pinned down, because then two careful readers can each be consistent and still disagree. That is the thing to check in Milestone 4: whether my own definitions leave a key word that loose. It matters most on my hardest boundary (`experience_request` vs. `risk_check`) and on the pair the matrix confuses most (`risk_check` called `rule_question`). This check does **not** stand in for criterion 4 (a second reader on my own 20 `cold` rows), which is a separate measurement.


---

## The Improvement

**What I changed:**
One training setting: `EPOCHS = 3` → `EPOCHS = 10`. Everything else is identical to the before run: the same `labels.csv`, taxonomy, criteria, base model (`distilbert-base-uncased`), learning rate (2e-5), batch size (16), max length (128) and seeds (42, 7, 2024).

**Which diagnosis pointed at it:**
Undertraining. In the before run, training loss only fell from about 1.35 to 1.18–1.21, against ln 4 ≈ 1.386 for a uniform guess over four labels. Validation loss was still falling at epoch 3, and the model's average top probability was about 0.40, barely above the 0.25 of a pure guess. Three epochs over 401 rows is about 78 update steps, which was enough to learn "say `rule_question`" but not the intent distinction. I didn't relabel, because the labelling evidence (29/30 staff agreement, and `experience_request` / `risk_check` barely trading posts) didn't point at my labels.

### Run Log — After

Run on `mps` (Apple Silicon), fine-tuned `distilbert-base-uncased`, **10 epochs**, lr 2e-5, batch 16, max length 128. Source: `results_three_seeds_after.json`. Accuracy moved 0.024 across the three seeds (0.581 to 0.605), still well under the ten-point warning line.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 on every seed | 0.593 | 0.581 | 0.605 | MISSED |
| 2. Per-label F1 (`rule_question` / `experience_request` / `risk_check` / `miscellaneous`) | first three ≥ 0.50, `miscellaneous` ≥ 0.40 | 0.657 / 0.600 / 0.607 / 0.000 | 0.636 / 0.564 / 0.621 / 0.000 | 0.677 / 0.683 / 0.561 / 0.000 | MISSED |
| 3. Balance: largest predicted-label share | ≤ 0.50 | 0.407 (`rule_question`) | 0.395 (`rule_question`) | 0.384 (`rule_question`) | MET |
| 4. Consistency: second reader agrees on the 20 `cold` rows | ≥ 15 / 20 | not run | not run | not run | NOT TESTED |
| 5. Confidence gap (most minus least confident third) | ≥ 0.12 | 0.179 | 0.143 | 0.429 | MET |

**Did it help, and how do I know:**

Yes, a lot, though it didn't turn every miss into a pass. Criterion 3 flipped from MISSED to MET. Criteria 1 and 2 are still MISSED, but they are now close rather than far.

- **The collapse is gone (criterion 3, MISSED → MET).** The largest predicted-label share dropped from 0.663–0.884 to 0.384–0.407 on every seed. The model now predicts `experience_request` 20–22 times and `risk_check` 30–32 times per seed, where before it predicted `experience_request` twice in all three seeds combined.
- **Accuracy (criterion 1) rose about 0.16–0.19 on every seed, but it still misses.** It went from 0.419 / 0.442 / 0.419 to 0.593 / 0.581 / 0.605. Seed 2024 clears 0.60. Seeds 42 and 7 don't (0.593 is seven-thousandths short), and the target has to hold on every seed, so it stays MISSED.
- **Per-label F1 (criterion 2) still misses, but only on one label.** `rule_question`, `experience_request` and `risk_check` now clear 0.50 on all three seeds; `experience_request` went from 0.000 / 0.000 / 0.190 to 0.600 / 0.564 / 0.683. `miscellaneous` is still 0.000 on every seed, because it was never predicted once. More training fixed the labels that had enough examples and did nothing for the one that doesn't.
- **The new errors are on the boundary I expected to be hard.** In seed 42, `experience_request` → `risk_check` is now 5, and `risk_check` → `experience_request` is 2. Those two labels now trade posts with each other instead of both draining into `rule_question`. The model got far enough to meet the real boundary.
- **Confidence (criterion 5) is still MET, and it now means more.** The gap is 0.179 / 0.143 / 0.429, all ≥ 0.12. The model's average top probability rose from about 0.40 to 0.63–0.75. Its sure third is no longer just the `rule_question` habit, because it is spread across three labels.
- **One warning sign: validation loss bottomed out early and then rose.** The lowest validation loss came at epoch 6 (seed 42), epoch 4 (seed 7) and epoch 6 (seed 2024). By epoch 10 it had risen, most on seed 7 (1.142 → 1.237), while training loss kept falling. The notebook keeps the last epoch, not the best one (`save_strategy="no"`). So 10 epochs is past the sweet spot, and the model is starting to memorise the training set. Seed 7, the most overfit, also has the lowest accuracy (0.581). Stopping earlier might push seeds 42 and 7 over 0.60, but that would be a second change, so I'm only noting it here.



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

**Criterion 1: overall accuracy (MISSED, 0.593 / 0.581 / 0.605).**
*What I'd do:* keep the best epoch instead of the last one. Validation loss bottomed out at epoch 6, 4 and 6 and then rose, while the notebook keeps the epoch-10 model (`save_strategy="no"`). I'd evaluate and save every epoch and load the best one (`load_best_model_at_end=True`, or simply train for about 6 epochs). Seeds 42 and 7 are 0.007 and 0.019 short, and seed 7 is both the most overfit and the least accurate. So stopping at the validation minimum is the change most likely to push all three over 0.60.
*Why I stopped:* this unit allows exactly one change, and I spent it on the number of epochs. Early stopping would be a second change, and its effect couldn't be separated from the first.

**Criterion 2: per-label F1 (MISSED, but only on `miscellaneous`, 0.000 on every seed).**
*What I'd do:* add at least 30 more `miscellaneous` posts, labelled with the same care as the first batch. It has only 61 rows (43 in training, 9 in test), and in all six runs, before and after, the model never predicted it once. The other three labels were fixed by more training; this one wasn't, which points at too few examples, not too little training. If more data alone didn't move it, the next step would be class weights in the loss, so that the smallest label isn't drowned out by the other 358 training rows.
*Why I stopped:* the one change went to the cause behind three failing criteria (the `rule_question` collapse), not to the single label left. Labelling 30 good `miscellaneous` posts from r/f1visa is also a full labelling session in its own right.

**Criterion 4: consistency (NOT TESTED).**
*What I'd do:* give one second reader the 20 `cold` texts and my README definitions and decision rule, with no labels showing, and count exact matches against my 15/20 target. That's the only direct test of whether my own labels are consistent. Until it runs, the "not a labelling problem" part of my diagnosis rests on indirect evidence (the 29/30 staff agreement and the shape of the confusion matrix).
*Why I stopped:* I couldn't get a second reader to do the labelling before this deadline.



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->

I meant my labels to capture intent, meaning what kind of answer would settle the post, but at 3 epochs the model learned topic and sent 66, 57 and 76 of 86 posts to `rule_question`; only at 10 epochs did it learn intent for three labels, with `experience_request` and `risk_check` now confusing each other instead of both draining into `rule_question`. It never learned `miscellaneous`, the one label defined by what a post *isn't* asking: in every after-run matrix all 9 `miscellaneous` posts went to the other three labels, so to the model "not a visa question" still looks like a visa question.



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
