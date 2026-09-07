# Putting the project on GitHub

The evaluation includes **Git/GitHub**, so the project needs to live in a real repository with
real commits. This takes about five minutes.

---

## Step 1 — Check Git is installed

Open a terminal (Command Prompt or PowerShell) **inside the project folder** and run:

```bash
git --version
```

If that fails, install Git from <https://git-scm.com/downloads> and reopen the terminal.

---

## Step 2 — Tell Git who you are (only needed once per computer)

```bash
git config --global user.name "Ashish Jain"
git config --global user.email "ashishjain3284@gmail.com"
```

---

## Step 3 — Create the repository

```bash
git init
git branch -M main
```

Before your first commit, **check that `.env` will not be committed**:

```bash
git status
```

`.env` must **not** appear in the list. It is excluded by `.gitignore`. If you ever see it
listed, stop and fix `.gitignore` first — committing an API key to GitHub means the key is
public and must be cancelled.

---

## Step 4 — Make commits

Rather than one giant commit, make a few small ones. This looks far better to an evaluator
because it shows how you actually work.

```bash
git add .gitignore requirements.txt .env.example
git commit -m "Add project setup and dependencies"

git add config.py models.py
git commit -m "Add configuration and Pydantic output schemas"

git add document_reader.py
git commit -m "Add document reader for txt, pdf and docx files"

git add ai_tasks.py
git commit -m "Add the three AI tasks with structured outputs"

git add workflow.py
git commit -m "Add LangGraph workflow with parallel generation steps"

git add app.py
git commit -m "Add batch runner, logging and CSV report"

git add test_basic.py .github/
git commit -m "Add tests and GitHub Actions workflow"

git add data/ README.md HOW_TO_DEMO.md GIT_SETUP.md SKILLS_CHECKLIST.md
git commit -m "Add sample documents and documentation"
```

Check the history:

```bash
git log --oneline
```

---

## Step 5 — Push to GitHub

1. Go to <https://github.com/new>
2. Repository name: `complaint-processor`
3. Choose **Public** (so the evaluator can open it)
4. Do **not** tick "Add a README" — you already have one
5. Click **Create repository**

Then copy the URL GitHub shows you and run:

```bash
git remote add origin https://github.com/YOUR-USERNAME/complaint-processor.git
git push -u origin main
```

Refresh the GitHub page. Your code is there, and under the **Actions** tab you will see the
tests running automatically.

---

## Step 6 — After any later change

```bash
git add .
git commit -m "Describe what you changed"
git push
```

---

## What to show the evaluator

- The repository page on GitHub, with the README displayed
- `git log --oneline` — the commit history
- The **Actions** tab with a green tick, proving the tests pass automatically
- `.gitignore` — and the point that `.env` and `output/` are deliberately excluded, so no API
  key and no generated files are ever committed

---

## If something goes wrong

**"fatal: not a git repository"** — you are in the wrong folder. `cd` into the folder
containing `app.py` and try again.

**Git asks for a password and rejects it** — GitHub no longer accepts account passwords. When
prompted, use a Personal Access Token instead: GitHub → Settings → Developer settings →
Personal access tokens → Tokens (classic) → Generate new token, tick `repo`, and paste the
token as the password.

**You committed `.env` by accident** — cancel that API key immediately at
<https://platform.openai.com/api-keys>, create a new one, and start the repository again with
a corrected `.gitignore`.
