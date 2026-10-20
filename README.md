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
| supported_claim | 64 | 32% |
| build_log | 66 | 33% |
| **Total** | **200** | 100% |

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

**Base model:**

**Settings:** <!-- epochs, learning rate, batch size, seed -->

**Anything I changed from the defaults, and why:**

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Pre-labelling disclosure:**

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



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
