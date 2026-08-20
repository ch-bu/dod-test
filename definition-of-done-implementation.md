# Definition of Done — Implementation Plan

**Scope:** `aai-institute/frontier-ai-academy` and `aai-institute/orientation-framework`. Same setup applies identically to both repos unless noted.

**Relationship to existing policy:** `frontier-ai-academy` already contains a company-level "Definition of Done for trustworthy AI" (`docs/DoD-trustworthy-ai.md`), referencing specific Company AI Policy articles. The 7 items below are this team's engineering-practice elaboration of that policy — each item notes which article it implements. Articles are cited by number, not by file path, since the file's location may change.

---

## 0. Cross-cutting setup (build this once, applies to every item below)

### 0.1 PR template
Add `.github/PULL_REQUEST_TEMPLATE.md` to both repos:

```markdown
## Summary
<!-- what changed and why -->

## Definition of Done
Mark each item `[x]` (done) or write `N/A — <reason>` next to it. Item 1 can never be N/A.

- [ ] **1. Commit & review** — committed with a clear PR title (used as the changelog entry), reviewed and approved by a teammate.
- [ ] **2. Human oversight** — any AI-generated/AI-assisted content checked by a human with topic knowledge, including language quality. *(Policy Art. 4.4)*
- [ ] **3. Transparency** — user-facing synthetic text/image/audio/video labeled "Created with the help of AI". *(Policy Art. 4.3)*
- [ ] **4. EU AI Act & Trustworthy AI compliance** — confirmed thoughtful/sustainable AI use (model choice, token use) and non-discrimination/accessibility; regulation-team sign-off obtained if this is a non-standard case (see criteria below). *(Policy Art. 4.11, GDPR)*
- [ ] **5. Subject matter / technical expert review** — reviewed by a subject matter expert where domain knowledge was needed; technical feasibility/best practice confirmed by the R&E team for technical items.
- [ ] **6. Data handling** — personal data (if any) collected/stored responsibly, access limited to those who need it. *(Policy Art. 4.8, GDPR)*
- [ ] **7. Internal testing** — reviewer has actually exercised the change (read the doc end-to-end / filled out the changed form / run the notebook) before approving.

## Reviewer routing (author's judgment — no automated routing)
<!-- If this PR needs SME, R&E, or regulation-team input, name them here and note how their sign-off was captured (link to Slack thread / email / doc). -->
```

### 0.2 Auto-labeling (GitHub Action, not a manual step)
PR titles must start with a conventional-commit-style prefix: `feat:`, `fix:`, `docs:`, or `chore:` (e.g. `feat: add tool-usage grid to survey`). Add `.github/workflows/auto-label.yml`, triggered on `pull_request` (opened/edited): it parses the title prefix and applies the matching label (`feature`/`fix`/`docs`/`chore`) automatically via the GitHub API — the author never picks a label by hand. If the title doesn't match any known prefix, the workflow fails its check instead of guessing, prompting the author to fix the title (a one-word fix, not a manual labeling step).

This is deliberately title-based, not path-based (e.g. via `actions/labeler`), for the same reason CODEOWNERS was rejected: the file layout isn't stable enough to map paths to categories reliably. A title prefix has no dependency on repo structure.

### 0.3 Hard enforcement (GitHub Action)
Add `.github/workflows/dod-check.yml` to both repos. On every `pull_request` (opened/edited/synchronize), it:

1. Parses the PR body and fails if any of the 7 checklist lines is neither `[x]` nor contains `N/A`.
2. Fails if item 1's line is not `[x]` (i.e., rejects `N/A` on item 1).
3. Re-validates the title prefix itself (`feat:`/`fix:`/`docs:`/`chore:`) — deliberately *not* by checking whether the label was actually applied. Tested this: `dod-check` and `auto-label` fire off the same `pull_request` event and can run in either order, and GitHub does not re-trigger a workflow off events caused by the default `GITHUB_TOKEN` — so a "label was added" trigger chain is unreliable. Re-checking the same regex independently avoids the dependency entirely.

Mark this check **required** in branch protection for `main` (Settings → Branches → Branch protection rule → "Require status checks to pass"), alongside:
- Require a pull request before merging.
- Require at least 1 approval.
- No direct pushes to `main`.

This is the concrete mechanism for item 1 ("committed... reviewed and approved") and the blank-checklist guard for the rest — everything else remains a human judgment call the Action can't verify (whether the review was any good), only that it was *not skipped*.

---

## 1. Commit & review

- **Mechanism:** Branch protection (above) + PR template + hard-enforcement Action. Never `N/A`.
- **Changelog:** No hand-maintained `CHANGELOG.md`. Instead:
  - PR titles must read as a changelog line for a human (not "fix bug" — "Fix duplicate survey submissions on resubmit"). This is enforced by review, not tooling.
  - Add `.github/release.yml` to both repos mapping labels → sections:
    ```yaml
    changelog:
      categories:
        - title: 🚀 Features
          labels: [feature]
        - title: 🐛 Fixes
          labels: [fix]
        - title: 📝 Documentation
          labels: [docs]
        - title: 🔧 Chores
          labels: [chore]
    ```
  - At the end of each sprint (~2 weeks, ending Thursday or Friday — whenever the sprint actually closes), whoever wraps up the sprint runs:
    ```
    gh release create v<next-version> --generate-notes
    ```
    This compiles all merged PR titles since the last tag, grouped by label, into the release notes — that's the changelog.
  - Versioning: bump according to whether the sprint's changes are breaking/feature/patch (simple judgment call, no semantic-release tooling needed for this team size).

