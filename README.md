# xtivdfreg
Stata code for IV Estimaton of Panel Data Models with Unobserved Common Factors

## Install
This is a development version, to download it and use, open Stata and run the following:

```
net install xtivdfreg, from(http://www.kripfganz.de/stata/) replace
```

## Description
This repository contains the Stata package xtivdfreg, which implements the Instrumental Variables approach of Cui, Norkute, Sarafidis and Yamagata (2021, 2022), for the estimation of panel data models with unobserved common factors. 
<br />
For a theorertical exposition of this approach, please refer to the following video and the references therein: [Panel Data Regression with Unobserved Factors: Novel IV Estimation & Stata Illustration (Part 1)](https://www.youtube.com/watch?v=VHX7zM2ui_I).
<br />
For an illustration of the Stata package itself, please refer to the following video: [Panel Data Regression with Unobserved Factors: Novel IV Estimation & Stata Illustration (Part 2)](https://www.youtube.com/watch?v=eCrXxa8u4JU).

<br />

## Illustration of Instrumentation Capabilities using a Balanced Panel

We make use of panel data from a random sample of 300 U.S. banks, each one observed over 56 quarters. Therefore, N=300 and T=56. I will use this dataset for two purposes: (i) to demonstrate how versatile and flexible our Stata package is regarding instrumentation; and (ii) in order to present an example with a regressor that is endogenous w.r.t. both error components, e.g. due to reverse causality.


Load into memory the (Stata-format) banking dataset from the web:

```
use http://www.kripfganz.de/stata/xtivdfreg_example
```

id and t denote the bank and time period identifiers, ROA stands for return on assets, CAR stands for capital adequacy ratio, liquidity and size are self-explanatory, and finally ROE stands for return on equity.

<br />
The simplest possible specification for the instruments involves using the same number of lags for all covariates as instruments:

```
xtivdfreg L(0/1).CAR size ROE liquidity, absorb(id t) iv(size ROE liquidity, lags(2))
```

The option absorb(.) transforms the model in terms of deviations from individual-specific and time-specific averages. This is in order to eliminate bank fixed effects and common time effects. Essentially, common factors are estimated conditional on bank fixed effects and time effects.
The option iv(.) combines all covariates together, specifies the same number of lags, and estimates jointly the factors that influence all these variables, for each lag separately. By default, the package estimates up to 4 factors and then selects the optimal number of factors based on the Eigenvalue ratio test of Ahn and Horenstein (2013). The default value can be overruled by adding the suboption factmax(#), where hash is greater than 4.

<br />
A different number of lags for each instrument can be specified as follows:

```
xtivdfreg L(0/1).CAR size ROE liquidity, absorb(id t) iv(size, lags(1))  iv(ROE liquidity, lags(2))
```

<br />
The number of factors extracted from each set of instruments can be set equal to a pre-specified value. If we wish to impose estimating 2 factors for size, we type

```
xtivdfreg L(0/1).CAR size ROE liquidity, absorb(id t) iv(size, lags(1) factmax(2) noeigratio) iv(ROE, lags(2)) iv(liquidity, lags(2)) 
```

Here two extra suboptions are included. Firstly, the maximum number of factors is set equal to 2, and then the algorithm is instructed not to use the Eigenvalue ratio test.


<br />
The number of factors can be set equal to zero as follows:

```
xtivdfreg L(0/1).CAR size ROE liquidity, absorb(id t) iv(size, lags(1) factmax(0) noeigratio) iv(ROE, lags(2)) iv(liquidity, lags(2))
```

<br />
One can extract factors based on a subset of covariates, and then defactor all variables based on those factors only. To achieve that the suboption fvar(.) is included:

```
xtivdfreg L(0/1).CAR size ROE liquidity, absorb(id t) iv(size ROE liquidity, lags(2) fvar(ROE liquidity)) 
```

<br />
One can also standardize the variables within the iv(.) option before the factors are extracted. To achieve this,the suboption std is included

```
xtivdfreg L(0/1).CAR size ROE liquidity, absorb(id t) iv(size, lags(1) std)  iv(ROE liquidity, lags(2)) 
```

#### Dealing with endogeneity w.r.t. the idiosyncraatic error

<br />
As an example, ROE is instrumented by ROA as follows:

```
xtivdfreg L(0/1).CAR size ROE liquidity, absorb(id t) iv(size, lags(1) std)  iv(ROE liquidity, lags(2)) 
```

<br />

## Illustration of Further Capabilities using an Unbalanced Panel
<br />
We make use of the macro-panel dataset of Eberhardt and Teal (2010) for estimating cross-country production functions in the manufacturing sector. The dataset is unbalanced. Below we explore 3 additional features: (i) how to test for uncorrelatedness of covariates w.r.t. the factors; (ii) how to deal with observed factors; and how to estimate heterogeneous models.

```
xtivdfreg lY lL lK, absorb(list year) iv(lL lK, lags(3)) factmax(3)
```

#### Testing for uncorrelatedness of covariates w.r.t. the unobserved common factors using a Hausman test

```
xtivdfreg lY lL lK, absorb(list year) iv(lL, lags(2)) iv(lK, lags(3)) factmax(3)
estimates store TSIV_robust
xtivdfreg lY lL lK, absorb(list year) iv(lL, lags(2)) iv(lK, lags(3) factmax(0)) factmax(3)
estimates store TSIV_efficient
hausman TSIV_robust TSIV_efficient
```

#### Dealing with observed factors

```
xtline lY if list<=12, xlabel(1970(5)2002, labsize(small) angle(45))
gen trend=year-1970+1
xtivdfreg lY lL lK trend, absorb(list) iv(lL, lags(2)) iv(lK, lags(3)) iv(trend, lags(0) factmax(0)) factmax(3)
```

#### Estimation of heterogeneous panels

```
xtivdfreg lY lL lK, absorb(list year) iv(lL, lags(2)) iv(lK, lags(3)) factmax(3) mg
xtivdfreg lY lL lK, absorb(list year) iv(lL, lags(2)) iv(lK, lags(3)) factmax(3) mg(1)
xtivdfreg lY lL lK, absorb(list year) iv(lL, lags(2)) iv(lK, lags(3)) factmax(3) mg
matrix beta_i =e(b_mg)
matrix list beta_i
We can also extract the corresponding standard errors by typing
matrix se_i =e(se_mg)
matrix list se_i
```

#### Estimation of heterogeneous panels with observed factors

```
xtivdfreg lY lL lK trend, absorb(list) iv(lL, lags(2)) iv(lK, lags(3)) iv(trend, lags(0) factmax(0) nodoubledefact) factmax(3) mg
```


## References
Cui, G., Norkute, M., Sarafidis, V., and T. Yamagata (2022). Two-Stage Instrumental Variable Estimation of Linear Panel Data Models with Interactive Effects, *The Econometrics Journal*, Vol. 25(2), 340-361.

Kripfganz, S., and V. Sarafidis (2021). Instrumental-variable estimation of large-T panel-data models with common factors, *The Stata Journal*, Vol. 21(3), 1-28.

Norkute, M., Sarafidis, V., Yamagata, T. and G. Cui (2021). Instrumental Variable Estimation of Dynamic Linear Panel Data Models with Defactored Regressors and a Multifactor Error Structure, *Journal of Econometrics*, Vol. 220(2), 416-446.
