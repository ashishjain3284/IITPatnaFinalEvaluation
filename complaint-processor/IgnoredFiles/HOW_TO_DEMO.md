# How to demonstrate this project

A 5-minute walkthrough you can follow live. Each step has a sentence you can actually say.

---

## Before you start (do this once, not during the demo)

```bash
pip install -r requirements.txt
```

Copy `.env.example` to `.env` and put your real OpenAI key in it. Then do one practice run so
you know it works:

```bash
python app.py
```

Delete the `output/` folder afterwards so the demo starts clean.

---

## Step 1 — Show the problem (30 seconds)

Open the `data/` folder.

> "Here are six customer complaint documents. They are in three different formats — PDF,
> Word and plain text. Today somebody has to read each one, type the details into a system,
> write a reply to the customer, and write a summary for their manager. That is what I have
> automated."

Open `data/complaint_002.pdf` so they can see it is real prose, not a form.

---

## Step 2 — Show the structure (1 minute)

> "There are six Python files. `app.py` is the one you run. `config.py` holds the settings,
> `models.py` defines what the AI must return, `document_reader.py` reads the files,
> `ai_tasks.py` has the three AI tasks, and `workflow.py` connects them together."

Open `models.py` and point at `ComplaintData`:

> "This is the key idea. I don't ask the AI for text and then try to parse it. I give it this
> Pydantic class and it has to return exactly these fields. Every `description` line here is
> actually part of the instruction sent to the model."

Open `workflow.py` and point at the diagram in the comment at the top:

> "The extraction has to happen first. But the email and the summary both only need the
> extracted data, so LangGraph runs those two in parallel."

---

## Step 3 — Run it (2 minutes)

```bash
python app.py
```

While it runs:

> "It found six documents. It's processing them one at a time, and for each one it makes
> three AI calls — extract, write the email, write the summary."

When it finishes, point at the table:

> "Six documents processed. Look at the ESCALATE column — the system worked out on its own
> which cases need to be escalated. And look at the last row: that document is just an
> enquiry, not a complaint, so `is_complaint` is No."

---

## Step 4 — Show the output (1 minute)

Open `output/final_report.csv` in Excel:

> "One row per document. This is what the operations team would actually use — they can sort
> by escalation or by priority."

Open `output/customer_emails/complaint_002.txt`:

> "This is the reply the customer would receive. It's specific to their case, and the prompt
> forbids the model from inventing a refund amount or a delivery date that wasn't in the
> original document."

Open `output/case_summaries/complaint_002.txt`:

> "And this is the internal note for the manager, with a recommended next action."

Open `output/structured_data/complaint_002.json`:

> "And here is the raw structured data, ready to be pushed into a CRM."

---

## Step 5 — Show the error handling (30 seconds, optional but impressive)

Create an empty text file called `data/broken.txt`, then run `python app.py` again.

> "One bad file doesn't stop the batch. It's reported, it's recorded in the CSV as failed
> with the reason, and the other documents still get processed."

Delete `data/broken.txt` afterwards.

---

## Step 6 — Show the tests and the repository (1 minute)

```bash
pytest -v
```

> "Fifteen tests, and none of them call OpenAI — so they run in under a second and cost
> nothing. They check the file reading, the Pydantic validation and the report building."

Then open your GitHub repository:

> "Every push runs these tests automatically through GitHub Actions — that's the green tick.
> And `.gitignore` keeps the `.env` file out of the repository, so my API key is never
> committed."

Also worth showing: `git log --oneline`, and `output/run.log` — the timestamped record the
run leaves behind.

---

## Questions you are likely to be asked

**"What stops the AI making things up?"**
Three things. The prompts all contain an explicit rule to use only what is in the document.
The structured output means the model fills in fixed fields instead of writing free prose. And
the generation tasks are given the extracted data, not the raw document, so they can only work
from facts that already passed through extraction.

**"Why LangGraph and not just three function calls?"**
Because the email and the summary don't depend on each other, so they can run at the same time
— and because keeping the steps separate makes it easy to add a fourth task later without
touching the first three.

**"How much does it cost to run?"**
About $0.03 for all six documents on `gpt-4o-mini` — three small calls per document.

**"What happens if the OpenAI API fails halfway through?"**
`ai_tasks.call_with_retry()` retries the call three times with a growing pause — 2 seconds,
then 4. If all three fail, that document is marked as failed in the CSV with the reason, and
the batch carries on with the next one.

**"What would you add next?"**
OCR, so scanned PDFs work too. Automatic routing of escalated cases to the right team. And a
small web interface so a non-technical user can drop files in.

**"Can it handle 1,000 documents?"**
The loop already handles any number. For that volume I'd process several documents at once
using a thread pool, since the program spends nearly all of its time waiting for the API.
