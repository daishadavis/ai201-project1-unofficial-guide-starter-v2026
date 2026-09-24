# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

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
A retrieval-augmented question-answering system over a corpus of campus-life advice threads (advice_threads). Documents are indexed as embeddings; when a question comes in, the system retrieves the most relevant chunks, checks whether any are close enough to be worth answering from, and if so, generates an answer grounded only in those chunks — citing the source file. Questions the corpus doesn't cover (e.g., general trivia, unrelated technical questions) are refused rather than answered from the model's own knowledge.


## Chunking Strategy

**Chunk size:** N/A — chunks are split structurally, not by character count.
**Overlap:** N/A — no overlap needed; replies don't share content with each other.

My corpus (`advice_threads`) is forum threads: a title line, then several
independently-voted replies. When I read these in Milestone 1, each reply was
already a complete, self-contained thought responding to the thread's
question — nothing like a long guide where a fixed window makes sense.

The starter's fixed 800-character windows were the wrong shape for this data:
on a short thread, a window could merge two unrelated replies into one chunk
(diluting the correct answer with an off-topic one). On a thread that didn't
divide evenly by 800 characters, it produced a chunk as short as 2 characters
— pure noise, the tail end of a document with no content in it.

Instead, `split_documents` splits each document on its `--- reply N (X votes)
---` markers, so one chunk = one reply. A reply alone still loses the
question it's answering (e.g. "Yeah. Cuts an 18 minute walk to about 6."
means nothing out of context), so I prepend the thread's title line to every
reply chunk to keep it self-contained.

Result: 26 chunks (fixed-window baseline, avg 487 characters, shortest 2,
longest 793) became 75 chunks avg 175 characters, shortest 105, longest
254 — smaller, far more uniform, and the 2-character debris chunk is gone.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `thread_bike_commute.txt#0` — produced by: `produced by: chunker.py::split_documents`

THREAD: Is a bike worth it for a 20 minute walk commute?

Yeah. Cuts an 18 minute walk to about 6. The thing nobody mentions is storage — covered bike parking exists at three buildings and is full by 9am at all three.


**Chunk 2** — source: `source: thread_first_gen.txt#1 ` — produced by: `produced by: chunker.py::split_documents`

THREAD: Anything specific for first-generation students?

The thing I'd say: the unwritten rules are the hard part, not the coursework. Ask about the unwritten rules explicitly. People are happy to explain them and nobody volunteers them.

**Chunk 3** — source: `thread_laptop_specs.txt#2 ` — produced by: `produced by: chunker.py::split_documents`

THREAD: How much laptop do I actually need for CS courses?

I did two years on an 8GB machine and it was fine until the last project, at which point it very much wasn't. 16 is the answer.


**Chunk 4** — source: `thread_parking.txt#1  ` — produced by: ` produced by: chunker.py::split_documents`

THREAD: Worth getting a parking permit?

Street parking on Verrill is legal and free and unmarked, which is why half the upper years do it.


**Chunk 5** — source: `thread_sleep_schedule.txt#1 ` — produced by: `chunker.py::split_documents`

THREAD: Everyone says fix your sleep. Does it actually matter?

The library being open until 2am is a trap. It's a resource, not a schedule.

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:**
Which building has it's own kitchenette?
**Answer:**

Fenwick has kitchenettes (thread_meal_plan_tier.txt).

Sources retrieved: thread_laundry_timing.txt, thread_meal_plan_tier.txt, thread_roommate_conflict.txt, thread_study_spots.txt, thread_winter_advice.txt


**My relevance cutoff:**

The cutoff (0.75) and the two distance groups (in-scope 0.322–0.715, out-of-scope 0.787–0.930)

| Question | In corpus? | Best distance |
|---|---|---|
|  |  |  |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->
    
**Chunking function (Milestone 3):** I described my corpus structure (forum
threads with a title and numbered `--- reply N (X votes) ---` blocks) and
asked for a replacement for `split_documents` that would split on reply
boundaries instead of fixed-size windows. Claude wrote a version using a
regex on the reply markers, with the thread title prepended to each reply so
it wouldn't lose context. It didn't include the `import re` the function
needed — I added that myself. Before trusting it, I ran `python chunker.py`
to check the actual numbers (26 → 75 chunks, shortest jumped from 2 to 105
characters) rather than assuming the strategy worked just because the code
ran without errors.

**Relevance cutoff (Milestone 4):** I ran my five test questions and the
five `OUT_OF_SCOPE` questions through `retrieve` and pasted the distances to
Claude to help find the gap. It suggested 0.75, sitting between my worst
in-scope distance (0.715, kitchenette) and my best out-of-scope distance
(0.787, World Cup). Rather than just setting it in `config.py` on that
recommendation, I re-ran both of those specific questions at 0.75 myself to
confirm the kitchenette question now passed the gate and the World Cup
question still got refused, before committing the number.


**1.**

**2.**

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
