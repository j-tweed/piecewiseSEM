# Piecewise Structural Equation Modeling

  Implementation of piecewise structural equation modeling (SEM) in R, including estimation of path coefficients and goodness-of-fit statistics. 
  
  For more information, see: 

    Shipley, Bill. "Confirmatory path analysis in a generalized multilevel context." Ecology 90.2 (2009): 
    363-368.

    Shipley, Bill. "The AIC model selection method applied to path analytic models compared using a 
    d-separation test." Ecology 94.3 (2013): 560-564.

Version: 0.1.4 (2014-08-04)

Author: Jon Lefcheck <jslefche@vims.edu>

##Examples

###Load package

```
# library(devtools)
# install_github("piecewiseSEM", "jslefche")
library(piecewiseSEM)
```

###Load data from Shipley 2009

```
data(Shipley)
```
The data is alternately hosted in Ecological Archives E090-028-S1 (DOI: 10.1890/08-1034.1).

###Create model set

The model corresponds to the following hypothesis (Fig. 2, Shipley 2009);

![Shipley 2009 Fig. 2](http://www.esajournals.org/na101/home/literatum/publisher/esa/journals/content/ecol/2009/00129658-90.2/08-1034.1/production/images/large/i0012-9658-90-2-363-f02.jpeg)

Models are constructed using a mix of the `nlme` and `lmerTest` packages, as in the supplements of Shipley 2009. 

```
# Load required libraries
library(lmerTest)
library(nlme)

Shipley.modlist = list(
  lme(DD~lat, random = ~1|site/tree, na.action = na.omit, 
  data = Shipley),
  lme(Date~DD, random = ~1|site/tree, na.action = na.omit, 
  data = Shipley),
  lme(Growth~Date, random = ~1|site/tree, na.action = na.omit, 
  data = Shipley),
  glmer(Live~Growth+(1|site)+(1|tree), 
  family=binomial(link = "logit"), data = Shipley) )
```


###Run Shipley tests

`get.sem.fit` returns a list of the following:
(1) the missing paths, whether these paths are conditional on any other variables in the model, and associated p-values;
(2) the Fisher's C statistic and p-value for the model (derived from a Chi-squared distribution);
(3) the AIC and AICc (corrected for small sample size) for the model.

The argument `add.vars` allows you to specify a vector of additional variables whose causal independence you also wish to test. This is useful if you are comparing nested models. Default is `NULL`.

The argument `adjust.p` allows you to adjust the p-values returned by the function based on the the total d.f. for the model (see supplementary material, Shipley 2013). Default is `FALSE` (uses the d.f. reported in the summary table).

(See ["p-values and all that"](https://stat.ethz.ch/pipermail/r-help/2006-May/094765.html) for a discussion of p-values from mixed models using the `lmer` package.)

```
get.sem.fit(Shipley.modlist)
```

The missing paths output differs from Table 2 in Shipley 2009. However, running each d-sep model by hand yields the same answers as this function, leading me to believe that updates to the `lme4` and `nlme` packages are the cause of the discrepancy. Qualitatively, the interpretations are the same.

###Extract path coefficients

Path coefficients can be either unstandardized or standardized (in units of standard deviation of the mean). Default is `FALSE`. The function returns a list of length up to 2, corresponding to non-significant and/or significant paths based. Default is `alpha = 0.05`.

```
get.sem.coefs(Shipley.modlist)
```

###Generate variance-covariance SEM using `lavaan`

Generate variance-covariance based SEM from the list of linear mixed models. The resulting object can be treated like any other model object constructed using the package `lavaan`.

```
lavaan.model = get.lavaan.sem(Shipley.modlist)
summary(lavaan.model)
```
The output shows that the variance-covariance SEM is a worse fit, indicating that a hierarchical piecewise approach is justified.