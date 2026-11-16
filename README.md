<div align="center">

<h1>🔎 Job Hunt</h1>

<h3><em>Find the right roles. Make every application your own.</em></h3>

<p>An open-source job-search assistant that finds relevant roles, prepares a
personalized digest, and helps you get ready to apply.</p>

<br />

<a href="https://www.python.org/"><img src="https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python 3.10+"></a>
<a href="https://www.greenhouse.com/"><img src="https://img.shields.io/badge/Greenhouse-jobs-16A34A?style=for-the-badge" alt="Greenhouse"></a>
<a href="https://www.lever.co/"><img src="https://img.shields.io/badge/Lever-jobs-0E7490?style=for-the-badge" alt="Lever"></a>
<a href="https://www.ashbyhq.com/"><img src="https://img.shields.io/badge/Ashby-jobs-7C3AED?style=for-the-badge" alt="Ashby"></a>
<a href="https://github.com/dipcoderr/job-hunt/actions"><img src="https://img.shields.io/badge/GitHub_Actions-scheduled-2088FF?style=for-the-badge&logo=githubactions&logoColor=white" alt="GitHub Actions"></a>
<a href="tests/"><img src="https://img.shields.io/badge/tests-63_passing-22C55E?style=for-the-badge&logo=pytest&logoColor=white" alt="63 tests passing"></a>

<br />

<p><a href="#-features">✨ Features</a> · <a href="#-quick-start">⚡ Quick start</a> · <a href="#️-configure-your-search">🛠️ Configure</a> · <a href="#-automate-your-digest">⏰ Automation</a></p>

<br />

<p><strong>Your search, your call:</strong> this tool finds, filters, scores, and drafts.<br />
<strong>It never submits an application.</strong></p>

</div>

---

## ✨ Features

| 👤 Your search | 🤖 Your workflow |
|---|---|
| 🌐 Find public roles on Greenhouse, Lever, and Ashby | 🎯 Filter by title, location, and posting age before AI screening |
| 🧭 Score jobs against a profile built from your resume | ✍️ Prepare tailored bullets, a fit summary, gaps, and a draft cover note |
| 📬 Get a clean HTML digest of selected jobs | 📊 Track applications and export a CSV |
| 🔁 Skip postings you have already seen | 🧪 Test the full pipeline offline—no API key needed |

## ⚡ How it works

```text
🌐 Public job boards
        ↓
🧹 Title · location · freshness filters
        ↓
🧠 Profile-based screening
        ↓
✍️ Draft kits for selected roles
        ↓
📬 HTML digest + 📊 application tracker
```

The prefilter is deterministic and runs before AI screening, helping reduce
unnecessary model calls. Results depend on the boards and filters you choose.
You review every draft and submit applications yourself.

## 🏁 Quick start

### 📋 Requirements

- Python 3.10 or later
- `pip`
- An AI provider key for profile generation and AI-powered screening
- Optional: SMTP credentials to email the digest

New to Python? Follow the [step-by-step setup guide](SETUP.md).

### 📥 Install

```bash
git clone https://github.com/dipcoderr/job-hunt.git
cd job-hunt

python -m venv .venv
# macOS/Linux:
source .venv/bin/activate
# Windows PowerShell:
# .venv\Scripts\Activate.ps1

python -m pip install -r requirements.txt
```

### 🎮 Run the offline demo

No API key or network access required:

```bash
python -m jobhunt run --mock --scorer keyword
```

Mock Greenhouse, Lever, and Ashby listings go through the real parsers. The
keyword scorer is **for development only**—it is not an AI assessment and
should not be used to judge real applications. Find the generated digest at
`out/digest.html`.

Run the tests:

```bash
python -m pytest tests -q
```

## 🛠️ Configure your search

### 🏢 Choose job boards

Add boards to [`companies.yaml`](companies.yaml). Use the company’s public
careers-board slug:

```yaml
companies:
  - {ats: greenhouse, slug: example-company, name: Example Company}
  - {ats: lever, slug: another-company, name: Another Company}
  - {ats: ashby, slug: third-company, name: Third Company}
```

| Board | Public jobs endpoint |
|---|---|
| 🟢 Greenhouse | `https://boards-api.greenhouse.io/v1/boards/{slug}/jobs?content=true` |
| 🔵 Lever | `https://api.lever.co/v0/postings/{slug}?mode=json` |
| 🟣 Ashby | `https://api.ashbyhq.com/posting-api/job-board/{slug}?includeCompensation=true` |

