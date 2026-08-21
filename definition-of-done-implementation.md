# Definition of Done — Implementation Plan

**Scope:** `aai-institute/frontier-ai-academy` and `aai-institute/orientation-framework`. Same setup applies identically to both repos unless noted. This `dod-test` repo is the sandbox used to design and test the automation before rollout — it doesn't contain either target repo's files, so the repo-specific details in the per-item sections below (`docs/DoD-trustworthy-ai.md`, `survey.gs`, ADRs, `CONTRIBUTING.md`) describe the target repos, not this sandbox.

**Relationship to existing policy:** `frontier-ai-academy` already contains a company-level "Definition of Done for trustworthy AI" (`docs/DoD-trustworthy-ai.md`), referencing specific Company AI Policy articles. The 6 items below are this team's engineering-practice elaboration of that policy. The checklist text itself no longer cites policy articles inline (dropped for brevity); this doc notes the article mapping in prose per item, where one is confirmed, for traceability.

---

## 0. Cross-cutting setup (build this once, applies to every item below)

### 0.1 PR template
`.github/PULL_REQUEST_TEMPLATE.md` (both repos):

```markdown
## Summary
<!-- what changed and why -->

## Definition of Done
Mark each item `[x]` (done) or write `N/A — <reason>` next to it.

- [ ] **1. Human oversight** — All AI-generated content has been checked by a human with topic knowledge.
- [ ] **2. Transparency** — User-facing synthetic text, images, audio and video have been correctly labeled.
- [ ] **3. Expert input** — Where additional domain expertise is needed (eg. technical or regulatory), at least one team member from the relevant team has reviewed the work.
- [ ] **4. Language norms [until automated]** — Language uses UK spelling for English, "Du" form for German, and terminology is in line with our Institute Glossary.
- [ ] **5. Data handling** — No personal data is contained in the PR.
- [ ] **6. Internal testing** — The deliverable has been tested and used internally before any public release.

## Reviewer routing (author's judgment — no automated routing)
<!-- Required if item 3 above is checked: name who reviewed it (SME / R&E / regulation team) and how their sign-off was captured (link to Slack thread / email / doc). Write your answer inside the quote block below — text outside it is ignored by the automated check. -->
> 
```

Every item, including item 1, can be marked `N/A`; there's no longer an item that must always be checked. "Committed with a clear title, reviewed and approved" is enforced separately, by branch protection (§0.3) rather than by a checklist line.

### 0.2 Auto-labeling (GitHub Action, not a manual step)
PR titles must start with a conventional-commit-style prefix: `feat:`, `fix:`, `docs:`, or `chore:`, optionally with a scope (e.g. `feat(survey): add tool-usage grid`). `.github/workflows/auto-label.yml`, triggered on `pull_request` (opened/edited/synchronize): parses the title prefix and applies the matching label (`feature`/`fix`/`docs`/`chore`) automatically via the GitHub API, first removing any stale category label the PR already carries — the author never picks a label by hand. If the title doesn't match any known prefix, the workflow fails its check instead of guessing, prompting the author to fix the title (a one-word fix, not a manual labeling step).

This is deliberately title-based, not path-based (e.g. via `actions/labeler`), for the same reason CODEOWNERS was rejected: the file layout isn't stable enough to map paths to categories reliably. A title prefix has no dependency on repo structure.

### 0.3 Hard enforcement (GitHub Action)
`.github/workflows/dod-check.yml` (both repos). On every `pull_request` (opened/edited/synchronize), it:

