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

# Capstone

## What this Extends 
In Unit 9, I will be extending the RAG system to add a measuring and logging feature.

## The Shape
I will be using shape A. I will be adding a structured JSON file that logs each query and includes timestamp, retrieval time, INFO or ERROR, and whether the questions was refused or not.

Out of scope: I am not building a dashboard, or changing any parameter of my system. I am not adding any features, changing or modifying chunking or retrieval.

## Running It
To run this script you will need Python 3.13. Then you will need to clone the repo. Then you will run
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt

This will install all the dependendencies, activate the virtual environment and make sure that when you run python test.py all tests clear. Then you will open another terminal. In the first terminal you will use:

curl -X POST http://localhost:5000/ask \
    -H 'Content-Type: application/json' \
    -d '{"question": "  YOUR QUESTION HERE?"}'

In the other terminal you will use:

python serve.py

## Instrumentation

2026-11-17 13:30:01,632 - INFO - 127.0.0.1 - - [17/Nov/2026 13:30:01] "POST /ask HTTP/1.1" 200 -
2026-11-17 13:36:52,014 - INFO - 127.0.0.1 - - [17/Nov/2026 13:36:52] "POST /ask HTTP/1.1" 200 -
2026-11-17 13:37:10,788 - INFO - 127.0.0.1 - - [17/Nov/2026 13:37:10] "POST /ask HTTP/1.1" 200 -
2026-11-17 13:37:24,423 - INFO - 127.0.0.1 - - [17/Nov/2026 13:37:24] "POST /ask HTTP/1.1" 200 -
2026-11-17 13:37:52,353 - INFO - 127.0.0.1 - - [17/Nov/2026 13:37:52] "[31m[1mPOST /ask HTTP/1.1[0m" 400 -
2026-11-17 13:38:19,511 - INFO - 127.0.0.1 - - [17/Nov/2026 13:38:19] "[31m[1mPOST /ask HTTP/1.1[0m" 400 -
2026-11-17 13:40:51,857 - ERROR - Rejected (400): empty question, after 0.0012 seconds
2026-11-17 13:40:51,858 - INFO - 127.0.0.1 - - [17/Nov/2026 13:40:51] "[31m[1mPOST /ask HTTP/1.1[0m" 400 -
2026-11-17 13:42:17,803 - INFO - 127.0.0.1 - - [17/Nov/2026 13:42:17] "POST /ask HTTP/1.1" 200 -
2026-11-18 13:16:00,070 - INFO - 127.0.0.1 - - [18/Nov/2026 13:16:00] "POST /ask HTTP/1.1" 200 -

## What It Cost
## What It Cost

**Per run: ~1.3 s** (mean of 6 runs; range 0.18–2.45 s).

**How it was measured:** Used the logs "Task completed in X seconds" line, across 6 runs on Nov 17–18, 2026. No warm-up runs were excluded.

**At 100 runs a day:** 100 × 1.3 s ≈ 132 s, about 2.2 minutes of
compute per day (18 s to 4 min across the observed range).

**Caveat:** timings cluster into two groups (~0.2 s and ~2.3 s). The fast ones are likely cached or warm runs, so I report the mean rather than the best case. Six runs is a small sample.

2026-11-17 13:30:01,631 - INFO - Task completed in 2.2910 seconds
2026-11-17 13:36:52,013 - INFO - Task completed in 0.5179 seconds
2026-11-17 13:37:10,787 - INFO - Task completed in 0.1837 seconds
2026-11-17 13:42:17,802 - INFO - Task completed in 2.2998 seconds
2026-11-18 13:16:00,066 - INFO - Task completed in 2.4503 seconds
2026-11-17 13:37:24,423 - INFO - Task completed in 0.1848 seconds




# Run Log - Before
| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. | Every log gets recorded | 5/5 | 5/5 | 5/5 | PASS |
| 2. | The requests takes less than 5s.| 4/5 | 4/5 | 5/5 | 4/5 | PASS |
| 3. | When the question is refused, the log records an error.| 2/5 | 1/5 | 0/5 | MISSED |
| 4. | When the question is empty, the log records an error. | 5/5 | 0/5 | 5/5 | MISSED | #if there is only a question mark, it won't mark it as empty
| 5. | The system works everytime a question inside the corpora is asked. | 4/5 | 4/5 | 4/5 | MET |

