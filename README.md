<!-- README.md is generated from README.Rmd. Please edit that file -->

brglm2
======

<!-- badges: start -->

[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/brglm2)](https://cran.r-project.org/package=brglm2)
[![R-CMD-check](https://github.com/ikosmidis/brglm2/workflows/R-CMD-check/badge.svg)](https://github.com/ikosmidis/brglm2/actions)
[![Codecov test
coverage](https://codecov.io/gh/ikosmidis/brglm2/branch/master/graph/badge.svg)](https://codecov.io/gh/ikosmidis/brglm2?branch=master)
[![Licence](https://img.shields.io/badge/licence-GPL--3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0.en.html)
<!-- badges: end -->

[**brglm2**](https://github.com/ikosmidis/brglm2) provides tools for the
estimation and inference from generalized linear models using various
methods for bias reduction ([Kosmidis,
2014](https://doi.org/10.1002/wics.1296)). **brglm2** supports all
generalized linear models supported in R, and provides methods for
multinomial logistic regression (nominal responses) and adjacent
category models (ordinal responses).

Reduction of estimation bias is achieved by solving either the mean-bias
reducing adjusted score equations in [Firth
(1993)](https://doi.org/10.1093/biomet/80.1.27) and [Kosmidis & Firth
(2009)](https://doi.org/10.1093/biomet/asp055) or the median-bias
reducing adjusted score equations in [Kenne et al
(2017)](https://doi.org/10.1093/biomet/asx046), or through the direct
subtraction of an estimate of the bias of the maximum likelihood
estimator from the maximum likelihood estimates as prescribed in
[Cordeiro and McCullagh (1991)](https://www.jstor.org/stable/2345592).
[Kosmidis et al (2019)](https://doi.org/10.1007/s11222-019-09860-6)
provides a unifying framework and algorithms for mean and median bias
reduction for the estimation of generalized linear models.

In the special case of generalized linear models for binomial and
multinomial responses (both ordinal and nominal), the adjusted score
equations return estimates with improved frequentist properties, that
are also always finite, even in cases where the maximum likelihood
estimates are infinite (e.g. complete and quasi-complete separation).
See, [Kosmidis & Firth (2020)](http://doi.org/10.1093/biomet/asaa052)
for the proof of the latter result in the case of mean bias reduction
for logistic regression (and, for more general binomial-response models
where the likelihood is penalized by a power of the Jeffreys invariant
prior).

**brglm2** also provides *pre-fit* and *post-fit* methods for the
detection of separation and of infinite maximum likelihood estimates in
binomial response generalized linear models (see `?detect_separation`
and `?check_infinite_estimates`).

Installation
------------

Install the development version from github:

    # install.packages("devtools")
    remotes::install_github("ikosmidis/brglm2")

Example
-------

Below we follow the example of [Heinze and Schemper
(2002)](https://doi.org/10.1002/sim.1047) and fit a logistic regression
model using maximum likelihood (ML) to analyze data from a study on
endometrial cancer (see `?brglm2::endometrial` for details and
references).

    library("brglm2")
    data("endometrial", package = "brglm2")
    modML <- glm(HG ~ NV + PI + EH, family = binomial(), data = endometrial)
    summary(modML)
    #> 
    #> Call:
    #> glm(formula = HG ~ NV + PI + EH, family = binomial(), data = endometrial)
    #> 
    #> Deviance Residuals: 
    #>      Min        1Q    Median        3Q       Max  
    #> -1.50137  -0.64108  -0.29432   0.00016   2.72777  
    #> 
    #> Coefficients:
    #>               Estimate Std. Error z value Pr(>|z|)    
    #> (Intercept)    4.30452    1.63730   2.629 0.008563 ** 
    #> NV            18.18556 1715.75089   0.011 0.991543    
    #> PI            -0.04218    0.04433  -0.952 0.341333    
    #> EH            -2.90261    0.84555  -3.433 0.000597 ***
    #> ---
    #> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    #> 
    #> (Dispersion parameter for binomial family taken to be 1)
    #> 
    #>     Null deviance: 104.903  on 78  degrees of freedom
    #> Residual deviance:  55.393  on 75  degrees of freedom
    #> AIC: 63.393
    #> 
    #> Number of Fisher Scoring iterations: 17

The ML estimate of the parameter for `NV` is actually infinite, as can
be quickly verified using the **detectseparation** R package

    # install.packages("detectseparation")
    library("detectseparation")
    update(modML, method = "detect_separation")
    #> Implementation: ROI | Solver: lpsolve 
    #> Separation: TRUE 
    #> Existence of maximum likelihood estimates
    #> (Intercept)          NV          PI          EH 
    #>           0         Inf           0           0 
    #> 0: finite value, Inf: infinity, -Inf: -infinity

The reported, apparently finite estimate
`r round(coef(summary(modML))["NV", "Estimate"], 3)` for `NV` is merely
due to false convergence of the iterative estimation procedure for ML.
The same is true for the estimated standard error, and, hence the value
0.011 for the *z*-statistic cannot be trusted for inference on the size
of the effect for `NV`.

Many of the estimation methods implemented in **brglm2** not only return
estimates with improved frequentist properties (e.g. asymptotically
smaller mean and median bias than what ML typically delivers), but also
return finite estimates and estimated standard errors in binomial
(e.g. logit, probit, and complementary log-log regression) and
multinomial regression models (e.g. baseline category logistic
regression for nominal responses and adjacent category logit models for
ordinal responses). For example, the code chunk below refits the model
on the endometrial cancer study data using mean bias reduction.

    summary(update(modML, method = "brglm_fit"))
    #> 
    #> Call:
    #> glm(formula = HG ~ NV + PI + EH, family = binomial(), data = endometrial, 
    #>     method = "brglm_fit")
    #> 
    #> Deviance Residuals: 
    #>     Min       1Q   Median       3Q      Max  
    #> -1.4740  -0.6706  -0.3411   0.3252   2.6123  
    #> 
    #> Coefficients:
    #>             Estimate Std. Error z value Pr(>|z|)    
    #> (Intercept)  3.77456    1.48869   2.535 0.011229 *  
    #> NV           2.92927    1.55076   1.889 0.058902 .  
    #> PI          -0.03475    0.03958  -0.878 0.379914    
    #> EH          -2.60416    0.77602  -3.356 0.000791 ***
    #> ---
    #> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    #> 
    #> (Dispersion parameter for binomial family taken to be 1)
    #> 
    #>     Null deviance: 104.903  on 78  degrees of freedom
    #> Residual deviance:  56.575  on 75  degrees of freedom
    #> AIC: 64.575
    #> 
    #> Number of Fisher Scoring iterations: 6

A quick comparison of the output from mean bias reduction to that from
ML reveals a dramatic change in the *z*-statistic for `NV`. The evidence
against the null of “NV” not contributing to the model in the presence
of the other covariates is now much stronger.

See `?brglmFit` and `vignettes(package = "brglm2")` for more examples
and the other estimation methods for generalized linear models,
including median bias reduction and maximum penalized likelihood with
Jeffreys’ prior penalty.

Solving adjusted score equations using quasi-Fisher scoring
-----------------------------------------------------------

The workhorse function in **brglm2** is
[`brglmFit`](https://github.com/ikosmidis/brglm2/blob/master/R/brglmFit.R),
which can be passed directly to the `method` argument of the `glm`
function. `brglmFit` implements a quasi [Fisher
scoring](https://en.wikipedia.org/wiki/Scoring_algorithm) procedure,
whose special cases result in a range of explicit and implicit bias
reduction methods for generalized linear models. Bias reduction for
multinomial logistic regression (nominal responses) can be performed
using the function `brmultinom`, and for adjacent category models
(ordinal responses) using the function `bracl`. Both `brmultinom` and
`bracl` rely on `brglmFit`.

The [iteration
vignette](https://cran.r-project.org/package=brglm2/vignettes/iteration.html)
and [Kosmidis et al (2019)](https://doi.org/10.1007/s11222-019-09860-6)
present the iteration and give mathematical details for the
bias-reducing adjustments to the score functions for generalized linear
models.

The classification of bias reduction methods into explicit and implicit
is as given in [Kosmidis (2014)](https://doi.org/10.1002/wics.1296).

References and resources
------------------------

**brglm2** was presented by [Ioannis Kosmidis](http://www.ikosmidis.com)
at the useR! 2016 international conference at University of Stanford on
16 June 2016. The presentation was titled “Reduced-bias inference in
generalized linear models” and can be watched online at this
[link](https://channel9.msdn.com/Events/useR-international-R-User-conference/useR2016/brglm-Reduced-bias-inference-in-generalized-linear-models).

Motivation, details and discussion on the methods that **brglm2**
implements are provided in

Kosmidis, I, Kenne Pagui, E C, Sartori N. (2020). Mean and median bias
reduction in generalized linear models. [*Statistics and
Computing*](https://doi.org/10.1007/s11222-019-09860-6) *30*, 43–59.
