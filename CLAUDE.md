# Project brief for Claude Code

Read this whole file before writing code. It is the source of truth for what this repository
builds. If anything here conflicts with a request in a session, ask before proceeding.

## What this is

A submission to the *Organization Science* special issue "AI-Assisted Frontiers in Organizational
Science" (Stage 1 due **Nov 1, 2026**; target submission **Oct 31**). Stage 1 needs three things:
- a 1,500–2,000-word proposal
- a technical appendix
- this repository, containing a working prototype

It is **not** a hypothesis-testing paper. Reviewers score novelty, feasibility, auditability,
ethicality, reproducibility, scalability, adoption and frontier expansion.

**Working name:** `forkpoint`.

**One-sentence purpose:** an AI-assisted workflow that finds coding alternatives in primary study
reports, judges them against a prespecified inferential aim and methodological adequacy, and
traces the admissible ones through a meta-analysis. It shows why each choice was accepted or
rejected, and how the unresolved choices affect conclusions.

The scientific contribution is making the path from **report → coding judgment → synthesis
result** systematic and inspectable. Efficiency is a side benefit, never the pitch.

## Conceptual basis (get this right)

- **The aim comes first.** The meta-analysis's purpose should settle coding choices wherever it
  can. Sensitivity analysis is only for alternatives that remain defensible after that assessment.
  - This rests on the co-author's framework for aligning a meta-analysis's inferential aim with its
    methods (a paper under review). Ask the lead author for it.
  - Don't invent the operational criteria; they come from the co-author.
- **Multiverse lineage.** This extends multiverse analysis to the coding level of meta-analysis.
  Don't claim to be the first multiverse meta-analysis or the first AI multiverse.
  - Plessen et al. (2023, *BMJ Mental Health*) varied inclusion criteria and estimators across
    4,281 meta-analyses; coding disagreements were settled by discussion.
  - Bertran, Fogliato & Wu (2026, *PNAS*) used AI analysts with an AI auditor on primary datasets.
  - Mazei et al. (2025, *JAP*) provide a process model for multiverse analysis.
  - **Our distinct piece:** identifying and validating study-level alternatives *from the primary
    reports*, then tracing them through the synthesis.

## Classification of every alternative

| Class | Code | Treatment |
|---|---|---|
| Extraction error | `extraction_error` | Correct it; exclude the erroneous version |
| Different estimand | `different_estimand` | Analyze separately if that target is relevant and the method is defensible |
| Same estimand, methodologically inadequate | `inadequate` | Exclude; document why |
| Same estimand, methodologically defensible | `defensible` | Keep for the sensitivity analysis; mark a preferred specification when justified |
| Insufficient information | `unresolved` | Record the missing evidence; never force a decision |

Defensible alternatives need not be equally preferable. Always report a justified **primary
specification** first, then sensitivity to the other admissible choices.

## Workflow (one CLI subcommand per step)

1. **`aim`.** Validate `aim.yaml`: target constructs, population, purpose, effect quantity,
   measurement corrections, and the features that separate the intended estimand from nearby
   targets. Nothing else runs without a valid aim file.
2. **`extract`.** Find alternatives in the primary reports. For each candidate, record:
   - `study_id`, `decision_type`, the proposed coding
   - a supporting passage (short), with its page or table location
   - why it is relevant

   Check passages automatically against the page text: normalise text, then fuzzy-match; a score
   of 90 or more on rapidfuzz `partial_ratio` counts as verified. Also check that the numbers
   belong to the intended sample, measure and analysis. Anything unverified is flagged, never
   silently accepted.
3. **`adjudicate`.** AI proposes a class and a rationale for each alternative. Humans certify the
   decision on a blind, shuffled sheet, against the prespecified aim and criteria.
   - **`adjudicate --freeze`** writes `adjudication.lock` (content hash + timestamp).
   - `synthesize` must refuse to run without a valid lock. This enforces "admissibility is
     settled before anyone sees pooled results".
4. **`configure`.** Build the admissible configurations from the `defensible` alternatives.
   - Rules for dependencies and incompatible choices live in `constraints.yaml`.
   - Each decision is either a **global rule** (applied to all studies) or **study-specific**.
   - Enumerate **unique** configurations. Repeated AI proposals must never add weight.
   - If enumeration is infeasible, use structured sampling and record the method, seed and size.
