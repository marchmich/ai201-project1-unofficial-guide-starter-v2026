# The Unofficial Guide

Michelle Marchesini, I chose the advice thread, which is a series of documents that contain students' answers to common questions about dorm/college life.

---

# Unit 1

## What This Does

I chose the advice threads corpus, and I asked questions about college/dorm life that freshmen would ask. The system reads through the threads and provides an answer to the question.


## Chunking Strategy

**Chunk size:** N/A — chunks are variable-length, one per forum reply (not a fixed character/token count)
**Overlap:** None — replies are disjoint by construction, so overlap would just duplicate whole replies rather than smoothing a cut

Reading the documents in the advice_thread, the fixed-size character window from `fallback_split` (chunk_size=800-ish, with overlap) was
clearly wrong for this corpus because these aren't long texts, they're short forum threads, averaging ~88 characters per reply, structured
as a title followed by `--- reply N (votes) ---` blocks. A fixed window either swallowed 5-10replies into one chunk (losing the one
sentence-per-idea structure) or, on a bigger document, cut a reply in half mid-sentence and produce a meaningless fragment paired
arbitrarily with the tail of the next one.

The natural unit here is the reply itself, not a byte count: each reply is
already a single, self-contained answer, and the `--- reply N (votes) ---`
marker is an explicit, reliable boundary the author (well, the platform)
already drew for us. So `split_documents` splits on that marker instead of
size, producing one chunk per reply, with the thread's title prepended to
each chunk so a reply like "Yes, but stack your courses" doesn't lose the
question it's answering. Overlap has no role here: since chunks track a
real semantic unit (one reply) rather than an arbitrary window, there's no
cut edge to smooth over — overlap would only add noise (duplicate reply
text) without adding safety.

## Sample Chunks

---
Chunk 1  |  source: thread_bike_commute.txt#0  |  produced by: chunker.py::split_documents

THREAD: Is a bike worth it for a 20 minute walk commute?
Yeah. Cuts an 18 minute walk to about 6. The thing nobody mentions is storage — covered bike parking exists at three buildings and is full by 9am at all three.

---

Chunk 2  |  source: thread_first_gen.txt#1  |  produced by: chunker.py::split_documents

THREAD: Anything specific for first-generation students?
The thing I'd say: the unwritten rules are the hard part, not the coursework. Ask about the unwritten rules explicitly. People are happy to explain them and nobody volunteers them.

---

Chunk 3  |  source: thread_laptop_specs.txt#2  |  produced by: chunker.py::split_documents

THREAD: How much laptop do I actually need for CS courses?
I did two years on an 8GB machine and it was fine until the last project, at which point it very much wasn't. 16 is the answer.

---
Chunk 4  |  source: thread_parking.txt#1  |  produced by: chunker.py::split_documents

THREAD: Worth getting a parking permit?
Street parking on Verrill is legal and free and unmarked, which is why half the upper years do it.

---
Chunk 5  |  source: thread_sleep_schedule.txt#1  |  produced by: chunker.py::split_documents

THREAD: Everyone says fix your sleep. Does it actually matter?
The library being open until 2am is a trap. It's a resource, not a schedule.

---
## Sample Answer

**Question:**
python app.py ask  "Do transfer credits actually count toward general requierements?"
  (best distance 0.343, cutoff 0.6)

**Answer:**
Yes, transfer credits count toward general requirements almost always. 

Source: `thread_transfer_credits.txt`

Sources retrieved: thread_bike_commute.txt, thread_changing_major.txt, thread_pass_fail.txt, thread_textbook_editions.txt, thread_transfer_credits.txt

1 model calls this session, 788 tokens (764 in, 24 out)


**My relevance cutoff:**




| Question  | In corpus? | Best distance |
|---|---|---|
|Are there any resources for first-generation students?" | Yes | 0.421 |
|When is laundry actually free in the dorms?" | Yes|  0.328 |
|Do transfer credits actually count toward general requierements | Yes | 0.343|
|Are there group study rooms available on campus? | Yes | 0.409|
|Are there free parking spots on campus? | Yes | 0.538|
|What is the capital of Japan? | No | 0.904|
|Should I attend this college? | No | 0.636|
|What is the national animal of Canada? | No | 0.828|
|"Who won the 1994 World Cup?" | No | 0.787|

## How I Used AI


**1.**
I used Claude to write the chunking function explaining how the documents were organized and the general structure. It was separating the chunks by /n, /s, which was not the case so I reexplained how the documents were formatted.

**2.**
I asked why the code gernerated replaced
return fallback_split(documents) for  return chunks 

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

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
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5  | 5/5  | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 |  5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  5/5 | 5/5  | 5/5  | MET |
| 4. All chunks contain a concrete answer, and an explanation. | | 4/5  | 4/5 | 4/5 | MET |
| 5. The answer clearly states whether the answer is a fact or a student's opinion. | 5/5 | 0/5 | 0/5| 0/5  | MISSED |

Street parking on Verrill is legal, free, and unmarked. 
Source: thread_parking.txt

Yes, there is an advising office with a specific programme for first-generation students that is opt-in and badly publicised, so you must ask for it by name. Additionally, there is an emergency fund for textbooks and travel that requires only a short form and is not means-tested. (Source: thread_first_gen.txt)

Laundry is actually free in the dorms on Tuesday and Wednesday mornings in every building (thread_laundry_timing.txt).

Yes, transfer credits count toward general requirements almost always. 
Source: `thread_transfer_credits.txt`

Yes, the library has group study rooms that can be booked by one person and used alone, and nobody checks (thread_study_spots.txt).



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
| 4. All chunks contain a concrete answer, and an explanation. | | | | | |
| 5. The answer clearly states whether the answer is a fact or a student's opinion. | | | | | |

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
