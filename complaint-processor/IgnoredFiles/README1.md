# AI Customer Complaint & Case Processing System

A GenAI application that reads customer complaint documents from a folder and, for each
one, uses an LLM to do three things:

1. **Extract structured information** — customer name, email, phone, category, issue,
   resolution, complaint Yes/No, escalation Yes/No, supporting document Yes/No, case status
2. **Write a customer reply email** — professional, based only on what the document says
3. **Write an internal case summary** — for the manager: overview, key issue, action taken,
   status, recommended next action

It then writes everything to an `output/` folder plus one consolidated `final_report.csv`.

**Run the whole thing with one command:** `python app.py`

---

## 1. How to run it

**Step 1 — install Python packages** (once):

```bash
pip install -r requirements.txt
```

**Step 2 — add your OpenAI key.** Copy `.env.example`, rename the copy to `.env`, and put
your real key inside it:

```
OPENAI_API_KEY=sk-your-real-key-here
```

**Step 3 — run it:**

```bash
python app.py
```

That's it. It processes every document in `data/` and writes the results to `output/`.

Expected cost: about ₹2–3 (roughly $0.03) for all six sample documents with `gpt-4o-mini`.

---

## 2. What you will see

```
============================================================================
  AI CUSTOMER COMPLAINT & CASE PROCESSING SYSTEM
============================================================================
12:10:37 | INFO    | Found 6 document(s) in data
12:10:37 | INFO    | [1/6] Processing complaint_001.pdf
12:10:41 | INFO    |       done in 3.8s
12:10:41 | INFO    | [2/6] Processing complaint_002.pdf
...
12:11:02 | INFO    | Final report written to output/final_report.csv

DOCUMENT              STATUS    CATEGORY          ESCALATE  CASE STATUS   PRIORITY
complaint_001.pdf     success   Billing           No        Resolved      Low
complaint_002.pdf     success   Delivery          Yes       Escalated     High
complaint_003.txt     success   Warranty          Yes       Escalated     High
complaint_004.docx    success   Account Access    Yes       Escalated     High
complaint_005.docx    success   Service Quality   No        Resolved      Low
enquiry_006.txt       success   Other             No        Closed        Low

  6 succeeded, 0 failed
```

And the `output/` folder will contain:

```
output/
├── structured_data/        complaint_001.json   <- the extracted fields
├── customer_emails/        complaint_001.txt    <- the reply to the customer
├── case_summaries/         complaint_001.txt    <- the internal note
├── final_report.csv        <- one row per document, opens in Excel
└── run.log                 <- a timestamped record of the run
```

---

## 3. The files, and what each one does

There are only **six** Python files. Read them in this order:

| File | Lines | What it does |
|---|---|---|
| **`app.py`** | ~230 | **The file you run.** Loops over the documents, calls everything else, saves the results, writes the CSV. |
| `config.py` | ~45 | All settings in one place: folders, model name, file types. |
| `models.py` | ~140 | The three Pydantic classes that define what the AI must return. |
| `document_reader.py` | ~90 | Turns a `.txt`, `.pdf` or `.docx` file into plain text. |
| `ai_tasks.py` | ~165 | The three prompts, the three LLM calls, and the retry logic. |
| `workflow.py` | ~110 | Connects the three AI tasks together with LangGraph. |

Every file starts with a comment block explaining what it is for.

