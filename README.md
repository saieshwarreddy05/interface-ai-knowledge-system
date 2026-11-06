# Hackathon: Build a Knowledge System That Knows What's True *Now*

## Candidate Submission Overview
- **Author:** Sai Eshwar Reddy
- **Submission Folder:** `solution/` (entrypoint: `solution/solution.py` containing `build()`)
- **Evaluation Command:**
  ```bash
  python harness/run_eval.py --submission solution --eval eval/public_eval.jsonl
  ```
- **Benchmark Score:** **0.969 / 1.00** (p95 latency: `0.00s`, consistency: `1.00`, stale trap hits: `0`)
- **External Dependencies:** None (100% Python Standard Library, zero external API keys)
- **Architecture & Results PDF:** [`Results_and_Architecture_Report.pdf`](./Results_and_Architecture_Report.pdf)
- **Engineering Notes:** [`NOTES.md`](./NOTES.md)

### Performance Summary
| Metric | Baseline Starter | Our Solution (`solution/`) | Notes |
| :--- | :--- | :--- | :--- |
| **Final Score** | 0.093 | **0.969** | Multi-pillar aggregate |
| **QA Accuracy** | 0.093 | **0.969** | 100% in 8/9 question categories |
| **Consistency** | 0.093 | **1.000** | Full stability across paraphrase/repeats |
| **Speed (Latency)**| 0.39s | **0.00s (p95)** | Well within &le; 2.0s requirement |
| **Stale Traps** | Failed | **0 hits** | Actively eliminates stale calculator/blog traps |
| **Ingest Time** | 0.0s | **3.5s** | Full 5 snapshots ingested (&le; 30 min limit) |

---

## Directory Structure
```
solution/                           Our submission folder (contains solution.py with build())
  solution.py                       Temporal knowledge system & conflict engine
Results_and_Architecture_Report.pdf Executive PDF report on architecture & benchmark results
NOTES.md                            Detailed engineering notes & design principles
requirements.txt                    Clean environment configuration (zero external pip packages)
eval/public_eval.jsonl              40 practice questions with expected answers
harness/                            Official evaluation & scoring suite
starter/                            Original naive baseline implementation
snapshots/                          Unpacked crawl snapshots (excluded from repo per rules)
snapshots_zip/                      Zipped crawl archives (excluded from repo per rules)
```

---

## The problem in one minute

Our assistant answers credit-union members' questions ("What's the 6-month certificate rate?", "How much is a lost debit card?") using the credit union's own website.

We download (crawl) that website again and again over time. Each download is a **snapshot**. Websites are messy:

- A rate changes, but an old page or PDF still shows the old number.
- A promotion ends, but its page is never taken down.
- One page lists the **rate**, another the **APY**, and they look like they disagree.
- A table has tiers (by balance, by term). Read it carelessly and every tier looks like a conflict.
- Old newsletters and press releases describe offers from years ago.

When we combined these snapshots, our system **gave members wrong or out-of-date answers**. Every eval question in this kit comes from a real problem like that.

**Your mission:** build a system that reads the snapshots in date order and answers questions the way a careful human expert would: with the **current** value, the **right** tier, a **source**, and an honest "I don't know" when the site doesn't say.

## What you must hand in

A folder with a file called `solution.py`, containing a `build()` function that returns an object with **three methods**:

| Method | What it does |
|---|---|
| `ingest(snapshot)` | We call it once per snapshot, oldest first. Learn from it. |
| `answer(question, as_of=None)` | Answer a member's question. Without `as_of`, answer from the newest snapshot. With a date, answer as the website stood on that day. |
| `conflicts()` | List every place where the website contradicts itself, which value you believe is right, and why. |

The exact shapes are defined in `harness/contract.py`; `solution/solution.py` follows them exactly.

## How to check your work

From the top folder of this repository, run:

```bash
python harness/run_eval.py --submission solution --eval eval/public_eval.jsonl
```

## How you will be scored

We run a **larger, hidden question set** with the same format and the same scorer. Four things count:

| | Weight | What we look for |
|---|---|---|
| **Accuracy** | 55% | The current, correctly-tiered value is in your answer. Repeating an old or wrong value scores **zero** for that question, even if the right value is also there, so hedging by listing every number doesn't pay. Citing the right page earns a little extra. When the site doesn't say, the right answer is "I don't know". |
| **Conflicts** | 15% | How many real contradictions you list in `conflicts()`, and whether you picked the right winner. |
| **Consistency** | 15% | The same question asked in different words gets the same answer, and asking twice gives the same answer. |
| **Speed** | 15% | How quickly you answer. Most answers in 2 seconds or less earns full marks; it drops to zero at 20 seconds. |