5. **`synthesize`.** Run the synthesis via R:
   - the preferred specification first
   - every configuration with its conventional estimate and CI (psychmeta Hunter–Schmidt; metafor
     random effects as a robustness check)
   - analytic settings held fixed, to isolate coding sensitivity
   - `different_estimand` alternatives as separate, labelled analyses
6. **`report`.** Write one self-contained HTML page with:
   - the preferred estimate
   - a descriptive specification curve
   - the range of pooled estimates
   - the share of configurations preserving a **prespecified** substantive conclusion (direction
     plus a meaningful magnitude; statistical significance reported separately)
   - which decisions are associated with the spread
   - which decisions the aim settled, which were rejected on methodological grounds, and which
     remained open

   Every number links to its configuration, adjudication record and source passage.

**Validation helpers:**
- `expert-packet` builds the independent expert's materials *without* any AI output: the aim,
  criteria, decision types, a study list and a blank form.
- `rate-sheet` builds a later, blind sheet for the expert to rate the AI-proposed alternatives.
- `validate` scores the comparisons.

### Statistical rules (non-negotiable)

- **No Rubin's rules and no "judgment interval".** Configurations are not draws from a
  data-generating model.
- The spread is **sensitivity to the defined decision set**, not a confidence distribution for the
  effect. A share like "80% of configurations" describes the included configurations; it is not a
  probability that the conclusion is correct.
- Equal weighting is one descriptive choice. Disclose any weighting, sampling and seed.

## Validation (independent expert benchmark)

An independent scholar works on the same 20–30 studies in two blind passes:
- **Pass A**, from the primary reports alone: identifies and adjudicates alternatives without
  seeing AI output or pooled results.
- **Pass B**, later: rates the AI-proposed alternatives, shuffled and blind to their source.

`validate` reports:
1. the legitimacy and adequacy of AI alternatives, including whether they preserve the estimand
2. the accuracy of passages and numbers, checked against their source location and context
3. how well the tool separates genuine ambiguity from extraction errors and insufficient reporting
4. coverage: legitimate alternatives the tool missed, especially consequential ones
5. whether the tool identifies the decisions that materially change the synthesis

Agreement with the original meta-analysis coders is one check, not ground truth.

Also keep a log, `docs/ai_contribution_log.md`, of cases where the AI surfaced a legitimate
alternative or evidence link that the original coding did not flag. The proposal needs this to show
what AI adds beyond applying a fixed coding rule.

## Scope for Nov 1

- One substantive relationship.
- About three decision types, chosen after checking the reports. Candidates must be shown to be
  legitimate; none is automatically eligible:
  - multiple measures of one construct
  - reliability estimates
  - composite vs. average
  - overlapping samples (never treat overlapping observations as independent)
- 20–30 studies, with 10–15 as the fallback floor if extraction or adjudication slips.
- One fully reproducible report.

**Stage 2, not now:** generalizing across relationships, the ranked-vs.-random review comparison,
and a generalizability-theory study.

## Hard constraints

- **Nothing to do with HubMeta.** No code, data, outputs, formats or adapters from it, and don't
  name it in the repo.
- **Open-weight models by default.** The call discourages proprietary models; commercial models are
  an optional comparison only.
  - Read `FORKPOINT_BASE_URL` and `FORKPOINT_API_KEY` from the environment (OpenAI-compatible endpoint).
  - Pin exact model IDs, temperature and seeds in `configs/`.
- **Frozen intermediates.** Every model call is cached as JSONL (model, params, prompt hash, input
  reference, raw output, parsed output, tokens, cost). `forkpoint replay` rebuilds every table,
  figure and the report from the cache and frozen adjudication files, with no model access.
- **Lawful sharing.**
  - Never commit PDFs. Keep them in `data/private/` (gitignored).
  - Passages stay short (under ~50 words).
  - The public demo uses open-access studies whose licence allows redistribution.
  - Don't commit the original human codes until the lead author confirms coauthor consent.
- **The repository is public, and review may be anonymous.** No author names, institutions or
  emails anywhere in the repo. Use "The forkpoint authors".
- **Never commit secrets.** Commit a `.env.example`; keep `.env` gitignored.
- **Ask before spending money.** Before any live run, print the call count and estimated cost.
  Run a 3–5 study pilot first.