There is also `test_basic.py` — 15 quick tests you never have to read to run the project.
See [section 6](#6-running-the-tests).

---

## 4. How it works

```
   data/complaint_001.pdf
            |
            v
   document_reader.py          reads the file, gives back plain text
            |
            v
   ┌──────────────────────────────────────────┐
   │            workflow.py                   │
   │                                          │
   │        [ 1. extract ]                    │   ai_tasks.extract_complaint_data
   │              |                           │
   │        +-----+-----+   these two run     │
   │        v           v   at the SAME TIME  │
   │   [ 2. email ] [ 3. summary ]            │   ai_tasks.write_customer_email
   │                                          │   ai_tasks.write_case_summary
   └──────────────────────────────────────────┘
            |
            v
   app.py saves 3 files + adds a row to final_report.csv
```

### The one idea worth understanding

We never ask the LLM for free text and then try to parse it. Instead:

```python
chain = EXTRACTION_PROMPT | llm.with_structured_output(ComplaintData)
result = chain.invoke({"document_text": text})
# result is a ComplaintData object - already validated
```

`ComplaintData` is a Pydantic class in `models.py`. `with_structured_output` sends that class
to OpenAI as the required answer format, so the model has to return exactly those fields, and
LangChain validates the answer before we ever see it. That is why the CSV is always clean.

### Why LangGraph instead of just calling three functions

The email and the summary both need only the extracted data — neither needs the other. So they
are drawn as two branches out of `extract`, and LangGraph runs them **in parallel**. Each
document finishes noticeably faster, and the three steps stay separate and easy to change.

---

## 5. Error handling

Errors are handled at three levels, so nothing ever fails silently:

**The API call** — `ai_tasks.call_with_retry()` retries a failed OpenAI call up to three times,
waiting 2 seconds then 4 seconds in between ("exponential backoff"). This recovers from
dropped connections and rate limits automatically.

**The file** — a corrupt PDF, a scanned image with no text, or an almost-empty file is
rejected by `document_reader.py` with a clear message.

**The batch** — `app.py` wraps each document in a `try/except`, so one bad file is reported
and skipped while the rest keep processing. Failed documents still get a row in
`final_report.csv` with `status = failed` and the reason, so nothing disappears.

Plus: unsupported file types (like `vendor_courier_notes.csv` in `data/`) are ignored, and if
the API key is missing the program says so clearly and stops before spending anything.

---

## 6. Running the tests

```bash
pip install pytest
pytest -v
```

15 tests, under a second, **no API key needed and no cost** — they check the file reading, the
Pydantic validation and the CSV building, none of which call OpenAI.

These same tests run automatically on GitHub every time you push, via
`.github/workflows/tests.yml`.

---

## 7. Putting it on GitHub

See **[GIT_SETUP.md](GIT_SETUP.md)** for step-by-step commands.

The important part: `.gitignore` excludes `.env` and `output/`, so your API key and the
generated files are never committed.

---

## 8. Trying it with your own documents

Put any `.txt`, `.pdf` or `.docx` file into the `data/` folder and run `python app.py` again.
Nothing else needs to change.

To start from a clean slate, delete the `output/` folder — it is recreated on every run.

---

## 9. Skills demonstrated

| Skill | Where it is in the code |
|---|---|
| Python | all six files + the tests |
| LLM API integration | `ai_tasks.py` — `build_llm()` / `ChatOpenAI` |
| Prompt engineering | `ai_tasks.py` — three prompts with a shared anti-hallucination rule |
| Structured outputs | `ai_tasks.py` — `with_structured_output(...)` |
| Pydantic | `models.py` — three schemas, `Literal` for fixed categories |
| Batch processing | `app.py` — the loop over `data/` plus the consolidated CSV |
| Sequential / parallel workflow | `workflow.py` — LangGraph; extract first, then two parallel branches |
| Error handling | retry in `ai_tasks.py`, validation in `document_reader.py`, `try/except` in `app.py` |
| Modular code | six small files, each with one job |
| File processing | `document_reader.py` — txt, pdf, docx in, JSON/TXT/CSV out |
| Git/GitHub | `.gitignore`, `GIT_SETUP.md`, `.github/workflows/tests.yml` |
| Logging | `app.py` — timestamped messages on screen **and** in `output/run.log` |

**[SKILLS_CHECKLIST.md](SKILLS_CHECKLIST.md)** has the detailed version, with exactly what to
point at for each one.

## 10. Sample data

The six documents in `data/` are **fictional** — invented names, invented order numbers. They
cover a duplicate billing charge, a delayed delivery that got escalated, a repeat warranty
failure, an account-access/security case, a service-quality complaint that was resolved, and
one plain enquiry that is deliberately **not** a complaint (so you can see `is_complaint = No`
in the report).
