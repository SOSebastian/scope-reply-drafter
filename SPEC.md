# SPEC: Scope Reply Drafter

**Status:** Draft
**Owner:** Sebastian
**Last updated:** 2026-11-11

## 1. Problem

Customers regularly push back on what is in or out of scope. A good reply has to anchor on the actual SOW wording before saying anything else. Drafting these by hand is slow, and it is easy to concede scope by accident.

## 2. Goal

A command-line tool that takes an SOW extract and a customer email, and produces a draft reply that is anchored on the SOW language, along with a short review of how well the draft holds scope.

## 3. Non-goals (for this version)

- No user interface. Command line only.
- No sending email. The tool only drafts.
- No reading from Outlook, SharePoint or any other system. Inputs are local text files.
- No deployment. Runs on my machine only.

## 4. Inputs

| Input | Format | Example |
|---|---|---|
| SOW extract | Plain text file | `examples/sow.txt` |
| Customer email | Plain text file | `examples/email.txt` |
| Tone (optional) | Command-line flag: `firm`, `collaborative` | default `collaborative` |

## 5. Outputs

Printed to the console and saved to `output/<timestamp>.md`:

1. **Scope analysis**: what the customer is asking for, and which SOW clauses it falls inside or outside of, quoting the clause numbers.
2. **Draft reply**: the email text, ready to edit.
3. **Review**: a second pass that checks the draft for accidental scope concessions, vague commitments, or claims not backed by the SOW, and lists any problems found.

## 6. How it works

1. Read both input files.
2. **Call 1 (analyse)**, model role `DEFAULT`: compare the request against the SOW and produce the scope analysis.
3. **Call 2 (draft)**, model role `DEFAULT`: write the reply using the analysis and the chosen tone.
4. **Call 3 (review)**, model role `DEEP`: critique the draft against the SOW.
5. If the review finds problems, redraft once using the review as feedback, then review again. Maximum 2 redrafts (set in config).
6. Save and print the final result, with token counts and approximate cost per run.

## 7. Usage

```
python -m src.scope_reply --sow examples/sow.txt --email examples/email.txt --tone firm
```

## 8. Acceptance criteria

- [ ] Runs end to end on the example files with a valid API key in `.env`.
- [ ] Output contains all three sections.
- [ ] Every scope claim in the analysis cites an SOW clause number.
- [ ] The redraft loop stops after the configured maximum.
- [ ] A clear error appears if an input file is missing or empty.
- [ ] A clear error appears if the API key is missing.
- [ ] Tests cover file loading, the loop limit, and output saving (API calls mocked in tests).

## 9. Build order (slices)

1. `src/config.py` and `src/llm.py`: one function that sends a prompt to a chosen model role and returns the text. Test it with a one-line prompt.
2. File loading and argument parsing.
3. Analyse step.
4. Draft step.
5. Review step.
6. Redraft loop with limit.
7. Saving output and cost summary.

## 10. Open questions

- Should the SOW extract support PDF or Word input later, or stay plain text?
- Should the tone options include a third, softer option?
- Is a cost cap per run worth adding (stop if a run exceeds a set amount)?
