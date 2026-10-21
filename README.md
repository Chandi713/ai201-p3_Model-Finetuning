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

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

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
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



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