- Mark clearly in the README what is demonstrated versus planned.

## Where this runs

- **Claude Code on the web (cloud, Ubuntu 24.04).**
  - Python is preinstalled; R is not. Install it with a setup script: `apt update && apt install -y r-base`.
  - R packages come from CRAN, which needs `cloud.r-project.org` in a **Custom** network allowlist,
    plus the model provider's API domain.
  - The cloud can't see local PDFs. Use it for building and testing, with the replay backend and
    the open-access demo.
- **The lead author's Windows machine.** This is where the live extraction runs on the private PDFs.
  - Use `pathlib` throughout and no shell-specific scripts.
  - Every task is a CLI subcommand, because `make` is not available on Windows.
- **Tests run offline:** pytest with the replay backend and fixture PDFs.

## Stack

- **Python 3.11+:** `pymupdf`, `pydantic`, `openai` (OpenAI-compatible client), `rapidfuzz`,
  `pandas`, `openpyxl`, `jinja2`, `matplotlib`, `typer`, `pyyaml`, `pytest`.
- **R 4.x:** `psychmeta`, `metafor`, `jsonlite`, pinned with `renv`. Run R once per synthesis over a
  long table of all configurations, never once per configuration.

## Repository layout

```
README.md  LICENSE  DATA_LICENSE (CC-BY-4.0)  CITATION.cff  .gitignore  .env.example
pyproject.toml  renv.lock
forkpoint/   cli.py aim.py extract.py verify.py adjudicate.py configure.py
             synthesize.py report.py validate.py models.py cache.py
R/synthesize.R
configs/     models.yaml run.yaml
prompts/     extract.md classify.md   (versioned)
examples/open_demo/     aim.yaml decisions.yaml constraints.yaml + open-access studies
examples/case_study/    aim.yaml decisions.yaml constraints.yaml, DOIs, cache/,
                        adjudication/, report.html   (no PDFs)
data/private/           gitignored
tests/
docs/        technical_appendix.md  ai_contribution_log.md  proposal/
```

`docs/proposal/` is drafted in a separate session. Don't edit it without asking.

## Build order

1. Skeleton, environments, `.gitignore`, `.env.example`, a CLI that runs, offline tests passing.
2. `aim` + the `decisions.yaml` and `constraints.yaml` schemas, with validators and the open-demo examples.
3. PDF ingest + `extract` + `verify`, with the replay backend.
4. `adjudicate` (export sheet → import → `--freeze` lock).
5. `configure`: constraints and unique enumeration.
6. `synthesize` + `R/synthesize.R`, checked against psychmeta's own example data.
7. `report`.
8. `expert-packet`, `rate-sheet`, `validate`.
9. README (demonstrated vs. planned; three uses: replay, rerun the demo, run your own) and a
   `docs/technical_appendix.md` draft.

## Timeline and critical path

The expert's Pass A needs only the aim, the criteria, the decision types and the PDFs. It runs in
**parallel** with the build.

| Dates | Work |
|---|---|
| Oct 13–15 | Pick the test meta-analysis and gather its PDFs. Co-author supplies the aim and adjudication criteria; fix the ~3 decision types. Build steps 1–2. |
| Oct 15–16 | `expert-packet` goes to the independent scholar; Pass A runs Oct 16–22. |
| Oct 16–20 | Build steps 3–5; 3–5 study pilot. |
| Oct 20–22 | Full extraction on 20–30 studies (lead author's machine). |
| Oct 22–25 | Author certification (blind); freeze; expert Pass B (rating AI alternatives). |
| Oct 25–27 | Steps 6–8: synthesis, report, validation. |
| Oct 27–30 | Proposal, technical appendix, README; co-author review. |
| Oct 31 | Zenodo release / anonymized mirror; submit. |

If the aim and criteria slip past Oct 16, cut to 10–15 studies rather than moving the end date.

## Open decisions (ask; don't guess)

- Which test meta-analysis, and where its files live.
- The aim and the adjudication criteria (from the co-author).
- Which three decision types.
- The model provider and the exact open-weight model IDs.
- Which Hunter–Schmidt corrections to apply, and the prespecified stability criterion.
- How to share the repo for anonymous review.
