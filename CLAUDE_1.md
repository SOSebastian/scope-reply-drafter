# CLAUDE.md

Standing instructions for Claude Code in this repository. Read this file at the start of every session.

## About me

- I am not a developer. I can read code, and I have AWS, GitHub and DevOps experience.
- I act as architect and reviewer. You act as the developer.
- Explain changes in plain English. Do not assume I know language-specific jargon; translate it.
- No em-dashes or emojis in anything you write (code comments, docs, commit messages, PR descriptions).

## How we work

1. **Plan before code.** For any new feature, read `SPEC.md` (or the relevant spec in `specs/`), then propose a plan and wait for my approval before writing code.
2. **Small slices.** Build one working piece at a time. Each slice must run on its own before moving on.
3. **Tests with every slice.** Every function that does real work gets a test in `tests/`. Run the tests before telling me a slice is done.
4. **One feature per branch.** Create a branch named `feature/<short-name>` before starting. Never commit directly to `main`.
5. **Commit often.** Small commits with clear messages describing what changed and why.
6. **Pull requests.** When a feature is complete, open a PR with a plain-English summary: what it does, how to test it, anything I should check.
7. **Ask, don't guess.** If the spec is unclear or contradicts itself, stop and ask me rather than inventing a requirement.

## Tech stack

- Language: Python 3.12+
- Claude access: the official `anthropic` Python SDK
- Dependencies: managed in `requirements.txt` (pin versions)
- Config: environment variables loaded from `.env` via `python-dotenv`
- Tests: `pytest`
- Formatting and linting: `ruff`

## Project layout

```
.
├── CLAUDE.md          # this file
├── SPEC.md            # what we are building (current feature)
├── specs/             # older or additional specs
├── src/
│   ├── llm.py         # shared Claude API wrapper (reused by every feature)
│   ├── config.py      # model IDs, defaults, settings loaded from env
│   └── <feature>.py   # feature code
├── prompts/           # prompt templates as .md or .txt files, not inline strings
├── tests/
├── .env.example       # lists required env vars with dummy values
└── requirements.txt
```

## Claude API rules

- **All API calls go through `src/llm.py`.** Feature code never imports `anthropic` directly. If `llm.py` is missing something, extend it there.
- **Model IDs live in `src/config.py` only**, never hardcoded in feature code. Use named roles rather than raw IDs:
  - `FAST` for cheap, simple tasks (classification, extraction)
  - `DEFAULT` for most work
  - `DEEP` for hard reasoning, used only where the spec says so
- **Prompts live in `prompts/`** as separate files so I can read and edit them without touching code.
- **Log every API call** (model used, input and output token counts, duration) to the console at INFO level. Never log the API key.
- **Handle failures.** Retry on rate limits and transient errors with backoff. Fail with a clear message on anything else.
- **Loops have limits.** Any loop that calls the API (agents, tool use, self-critique) must have a maximum iteration count set in config.

## Security rules (non-negotiable)

- Never hardcode API keys, tokens, passwords or account IDs.
- `.env` must be in `.gitignore`. Only `.env.example` is committed.
- Never print or log secrets, even partially.
- Do not add a new dependency without telling me what it is and why.
- No real customer data in tests or examples. Use invented data.

## Definition of done

A slice is done when:
- the code runs,
- tests pass (`pytest`),
- lint passes (`ruff check .`),
- I have seen a plain-English summary of what changed.

## Lessons learned

Add a line here whenever you get something wrong twice, so it does not happen again.

- (none yet)
