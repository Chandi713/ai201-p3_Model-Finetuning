# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy

The `accuracy` value in `results.json` for the held-out test split is
**≥ 0.60** as a raw number (not rounded up). This must hold separately for
each of seeds **42, 7, and 2024**. A run of 0.599 fails.

**Why this target:** I picked 0.60 because `rule_question` is already 214 of
573 posts (37%), so a model that only guesses that label would score 0.37. A
target of 0.60 is the lowest number I would still call useful on this
community, and I did not pick 0.70 because the `experience_request` /
`risk_check` boundary will cost points across three seeds.

---

## 2. Per-label performance

Using the per-label `f1` fields in `results.json` for the held-out test
split, all three of `rule_question`, `experience_request`, and `risk_check`
are **≥ 0.50**, and `miscellaneous` is **≥ 0.40**. Each comparison uses the
raw F1, not a rounded value. This must hold separately for each of seeds
**42, 7, and 2024**. A missing label in the test split (support 0) is a miss
for that label, not a skip.

**Why this target:** I picked 0.40 F1 for `miscellaneous` because it is my
smallest label and I only have 61 examples of it, so about nine land in the
test split. I picked 0.50 for the other three because each of those has at
least 126 rows, enough that a 0.50 floor is a real miss and not a noisy
draw.

---

## 3. Balance

On the held-out test split, let *n* be the number of test rows and let
*count(y)* be how many times the model predicted label *y*. For every label
*y*, *count(y) / n* is **≤ 0.50**. Ties at exactly 0.50 pass; 0.501 fails.
Use predicted labels (confusion-matrix column sums), not true labels. This
must hold separately for each of seeds **42, 7, and 2024**.

**Why this target:** I picked 0.50 because `rule_question` is already 37% of
my labels. If more than half of the test guesses are one label, the model
learned to dump everything on `rule_question` and `miscellaneous` (61 rows)
never gets used. 0.50 is above 37%, so a little over-prediction is allowed;
a collapse is not.

---

## 4. Consistency

Take the 20 rows in `labels.csv` whose `note` contains `cold`. A second
reader, using only the README definitions and the written decision rule,
assigns one of the four labels to each of those 20 texts. Agreement is the
number of rows where the second reader's label string matches mine exactly.
That count is **≥ 15** (0.75). Near-misses, “either label is fine,” and
agreement on a different 20-row sample do not count.

**Why this target:** I picked 15/20 because the hardest boundary in this
taxonomy is `experience_request` vs `risk_check` — “has anyone done this?”
vs “should I do this?” I expect a second reader to split with me on about
five of those and still agree on the clean rule and miscellaneous posts. I
did not pick 18/20 because that would pretend the decision rule removed a
boundary I already know is hard.

---

## 5. Confidence

Using the held-out test split, let *A_high* be
`confidence.accuracy_most_confident_third` and *A_low* be
`confidence.accuracy_least_confident_third` in `results.json` (the
notebook's ranking of test rows by predicted probability, split into
thirds). *A_high − A_low* is **≥ 0.12** as a raw difference, not rounded.
This must hold separately for each of seeds **42, 7, and 2024**. A higher
sure-third accuracy that still leaves the gap under 0.12 fails this
criterion even if criterion 1 passes.

**Why this target:** I picked a gap of 0.12 because the held-out set is
about 86 posts, so each third is about 28 rows — 0.12 is roughly three extra
correct guesses in the sure third. On this community a high-confidence
`risk_check` is only worth trusting if sure guesses are actually righter
than unsure ones. Absolute accuracy is already criterion 1.



---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
