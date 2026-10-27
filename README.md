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

TakeMeter classifies posts from [r/AI_Agents](https://www.reddit.com/r/AI_Agents/), a public Reddit community about building and using AI agents. Anyone can read it. There is enough text there to collect 200 posts.

I read the current hot feed: 15 text posts, skipping ads and the weekly megathreads. These three distinctions showed up in the posts, and they are pointable in the text.

1. **The post asks for an answer.** The body is a question or a request for a tool. "How realistic is it to build a small custom CRM/ERP with AI?" asks how long a custom build takes, what maintenance looks like, and whether an existing product is better. "Best AI Agent for distinguishing between voices?" asks for a tool for an upcoming meeting.

2. **The post makes a claim and gives a checkable detail.** A count, a price, or a named event is in the post. "Lessons from trying to buy from ~1,900 x402 APIs in one day (1,255 delivered)" reports about 1,900 attempts, 1,255 delivered, and $12.19 spent, then lists what failed (151 extra 402 responses, listed prices up to 5x higher live). "Accenture just capped Claude spend at $29/person after some staff hit $1,000 months" hangs the claim on those dollar figures.

3. **The post is a build status.** It lists what the author has shipped or is building, and that list is the point of the post. "Milo Road Map" goes through what is done (brain, voice, and memory) and what is next.

Those distinctions are now the labels `question`, `supported_claim`, and `build_log`. Posts that are only a bare opinion, with no question, no checkable detail, and no build status, stay out of the dataset.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `question`

**Definition:** The post's main purpose is to get an answer, a recommendation, or a tool from other people.

**Example 1:**
> "How realistic is it to build a small custom CRM/ERP with AI?" asks how long a custom build takes, what maintenance looks like, and whether an existing product is better.

**Example 2:**
> "Best AI Agent for distinguishing between voices?" asks for a tool for an upcoming meeting.

### `supported_claim`

**Definition:** The post's main purpose is to report a result, and that result includes a count, a price, or a named event.

**Example 1:**
> "Lessons from trying to buy from ~1,900 x402 APIs in one day (1,255 delivered)" reports about 1,900 attempts, 1,255 delivered, and $12.19 spent.

**Example 2:**
> "Accenture just capped Claude spend at $29/person after some staff hit $1,000 months." The post is about that cap and those amounts.

### `build_log`

**Definition:** The post's main purpose is to say what the author's own system does now, or what stage that system is in.

**Example 1:**
> "Milo Road Map" lists what is done (brain, voice, and memory) and what is being built next.

**Example 2:**
> "A Cloud Desktop Computer for your AI Agent" describes Cubicles, a cloud desktop the author built so an agent is not running on their laptop.

### The hardest boundary

**Which two labels:** `question` and `supported_claim`

**The post that sits on it:** "What's actually worth automating in customer support with AI agents?" It states a rule (automate low-risk requests, escalate irreversible ones) and then asks what people run in production. The details in it are hypothetical examples, not a result the author measured.

**The decision rule I used every time:** If the count, price, or named event is a result the author is reporting as the point of the post, it is `supported_claim`, even when the last line asks a question. If that detail is only an example inside a request for advice, it is `question`. If the body is a status of the author's own system, it is `build_log` even when that status includes a number or a trailing question.



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Public text posts on r/AI_Agents (https://www.reddit.com/r/AI_Agents/). Each row is the title plus the full body. Posts were not trimmed, and the file is not split.

**How I labelled them:** Pre-labelled from the definitions and the decision rule in Label Taxonomy, then checked. Every note says `pre-labelled`. None are marked `cold`. The 20 cold labels were supposed to be done with no tools, and these were not, so they are not marked that way.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| question | 70 | 35% |
| supported_claim | 56 | 28% |
| build_log | 74 | 37% |
| **Total** | **200** | 100% |

Unit 5 filed this file as 70 / 64 / 66. The Improvement moved 8 rows from `supported_claim` to `build_log`. The before run log uses the original counts.

**Three hard cases**

**1.**
> *The post:* "I audited 73 public agent trajectories for wasted tokens — 74% had measurable waste." The body also says the author built the chewme-audit CLI that did the audit.
>
> *Could have been:* `supported_claim` or `build_log`
>
> *I chose `supported_claim`, because:* the title's point is the measured result (73 trajectories, 74%). The tool is how they measured it.

**2.**
> *The post:* "I built MCP tools for AI agents, and the agents got good enough that they don't need them." It has been live about 4 months and nobody is using it.
>
> *Could have been:* `build_log` or `supported_claim`
>
> *I chose `build_log`, because:* the post is the status of their own API. The 4 months is part of that status, not a separate measurement study.

**3.**
> *The post:* "What's actually worth automating in customer support with AI agents?" It states a rule (automate low-risk requests, escalate irreversible ones) and asks what people run in production. The order and refund examples are hypothetical.
>
> *Could have been:* `question` or `supported_claim`
>
> *I chose `question`, because:* the details are examples inside a request for advice, not a result the author measured.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, 128 tokens per post, seed 42. Defaults, unchanged.

**Anything I changed from the defaults, and why:** Nothing. The label list is `question`, `supported_claim`, `build_log`.

**Device:** `mps` (Apple Silicon GPU). Torch 2.14.1. `results.json` records the same device.

**Split sizes:** train 139, validation 31, test 30. Test counts: `question` 10, `supported_claim` 10, `build_log` 10. None is under 8.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* Turn the three distinctions from the r/AI_Agents hot feed into labels, and stress-test them with posts that sit on the boundaries.
- *What came back:* `question`, `supported_claim`, and `build_log`, plus the decision rule. A measured result is `supported_claim` even if the last line asks a question. A detail that is only an example inside a request is `question`. A status of the author's own system is `build_log`.
- *What I changed:* One stress-test post ("Agents are dead. Nobody serious uses them.") fit none of the three. I did not add a fourth label. Posts like that stay out of the dataset.

**Moment 2**

- *What I asked for:* Collect 200 public r/AI_Agents posts and label them with that rule.
- *What came back:* A first pass of 70 `question`, 65 `supported_claim`, and 65 `build_log`.
- *What I changed:* After a spot check, "I'm building a memory agent for AI agents. Here's what I learned about why small models fail at memory" moved from `supported_claim` to `build_log`. The post describes the author's system. It does not report a measured result. Final counts are 70, 64, and 66.

**Moment 3**

- *What I asked for:* After the seed-42 matrix, which posts labeled `supported_claim` are actually the status of the author's own system under the decision rule.
- *What came back:* A list of posts whose titles lead with "we built" or "my agent now does X", with a number attached.
- *What I changed:* Eight of those moved to `build_log`. Posts whose point is the measurement stayed `supported_claim`, including the 73-trajectory audit and the voice agent post where 50% hung up. That change is The Improvement. It made the scores worse.

**Pre-labelling disclosure:** All 200 rows were pre-labelled from the written rules and then spot-checked. They were not labelled cold. Every note says `pre-labelled`. No row is marked `cold`. Eight notes also say they were relabelled in unit 6.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.567 | 0.533 | -0.033 |
| Macro F1 | 0.451 | 0.434 | -0.016 |
| F1 — `build_log` | 0.000 | 0.583 | +0.583 |
| F1 — `question` | 0.583 | 0.720 | +0.137 |
| F1 — `supported_claim` | 0.769 | 0.000 | -0.769 |

**What I predicted before I looked:**

The trained model should beat a definition-only baseline on `question` and `build_log`, where the post has an obvious shape, and lose on `supported_claim`. The Unit 5 run never predicted that label, so a model that only reads the definition may do better there than the fine-tune. This sentence was written before `baseline.py` finished. The Unit 5 trained numbers were already in `results.json`.

**What the gap actually means:**

Fine-tuning did not make the classifier more accurate overall. On the same 30 posts the baseline is at 0.567 and the trained model is at 0.533. What training did was swap which label gets learned. The baseline, reading only the definitions, got every `supported_claim` right and every `build_log` wrong (7 called `question`, 3 called `supported_claim`). The trained model learned `build_log` (F1 0.583) and got a bit better at `question`, and it never predicted `supported_claim`. The prediction above matches that split. The overall gap is a trade, not a win.



---

## Run Log — Before

Device for all three seeds: `mps`, Apple Silicon GPU, torch 2.14.1, `MAX_LENGTH` 128. Accuracy moved 0.067 across seeds.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 | 0.533 | 0.500 | 0.567 | MISSED |
| 2. Every label F1 | ≥ 0.50 | 0.000 (`supported_claim`) | 0.375 (`question`) | 0.526 (`build_log`) | MISSED |
| 3. No label over 50% of `labels.csv` | ≤ 50% | 35% | 35% | 35% | MET |
| 4. Match staff labels | ≥ 21/30 | — | — | — | waiting on `staff_labels.csv` |
| 5. Confident third beats unsure third | ≥ 0.10 | 0.600 | 0.100 | 0.500 | MET |

Criterion 2's seed columns are the worst label that seed, which is the number the target has to clear. The other labels on those seeds: seed 42 `question` 0.720 and `build_log` 0.583; seed 7 `supported_claim` 0.526 and `build_log` 0.560; seed 2024 `question` 0.632 and `supported_claim` 0.545. Criterion 3 is a property of the file (70 / 64 / 66 of 200), so it does not change with the seed. Criterion 5 on seed 7 is 0.600 versus 0.500, which meets "at least 0.10" on the line.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Seed 42, the first of the three. Rows are the true label.

| true \ predicted | question | supported_claim | build_log |
|---|---|---|---|
| **question** | 9 | 0 | 1 |
| **supported_claim** | 4 | 0 | 6 |
| **build_log** | 2 | 1 | 7 |

**My biggest off-diagonal number, and what it means:**

On seed 42, 6 real `supported_claim` posts were called `build_log`, and only 1 real `build_log` was called `supported_claim`. The model leans toward build status on that pair. The other seeds do not settle it: seed 7 sends 4 claims to `build_log`, and seed 2024 sends 4 build logs to `supported_claim`. The boundary moves with the split.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.60 on every seed | MISSED | 0.533, 0.500, 0.567. All three are under 0.60. |
| 2 | Every label F1 | ≥ 0.50 on every seed | MISSED | Seed 42 `supported_claim` is 0.000. Seed 7 `question` is 0.375. One miss on any seed is a miss. |
| 3 | No label over 50% | ≤ 50% of `labels.csv` | MET | The before file is 70 / 64 / 66. The largest share is 35%. |
| 4 | Match the staff labels | ≥ 21 of 30 | not called | `my_staff_labels.csv` is saved. `data/staff_labels.csv` is not in the repo yet. |
| 5 | Confident third beats the unsure third | ≥ 0.10 on every seed | MET | Gaps are 0.600, 0.100, and 0.500. Seed 7 is exactly 0.10, which meets "at least". |

**Diagnoses**

`staff_labels.csv` was not here when these were written, so the agreement report could not yet separate a loose rule from a hard pair. The evidence below is the matrix and a re-read of the decision rule. The agreement section will be filled when the file arrives.

**1. Overall accuracy.** Missed on every seed, and the spread is only 0.067, so this is not one unlucky split. The model is right about half the time on 30 posts. The seed-42 matrix shows where those errors go: `supported_claim` is not predicted, and 6 of those 10 posts are called `build_log`. The baseline, which reads the full post and the definition, scores 0.567 and gets every `supported_claim` right, so the signal is in the text. The fine-tune is not using it.

**2. Every label F1.** Two different labels fail on two different seeds. `supported_claim` is 0.000 on seed 42 and about 0.53 on the other two, so ten test posts are enough for that F1 to swing from nothing to just over the line. `question` is 0.720 on seed 42 and 0.375 on seed 7. `build_log` holds on all three (0.583, 0.560, 0.526). The unstable pair is `supported_claim` against `build_log`. I re-read the `supported_claim` rows against the written rule and found posts whose body is the status of the author's system, with a number attached. That is the loose boundary the matrix is pointing at. It is also a hard pair: a build update and a measured result often share the same shape. Both can be true. The improvement tests the loose-boundary reading.



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** not scored yet. `my_staff_labels.csv` is 30 rows, text copied unchanged from `data/staff_posts.csv`, labeled under `data/staff_taxonomy.md` (`analysis` 10, `hot_take` 10, `reaction` 10). `data/staff_labels.csv` is not in the repo. I asked for it and I have not opened an answer key.

**Disagreements**

Not written yet. The script has nothing to join. I will adjudicate each disagreement from the staff rules when the file is here, and I will not mark them all "staff is right" by default.

**What the pattern in my disagreements tells me:**

Nothing yet. The diagnosis above says so.



---

## The Improvement

**What I changed:** Eight rows in `labels.csv` moved from `supported_claim` to `build_log`. Counts went from 70 / 64 / 66 to 70 / 56 / 74. Nothing else changed: same model, 3 epochs, learning rate 2e-5, batch 16, `MAX_LENGTH` 128, same three seeds.

The eight are posts whose body is the status of the author's own system, which the decision rule already calls `build_log` even when a number is attached: the day-3 OSS Radar experiment, the Hermes cleanup walkthrough, both copies of the SurrealDB Agent Memory launch, the planner that can no longer write files, the "I turned off Muse" stack change, the one-MCP sales agent, and the offline phone agent. Posts whose point is the measurement stayed put, including the 73-trajectory audit and the voice-agent post where 50% of people hung up.

**Which diagnosis pointed at it:** Criterion 2. On seed 42, 6 real `supported_claim` posts were called `build_log` and only 1 went the other way. That direction usually means the boundary was applied loosely. I applied the written rule once, on that pair only.

### Run Log — After

Same device: `mps`, Apple Silicon GPU, torch 2.14.1, `MAX_LENGTH` 128. Accuracy moved 0.033 across seeds.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 | 0.467 | 0.467 | 0.433 | MISSED |
| 2. Every label F1 | ≥ 0.50 | 0.000 (`supported_claim`) | 0.000 (`supported_claim`) | 0.000 (`supported_claim`) | MISSED |
| 3. No label over 50% of `labels.csv` | ≤ 50% | 37% | 37% | 37% | MET |
| 4. Match staff labels | ≥ 21/30 | — | — | — | waiting on `staff_labels.csv` |
| 5. Confident third beats unsure third | ≥ 0.10 | 0.700 | 0.400 | 0.500 | MET |

The other F1s: seed 42 `question` 0.400 and `build_log` 0.595; seed 7 `question` 0.429 and `build_log` 0.579; seed 2024 `question` 0.267 and `build_log` 0.595. Test supports shifted with the labels. On seed 42 they are `question` 11, `supported_claim` 8, `build_log` 11.

Seed 42 confusion matrix after the change:

| true \ predicted | question | supported_claim | build_log |
|---|---|---|---|
| **question** | 3 | 0 | 8 |
| **supported_claim** | 1 | 0 | 7 |
| **build_log** | 0 | 0 | 11 |

**Did it help, and how do I know:**

No. Accuracy fell on every seed (0.533 to 0.467, 0.500 to 0.467, 0.567 to 0.433). `supported_claim` F1 went to 0.000 on all three seeds, including the two seeds where it had been just over 0.50. The model now calls almost everything `build_log`: on seed 42, 8 real questions and 7 real claims were called `build_log`, and it never predicted `supported_claim`. Moving eight system-status posts into `build_log` made that label the easier guess. It did not teach the model the measurement.



---

## What's Still Broken

**1. Overall accuracy.** Still missed, and worse after the only change I am allowed. What I would do next is add posts whose point is a measurement and whose body is not a build update, so `supported_claim` is not just "a build log with a number in it." I stopped because this unit is one change.

**2. Every label F1.** Still missed. `supported_claim` is now 0.000 on every seed, and `question` dropped as well because those posts are being called `build_log`. Same next step as criterion 1. I am not lowering the 0.50 target. The number was measurable and I missed it.

**4. Staff agreement.** Not called. `my_staff_labels.csv` is committed. The verdict waits on `data/staff_labels.csv`.

**The gap between what I meant my labels to capture and what the model learned:**

The labels were meant to separate a reported measurement from the status of someone's system, and a question from both. The after matrix shows the model learned one habit: call it `build_log`. On seed 42 it did that for 8 of 11 questions and 7 of 8 claims, and the `supported_claim` column is all zeros. The number in a post was never enough, on its own, to make a class.



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
