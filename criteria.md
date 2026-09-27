# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**
One of my questions is about a topic that is formulated in a way that looks like one outside the scope of the chunks, so
     I expect that one to be be hard to answer. 

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
 Because the pipeline only ingests and load documents that it was fed into, it can only uses those to answer the questions. The GROUNDING_INSTRUCTIONS in generate.py strictly looks at the information in the documents and if the documents don't cover the question, it says you don't have not enough information.



---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:**
"The cutoff is equal to 0.6. If the  distance between the chunks embeddings and the question is greater than the cutoff number, that's why out of scope questions gave that answer for 4 out of 5 questions. The one question that it didn't account for might be a very rare edge case where the question text share a coincidential semantic dimension with the chunks"

---

## 4. No chunk in the corpus is shorter than 100 characters or longer than 800 characters.

<!-- YOU WRITE THIS ONE.

     How would you know if your chunks were the right size? Name something
     countable or observable.

     Examples of the right shape — don't copy these, they should come from
     what you actually saw in Milestone 3:
       - "At least 4 of 5 sampled chunks read as a complete thought, with no
          sentence cut in half at either end."
       - "No chunk is shorter than 200 characters, since anything below that
          in my corpus turned out to be a heading with no content under it." -->



**Why this target:**
The campus_life corpus consists of 88 documents of short student posts. No chunk has a character count of less than 100 and more than 800. The 100 floor prevents posts to contain only headers, blank spaces to be standalone chunks, while the 800 ceiling makes sure unrelated posts don't share the same chunk


---

## 5. For all 5 of my answered test questions, the document cited in the final answer actually contains the specific claim made in the response, with 0 hallucinated source citations.

<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. It could be about
     speed, about refusals, about a particular kind of question your corpus
     handles badly, about source attribution being correct rather than merely
     present — anything, as long as it names a number or an observable
     outcome. -->



**Why this target:**

As mentioned in criterion 2, an answer can come up with a source for the generated answer. But the cited file may not contain the information the answer claims it does. I chose 5 of 5 because attributing a fact to the wrong document completely breaks user trust, which goes against the rules of our system and verifying that the claim actually exists in the named .txt file is either 100% correct or it is not.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
