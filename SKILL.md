---
name: benchmark-builder
description: Helps a user design, structure, document, and self-score a scientific ML benchmark that conforms to the MLCommons Science Benchmarks Ontology (five-element definition, six-category rubric, arXiv:2511.05614), generating a reproducible repo, verifying reference solutions, and packaging a validated Codabench competition. Use whenever the user wants to build, formalize, publish, or audit a benchmark, dataset+task+metric suite, or reproducibility package, including LLM evaluation benchmarks, even if they never say "MLCommons," "ontology," or "Codabench." Trigger on "turn my model comparison into a proper benchmark," "structure this repo so others can reproduce our results," "grade our benchmark documentation," or "make this a Codabench competition." Also self-scores an existing benchmark, writes an MLCommons corpus entry or DOE GEAR/Genesis cards, and runs benchmarks on Slurm or PBS clusters, loading Genesis Mission skills for HPC sites, lm-eval tasks, data/model cards and AmSC MLflow tracking.
license: Apache-2.0
---

# Building an MLCommons-ontology-conformant benchmark

A benchmark, per this ontology, is "a carefully defined standardized version of a
scientific application used for making quantifiable comparisons of solutions." Most things
called benchmarks in scientific ML are partial: a dataset with no defined task, a task with
undefined metrics, code only the original author can run. This skill walks the user
through the five required elements, produces real files (not just advice), and scores the
result honestly against the rubric MLCommons uses.

## Read first

- **`references/ontology.md`, in full, before anything else.** It has the exact wording of
  the five elements and six rubric categories. You need the precise wording, not a
  paraphrase, both to interview well and to score correctly.
