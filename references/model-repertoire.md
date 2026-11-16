# Proposed File: references/model-repertoire.md

> This document is the deployable file content for the statistical and ecological model repertoire.
> Initialization date: 2026-11-16. Sources: Official R package documentation and primary literature cited below.
> All code blocks are transferable blueprints authored by this team: primary interfaces have been verified against official documentation, but code has not been executed in the current runtime environment or validated against user datasets, and must not be claimed as a completed replication.

## Table of Contents

- Selection Protocol and Observation Distributions
- ECO-M001 Baseline Regression and Hierarchical Dependence
- ECO-M002 Generalized Linear Mixed Models and Observation Processes
- ECO-M003 Bayesian, Phylogenetic, and Multivariate Models
- ECO-M004 Nonlinearity, Thresholds, and Spatiotemporal Structure
- ECO-M005 Directed Acyclic Graphs and Structural Equation Modeling
- ECO-M006 Hierarchical Variance Partitioning
- ECO-M007 Community Ordination, Joint Species Distribution Models, and Networks
- ECO-M008 Multilevel Meta-Analysis
- ECO-M009 Plant-Soil Feedback and Coexistence Modeling
- ECO-M010 Statistical Power and Design Simulation
- Update Contract

## Selection Protocol and Observation Distributions

This repertoire is an open index rather than a closed whitelist; external methods may be introduced whenever warranted by a new research question. Define the scientific estimand first, then select the statistical model.
For every analysis, document upfront: experimental or observational design, treatment assignment unit, independent replicates, nested/crossed hierarchy, spatial and temporal coordinates, response support domain, missingness mechanism, measurement error, and target population.
Distinguish average treatment effects, conditional associations, mechanistic parameters, out-of-sample predictive error, and variance partitioning; these estimands are not interchangeable.
Specify identifiable core models and a bounded set of sensitivity analyses upfront; never iterate across model specifications by P-value screening to chase statistical significance.
Effective sample size is governed by independent experimental units and information structure rather than arbitrary total-row thresholds.
Verify missing values, mortality, non-detections, structural zeros, and sampling zeros item by item; never carry over coding conventions from previous projects automatically.
Placeholders such as `dat`, `y`, `x`, `site`, and `plot` represent user variables to be mapped per dataset; modules are independent reference blueprints rather than a sequential script.

| Response Observation Mechanism | Candidate Models | Critical Verification Checks |
|---|---|---|
| Continuous measurement with approximately symmetric conditional errors | Gaussian; Student-t for heavy tails | Diagnose conditional residuals rather than applying mechanical normality tests to raw marginal responses |
| Strictly positive and right-skewed continuous measurement | Gamma (log link), lognormal | Distinguish link-scale parameters from response-scale marginal means; never log-transform zeros directly |
| Count data with known sampling effort | Poisson, Negative Binomial (with `offset(log(effort))` when effort varies) | Check overdispersion, imperfect detection, and strictly positive effort; do not treat effort as a precision weight |
| Successes out of known trials | Binomial, Beta-Binomial | Preserve trial denominators (`cbind(success, failure)`) and account for within-trial correlation |
| Continuous proportions $(0, 1)$ | Beta; Ordered-Beta (`ordbeta`) or explicit boundary process when true $0$ or $1$ values occur | Continuous proportions differ from binomial counts; avoid arbitrary ad-hoc boundary squeezing if boundary mass is substantive |
| Exact zeros combined with positive continuous values | Tweedie, hurdle, or explicit two-part model | Identify whether zeros represent true absence, non-detection, or threshold processes and verify separate identifiability |
| Multi-category abundances or compositional proportions | Multinomial, compositional models, or joint species models | Account for constant-sum closure constraints, sequencing depth, zero replacement, and absolute abundance scaling |
| Time-to-event, detection limits, or interval-censored observations | Survival or censored regression models | Never replace censored values with zero or half the detection limit and treat them as exact observations |

## ECO-M001 Baseline Regression and Hierarchical Dependence

