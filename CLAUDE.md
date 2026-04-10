# CLAUDE.md — OpenTelemetry End-User SIG

This file provides guidance for AI assistants (Claude Code and similar tools) working in this repository.

## What This Repository Is

This is the **OpenTelemetry End-User Special Interest Group (SIG)** repository. It is a **community research and documentation repository**, not a software product. Its primary purpose is:

1. Collecting and analyzing end-user feedback about OpenTelemetry (surveys, interviews)
2. Managing YouTube video transcripts from the OTel community channel
3. Documenting processes and best practices for community engagement

Most content is Markdown documentation and data files. The only substantive code is a Python script for fetching YouTube transcripts.

## Repository Structure

```
sig-end-user/
├── end-user-surveys/           # Survey data, results, and analysis guides
│   ├── README.md               # Survey best practices and pipeline guidance
│   ├── README.template.md      # Template for new survey directories
│   ├── analysis-guides/        # Step-by-step data analysis tutorials (Google Sheets)
│   └── <survey-name>/          # One directory per survey (e.g., otel-collector/)
│       ├── README.md           # Survey metadata, purpose, resources
│       ├── *.csv               # Raw response data
│       └── *.pdf               # Published report
├── end-user-interviews/        # Interview program documentation
│   └── README.md               # Interview program FAQ and structure
├── video-transcripts/          # YouTube transcript automation
│   ├── transcripts.py          # Main Python script
│   ├── test_transcripts.py     # Unit tests
│   ├── requirements.in         # Direct Python dependencies (edit this)
│   ├── requirements.txt        # Locked dependencies (auto-generated, do not edit)
│   └── transcripts/            # 200+ generated markdown transcript files
├── processes/                  # Operational process documentation
│   └── cncf-event-creation/    # Guide for creating CNCF virtual events
├── architecture/               # OTel blueprint templates
├── assets/                     # Static assets
├── .github/
│   ├── CODEOWNERS              # @open-telemetry/sig-end-user-approvers own all files
│   ├── renovate.json5          # Renovate bot config for dependency updates
│   ├── ISSUE_TEMPLATE/         # GitHub issue templates (survey, Q&A, in-practice)
│   └── workflows/
│       ├── video-transcripts-tests.yml  # CI for Python tests
│       ├── fossa.yml                    # License scanning
│       └── ossf-scorecard.yml           # Security scorecard
├── README.md                   # Main project overview
├── sig-end-user-charter.md     # Formal SIG charter and scope
└── LICENSE                     # Apache 2.0
```

## Technology Stack

- **Python 3.14**: Only for the `video-transcripts/` script
- **Markdown**: All documentation and survey materials
- **YAML**: GitHub Actions workflows and issue templates
- **pip-tools**: Python dependency management

Key Python dependencies (`video-transcripts/`):
- `google-api-python-client` — YouTube Data API v3
- `youtube-transcript-api` — Fetch video transcripts
- `openai` — Optional transcript summarization (GPT-4o-mini)
- `python-dotenv` — Load `.env` environment variables
- `python-slugify` — Generate URL-safe filenames

## Development Workflows

### Video Transcripts — Setup

```bash
cd video-transcripts
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Create a `.env` file in `video-transcripts/`:
```
API_KEY=<your YouTube Data API v3 key>
OPENAI_API_KEY=<optional, for transcript summarization>
```

To get a YouTube API key: enable the YouTube Data API v3 in a GCP project, then create an API key.

### Video Transcripts — Running the Script

```bash
# By channel (most common usage)
python3 transcripts.py \
    --channel UCHZDBZTIfdy94xMjMKz-_MA \
    --path ./transcripts

# By playlist
python3 transcripts.py \
    --playlist <PLAYLIST_ID> \
    --path ./transcripts

# With date range (ISO 8601 format)
python3 transcripts.py \
    --channel UCHZDBZTIfdy94xMjMKz-_MA \
    --start 2024-01-01T00:00:00Z \
    --end 2024-12-31T00:00:00Z \
    --path ./transcripts