1. Parses the PR body and fails if any of the 6 checklist lines is neither `[x]` nor contains `N/A`.
2. Re-validates the title prefix itself (`feat:`/`fix:`/`docs:`/`chore:`, optional scope) — deliberately *not* by checking whether the label was actually applied. Tested this: `dod-check` and `auto-label` fire off the same `pull_request` event and can run in either order, and GitHub does not re-trigger a workflow off events caused by the default `GITHUB_TOKEN` — so a "label was added" trigger chain is unreliable. Re-checking the same regex independently avoids the dependency entirely.
3. Fails if item 3 ("Expert input") is checked `[x]` but the "Reviewer routing" section is empty (only the placeholder HTML comment, or nothing). Caught in testing: a PR author could check "an expert reviewed this" done without naming anyone, since there was nothing tying the checkbox to actual evidence. This doesn't verify the sign-off is *genuine* — it can't — but it stops the specific case of claiming the work happened while leaving zero trace of who did it.
4. Only counts text written inside a blockquote (`> ...`) or a fenced code block (` ``` `) within the routing section as the actual answer — plain prose typed elsewhere in that section, or unrelated content someone adds further down the PR body, is ignored. Otherwise "everything until the next `##` heading" is too loose a boundary: it's ambiguous whether trailing content belongs to the answer, and the check would treat any stray text as satisfying the requirement without the author actually answering it.

Mark this check **required** in branch protection for `main` (Settings → Branches → Branch protection rule → "Require status checks to pass"), alongside:
- Require a pull request before merging.
- Require at least 1 approval.
- No direct pushes to `main`.

Branch protection is the actual mechanism ensuring every change is committed with a clear title, reviewed, and approved — it's no longer a numbered checklist item. `dod-check`'s job is narrower and can't verify a review was any good; it only catches a blank checklist or a claimed-but-unsupported routing answer.

> **Branch protection on personal free-plan repos:** GitHub blocks branch protection for private repos on free personal accounts ("Upgrade to GitHub Pro or make this repository public"). `aai-institute` is on the Team plan, where this works natively regardless of repo visibility — confirmed via `gh api orgs/aai-institute`. Not a concern for the real rollout.

### 0.4 CLAUDE.md (guards PR creation done via an AI agent)
`CLAUDE.md` (both repos):

```markdown
# CLAUDE.md

## Creating pull requests

Before running `gh pr create`, read `.github/PULL_REQUEST_TEMPLATE.md` and use it as `--body-file`. Fill in the Summary section with a factual description of the change.

Never pre-fill the Definition of Done section. Leave every `- [ ]` box unchecked, and leave the Reviewer routing `>` line empty, exactly as the template has them. Whether an item is done, N/A, or needs a named reviewer is a human judgment call about a change Claude made — it is Claude's change to describe, not Claude's compliance to certify. A human fills these in before merge, even if it means `dod-check` fails until they do.

`--fill`/`--fill-first` and a hand-written `--body` both skip the template entirely. Never use them in this repo.
```

This reverses the original design. The first version had Claude mark each item `[x]`/`N/A` and fill in reviewer routing itself, reasoning that an unfilled checklist wastes a round trip fixing something `dod-check` would catch anyway. That was deliberately changed: whether oversight happened, content is labeled, or an expert reviewed the work is a judgment call about *Claude's own change* — Claude certifying its own compliance defeats the point of the checklist. Claude now only writes the Summary and leaves the entire Definition of Done section untouched; a human fills it in before merge, and it's expected (not a failure state to work around) that `dod-check` stays red until they do.

The `--fill`/`--body` guard is unchanged from the original design: `gh pr create --body "..."` or `--fill` silently skips the repo's PR template (GitHub only auto-populates the template when no explicit body is passed), so PR creation must always read the template and pass it via `--body-file`.

### 0.5 Changelog & releases (cross-cutting, no longer tied to a numbered item)
No hand-maintained `CHANGELOG.md`. Instead:
- PR titles must read as a changelog line for a human (not "fix bug" — "Fix duplicate survey submissions on resubmit"). Enforced by review, not tooling.
- `.github/release.yml` (both repos) maps labels → changelog sections:
  ```yaml
  changelog:
    categories:
      - title: "🚀 Features"
        labels: [feature]
      - title: "🐛 Fixes"
        labels: [fix]
      - title: "📝 Documentation"
        labels: [docs]
      - title: "🔧 Chores"
        labels: [chore]
  ```
- At the end of each sprint (~2 weeks, ending Thursday or Friday — whenever the sprint actually closes), whoever wraps up the sprint runs:
  ```
  gh release create v<next-version> --generate-notes
  ```
  This compiles all merged PR titles since the last tag, grouped by label, into the release notes — that's the changelog.
- Versioning: bump according to whether the sprint's changes are breaking/feature/patch (simple judgment call, no semantic-release tooling needed for this team size).

---

## 1. Human oversight

- **Trigger:** Any PR that introduces or edits AI-generated or AI-assisted content (docs drafted with AI help, survey copy, analysis narrative, etc.).
- **Mechanism:** PR template item 1, checked off by a human reviewer with topic knowledge of what's being reviewed (not just a code-correctness pass) — never by Claude itself (§0.4).
- **Routing:** PR author names the right reviewer in the "Reviewer routing" section if the default reviewer doesn't have the topic knowledge needed; no automated path-based routing (deliberately — see item 3).
- **Policy link:** Art. 4.4.

---

## 2. Transparency

- **Trigger:** Any user-facing synthetic text, image, audio, or video (i.e., content a reader/survey participant/stakeholder outside the immediate team will see) that was AI-generated or substantially AI-assisted.
- **Mechanism:** Label such content exactly: **"Created with the help of AI"** (the wording company policy specifies, Art. 4.3) — e.g., a footer note in a doc, a line in survey instructions, a caption on generated media. The checklist item's wording was shortened to "correctly labeled"; the exact required label text is still governed by Policy Art. 4.3.
- **N/A case:** Internal-only content not shared beyond the immediate team, or content with no AI involvement.
- **Note:** Neither repo currently has a live AI-generation feature exposed to end users; this item currently applies to AI-assisted authoring of docs/survey content, not a runtime product feature. Revisit if that changes.

---

## 3. Expert input

- **Domain content** (docs, survey design, analysis interpretation): reviewed by a subject matter expert from the relevant team when domain knowledge beyond the core team is needed.
- **Technical/architecture items** (e.g. `survey.gs` logic, `analysis/` notebooks, ADRs in `orientation-framework`): feasibility and best practice confirmed by the **R&E team** specifically.
- **Regulatory items:** where a change touches something the regulation team should weigh in on (a new category of AI-generated content, a new type of personal-data collection, or anything a SME/R&E reviewer flags as uncertain), the author routes it to them the same way. Earlier drafts of this plan had a separate checklist item with a formal "non-standard case" trigger list forcing this escalation; that's now folded into this single item as an author judgment call rather than a separately enforced rule. Keep the trigger list above as guidance for that judgment, since nothing else prompts the author to think of it.
- **Routing:** Deliberately manual — the PR author decides who the right expert is and requests them directly (named in the PR body), rather than automated path-based routing (CODEOWNERS), since the repo structure isn't stable enough yet for reliable path-to-owner mapping. Revisit CODEOWNERS once the codebase/file layout stabilizes.
- **Sign-off capture:** The regulation team doesn't use GitHub day-to-day. The PR author reaches them (or the SME/R&E reviewer) via their normal channel (email/Slack/shared doc), gets a written confirmation, and pastes a short quote/link into the "Reviewer routing" section as auditable evidence before merging — `dod-check` fails if item 3 is checked and this is empty (§0.3).
- **Open question:** the sustainable/proportionate-AI-use and non-discrimination/accessibility sub-checks that used to live under a dedicated "EU AI Act & Trustworthy AI compliance" item have no checklist home anymore. Revisit whether they need one, or whether they're adequately covered by item 1 (human oversight) in practice.

---

## 4. Language norms [until automated]

- **Trigger:** Any PR that adds or changes user-facing English or German prose (docs, survey copy, UI text).
- **Mechanism:** Reviewer manually confirms UK spelling for English content, "Du" form for German content, and that terminology matches the Institute Glossary. The "[until automated]" tag flags this as a manual stand-in for tooling (e.g. a style-linter Action) that doesn't exist yet — replace this with automation when one is built, rather than leaving it a permanent manual step.
- **N/A case:** PRs with no English/German prose changes (pure code, config, or data changes).

---

## 5. Data handling

- **`frontier-ai-academy`:** No personal data in this repo (pure documentation) — mark `N/A` on this item for PRs here unless that changes.
- **`orientation-framework`:** In scope. Its own `CONTRIBUTING.md` states assessment results contain personal data; policy Art. 4.8 and GDPR apply. Existing controls to keep enforced and documented:
  - `data/` stays out of git — protected by `.gitignore`; never work around it.
  - No email/identity collection in the survey (per ADR-0006) — `setCollectEmail` must not be reintroduced without an explicit new ADR re-approving that trade-off.
  - `setLimitOneResponsePerUser(true)` uses Google sign-in only to prevent duplicate submissions — this is not identity collection and must not be extended into one.
  - PR template item 5 requires the author to confirm no personal data is being newly introduced into tracked files, and that access to any personal data (e.g., raw survey exports) stays limited to people who need it.

---

## 6. Internal testing

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
- [ ] Add `.github/workflows/dod-check.yml` hard-enforcement Action, checking items 1–6 and gating "Reviewer routing" on item 3 (§0.3)
- [ ] Add `CLAUDE.md` with PR-creation instructions, so an AI agent always sources the PR body from the template and never fills in the Definition of Done section itself (§0.4)
- [ ] Add `.github/release.yml` label-to-category mapping (§0.5)
- [ ] Create labels: `feature`, `fix`, `docs`, `chore` (targets for auto-labeling, not for manual picking)
- [ ] Enable branch protection on `main`: require PR + 1 approval + the auto-label and DoD status checks; disallow direct pushes
- [ ] Agree who cuts the sprint-end release (`gh release create --generate-notes`) and confirm the (~2-week, Thu/Fri-ish) cadence with the team
- [ ] Share this document with the regulation team and internal SME/testing contacts so they know when and how they'll be looped in