**Use Case & Inputs:** Single-response treatment effects, silvicultural gradients, or trait associations with clearly defined randomization hierarchy, blocking, and repeated-measurement units.
**Target Estimand:** Prespecified treatment contrasts or slopes; random effects parameterize dependency structure and variance components rather than arbitrary column labels.

```r
# Baseline linear model for truly independent observations; invalid when hierarchical clustering exists.
m_lm <- stats::lm(y ~ treatment * x, data = dat)
# Plot nested within site; use only when the actual experimental design matches this hierarchy.
m_lmm <- lme4::lmer(y ~ treatment * x + (1 | site/plot),
                   data = dat, REML = TRUE)
lme4::isSingular(m_lmm)
```

**Diagnostics:** Residual distribution, heteroscedasticity, influential observations, singular fit (`isSingular`), and cluster support for random effects; subplots cannot serve as independent replicates when treatments are assigned at the stand level.
**Boundaries:** Random intercepts do not automatically remove temporal autocorrelation; random slopes require sufficient within-group variation and replication. Complete confounding cannot be fixed by adding random effects.
Account for ML vs. REML rules when comparing Gaussian mixed models with differing fixed effects, and ensure identical observation sets across compared models.
**Reporting:** Plot raw independent-unit data alongside prespecified marginal contrasts and intervals, stating the averaging grid and target scale explicitly.
**Official Source:** [Documentation](https://lme4.github.io/lme4/reference/lmer.html)
**Diagnostic Source:** [Documentation](https://lme4.github.io/lme4/reference/isSingular.html)
**Validation Status:** Official interface verified; unexecuted.

## ECO-M002 Generalized Linear Mixed Models and Observation Processes

**Use Case & Inputs:** Seedling recruitment counts, incidence, biomass, litter decomposition loss, and non-Gaussian responses governed by specific mean-variance relationships and zero-generating processes.
**Target Estimand:** Link-scale (`log`/`logit`) parameters vs. response-scale marginal effects, separating zero-inflation/hurdle processes from positive conditional responses.

```r
stopifnot(all(dat$effort > 0), all(dat$count >= 0),
          all(dat$count == floor(dat$count)))
m_count <- glmmTMB::glmmTMB(
  count ~ treatment * x + offset(log(effort)) + (1 | site),
  family = glmmTMB::nbinom2(link = "log"), data = dat)
sim_res <- DHARMa::simulateResiduals(m_count, n = 1000, seed = 1)
plot(sim_res)
# Substitute family according to data-generating mechanism; never batch-screen families for P-values.
# glmmTMB::tweedie("log"), beta_family("logit"), ordbeta("logit")
```

**Diagnostics:** Optimizer convergence, positive-definite Hessian, DHARMa simulated quantile residuals, dispersion tests, zero-count calibration, outliers, and spatial/temporal residual autocorrelation.
**Boundaries:** High zero counts do not automatically imply zero inflation; omitted covariates or unmodeled clustering can also distort residuals. Verify linear (`nbinom1`) vs. quadratic (`nbinom2`) mean-variance scaling.
Model heteroscedasticity via theoretically justified `dispformula` and zero processes via `ziformula` rather than adding complex submodels by default.
Document missingness, censoring, and plant mortality explicitly rather than masking them through distributional choices.
**Official Source:** [Documentation](https://glmmtmb.github.io/glmmTMB/reference/nbinom2.html)
**Diagnostic Source:** [Documentation](https://glmmtmb.github.io/glmmTMB/articles/troubleshooting.html)
**Residual Source:** [Documentation](https://cran.r-project.org/web/packages/DHARMa/vignettes/DHARMa.html)
**Validation Status:** Official interface verified; unexecuted.

## ECO-M003 Bayesian, Phylogenetic, and Multivariate Models

**Use Case & Inputs:** Partial pooling across hierarchies, measurement error, cross-species phylogenetic non-independence, and multiple response variables sharing latent group structure; requires weakly informative priors scaled to the variables and identifiable parameters.
**Target Estimand:** Joint posterior distributions of effects and predictions; Bayesian estimation cannot manufacture independent replication where none was sampled.

```r
library(brms)
A <- ape::vcv.phylo(tree, corr = TRUE)
sp <- levels(factor(dat$species))
stopifnot(all(sp %in% rownames(A)))
A <- A[sp, sp, drop = FALSE]
dat$species <- factor(dat$species, levels = sp)
# y_z and x_z are standardized with recorded center/scale; priors below are starting templates to be calibrated.
priors_z <- c(prior(normal(0, 1), class = "b"),
              prior(normal(0, 1), class = "Intercept"),
              prior(exponential(1), class = "sd"),
              prior(exponential(1), class = "sigma"))
m_phy <- brm(y_z ~ x_z + (1 | gr(species, cov = A)) + (1 | site),
             data = dat, data2 = list(A = A), family = gaussian(),
             prior = priors_z, chains = 4, seed = 1)
# Multivariate Gaussian formula blueprint; requires response-specific priors.
f_mv <- bf(y1_z ~ x_z + (1 | p | site)) +
        bf(y2_z ~ x_z + (1 | p | site)) + set_rescor(TRUE)
```

**Diagnostics:** Prior predictive checks before fitting; posterior predictive checks (`pp_check`), split-$\hat{R}$, bulk/tail ESS, divergent transitions, energy diagnostics (BFMI), and prior sensitivity after fitting.
**Boundaries:** `set_rescor(TRUE)` is not universally supported across arbitrary non-Gaussian families; residual correlation across responses does not establish causal direction.
Propagate phylogenetic tree uncertainty, intraspecific trait measurement error, and spatial covariance when scientifically relevant; align cross-validation folds (`loo` / grouped K-fold) with the target prediction level (e.g., new species vs. new sites).
**Official Source:** [Documentation](https://paulbuerkner.com/brms/articles/brms_phylogenetics.html)
**Multivariate Source:** [Documentation](https://paulbuerkner.com/brms/articles/brms_multivariate.html)
**Diagnostic Source:** [Documentation](https://mc-stan.org/learn-stan/diagnostics-warnings.html)
**Validation Status:** Official interface verified; unexecuted.

## ECO-M004 Nonlinearity, Thresholds, and Spatiotemporal Structure

**Use Case & Inputs:** Continuous resource gradients, disturbance intensity, microclimatic curves, and seasonal dynamics with sufficient gradient coverage and independent replication.
**Target Estimand:** Smooth functional responses vs. theoretically prespecified structural breakpoints (which address distinct scientific questions).

```r
library(mgcv)
dat$site <- factor(dat$site)
# Basis dimension k must be chosen based on unique x values, replication hierarchy, and gam.check diagnostics.
m_gam <- gam(y ~ treatment + s(x, k = 6) + s(site, bs = "re"),
             data = dat, method = "REML")
gam.check(m_gam)
concurvity(m_gam)
# Illustrative piecewise linear breakpoint model for independent observations; account for hierarchy when clustered.
m_break <- segmented::segmented(stats::lm(y ~ x, data = independent_dat),
                                seg.Z = ~ x)
```

**Diagnostics:** Basis dimension adequacy (`gam.check`), smooth concurvity, boundary extrapolation risk, residual autocorrelation, breakpoint confidence intervals, and data density across the gradient.
**Boundaries:** Smooth curvature does not automatically prove an ecological threshold or tipping point; searching for breakpoints and testing significance on the same dataset requires transparent reporting of selection uncertainty.
Select spatial smooths, autoregressive structures, or Gaussian processes based on actual sampling coordinates and time intervals; spatial terms cannot rescue treatments that are completely confounded with space.
**Official Source:** [Documentation](https://stat.ethz.ch/R-manual/R-devel/library/mgcv/html/gam.html)
**Threshold Source:** [Documentation](https://cran.r-project.org/web/packages/segmented/refman/segmented.html)
**Covariance Source:** [Documentation](https://glmmtmb.github.io/glmmTMB/articles/covstruct.html)
**Validation Status:** Official interface verified; unexecuted.

## ECO-M005 Directed Acyclic Graphs and Structural Equation Modeling

**Use Case & Inputs:** Testing hypothesized multi-step process chains where temporal ordering, common causes, post-treatment intermediates, and unmeasured confounders are explicitly mapped beforehand.
**Target Estimand:** Total effects, direct effects, and indirect (mediated) effects defined via a prespecified DAG; causal structure is a theoretical identification assumption, never an output discovered by goodness-of-fit statistics.

```r
# Illustrative piecewise SEM interface on complete, aligned observations with linear Gaussian submodels.
# The mediator must be empirically measured, and covariate adjustment must be justified by the DAG.
sem_fit <- piecewiseSEM::psem(
  lme4::lmer(mediator ~ treatment + covariate + (1 | site), data = dat),
  lme4::lmer(y ~ treatment + mediator + covariate + (1 | site), data = dat),
  data = dat)
summary(sem_fit)
```

**Diagnostics:** Validate each component model independently first; evaluate tests of directed separation (d-sep), omitted paths, measurement scaling, and observationally equivalent alternative DAGs.
**Boundaries:** Passing global goodness-of-fit tests (e.g., Fisher's $C$ with $P > 0.05$) does not prove causality; never reverse arrows or invent post-hoc pathways merely to inflate fit indices.
Under nonlinear links, interactions, or non-Gaussian families, simple products of path coefficients do not generally equal natural indirect effects; define mediation estimands and standardization scales explicitly.
Multivariate `brms` models can estimate simultaneous equations, but latent constructs, measurement error, and sequential ignorability assumptions still require explicit justification.
**Official Source:** [Documentation](https://jslefche.github.io/piecewiseSEM/reference/psem.html)
**DAG Source:** [Documentation](https://dagitty.net/learn/)
**Validation Status:** Official interface verified; unexecuted.

## ECO-M006 Hierarchical Variance Partitioning

**Use Case & Inputs:** Quantifying the relative contribution of predefined predictor groups (e.g., microclimate, edaphic properties, functional traits, spatial structure) to explained variance in univariate or multivariate responses.
**Target Estimand:** Total explained variance ($R^2$), unique components, shared components, and hierarchically partitioned relative importance—never interpreted as causal effect magnitudes.

```r
# Assumes `fit` has already passed model diagnostics and predictor groupings match formula terms.
hp_single <- glmm.hp::glmm.hp(fit)
# Rows of Y_hel and explanatory matrices E and T must be aligned to identical sample IDs.
hp_community <- rdacca.hp::rdacca.hp(
  dv = Y_hel, iv = list(environment = E, traits = T),
  method = "RDA", type = "adjR2", var.part = TRUE)
```

**Diagnostics:** Multicollinearity within and across predictor sets, group definition sensitivity, negative shared variance fractions, unadjusted vs. adjusted $R^2$, and stability under leave-one-cluster-out resampling.
**Boundaries:** Relative importance rankings depend entirely on the candidate variables included and how shared variance is allocated; evaluate submodel combinatorics and overfitting when predictor counts are large.
Keep interaction terms and grouped predictors conceptually coherent; never use unrestricted row permutations that violate plot or temporal clustering to generate artificially narrow p-values or intervals.
**Official Source:** [Documentation](https://cran.r-project.org/web/packages/glmm.hp/refman/glmm.hp.html)
**Multivariate Source:** [Documentation](https://cran.r-project.org/web/packages/rdacca.hp/refman/rdacca.hp.html)
**Variance Partitioning Source:** [Documentation](https://vegandevs.github.io/vegan/reference/varpart.html)
**Validation Status:** Official interface verified; unexecuted.

## ECO-M007 Community Ordination, Joint Species Distribution Models, and Networks

**Use Case & Inputs:** Sample-by-species matrices paired with environmental covariates, functional traits, and phylogenies; verify row/column alignment, all-zero rows, and sampling effort standardization first.
**Target Estimand:** Match the method to the exact question: community composition shifts, species-level environmental responses, out-of-sample prediction, or conditional residual associations.

```r
stopifnot(identical(rownames(Y), rownames(env)))
Y_hel <- vegan::decostand(Y, method = "hellinger")
ord <- vegan::rda(Y_hel ~ moisture + temperature, data = env)
D <- vegan::vegdist(Y, method = "bray")
# Valid only when treatments are randomized within blocks and rows represent independent experimental units.
perm_design <- permute::how(blocks = env$block, nperm = 999)
permanova <- vegan::adonis2(D ~ treatment, data = env,
                           permutations = perm_design)
dispersion <- vegan::betadisper(D, env$treatment)
# Formal permutation tests of multivariate dispersion must also respect the experimental exchangeability structure.
```

**Diagnostics:** Sensitivity to distance metrics and transformations, within-group multivariate dispersion (`betadisper`), ordination axis fit, design-constrained permutation schemes (`permute::how`), and cluster-blocked cross-validation.
**Boundaries:** Unrestricted within-block permutation is invalid when treatments are assigned at the whole-stand or whole-block level; subsamples within a plot must never be permuted as independent units.
Joint Species Distribution Models (`Hmsc`) integrate environment, traits, phylogeny, and random effects, but residual species-to-species associations reflect unmeasured environmental covariates as well as biotic interactions and must not be equated with direct ecological interactions.
Microbiome/amplicon datasets require explicit handling of compositionality, prevalence filtering, sequencing depth, and batch effects. Evaluate network stability against estimation uncertainty, thresholding, and sampling variability.
`linkET` visualizes correlation matrices and Mantel tests (note: network functions were migrated by the author to `netET`); visual correlation webs do not substitute for formal network inference.
**Ordination Source:** [Documentation](https://vegandevs.github.io/vegan/reference/cca.html)
**Permutation & Dispersion Source:** [Documentation](https://vegandevs.github.io/vegan/reference/adonis.html) ; [Documentation](https://vegandevs.github.io/vegan/reference/betadisper.html)
**Joint Model Source:** [Documentation](https://search.r-project.org/CRAN/refmans/Hmsc/html/Hmsc.html)
**Association Interpretation Source:** [Documentation](https://search.r-project.org/CRAN/refmans/Hmsc/html/Hmsc-package.html)
**Visualization Tool Source:** [Documentation](https://github.com/Hy4m/linkET)
**Validation Status:** Official interface verified; unexecuted.

## ECO-M008 Multilevel Meta-Analysis

**Use Case & Inputs:** Synthesizing comparable effect sizes and sampling variances across studies while accounting for within-study clustering, shared control groups, repeated time points, and phylogenetic relatedness.
**Target Estimand:** Pooled grand mean effect, between- and within-study heterogeneity ($\tau^2$, $I^2$), prespecified moderator slopes, and 95% prediction intervals with consistent directional sign conventions.

```r
# Order of yi and rows/columns of V must match exactly; V encodes within-study sampling error covariance.
stopifnot(nrow(V) == nrow(meta_dat), ncol(V) == nrow(meta_dat))
m_meta <- metafor::rma.mv(
  yi = yi, V = V, mods = ~ moderator,
  random = ~ 1 | study/effect, data = meta_dat, method = "REML")
```

**Diagnostics:** Positive-definiteness of the variance-covariance matrix $V$, study-level leverage and influence, residual heterogeneity, moderator collinearity, and sensitivity to assumed within-study correlation coefficients ($\rho$) or clubSandwich cluster-robust variance estimation.
**Boundaries:** Study-level random intercepts capture outcome heterogeneity but do not replace correlated sampling errors in $V$ when multiple treatments share a single control group.
Compute design-matched effect sizes via `escalc` and construct approximate sampling covariance matrices via `vcalc` paired with sensitivity analyses across plausible $\rho$ values.
Conduct explicit sensitivity checks for small numbers of independent studies, publication bias, and imputed variances; never report the total number of effect sizes ($k$) as if it were the number of independent studies ($n_{\text{studies}}$).
**Official Source:** [Documentation](https://wviechtb.github.io/metafor/reference/rma.mv.html)
**Input Source:** [Documentation](https://wviechtb.github.io/metafor/reference/escalc.html) ; [Documentation](https://wviechtb.github.io/metafor/reference/vcalc.html)
**Validation Status:** Official interface verified; unexecuted.

## ECO-M009 Plant-Soil Feedback and Coexistence Modeling

**Use Case & Inputs:** Two-phase conditioning and feedback experiments tracking conditioning species, focal responder species, inoculum source replication, and separate abiotic vs. microbial pathways.
**Target Estimand:** Estimate the full species-by-soil response matrix first, then derive prespecified pairwise interaction or home-vs.-away feedback contrasts ($I_s$) and map parameters onto formal coexistence criteria when supported by the design.

```r
# Illustrative specification for strictly positive biomass responses; select an appropriate zero-capable family if mortality/zeros occur.
m_psf <- glmmTMB::glmmTMB(
  growth ~ focal * conditioner * environment +
           (1 | inoculum_batch) + (1 | block),
  data = psf_dat, family = Gamma(link = "log"))
```

**Diagnostics:** Verify independent replication across Phase 1 conditioning and Phase 2 feedback (avoiding pseudoreplication from pooled soil inocula), control treatment structure, sample support across species pairs, and uncertainty propagation into composite feedback indices.
**Boundaries:** Negative home-soil feedback alone is insufficient to prove long-term species coexistence; stabilizing niche differences and average fitness differences must be evaluated jointly under the chosen theoretical framework.
Single-harvest final biomass does not automatically equate to per-capita invasion growth rates when rare; competition and coexistence claims require density gradients, demographic rates, or explicit model justification.
Autoclaving or gamma-sterilization can alter soil nutrient availability alongside microbiota; isolating specifically "microbial" feedback requires appropriate inoculum-addition or filtrate controls rather than labels alone.
**Primary Literature Source:** [Documentation](https://doi.org/10.1073/pnas.2122088119)
**Author Data & Code:** [Documentation](https://doi.org/10.5281/zenodo.6513066)
**Validation Status:** Theoretical scope and sources verified; the formula above is an illustrative team blueprint, unexecuted, and not a replication of the cited paper.

## ECO-M010 Statistical Power and Design Simulation

**Use Case & Inputs:** Pre-experiment comparison of sample allocation across independent stands/sites, nested plots, and sampling frequencies given biologically meaningful minimum effect sizes, variance components, and intraclass correlations.
**Target Estimand:** Statistical power, confidence/credible interval width, coverage probability, or Bayesian assurance defined relative to concrete scientific decision thresholds.

```r
# simulation_model is parameterized a priori from pilot/literature variance and target effect size, never from post-hoc observed p-values.
power_result <- simr::powerSim(
  simulation_model, test = simr::fixed("x", method = "lr"),
  nsim = 1000, seed = 1)
# For models unsupported by simr, write an explicit "simulate data -> fit model -> extract decision -> summarize" loop.
```

**Diagnostics:** Monte Carlo standard error of power estimates, convergence/singularity failure rates across simulated datasets, and sensitivity curves across plausible ranges of between-site variance, zero proportions, and autocorrelation.
**Boundaries:** Adding subsamples or subplots within existing stands does not substitute for increasing the number of independent stands when treatments operate at the stand level; never compute post-hoc "observed power" from a study's own p-value to excuse non-significant findings.
Incorporate realistic randomization constraints, missing data rates, and observation distributions into the data-generating simulation, and document all prior variance sources transparently.
**Official Source:** [Documentation](https://search.r-project.org/CRAN/refmans/simr/html/powerSim.html)
**Validation Status:** Official interface verified; unexecuted.

## Update Contract

Assign stable sequential IDs starting from `ECO-M011` to newly internalized models, recording primary literature / official package documentation URLs, ingestion dates, boundary conditions, and revision lineage.
Maintain strict separation across the three origin labels (`reported`, `reconstructed`, `proposed`) and three code validation states (`unexecuted`, `example-passed`, `dataset-validated`).
Verify official documentation and run a minimal targeted test before adopting any newly released package; log `sessionInfo()`, data version, random seed, and diagnostics rather than equating package installation with model validation.
Deduplicate and merge overlapping entries during updates, preserving boundary conditions when empirical studies conflict and retaining full version history when superseding older entries.
When converting blueprint snippets into project functions, declare required column names, physical units, missing/zero-value handling, returned effect scales, and explicit stop conditions; never build automated wrapper loops that silently hunt for statistically significant model specifications.
