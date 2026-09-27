# The Unofficial Guide

Samuel Akproh: campus_life corpus

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does
This system is an unofficial campus guide built on a Retrieval-Augmented Generation (RAG) architecture using the `campus_life` corpus. It indexes 88 student posts covering dorm life, course workloads, dining options, and administrative policies (such as add/drop deadlines and the housing lottery). Given a plain question, it retrieves semantically relevant text passages from ChromaDB and prompts Gemini to produce grounded answers strictly citing the source files.


## Chunking Strategy
The starter's fixed 800-character window did not split anything because most `campus_life` posts are under 500 characters, turning 88 documents into 88 whole posts. To preserve complete thoughts rather than slicing arbitrarily, I replaced it with a paragraph-based chunker in `chunker.py::split_documents`. It splits on double-newlines (`\n\n`), discards isolated fragments under 40 characters, and preserves entire multi-sentence paragraphs as cohesive thoughts so each chunk contains sufficient context to stand on its own.


**Chunk size:** Variable, paragraph-based (natural post boundaries, min 40 characters up to ~500 characters)

**Overlap:** 0 characters

*Why these values:* The `campus_life` corpus consists of 88 short, self-contained student forum posts rather than long multi-page policy manuals. When reading the files in Milestone 1, I observed that a single post or paragraph rarely exceeds 500 characters and captures one cohesive idea. By splitting on double-newlines (`\n\n`), each chunk captures an entire post or cohesive thought without needing an artificial character window. Overlap is set to 0 because distinct forum posts and paragraphs are independent—duplicating lines across independent posts would add redundant tokens to the prompt without adding context.

## Sample Chunks

**Chunk 1** —  source: admin_add_drop_deadline.txt#0  by: chunker.py::split_documents



```
You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.

```

**Chunk 2** — source:course_cs_340_workload.txt#1  — produced by: chunker.py::split_documents



```
It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 3** — source:  course_phys_130_exams.txt#1 — produced by: chunker  py::split_documents


```
The lab practical is worth 20% and almost nobody prepares for it.
```

**Chunk 4** —  source: dining_verrill_street_grill_followup.txt#0 — produced by:  chunker.py::split_documents



```
Adding to what people have said about Verrill Street Grill. The wait figure of up to 30 minutes on Friday evenings matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.
```

**Chunk 5** — source: housing_morrow_house.txt#1  — 
produced by: chunker.py::split_documents



```
The good: cheapest housing tier by about $900 a year, and the singles are real singles.
```

## Sample Answer

**Question:**
Is the housing lottery completely random for all students?
**Answer:**

No, the housing lottery is not completely random for all students. While rising sophomores get a number drawn at random, juniors and seniors are ordered by accumulated credit hours first, with random tie-breaking used only for ties.

Source: `admin_housing_lottery.txt`


```Sources retrieved: admin_housing_lottery.txt, advising_registration.txt, housing_aldridge_hall.txt, housing_morrow_house.txt, housing_tamsin_court.txt

1 model calls this session, 414 tokens (355 in, 59 out)
```

**My relevance cutoff:** `0.70`

*Analysis:* My campus life questions produced semantic distances ranging from `0.179` to `0.621`. The off-topic questions in `OUT_OF_SCOPE` produced much higher distances between `0.821` and `0.885`. I set the cutoff to `0.70`, right in the gap between `0.621` and `0.821`. This permits all legitimate campus queries to pass through while cleanly refusing all 5 out-of-scope questions.

| Question | In corpus? | Best distance |
|---|---|---|
| Is the housing lottery completely random for all students? | Yes | 0.197 |
| How are rising sophomores assigned numbers in the housing lottery? | Yes | 0.179 |
| Can freshmen get a campus parking permit? | Yes | 0.621 |
| When do study abroad applications open? | Yes | 0.307 |
| What are the common student complaints about Morrow House? | Yes | 0.562 |
| What is the capital of Mongolia? | No | 0.821 |
| How do I change the oil in a diesel engine? | No | 0.885 |
| Who won the 1994 World Cup? | No | 0.874 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.824 |
| How do I write a for loop in Rust? | No | 0.831 |

## How I Used AI


<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

1. **Developing the chunker function(`chunker.py`):**
   - **What I asked:** I explained that `campus_life` contains short forum posts where the starter's 800-character fixed window produced 88 identical whole-document chunks. I asked Gemini to generate a splitting function that respects natural paragraph boundaries instead of character counts.
   - **What came back:** Gemini suggested splitting strings on `\n\n` into paragraph chunks and adding a basic length check.
   - **What I changed:** I modified the code to ensure `produced_by` correctly reported `"chunker.py::split_documents"`, added a fallback to keep short single-paragraph posts intact if no double newlines existed, and tuned the length filter to skip isolated heading lines under 40 characters so they wouldn't become fragmented chunks.

 
2. **Selecting and analyzing the relevance cutoff (`config.py`):**
   - **What I asked:** After running `run_eval.py`, I shared my evaluation distance scores (campus question distances from 0.179 to 0.621 vs. out-of-scope question distances from 0.821 to 0.885) and asked where to place the cutoff threshold.
   - **What came back:** Gemini analyzed the distribution gap and recommended setting `RELEVANCE_CUTOFF` to 0.70 to create a safe boundary between the two clusters.
   - **What I changed:** Rather than blindly accepting the default 0.60 (which was slightly too tight and would have blocked my valid freshman parking question at 0.621), I verified the math against my test logs, updated `config.py` to 0.70, and confirmed that all 5 out-of-scope questions were still cleanly refused by the gate.
<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
