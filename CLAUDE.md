# CLAUDE.md

## Creating pull requests

Before running `gh pr create`, read `.github/PULL_REQUEST_TEMPLATE.md` and use it as `--body-file`. Fill in the Summary section with a factual description of the change.

Never pre-fill the Definition of Done section. Leave every `- [ ]` box unchecked, and leave the Reviewer routing `>` line empty, exactly as the template has them. Whether an item is done, N/A, or needs a named reviewer is a human judgment call about a change Claude made — it is Claude's change to describe, not Claude's compliance to certify. A human fills these in before merge, even if it means `dod-check` fails until they do.

`--fill`/`--fill-first` and a hand-written `--body` both skip the template entirely. Never use them in this repo.
