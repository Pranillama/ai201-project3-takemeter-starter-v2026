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

On the held-out set, accuracy is at least 0.60 on every one of the three seeds.

**Why this target:** Three labels means a guesser gets about 0.33. 0.60 is the point where I would actually let this sort a feed. It has to hold on every seed, so one lucky run does not count.

---

## 2. Per-label performance

On the held-out set, every label has an F1 of at least 0.50 on every one of the three seeds.

**Why this target:** `build_log` was the rarest of the three in the posts I read, so its test split will be small and its F1 will jump. 0.50 is looser than the overall 0.60 on purpose, and it is still above a 0.33 guess. A model that only learns `question` fails this.

---

## 3. Balance

In `labels.csv`, no label is more than 50% of the rows.

**Why this target:** The hot feed was mostly people asking for help. The course already rejects a file where one label is over 70%. 50% is tighter than that, because `supported_claim` and `build_log` have to be real parts of the set, not leftovers.

---

## 4. Consistency

When I label the 30 staff posts under the staff definitions in `data/staff_taxonomy.md`, I match the staff label on at least 21 of the 30 (70%).

**Why this target:** This is someone else's taxonomy, which is the check on whether a written rule can be applied twice. 21 of 30 is "usually the same label." I did not pick 27 of 30, because their analysis / hot_take boundary is the same kind of hard call as `question` / `supported_claim`.

---

## 5. Confidence

On the held-out set, for every seed, accuracy on the most-confident third of predictions is at least 0.10 higher than accuracy on the least-confident third.

**Why this target:** If a confident guess is not right more often than an unsure one, the score is not usable for skipping past posts. A third of a ~30-post test split is about 10 posts, so 0.10 is a gap I can actually see, and I am not asking the model to be sure of everything.



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
