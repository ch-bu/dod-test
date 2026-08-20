# CLAUDE.md

## Creating pull requests

Before running `gh pr create`, read `.github/PULL_REQUEST_TEMPLATE.md` and build `--body-file` from it: keep every checklist item, mark each one `[x]` or `N/A — <reason>` based on the actual change, and — when item 4 or 5 is checked — put the SME/R&E/regulation sign-off inside a `>` blockquote (or a fenced code block) under "Reviewer routing"; text outside a quote/fence there doesn't count toward `dod-check`.

`--fill`/`--fill-first` and a hand-written `--body` both skip the template entirely. Never use them in this repo.