## Output
1. 
     ``` curl -X POST http://localhost:5000/ask \
     -H 'Content-Type: application/json' \
     -d '{"question": "What is the weather?"}' 

     "answer": "I don't have enough information about that.",
     "best_distance": 0.8546,
     "corpus": "advice_threads",
     "elapsed_time": 2.4788,
     "question": "?",
     "refused": true,
     "sources": [],
     "threshold": 0.5 ```

     2026-11-19 13:25:08,639 - INFO - Task completed in 2.4788 seconds //
     2026-11-19 13:25:08,640 - INFO - 127.0.0.1 - - [19/Nov/2026 13:25:08] "POST /ask HTTP/1.1" 200 -

2. 
All request took less than 5s 
```
2026-11-17 13:30:01,631 - INFO - Task completed in 2.2910 seconds
2026-11-17 13:36:52,013 - INFO - Task completed in 0.5179 seconds
2026-11-17 13:37:10,787 - INFO - Task completed in 0.1837 seconds
2026-11-17 13:42:17,802 - INFO - Task completed in 2.2998 seconds
2026-11-18 13:16:00,066 - INFO - Task completed in 2.4503 seconds
2026-11-17 13:37:24,423 - INFO - Task completed in 0.1848 seconds
```
3. 

     ```-H 'Content-Type: application/json' \
     -d '{"question": "What is the capital of Italy?"}'
     {
     "answer": "I don't have enough information about that.",
     "best_distance": 0.8835,
     "corpus": "advice_threads",
     "elapsed_time": 0.3295,
     "question": "What is the capital of Italy?",
     "refused": true,
     "sources": [],
     "threshold": 0.5
     }

     Not recorded as an error
     2026-11-19 14:44:54,193 - INFO - Task completed in 0.3295 seconds
     2026-11-19 14:44:54,194 - INFO - 127.0.0.1 - - [19/Nov/2026 14:44:54] "POST /ask HTTP/1.1" 200 -
     ```
4. ``` 
     -H 'Content-Type: application/json' \
     -d '{"question": ""}'                             
     {
     "error": "Send JSON with a question in it, like {\"question\": \"is the housing lottery random?\"}"
     }

     2026-11-19 14:46:26,674 - ERROR - Rejected (400): empty question, after 0.0010 seconds
     2026-11-19 14:46:26,675 - INFO - 127.0.0.1 - - [19/Nov/2026 14:46:26] "[31m[1mPOST /ask HTTP/1.1[0m" 400 -
```

5. 
```
 -H 'Content-Type: application/json' \
    -d '{"question": "should i sleep?"}'
{
  "answer": "I don't have enough information about that.",
  "best_distance": 0.5058,
  "corpus": "advice_threads",
  "elapsed_time": 0.3756,
  "question": "should i sleep?",
  "refused": true,
  "sources": [],
  "threshold": 0.5
} ```



## Diagnosis

The following criteria missed.
| 3. | When the question is refused, the log records an error.
| 4. | When the question is empty, the log records an error. | #if there is only a question mark, it won't mark it as empty

## 3. When the question is refused, the log records the refusal.
For 5/5 refused requests, a log entry is created with the gate's decision and the reason for the refusal, but it is not labeled as "ERROR",

**What I got wrong:** The criterion expected an error to be logged. The stage is the gate, and the mechanism is that a refusal is the gate working correctly: it rejected an input that was out of scope, and nothing in the system failed. My criterion treated a user-input outcome as a system fault. Logging refusals as errors would also bury real failures in noise. I changed the target to record the refusal as an ordinary, labeled log entry (for example, level INFO with a "refused" status and a reason) and kept 5/5, since this is still deterministic logging.

## 4. When the input is empty, the log records that it was empty.
For 5/5 empty requests, a log entry is created explaining that no question was provided.

**What I got wrong:** The criterion assumed "empty" meant no characters. The stage is input validation, and the mechanism is that it checks for a question rather than for content. A request containing only a question mark still counts as a question, so it passes the empty check and moves on to the gate. The criterion never fired because the input I called empty was not empty by the system's definition. I changed the target to match that definition: only truly blank input (no characters or only whitespace) is logged as empty. A bare "?" is treated as a question and handled by the gate like any other.

## The Improvement

I have edited criterion 3 so a refused question does not get labeled as an error.
For criterion 4, I have made edits to serve.py so that "?" is considered an empty question.

## What's Still Broken 

I still haven't added a token count.

## How I Used AI

I used AI to review my criteria, and assist in the creation of the logging file. 