---

## 2. Human oversight

- **Trigger:** Any PR that introduces or edits AI-generated or AI-assisted content (docs drafted with AI help, survey copy, analysis narrative, etc.).
- **Mechanism:** PR template item 2, checked off by the reviewer — who must have topic knowledge of what's being reviewed (not just a code-correctness pass). Reviewer also checks language quality.
- **Routing:** PR author names the right reviewer in the "Reviewer routing" section of the PR body if the default reviewer doesn't have the topic knowledge needed; no automated path-based routing (deliberately — see item 5).
- **Policy link:** Art. 4.4.

---

## 3. Transparency

- **Trigger:** Any user-facing synthetic text, image, audio, or video (i.e., content a reader/survey participant/stakeholder outside the immediate team will see) that was AI-generated or substantially AI-assisted.
- **Mechanism:** Label such content exactly: **"Created with the help of AI"** (the wording company policy specifies, Art. 4.3) — e.g., a footer note in a doc, a line in survey instructions, a caption on generated media.
- **N/A case:** Internal-only content not shared beyond the immediate team, or content with no AI involvement.
- **Note:** Neither repo currently has a live AI-generation feature exposed to end users; this item currently applies to AI-assisted authoring of docs/survey content, not a runtime product feature. Revisit if that changes.

---

## 4. EU AI Act & Trustworthy AI compliance

- **Scope:** Includes policy rows not separately named in this 7-item list — thoughtful/sustainable AI use (is the model/tool proportionate to the task; token consumption considered) and non-discrimination/accessibility of any AI-generated content — folded in here as sub-checks, not separate checklist lines.
- **Non-standard case (triggers mandatory regulation-team sign-off):** any of —
  1. A new *category* of AI-generated content not previously covered.
  2. A new *type* of personal-data collection not previously covered.
  3. Anything the SME or R&E reviewer explicitly flags as uncertain during their review.
- **Sign-off capture:** The regulation team doesn't use GitHub day-to-day. The PR author reaches them via their normal channel (email/Slack/shared doc), gets a written confirmation, and pastes a short quote/link into the PR body as auditable evidence before merging. This is a manual step — no automation possible for the actual judgment, only for detecting when it's needed (the checklist question itself).

---

## 5. Subject matter & technical expert review

- **Domain content** (docs, survey design, analysis interpretation): reviewed by a subject matter expert from the relevant team when domain knowledge beyond the core team is needed.
- **Technical/architecture items** (e.g. `survey.gs` logic, `analysis/` notebooks, ADRs in `orientation-framework`): feasibility and best practice confirmed by the **R&E team** specifically (per policy row 6, "R&E team consulted" for technical items).
- **Routing:** Deliberately manual — the PR author decides who the right expert is and requests them directly (named in the PR body), rather than automated path-based routing (CODEOWNERS), since the repo structure isn't stable enough yet for reliable path-to-owner mapping. Revisit CODEOWNERS once the codebase/file layout stabilizes.
- **Sign-off capture:** Same as item 4 — captured wherever the expert naturally works, referenced in the PR.

---

## 6. Data handling

- **`frontier-ai-academy`:** No personal data in this repo (pure documentation) — mark `N/A` on this item for PRs here unless that changes.
- **`orientation-framework`:** In scope. Its own `CONTRIBUTING.md` states assessment results contain personal data; policy Art. 4.8 and GDPR apply. Existing controls to keep enforced and documented:
  - `data/` stays out of git — protected by `.gitignore`; never work around it.
  - No email/identity collection in the survey (per ADR-0006) — `setCollectEmail` must not be reintroduced without an explicit new ADR re-approving that trade-off.
  - `setLimitOneResponsePerUser(true)` uses Google sign-in only to prevent duplicate submissions — this is not identity collection and must not be extended into one.
  - PR template item 6 requires the author to confirm no personal data is being newly introduced into tracked files, and that access to any personal data (e.g., raw survey exports) stays limited to people who need it.

---

## 7. Internal testing

- **Trigger:** Any change that's user-facing — affects colleagues reading a doc, survey participants, or anyone relying on the analysis outputs.
- **Mechanism:** Folded into the review step itself, not a separate release gate — neither repo has a distinct "public release" moment or a preview-deploy environment. The reviewer must actually perform the relevant action before approving:
  - Doc changes: read the changed doc end-to-end.
  - Survey changes: fill out the changed form (`survey.gs` output) as a respondent would.
  - Analysis changes: run the updated notebook and sanity-check the output.
- **N/A case:** Internal-only refactors/chores with no observable effect for anyone outside the PR author.

---

## Setup checklist (one-time, per repo)

- [ ] Add `.github/PULL_REQUEST_TEMPLATE.md` (§0.1)
- [ ] Add `.github/workflows/auto-label.yml` — auto-applies label from PR title prefix (§0.2)
- [ ] Add `.github/workflows/dod-check.yml` hard-enforcement Action (§0.3)
- [ ] Add `.github/release.yml` label-to-category mapping (§1)
- [ ] Create labels: `feature`, `fix`, `docs`, `chore` (targets for auto-labeling, not for manual picking)
- [ ] Enable branch protection on `main`: require PR + 1 approval + the auto-label and DoD status checks; disallow direct pushes
- [ ] Agree who cuts the sprint-end release (`gh release create --generate-notes`) and confirm the (~2-week, Thu/Fri-ish) cadence with the team
- [ ] Share this document with the regulation team and internal SME/testing contacts so they know when and how they'll be looped in