- **`references/wa-hls4ml-example.md`**, skimmed, whatever the domain. It covers
  [axess-benchmark](https://github.com/ben-hawks/axess-benchmark), a real benchmark built
  with this skill (5.00/5, a validated Codabench bundle, reproduced on NERSC Perlmutter),
  and the mistakes found while building it. Those mistakes are why the verification steps
  below exist.
- **`references/repo-structure.md`, in full, before generating any benchmark files.** It
  specifies the recommended layout and the pipeline contract.
- **`references/genesis-skills.md` before the first step that uses a Genesis Mission
  skill** (HPC sites and schedulers, the lm-eval-harness chain, `card-eval-updater`,
  `datacard-generator`, `croissant-validator`, `uq-metrics-evaluator`,
  `literature-search`, `amsc-mlflow`). It says how to load each one, what to hand it, and
  what comes back. Follow the loaded skill's own workflow; don't paraphrase it from memory.

Later deliverables each have their own reference; read it in full before starting:

| Deliverable | When | Read |
|---|---|---|
| MLCommons corpus entry | once the benchmark and its score exist | `references/mlcommons-corpus-format.md` |
| DOE GEAR / Genesis Model, Agent, Data cards | DOE/Genesis work, or the reference solution is a model or agent | `references/doe-gear-cards.md` |
| Codabench competition bundle | the user wants participants to submit | `references/codabench.md` |
| Cluster runs | the benchmark runs on Slurm or PBS | `references/hpc.md` |
| LLM benchmark | the task evaluates language models | `references/llm-benchmarks.md` |
| AmSC MLflow tracking and model registry | the user opts in (Genesis/AmSC work, or wants experiment tracking) | the `amsc-mlflow` notes in `references/genesis-skills.md` |

## Gather artifacts before interviewing

Don't open with a blank interview. Most of the five elements are already answered in a
paper, a repo, a dataset sample, a README or a training script. The first pass at
wa-hls4ml was built entirely from the paper and repo. But reading isn't verifying: the
paper was wrong about which label its checkpoints were trained on. Treat extracted claims
about the reference solution as drafts until element D's checks confirm them.

Before asking a single interview question:

1. **Check what's already in front of you**: files in the conversation, and the working
   directory (PDFs, READMEs, notebooks, `data/` or `src/`, dataset files). If the request
   names a repo, paper or dataset, read it first.
2. **Ask for what's missing, once.** List what you found, and ask specifically for what
   would close which element ("a link to the training code would answer the Reference
   Solution and Software Environment questions"). If related work matters and the user has
   none, offer the `literature-search` skill.
3. **Extract a draft answer for each element**, with evidence (file, section, line). An
   element the material doesn't address is a real gap; don't guess past the evidence.
4. **Carry the draft into the interview as confirmation** ("here's what I found for the
   splits -- still current?"), and ask real questions only for what's left open.

If the user genuinely has nothing yet, step 3 comes up empty and the interview runs in full.
The point is never to ask for something already sitting in front of you.

## Figure out which mode you're in

Confirm rather than asking cold:

1. **New benchmark from scratch.** The artifact pass came up mostly empty; interview in
   full.
2. **Formalize an existing project.** Confirm the draft, interview only real gaps, and
   propose moving the code into the recommended structure. Whether to reorganize or only
   add the missing pieces is the user's call.
3. **Score/audit an existing benchmark.** Skip to "Score against the rubric" once you have
   evidence per rubric item. An un-evidenced score is false confidence.

Modes 2 and 3 often turn into mode 1 output once the gaps are visible.

**Is it an LLM benchmark?** If the task evaluates language models on text (QA, reasoning,
extraction, generation), read `references/llm-benchmarks.md` now. The elements and rubric
are the same; the task becomes an lm-evaluation-harness task, the reference solutions are
named models, and model cards come from `card-eval-updater`.

## Interview (confirm the draft, fill the gaps)

Work through the five elements in order, conversationally, not as a questionnaire. Each
maps onto rubric categories, so getting them right is what makes the score legitimate.

**A. Problem Specification and Constraints.** The input (image, time series, graph, text,
tabular, ...), the output (regression, classification, generation, anomaly flag, control
action), and the fixed constraints that are *not* optimized (target hardware, power,
latency ceiling, software version). "None" is a valid answer; don't invent constraints.
For LLM benchmarks, the prompt, few-shot count and answer format belong here.

**B. Dataset.** Does it exist, or must it be built? Push on each FAIR principle concretely:
*Findable* (is every instance uniquely identified?), *Accessible* (hosted where, which
license, which protocol?), *Interoperable* (what format; a community standard?),
*Reusable* (versioned? is the code that generated it public?). Then the split: sizes,
construction (random / stratified / held out by construction), non-overlap, and whether a
held-out generalization set makes sense (wa-hls4ml's "exemplar" set).

Settle what `data.py` and `data/SCHEMA.md` need, from **this** dataset, not from defaults
(details and axess's answers in `references/repo-structure.md`):
- **Sample IDs.** What uniquely identifies a sample within a split, and are there
  fallbacks? (axess: one subset had `model_id` instead of `uuid`.)
- **Ground truth and forbidden inputs.** Which field is the ground truth, and which fields
  are **not valid inputs** because they're labels or derived from them.
- **Missing or invalid ground truth.** Excluded, imputed, or scored as failures? Ask, and
  document the choice. Whatever it is, report coverage.
- **Training filter vs scoring filter.** If anything reproduces upstream training, did
  upstream select samples differently from how the benchmark scores? If so, keep both.
- **Size and format.** Does it fit in memory? Is preprocessing expensive enough to cache?

For checkable FAIR evidence, `croissant-validator` generates or validates `croissant.json`
(on Hugging Face, validate the Hub's own), and for DOE/Genesis work `datacard-generator`
writes the data card.

**C. Performance Metric(s).** What's measured, and does it capture what matters, not just
what's easy to compute? This category has two sub-scales (`references/ontology.md`). Pin
down the exact formula, edge cases (division by zero, empty or unparseable predictions),
per-target vs aggregated, and whether one metric is enough or it's a multi-dimensional
benchmark (e.g. accuracy under a latency bound).

**Check that published numbers reproduce with the *stated* formula.** When a paper or
README reports results:
1. Recompute at least one published table cell from raw predictions, using the formula
   exactly as documented.
2. If it doesn't match, find the variant that does, then ask the user which one the
   benchmark defines. Document the discrepancy either way.
3. Look for internal-consistency failures visible without code: a value that contradicts
   another column, or an aggregate row that can't be reconciled with its subgroups.

axess's paper states SMAPE with ε = 1,
but its numbers only reproduce with ε = 1e-8, and its dense DSP R² is printed as −0.74
while its own RMSE implies −111.74. The benchmark implements the stated ε = 1 and
documents all three findings in its `<repo>/docs/VALIDATION.md`.

**Metric choice follows the benchmark's own AI/ML motif; there is no default suite.**
Check the motif against `references/ontology.md` Part 3 ("What metrics do comparable
existing benchmarks actually use?"), then whether that fits this benchmark's question.
`scripts/metrics.py` covers three families only: regression (`r_squared`, `smape`,
`rmse`, `mae`, `relative_l2_error`, ...), classification and anomaly detection
(`classification_report`, `roc_auc`), and calibration (`references/metrics-and-uq.md`).
Offer them only when the task really is one of those, picking the function that fits
(their docstrings say when), and write bespoke metric code for every other motif. LLM benchmarks get their metrics from the
lm-eval task. A metric that doesn't fit its task loses Metric Quality credit however well
it's implemented.

Also, per `references/metrics-and-uq.md`: recompute standard metrics independently with
`uq-metrics-evaluator` and record agreement in `<repo>/docs/VALIDATION.md`; and if a
reference model outputs uncertainties or probabilities, ask whether calibration belongs in
the metrics.

**D. Reference Solution.** It needs public code, full documentation (architecture,
hyperparameters, training procedure), hardware/software requirements, and every element C
metric evaluated for it. A simple baseline plus a stronger model strengthens the
benchmark.

**Ask: "Does the benchmark evaluate only given or pretrained models, or does it also train
or reproduce training?"** Don't assume; score-only was axess's choice, not a rule. The
answer shapes the repo (`references/repo-structure.md`, "Score-only vs includes
training"): score-only has no `train.py` and takes weights from `weights/MANIFEST.json`;
including training adds `train.py`, training jobs and training-reproduction checks. It also
feeds the Codabench mode decision. The `predict → score_all.sh` contract is the same either
way.

**Verify the reference solution before building on it**: run every check in
`references/reference-solution-checks.md` and record the results in
`<repo>/docs/VALIDATION.md`. In short:
1. identity and weights;
2. training label vs scored label, on data;
3. the whole inference procedure;
4. per-sample agreement, then golden outputs;
5. reference vs auxiliary models.

For an LLM benchmark they take the form in `references/llm-benchmarks.md`: model identity
and revision, the answer key, the whole prompt-to-answer procedure, a full (not
sample-limited) run, and a weak control model.

**E. Documentation and Reproducible Protocol.** Can someone outside the team reproduce the
numbers from a clean environment? That takes numbered reproduction steps, a pinned or
containerized environment, and prose on motivation and scientific background, not just API
docs. No paper yet is one specific gap ("an academic paper about the benchmark exists"),
not a vague weakness.

For citations (details in `references/repo-structure.md`, "Conventions"):
- prefer the published DOI over the preprint once one exists;
- fetch the metadata from `https://doi.org/<doi>` with `Accept: application/x-bibtex`
  rather than typing it in;
- check section and equation numbering against the published version.

**Ask which machine(s) it will run on**: a laptop, a workstation, or which cluster(s).
Don't assume any site. As soon as the user names a cluster, load its genesis HPC skill
(site plus `slurm` or `pbs`, latest version), pre-fill the machine profile from it, and ask
only what's left (`references/hpc.md`, "What to ask per machine"). Generate the generic
Slurm or PBS templates, with one machine profile per cluster.

Also ask about **motif tagging** (`references/ontology.md` Part 3): scientific domain(s)
and exactly one AI/ML motif. It isn't scored, but it makes the benchmark findable.

## Build the artifacts

Generate real files as each element is answered, not all at the end; later answers often
change earlier files. Paths written `<repo>/...` are files in the benchmark repository you
generate, not in this skill.

**Propose the recommended structure by default, and explain what it buys:**
- one uniform pipeline for every model;
- participants plug in by writing their prediction files;
- golden tests make a new machine verifiable before any job runs;
- Codabench submissions and the bundle come from the same truth function, so they can't
  drift from the benchmark.

Each file's contract is in `references/repo-structure.md`. In summary:

```
<repo>/
├── README.md, SUBMISSION.md, CITATION.cff, LICENSE, rubric.yaml, SCORE_REPORT.md
├── pyproject.toml, requirements*.txt (one per stack that must stay separate), .gitattributes
├── data/SCHEMA.md, [data/croissant.json]
├── docs/  VALIDATION.md, <MACHINE>.md
├── reference_solution/README.md          documentation only
├── reference_results/                    committed LEADERBOARD.md + <split>/<model>/metrics
├── weights/MANIFEST.json (+ small derived artifacts)
├── src/<pkg>/  data, [cache, features, stats], models/, [train], predict, truth, score, report, submission
├── scripts/    fetch_data.py, fetch_weights.py, score_all.sh
├── [tasks/<task>/]                       LLM benchmarks: the lm-eval task
├── <hpc>/      (optional) generic Slurm or PBS, tuned per machine
├── codabench/  (optional) README, build_bundle.py, bundle_src/
└── tests/      score, golden pipeline, [features equivalence], submission, fixtures/
```

Every run follows the same **pipeline contract**:

```
fetch_data → [cache] → [train] → predict (per model, per split) → score_all.sh (truth, score, leaderboard, submissions)
```

A participant's model plugs in by writing its prediction files (by default a CSV of
`sample_id` plus one column per output); nothing else changes. Tell the user that in those
words.

**Generate the tree; don't copy a skeleton**, and **decide the contents per benchmark.**
Which optional pieces exist, the models, splits, metrics, preprocessing, stack workarounds,
test tolerances and job sizes all come from this benchmark's answers and measurements.
Adapt the snippets in `assets/repo/`, `assets/hpc/` and `assets/codabench/` where they fit.
A fix found while building a benchmark goes into that benchmark's code and docs, never into
this skill's templates.

The structure is recommended, not required. If the user wants a different layout, or
reorganizing existing code costs more than it buys, deviate; keep the pipeline contract
anyway, and note the deviation in the README.

## Score against the rubric

1. Copy `assets/rubric_template.yaml` into the benchmark (e.g. `rubric.yaml`).
2. Fill in every item with `met: true` or `met: false` and a one-line `evidence:` pointing
   at the file/section that justifies it. For judgment calls ("well documented"), make the
   call and say why; when really unsure, ask the user. An inflated self-score misleads
   everyone who relies on it, including MLCommons endorsement reviewers.
3. For **Performance Metrics**, set `definition_level` (0-3) and `quality_level` (0-2) per
   `references/ontology.md`; it isn't a flat checklist.
4. Run `python scripts/score_benchmark.py path/to/rubric.yaml --out SCORE_REPORT.md`. It
   refuses to score a category with unanswered items; the error names them.
5. Present the overall score, whether it clears the 4.5 endorsement threshold, and the gap
   list, prioritizing the cheapest fixes (a missing environment file before "write a
   paper").

## Iterate

Ask whether to act on the gaps now, and re-run the scorer after each fix so the user sees
the number move. Stop when the user is satisfied; 5/5 everywhere is their call, not a
target to impose.

**Keep the self-score honest over time.** Whenever a verified fact changes (a dataset gets
published, a cluster run completes, a reference model is replaced, a table turns out not to
reproduce), re-read **every** `evidence:` string, not just the item you worked on. In axess,
"HF repo pending", "not yet on Perlmutter" and "paper Table 4 reproduced" all went stale.
Keep dated caveats in `notes:`. The scorer lists evidence with provisional wording
("pending", "not yet", "TODO", "will be") under "Evidence to re-check"; resolve or re-confirm
each.

## Additional deliverables

Optional, separate from the benchmark package. Read the reference in full first.

- **MLCommons corpus entry** (`references/mlcommons-corpus-format.md`, "Step by step"), once
  the benchmark and its score exist. Ask about `url` and `contact`; set
  `valid`/`reproducible`/`benchmark_ready` only if verified; always validate.
- **DOE GEAR / Genesis cards** (`references/doe-gear-cards.md`), when the reference solution
  is a model or agent, or the user works under a DOE/Genesis RFA. Ask which cards apply.
  Model Card results are written by script, never retyped (`card-eval-updater` for LLM
  benchmarks, `scripts/render_card_results.py` otherwise).
- **AmSC MLflow tracking** (`amsc-mlflow`, via `references/genesis-skills.md`): log
  `reference_results/` (dry run first) and record the run ID in
  `<repo>/docs/VALIDATION.md`; register reference weights as `staging`, and move
  `production` only after verification and with the user's go-ahead. The user sets their
  MyAmSC token themselves; never ask for it.

## Package for Codabench

If the user wants participants to submit, package a Codabench competition bundle. Start
only once the standalone benchmark exists (dataset, metrics, reference solution, docs);
without those a bundle is an empty shell. Read `references/codabench.md` in full first; its
"Workflow" section has the steps.

- **Ask code submission vs results submission** before generating anything. Score-only
  benchmarks with public ground truth, and LLM benchmarks, are usually results submissions.
- **Never author real participation terms.** Generate an obviously-marked placeholder and
  say it must be replaced before going live.
- **Get explicit confirmation before every `docker push`.**
