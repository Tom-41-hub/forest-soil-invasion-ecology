# 5-Dimensional Literature Digestion Card v3.0

This file is a blank template and does not imply that any paper has already been ingested. Fields may be marked as `Not reported`, `Unavailable`, or `Not applicable`. Never fabricate or extrapolate unstated claims merely to fill every cell.

## 0. Identity, Material Scope, and Status

| Field | Required Entry |
|---|---|
| `source_key` / `card_id` | Normalized DOI or verified `author-year-title` key; stable card ID |
| Title, Authors, Year | Original bibliographic metadata; distinguish preprint vs. published journal version |
| DOI / Official URL / Version | Canonical link, version date, erratum or retraction status (if verified) |
| Materials Actually Read | Full text / Abstract only / Supplementary Information / Author code / Tutorial; list missing components explicitly |
| Date & Reader Roles | Date processed; specialist roles activated |
| Study System & Scope | Ecosystem type, focal taxa, spatial/temporal scales, treatment range, observational vs. experimental design |
| Independent Units & Dependency | Level of treatment randomization, number of independent replicates, subsamples / repeated measures / shared controls |
| Target Estimand | Total effect, conditional association, prediction, indirect/mediated effect, heterogeneity, or other |
| Update Operation | `added` / `merged` / `revised` / `contested` / `superseded` / `no-change` |

Tag all substantive items along three independent status axes:
- **Origin**: `reported` (explicitly stated in source) / `reconstructed` (rebuilt from methods text) / `proposed` (formulated by our team).
- **Applicability**: `candidate` / `scoped` (boundary conditions established) / `contested` (conflicting evidence exists) / `superseded`.
- **Code Validation**: `not-applicable` / `unexecuted` / `example-passed` / `dataset-validated` (record exact replication scope separately only if original study results were reproduced).

## 1. Theoretical Tension and Hypothesis Structure

- What prior consensus, assumption, or expectation existed? Which empirical observation, scale mismatch, or theoretical prediction created tension with it?
- What specific questions or hypotheses did the authors pose? Distinguish prespecified hypotheses, post-hoc explanations, and new predictions proposed by our team; do not force artificial H1-H3 numbering.
- What empirical outcomes would support or falsify each explanation? Which alternative mechanisms could generate the exact same observed pattern?
- **Source Evidence Locator**: Exact section, page number, and phrase locator.
- **Transferable Structure**: Express the tension using generalized ecological variables; specify applicable ecosystems and boundary conditions where the logic breaks down.

## 2. Cross-Domain Indicators and Mechanisms

| Mechanistic Link / Arrow | Measured Indicator & Unit | Direct Process Rate vs. Static Proxy | Spatial / Temporal Scale | Evidence Locator | What It Supports vs. Cannot Support |
|---|---|---|---|---|---|
| Populate strictly from source material | To be filled | To be filled | To be filled | To be filled | To be filled |

Isolate unmeasured mechanisms under "Candidate Explanations" rather than embedding them inside verified causal chains. Where relevant to the material, distinguish concentration vs. pool/stock, potential enzyme activity vs. in situ flux, taxonomic composition vs. functional activity, and short-term growth vs. lifetime fitness.

Provide one transferable mechanistic rule: **Boundary Condition -> Predicted Direction/Magnitude -> Observable Indicator -> Key Alternative Explanation -> Discriminating Measurement or Experiment**.

## 3. Statistical and Modeling Architecture

| Field | Recording Requirement |
|---|---|
| Response & Support Domain | Continuous / count / proportion / multivariate; define zeros, missingness, censoring, and detection limits explicitly |
| Model Specification | Distribution family, link function, fixed effects, interactions, random effects, correlation structure, offset |
| Design & Identification | Randomization level, independent replication, covariate adjustment, confounding, causal identification assumptions |
| Estimation & Contrasts | Effect size metric, link vs. response scale, target contrasts, interval type, multiple-testing family |
| Computational Implementation | Software, packages, and versions reported by the authors; do not guess if unstated |
| Diagnostics & Robustness | Separate checks actually performed by the authors from additional checks recommended by our team |
| Source Locator | Methods page number, supplementary table, or script filename and function |
| Transferability Scope | Data prerequisites, disqualifying conditions, and parameters requiring re-tuning on new datasets |
| Validation Evidence | Execution environment, random seed, input data, output summary, and failure logs; mark `unexecuted` if not run |

Formula and code metadata block:
```text
model_id: M-...
origin: reported / reconstructed / proposed
code_status: unexecuted / example-passed / dataset-validated
source_locator: ...
required_inputs: ...
estimand: ...
formula_blueprint: ...
adaptation_required: ...
```

Formulas written solely from prose descriptions must be labeled `reconstructed` ("formula skeleton reconstructed from Methods text") and never presented as the authors' original code. Never fabricate execution success when input data are unavailable. Reusable wrapper functions must declare input constraints, return values, diagnostic outputs, failure modes, and minimal working examples; upgrade `code_status` only after actual execution passes.

## 4. Figure Architecture Blueprint

| Original Figure / Panel | Scientific Question Answered | Plotted Data & Summary Statistics | Axes & Units | Grouping / Color / Faceting | Uncertainty & Sample Hierarchy | Transferable Visual Logic |
|---|---|---|---|---|---|---|
| Populate only for inspected figures | To be filled | To be filled | To be filled | To be filled | To be filled | To be filled |

Distinguish SD, SE, confidence intervals, and credible intervals; specify whether points represent raw observations, plot-level aggregates, marginal predictions, or posterior draws. Never invent panel layouts or color schemes when original figures were not inspected. Extract how the visual encoding enables readers to evaluate effect magnitude, heterogeneity, or mechanism rather than appealing to vague "top-journal aesthetics".

## 5. Argumentation Structure and Paraphrase Templates

Record each rhetorical move as: **Argumentative Function -> Paraphrased Template -> Required Evidence -> Inapplicable Contexts -> Source Locator**.

Examples below are internal team templates, not verbatim quotes from external papers:
- Bounded effect reporting: "Under [measured conditions], [response] changed by [effect size and interval], indicating [bounded ecological consequence]."
- Candidate mechanism framing: "This pattern is consistent with [mechanism]; distinguishing it from [alternative] requires [discriminating measurement or intervention]."
- Low precision / wide interval: "The estimate spans [ecologically meaningful range], leaving both [scenario A] and [scenario B] compatible with the data."
- Prespecified equivalence supported: "The interval falls within the prespecified equivalence bounds for [estimand]."

Non-significance alone never proves zero effect, mechanistic cancellation, or decoupling; equivalence claims require prespecified scientific bounds and formal equivalence testing. State uncertainty concisely without padding every paragraph with defensive disclaimers.

## 6. Internalized Increments and Persistence Verification

| Entry ID | Content Type | Prior Status -> New Status | Source / Card ID | Scope Conditions | Operation | Target File |
|---|---|---|---|---|---|---|
| To be filled | Method / Mechanism / Visual / Rhetoric | To be filled | To be filled | To be filled | `add` / `merge` / `revise` / `contest` | To be filled |

- Does this duplicate, supplement, or correct an existing source or entry?
- Does counter-evidence or context-dependence exist? Preserve both records and cross-reference them.
- Did file persistence and read-back verification succeed? Mark `Not yet persisted` if saving failed or was skipped.
- Provide a 3-5 sentence user-facing summary covering: the most valuable transferable insight, what new research problems it informs, what cannot be inferred from it, and the exact persisted increments.
