# Job Hunt

### Find relevant roles. Review every application yourself.

An open-source Python tool that checks public careers boards, filters and ranks
job postings against a candidate profile, then prepares a daily digest and
application drafts.

**It never submits applications.** You decide which roles to pursue, review
every generated draft, and apply yourself.

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-scheduled-2088FF?logo=githubactions&logoColor=white)](https://github.com/dipcoderr/job-hunt/actions)
[![Tests](https://img.shields.io/badge/tests-63%20passing-brightgreen)](tests/)

[Features](#features) · [Technology](#technology) · [Get started](#get-started) · [Configuration](#configuration) · [Scheduling](#scheduled-digests)

---

## Features

- **Collects public job listings** from Greenhouse, Lever, and Ashby boards.
- **Filters before calling an AI model** using configurable title, location,
  and posting-age rules.
- **Scores matching roles** against a profile derived from your resume.
- **Drafts an application kit** for selected jobs: a fit summary, tailored
  resume bullets, potential gaps, a cover note, and questions to ask.
- **Creates an HTML digest** and CSV application tracker.
- **Avoids repeat listings** with a local, private `seen.json` store.
- **Runs offline for development** using realistic fixtures and a clearly
  labeled keyword-only scoring stub.
- **Can send a scheduled weekday digest** using GitHub Actions and your own
  configured secrets.

## How it works

```text
Public job boards → title/location/age filters → profile-based screening
                                                       ↓
                                       selected roles and draft kits
                                                       ↓
                                       HTML digest + application tracker
```

The local prefilter is deterministic and does not use an LLM. This reduces the
number of postings sent for screening. The exact counts depend on the boards
and filters you configure.

## Technology

| Area | Tools |
|---|---|
| Language | Python 3.10+ |
| Job sources | Greenhouse, Lever, and Ashby public board APIs |
| Configuration | YAML (`config.yaml`, `companies.yaml`) |
| AI providers | Anthropic, Google Gemini, Groq, OpenAI-compatible APIs, and Ollama |
| Email | SMTP |
| Automation | GitHub Actions |
| Tests | pytest; parsers and pipeline tests use fixtures and stubs |

## Get started

### Requirements

- Python 3.10 or later
- `pip`
- An AI provider key for profile generation and AI-powered screening
- Optional: SMTP credentials if you want to send the digest by email

For detailed first-time setup, see the [step-by-step setup guide](SETUP.md).

### Install

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

### Try the offline demo

No API key or network access is needed for this run:

```bash
python -m jobhunt run --mock --scorer keyword
```

The mock mode sends bundled Greenhouse, Lever, and Ashby examples through the
normal parsers. The keyword scorer is intended only to test the pipeline; it is
not an AI assessment and should not be used to judge real applications. The
generated digest is written to `out/digest.html`.

Run the test suite with:

```bash
python -m pytest tests -q
```

## Configuration

### Choose the job boards

Edit [`companies.yaml`](companies.yaml) to add companies you want to follow.
Each entry identifies the board type, its public board slug, and a display
name:

```yaml
companies:
  - {ats: greenhouse, slug: example-company, name: Example Company}
  - {ats: lever, slug: another-company, name: Another Company}
  - {ats: ashby, slug: third-company, name: Third Company}
```

The slug is the last part of the company's board URL. Confirm each board is
active before relying on its results; companies can change providers or board
slugs.

| Provider | Public jobs endpoint |
|---|---|
| Greenhouse | `https://boards-api.greenhouse.io/v1/boards/{slug}/jobs?content=true` |
| Lever | `https://api.lever.co/v0/postings/{slug}?mode=json` |
| Ashby | `https://api.ashbyhq.com/posting-api/job-board/{slug}?includeCompensation=true` |

The parsers normalize these different formats into a shared job record.
Unlisted Ashby drafts are skipped, and each record gets a board-qualified ID
for deduplication.

### Tune the filters

[`config.yaml`](config.yaml) controls the deterministic prefilter, screening,
drafting, and output paths. Start by adjusting the title patterns and locations
to match the roles you want:

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

Use `--limit` while tuning to cap the number of jobs sent to an AI provider.
The shipped company list and filters are examples; review and personalize
them before a real run.

### Create a candidate profile

Provide a resume file in PDF, TXT, or Markdown format and run:

```bash
python -m jobhunt profile --resume path/to/resume.pdf
```

The command writes `profile.json` by default. Review the generated profile and
correct it before using it. The profile and resume are private, gitignored
files; do not commit them.

For AI features, configure the provider and corresponding key in your local
environment (for example in a local `.env` file):

```dotenv
LLM_PROVIDER=gemini
GEMINI_API_KEY=your-key
```

Available provider names include `anthropic`, `gemini`, `groq`,
`openai-compatible`, and `ollama`. Set `SCREEN_PROVIDER` or `DRAFT_PROVIDER`
to select a different provider for an individual stage; `SCREEN_MODEL` and
`DRAFT_MODEL` choose the model IDs. See [SETUP.md](SETUP.md) for provider
details and environment-specific instructions.

## Run the application

```bash
# Build the digest without sending email
python -m jobhunt run

# Limit provider usage while testing
python -m jobhunt run --limit 10

# Screen jobs without generating draft kits
python -m jobhunt run --no-draft

# Send the digest through configured SMTP
python -m jobhunt run --send
```

The tool only finds, ranks, and drafts. It does not submit applications.
Review every generated statement for accuracy before using it.

## Application tracking

`seen.json` records previously surfaced jobs and application status. It is
local-only and excluded from Git so each user's first run starts with an empty
tracker.

```bash
python -m jobhunt applied "greenhouse:example-company:12345"
python -m jobhunt stats
```

The stats command summarizes the tracker and exports `out/tracker.csv`, which
can be opened in a spreadsheet application.

## Scheduled digests

The [daily workflow](.github/workflows/daily.yml) is configured for weekdays at
06:00 India Standard Time. GitHub Actions scheduling is best-effort, so a run
may start later than its scheduled time.

To enable it:

1. Add your reviewed profile as the `PROFILE_JSON` Actions secret.
2. Add the AI provider key you use, such as `GEMINI_API_KEY`,
   `ANTHROPIC_API_KEY`, or `GROQ_API_KEY`.
3. To email the digest, configure `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`,
   `SMTP_PASS`, and `MAIL_TO` as Actions secrets.
4. Optionally add provider/model selections as Actions variables:
   `LLM_PROVIDER`, `SCREEN_PROVIDER`, `DRAFT_PROVIDER`, `SCREEN_MODEL`, and
   `DRAFT_MODEL`.
5. Enable Actions, then run **daily job digest** manually with `dry_run`
   selected to test without sending email.

The workflow preserves the deduplication store using the Actions cache and
uploads generated digest files as a short-lived artifact. It does not commit
personal tracking data back to the repository.

## Project layout

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

## Privacy and safety

- Never commit `.env`, API keys, SMTP passwords, your resume, or `profile.json`.
- Generated drafts may be wrong or misleading. Verify every detail before
  sharing them.
- Job descriptions and your profile are provided to whichever AI provider you
  configure. Review that provider's data policies before use.
- Applications are never submitted automatically; you remain in control.

## Contributing

Issues and pull requests are welcome. When adding an ATS parser, keep HTTP
fetching separate from parsing and add fixture-based tests so parser behavior
can be checked without network access.
