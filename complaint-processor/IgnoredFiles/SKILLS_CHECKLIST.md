# Evaluation checklist — where each skill is demonstrated

The twelve skills the project is assessed on, and exactly where to find each one.

| # | Skill | Where it is | What to point at |
|---|---|---|---|
| 1 | **Python** | all 6 files + `test_basic.py` | Type hints on every function, docstrings, f-strings, `pathlib`, list comprehensions, context managers, `@lru_cache`. |
| 2 | **LLM API integration** | `ai_tasks.py` → `build_llm()` | `ChatOpenAI` from `langchain-openai`, configured from `config.py`, key read from `.env` and never hard-coded. |
| 3 | **Prompt Engineering** | `ai_tasks.py` → the three `ChatPromptTemplate`s | System/human message split; a shared `NO_MAKING_THINGS_UP` rule; task-specific rules (word counts, tone, "do not promise a refund"); low `temperature` for factual work. |
| 4 | **Structured Outputs** | `ai_tasks.py` → `.with_structured_output(...)` | The model must return the exact fields of a Pydantic class. No text parsing anywhere in the project. |
| 5 | **Pydantic** | `models.py` | Three `BaseModel` classes; `Field(description=...)` doubles as the instruction to the model; `Literal[...]` restricts categories and statuses to a fixed list; `Optional[...]` for facts that may be absent. |
| 6 | **Batch Processing** | `app.py` → `main()` | Discovers every document in `data/` and processes them in one run; each result saved individually plus one consolidated `final_report.csv`. |
| 7 | **Sequential / Parallel Workflow** | `workflow.py` → `build_workflow()` | LangGraph. Extraction runs **first** (sequential dependency). The email and summary have two edges out of `extract`, so LangGraph runs them **in parallel**. The diagram is in the comment at the top of the file. |
| 8 | **Error Handling** | `ai_tasks.py` → `call_with_retry()`, `document_reader.py`, `app.py` | Three layers: retry with exponential backoff on API failures; validation on unreadable/empty/oversized files; a `try/except` per document so one bad file never stops the batch — and failed documents still get a CSV row with the reason. |
| 9 | **Modular Code** | the file layout | Six files, one job each: settings, schemas, file reading, AI tasks, orchestration, runner. `app.py` contains no prompts and `ai_tasks.py` contains no file handling. |
| 10 | **File Processing** | `document_reader.py` | Three formats (`.txt`, `.pdf`, `.docx`) with a separate function for each; unsupported types skipped; output written as JSON, TXT and CSV. |
| 11 | **Git/GitHub** | `.gitignore`, `GIT_SETUP.md`, `.github/workflows/tests.yml` | `.env` and `output/` excluded so no API key is ever committed; a commit-by-commit setup guide; GitHub Actions runs the tests on every push. |
| 12 | **Basic logging** | `app.py` → `logging.basicConfig` + `start_log_file()` | Timestamped `INFO`/`WARNING`/`ERROR` messages for every step, printed to the screen **and** saved to `output/run.log`. |

---

## Extra credit, if it comes up

- **Testing** — `test_basic.py`, 15 tests, no API key needed, runs in under a second.
- **Continuous integration** — `.github/workflows/tests.yml` runs those tests on every push.
- **Cost control** — long documents are truncated (`config.MAX_CHARACTERS`), and the model is
  the cheap `gpt-4o-mini` by default. About $0.03 for the whole sample batch.
- **Security** — the API key lives in `.env`, which is git-ignored. Nothing sensitive is
  written into the code or the outputs.

---

## Three-sentence summary if you are asked to describe the project

> It processes a folder of customer complaint documents in PDF, Word and text format. For each
> one it uses an LLM to extract structured case data, write a reply to the customer, and write
> an internal summary for the manager — with the extraction step first and the two writing
> steps running in parallel through LangGraph. Every result is saved as a file and the whole
> batch is consolidated into one CSV report.