# Limit number of videos (useful for testing)
python3 transcripts.py --channel UCHZDBZTIfdy94xMjMKz-_MA --limit 5 --path ./transcripts
```

The script skips videos that already have a transcript file (idempotent).

### Video Transcripts — Running Tests

```bash
cd video-transcripts
python -m unittest test_transcripts.py -v
```

Tests use `unittest.mock` to mock YouTube API and OpenAI calls. CI runs these on Python 3.14 via GitHub Actions on any PR or push to `main` that touches `video-transcripts/**`.

### Dependency Management

Dependencies are managed with `pip-tools`:

1. Edit `requirements.in` to add/change a direct dependency
2. Regenerate the lockfile: `pip-compile requirements.in -o requirements.txt`
3. Install: `pip install -r requirements.txt`

**Never manually edit `requirements.txt`** — it is auto-generated. Renovate bot handles routine version updates automatically.

## File Naming Conventions

### Transcript Files

Generated transcript filenames follow this pattern:
```
<ISO8601-datetime>-<slugified-title>.md
```
Example: `20230913T192610Z-opentelemetry-q-a.md`

- Datetime is compacted (dashes and colons removed) for Windows compatibility
- Title is lowercased and hyphenated via `python-slugify`

### Survey Directories

Survey directories use kebab-case matching the survey topic:
```
end-user-surveys/otel-collector/
end-user-surveys/developer-experience/
```

## Content Conventions

### Adding a New Survey

1. Create a new issue using the `Survey` GitHub issue template
2. Copy `end-user-surveys/README.template.md` into a new directory
3. Fill in the README with: purpose, open date, close date, Google Form link, results link
4. After closing, add the raw CSV data and PDF report to the directory
5. Follow the [analysis guides](end-user-surveys/analysis-guides/) for data processing

**Survey design rules** (from `end-user-surveys/README.md`):
- Maximum ~15 short, concise questions
- Include standard demographic questions (org size, industry)
- No PII collection
- Avoid leading, loaded, or double-barreled questions
- Only one open-ended question max
- Surveys run for ~1 month; max 6 per year; never concurrent

### Adding a New Interview Session

Document outcomes in `end-user-interviews/` following the existing format. See `end-user-interviews/README.md` for the interview program structure and FAQ.

## CI/CD and Automation

| Workflow | Trigger | Purpose |
|---|---|---|
| `video-transcripts-tests.yml` | PR/push to `video-transcripts/**` | Run Python unit tests |
| `fossa.yml` | Push to `main` | License compliance scanning |
| `ossf-scorecard.yml` | Weekly + manual dispatch | Security scorecard |

All workflow action versions are pinned to full commit SHAs for reproducibility. When updating actions, always use the commit SHA, not a tag.

Renovate bot (`renovate.json5`) automatically opens PRs to update `requirements.txt` with patch updates (weekday mornings) and minor/major updates (Monday mornings).

## Key Constraints for AI Assistants

1. **This is documentation-first**: Most changes involve Markdown files, not code. Respect the community tone and factual accuracy of all content.

2. **Do not edit `requirements.txt` directly**: It is auto-generated by `pip-compile`. Only edit `requirements.in`.

3. **Transcript files are generated**: Do not manually create or edit files in `video-transcripts/transcripts/`. They are produced by the script.

4. **Survey CSV data is raw community data**: Do not modify CSV files. These are the source of truth for analysis.

5. **No PII**: Never add personally identifiable information to any file in this repository.

6. **CODEOWNERS**: All files are owned by `@open-telemetry/sig-end-user-approvers`. Changes require approval from that team.

7. **Python version**: The CI uses Python 3.14. Avoid using APIs or syntax not available in that version.

8. **Action pinning**: GitHub Actions must be pinned to full commit SHAs, not floating tags.

## Environment Variables

| Variable | Required | Purpose |
|---|---|---|
| `API_KEY` | Yes (for transcript script) | YouTube Data API v3 key |
| `OPENAI_API_KEY` | No | Enables transcript summarization via GPT-4o-mini |

Store these in `video-transcripts/.env` (this file is gitignored).

## Communication & Governance

- **Slack**: `#otel-sig-end-user` on CNCF Slack
- **Meetings**: Every two weeks, Thursdays 10:00 PT / 13:00 ET / 19:00 CET
- **Meeting notes**: Public Google Doc (linked in README)
- **Maintainers**: @AndrejKiri, @avillela, @danielgblanco, @reese-lee