Check that each board is active before relying on its results—companies can
change providers or slugs. The parsers normalize jobs into a shared format,
skip unpublished Ashby drafts, and create unique IDs for deduplication.

### 🎚️ Tune your filters

[`config.yaml`](config.yaml) controls the prefilter, screening, drafting, and
output paths. Customize the title patterns and locations for your search:

```yaml
filters:
  include_titles: [...]
  exclude_titles: [...]
  locations: [...]
  allow_remote: true
  max_age_days: 30

score_threshold: 4.0
max_per_digest: 5
```

The included company list and filters are examples. Review them before a real
run, and use `--limit` while tuning to cap provider usage.

### 🪪 Create your candidate profile

Build a profile from a PDF, TXT, or Markdown resume:

```bash
python -m jobhunt profile --resume path/to/resume.pdf
```

This writes `profile.json` by default. Review and correct the generated profile
before using it. Your resume and profile are private, gitignored files—**never
commit them**.

### 🧠 Pick an AI provider

Set a provider and its key in your local environment (for example, in `.env`):

```dotenv
LLM_PROVIDER=gemini
GEMINI_API_KEY=your-key
```

Supported providers: `anthropic`, `gemini`, `groq`, `openai-compatible`, and
`ollama`. Use `SCREEN_PROVIDER` and `DRAFT_PROVIDER` to choose providers for
each stage, and `SCREEN_MODEL` and `DRAFT_MODEL` to set model IDs. See
[SETUP.md](SETUP.md) for provider-specific setup.

## 🚀 Run it

```bash
# Build the digest without sending an email
python -m jobhunt run

# Cap provider usage while testing
python -m jobhunt run --limit 10

# Screen jobs without generating draft kits
python -m jobhunt run --no-draft

# Send the digest using configured SMTP
python -m jobhunt run --send
```

> **Human in the loop:** review every generated claim for accuracy. The tool
> helps you prepare; it never submits applications.

## 📊 Track your applications

`seen.json` keeps track of jobs already surfaced and application status. It is
private and gitignored, so every new user starts with an empty tracker.

```bash
python -m jobhunt applied "greenhouse:example-company:12345"
python -m jobhunt stats
```

The stats command summarizes your tracker and exports `out/tracker.csv` for
your favorite spreadsheet app.

## ⏰ Automate your digest

The [daily GitHub Actions workflow](.github/workflows/daily.yml) is scheduled
for weekdays at **06:00 IST**. GitHub scheduling is best-effort, so a run may
start a little later.

### 🔐 Set up repository secrets

1. Add your reviewed profile as the `PROFILE_JSON` Actions secret.
2. Add the AI provider key you use: `GEMINI_API_KEY`, `ANTHROPIC_API_KEY`, or
   `GROQ_API_KEY`.
3. To email the digest, add `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`,
   and `MAIL_TO`.
4. Optionally add provider/model selections as Actions variables:
   `LLM_PROVIDER`, `SCREEN_PROVIDER`, `DRAFT_PROVIDER`, `SCREEN_MODEL`, and
   `DRAFT_MODEL`.
5. Enable Actions, then run **daily job digest** manually with `dry_run`
   selected to test without sending email.

The workflow caches the deduplication store and uploads digest files as a
short-lived artifact. It does **not** commit personal tracking data to the
repository.

## 🗂️ Project structure

```text
jobhunt/
  cli.py        Command-line interface and daily pipeline
  fetch.py      Shared job record and ATS fetchers/parsers
  prefilter.py  Title, location, and freshness filters
  providers.py  AI provider implementations
  llm.py        Screening, drafting, and profile generation
  digest.py     HTML digest rendering
  mailer.py     SMTP delivery
  store.py      Deduplication and application tracking
  mock.py       Offline fixtures
config.yaml     Filters, model limits, and output paths
companies.yaml  Job boards to poll
tests/          Parser and pipeline tests
```

## 🔒 Privacy & safety

- Never commit `.env`, API keys, SMTP passwords, your resume, or `profile.json`.
- AI-generated drafts can be wrong. Check every detail before sharing them.
- Job descriptions and your profile are sent to the AI provider you configure;
  review that provider’s data policies before use.
- **Applications are never submitted automatically.** You stay in control.

## 🤝 Contributing

Issues and pull requests are welcome! When adding an ATS parser, keep HTTP
fetching separate from parsing and add fixture-based tests that run without
network access.
