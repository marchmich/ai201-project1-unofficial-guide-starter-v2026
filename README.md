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
| 4. All chunks contain a concrete answer, and an explanation. | 4/5 | 3/5  | 3/5 | 3/5 | MISSED |
| 5. The answer clearly states whether the answer is a fact or a student's opinion. | 5/5 | 0/5 | 0/5| 0/5  | MISSED |

Street parking on Verrill is legal, free, and unmarked. 
Source: thread_parking.txt

Yes, there is an advising office with a specific programme for first-generation students that is opt-in and badly publicised, so you must ask for it by name. Additionally, there is an emergency fund for textbooks and travel that requires only a short form and is not means-tested. (Source: thread_first_gen.txt)

Laundry is actually free in the dorms on Tuesday and Wednesday mornings in every building (thread_laundry_timing.txt).

Yes, transfer credits count toward general requirements almost always. 
Source: `thread_transfer_credits.txt`

Yes, the library has group study rooms that can be booked by one person and used alone, and nobody checks (thread_study_spots.txt).



## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer |  MET |  All my responses contained an answer to my question|
| 2 |  Every answer names a source  | MET | Every answer contained the source file from where it obtained the information |
| 3 | Gate stops out-of-corpus questions | MET  | All the out-of-corpus questions were refused at the gate  |
| 4 | All chunks contain a concrete answer, and an explanation. | MISSED | Answers to question 1 did not include "Yes,..." in the response. |
| 5 | The answer clearly states whether the answer is a fact or a student's opinion. | MISSED | None of the questions answered if this was a students opinion's or facts. |

## Diagnoses

Criteron 4 missed because the answers to question 1 did not explicitly include "yes". I believe the mistake is in the generation-stage because maybe the answer doesn't explicitly say "Yes" so when the LLM paraphrases it, it does not include the concrete wording I am looking for.

Criteron 5 missed because the answers don't state whether it is a fact or an opinion. I believe the mistake is in the generation stage because the generation prompt does not explicitly ask for it.

## The Improvement

**What I changed:**
I added a grounding instruction to fix criteron 5. At first, I added "Clearly state when the information is an opinion or an opinion", but I realized this was changing all my answers in a way that I did not want so I edited to "When the information is a student's opinion and not a fact, clearly state that it is a student's opinion."

**Why I picked it:**

I picked to add another grounding instruction because this would fix the problem since the issue was in the generation stage.

### Run Log — After

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5  | 5/5 | PASS |
| 2. Every answer names a source | 5 of 5 |  5/5 | 5/5  | 5/5 | PASS |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  5/5 | 5/5  | 5/5 | PASS |
| 4. All chunks contain a concrete answer, and an explanation. | 4/5 | 2/5 | 4/5 | 3/5 | MISSED |
| 5. The answer clearly states whether the answer is a fact or a student's opinion. | 5/5 | 0/5 | 0/5| 0/5  | MISSED |

**Did it help?**

It helped for criteron 4, even though the changes were intended for criteron 5. However, this made me realize that the criteron 5 could have been written differently, or the questions could have been more specific. Criteron 4 improved, and now more questions start with "Yes,...". 

## What's Still Broken

Criteron 5 is still broken. I need to rewrite it to be able to holistically evaluate my model and understand what's not working. Additionally, I could also enhance my questions so that my model can answer whether this is student advice or a fact.

## What I'd Do Differently

I would formulate my questions better and re-edit the grounding instructions to fit my criteria specifically.
