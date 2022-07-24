# StataExamples

In this teaching page we provide some useful examples for implementation in Stata depending on the application of interest. 

# [A]. Introduction to Econometrics 

```Stata

//----------------------------------------------------
// Creating variables and Summary Statistics in STATA
//----------------------------------------------------

// We can create a new categorical variable
// The categorical variable xcat takes the values 1, 2 and 3 as defined below 

generate xcat  = 2
replace  xcat  = 1 if ( x < 200 )
replace  xcat  = 3 if ( x > 500 )

// Obtain Summary statistics
summarize
summarize, detail

// Obtain a table for the categorical variable xcat
tab xcat

// We can ontain a summary statistics table for xcat which corresponds to x only
table xcat, contents (n y mean y sum y sd y) row

// We can ontain a summary statistics table for xcat which corresponds to y only
table xcat, contents (n x mean x sum x sd x) row

// Creating Graphs
graph twoway scatter y x

// Obtain the correlation coefficient between y and x
correlate y x

// Obtain a regression model for y on x
regress y x

//----------------------------------------------------
// Creating a log-file in STATA to save commands
//----------------------------------------------------

log using workshop.log
infile y x using hospital.txt, clear
label variable y "Number of patients discharged"
label variable x "Number of beds"

sort x

egen xstrat1    = cut(x), group(5)
replace xstrat1 = xstrat1 + 1
tab xstrat1

//----------------------------------------------------
// Simple Random Sampling from a Finite Population
//----------------------------------------------------

// Step 1: Obtain a Simple Random Sample of size n = 32 from the population of hospitals, N = 393 
sample 32, count

// Step 2:  Specifying the sample design
generate n  = 32
generate N  = 393
generate sampfrac = n/N
generate weight   = N/n
svyset [pweight = weight], fpc(sampfrac)

// Creating Estimates of Population Quantities
svy: total y


```

## Example 1: Simple Linear Regression 

We begin by considering a small simulation study for the parameter of a linear regression model below

$$ y_i = \beta x_i + e_i, \ \ \ \text{for} \ \ i = 1,...,n.$$

Suppose that $e_i$ is an i.i.d sequence of random variables that have a N(0,1) distribution. Write a short programme in Stata for a simulation study using the following parameters

$$n = 50, \ \ \beta = ( 0.5, 0.7, 0.9 ) \ \ \ \text{and} \ \ B = 500.$$

```Stata

set seed 1234
clear all

// We define a Stata program as below
program define mcexample, rclass
drop _all

set obs 50
generate e = rnormal(0,1)
generate x = rnormal(0,1)

generate y = beta*x + e
regress y x 

return scalar beta  = _b[x]
return scalar se = _se[x]

end
// end of Stata program

// Now we can run our Stata program using the build-in Stata function "simulate"
// Each sample is of size n = 50, while we repeat the above for B = 500 times

simulate beta=r(beta) se=r(se), reps(500): mcexample
count if (t < - 1.96) | (t > 1.96)

```

### Remarks:

1. A small simulation study allows us to calcuate the empirical size and empirical power of a test statistic (such as the t-test for a coefficient of the linear regression). In general, the significance level (alpha) of a test or size of the test is given by alpha = P(Type I Error), that is, the probability of doing a Type I Error i.e., a = P(reject H_0 when H_0 is true). Therefore, specifically when using the t-test as a test function to assess the statistical significance of the slope coefficient of the SLR, we basically test the following hypotheses: H0: beta = 0 versus H1: beta not equal to zero (for a two-sided test). Thus, in order for the covariate x to be a statistical significant predictor for the dependent variable y we aim to find statistical evidence that allow us to reject the null hypothesis.

2. To do this, we simulate a Data Generating Process (DGP) which represents the linear regression model. The way to generate the linear regression model we start with the error sequence which imposes the distributional assumption to the model, then we generate the x variable and finally y inherits the properties of the error term as explained in PS2 Q3, since the error is i.i.d normal Random Variable and therefore y1,...,yn is also assumed to be i.i.d. Normal with first and second moments estimated by taking the Expected Value and Variance, that is, E[y] and Var[y]. 

3. Therefore, the Monte Carlo simulation step requires to obtain an estimate of the t-test (e.g., t = beta_hat / s.e(beta_hat) ) in each replication. Then, the given value of the t-test which is considered to be a realization of the corresponding asymptotic distribution can be used to decide whether to accept or reject the null hypothesis, by appropriately (depending on the type of the test) comparing with the associated critical value or estimating the corresponding p-value. Finally, if the replication step is B = 100, then based on a significance level alpha = 5%, we should obtain the frequency of non-rejections of the null hypothesis to be close to the nominal size, which is the probability of a Type I Error to occur. 


## Example 2: First Order Autoregressive Regression 

Consider the first-order autoregressive time series model below

$$ y_i = \beta y_{i-1} + u_i, \ \ \ \text{for} \ \ i = 1,...,n.$$

Suppose that ui is an i.i.d sequence of random variables that have a N(0,1) distribution. Write a short programme in Stata for a simulation study using the following parameters

$$n = 200, \ \ \beta = ( 0.5, 0.7, 0.9 ) \ \ \ \text{and} \ \ B = 500.$$


```Stata

set seed 1234

program define myprog, rclass
// begin of Stata program

drop _all
set obs 100
generate u = rnormal(0,1)

generate y = 0 in 1 
forvalues j = 2/200{
replace y = 0.5*y[_n-1]+u in `j'
}

generate y_1=y[_n-1]
reg y y_1 , nocon
return scalar b  = _b[y_1]
return scalar se = _se[y_1]
return scalar t  = _b[y_1]/_se[y_1]
end
// end of Stata program

// Now we can run our Stata program using the build-in Stata function "simulate"
// Each sample is of size n = 200, while we repeat the above for B = 500 times

simulate b= r(b) se= r(se) t= r(t) , reps(500) : myprog

```

### Remarks:

- Both Example 1 and Example 2 which implement a short simulation study are very important in helping us understand the large sample properties of parameter estimates in a Simple Linear Regression model as well as in a first-order AR(1) regression model. Additionally, the main idea behind the construction of the above computer coding procedure provides some useful insights regarding the use of logical analysis and inductive reasoning which is also very useful when considering the validity of inference assumptions as well as the appropriatness of statistical models under different econometric conditions.

- According to [Bekkerman (2015)](https://wires.onlinelibrary.wiley.com/doi/10.1002/wics.1342) "Simulations provide an opportunity to fully design the data generation process, introduce aspects that mimic common empirical problems, and assess estimators' quality under numerous alternative conditions". 

Furthermore, many applications in classical econometric analysis require the use of multiple regression models. In particular, data applications and various examples from empirical studies based on the multiple regression model in Stata can be found in the  book of Wooldridge (2015). Here, we present some key applications for the purpose of illustration.  

## Example 3: Multiple Regression Analysis 

### Application 1: Wage regression

```Stata

// Consider the linear regression model with multiple regressors
// wage = b0 + b1 * female + b2 * educ + u

regress wage female educ

// Consider the variable 'brthord' as an IV for 'educ' in the simple wage regression
// log(wage) = b0 + b1 * educ

regress l_wage educ

// Consider the reduced form equation such that 
// educ = a0 + a1 * sibs + a2 * brthord + v

// Notice that we need that a2 to be different than zero so that the parameter beta in the structural econometric model to be identified. 
// Thus, we need to check whether the identification assumption holds by checking the t-test under the null hypothesis, H0: a2 = 0.

regress educ sibs brthord

```

### Application 2: Panel Data regression 

```Stata

// Consider the Crime dataset in Stata
// In particular we are interested to study the effect of Medical Marijuana Laws on Crime

egen Murder_crime_ave  = mean( Murder_crime),by(Year)
egen Violent_crime_ave = mean( Violent_crime),by(Year) 

```

### References

- Bekkerman, A. (2015). The role of simulations in econometrics pedagogy. Wiley Interdisciplinary Reviews: Computational Statistics, 7(2), 160-165.

- Morris, R. G., TenEyck, M., Barnes, J. C., & Kovandzic, T. V. (2014). The effect of medical marijuana laws on crime: evidence from state panel data, 1990-2006. PloS one, 9(3), e92816.


## Further Reading 

- Stock, J. & Watson, M. (2016). Introduction to econometrics. London: Addison-Wesley.
- Wooldridge, J.M. (2020). Introductory econometrics: A modern approach (7th ed.).
- Robert, C. P., Casella, G., & Casella, G. (2010). Introducing monte carlo methods with r (Vol. 18). New York: Springer.

Greek Bibliography:
- Δριτσάκη, Χ. Ν., & Δριτσάκη, Μ. Ν. (2013). Εισαγωγή στην Οικονομετρία με τη χρήση του λογισμικού EVIEWS. Εκδόσεις Κλειδάριθμος.


# [B]. Applied Econometrics

## Application B1: Resampling Techniques

Consider the Treatment Effect Linear Regression Model (TELRM), with no covariates given by

$$y_i = \gamma D_i + e_i, \ \ \ \text{for} \ \ i = 1,...,n,$$

where y represents a health outcome for the i-th survey participant and D is a binary variable which indicates the participation to a RCT study. We are particularly interested to assess the finite-sample validity of the treatment estimator. 

First we simulate a data generating process for the TELR model using the following code

```Stata

set obs 25
generate d = (rnormal(0,1)<0)
generate e = rnormal(0,1)
generate y = 0.8*d + e
regress y d

```

Second consider the TELRM under the experimental condition of covariates imbalance 

```Stata

generate d = (rnormal(0,1)<0)
generate X1 = rnormal(0,2)
generate rand = runiform()
generate X2 = (rand>0.7) + 1
generate e = rnormal(0,1)
generate y = 0.8*d + 0.20*X1 + 0.05*X2 + e

// The statistical validity of the treatment estimator can be assessed using a permuation test techique
permtest y, treat(D) np(`np') ipwcovars1(W)

```

Next consider a small simulation study to assess the validity of the treatment effect estimator for the TERLM with no covariates and Cauchy errors.

```Stata

cap program drop mcexample

program define mcexample, rclass
args gamma N
drop _all
set obs `N'
gen d = (rnormal(0,1)<0)
gen e = rcauchy(1,0.25)
gen y = `gamma'*d + e

permtest y, treat(d) np(1000)
mat MATmatrix1 = r(pval_asym1s)
mat MATmatrix2 = r(pval_perm)
return scalar pvalue1 = MATmatrix1[1,1]
return scalar pvalue2 = MATmatrix2[1,1]
end

simulate pvalue1=r(pvalue1) pvalue2=r(pvalue2), reps(1000) : mcexample 0.05 20
count if (pvalue1<=0.05)
count if (pvalue2<=0.05)

```

### Remarks:

'permtest' is a build-in function in Stata that implements a permutation test as a statistical siginificance technique to validate the treatment effect estimator. The ado file for the particular command can be found in the Github page of [permtest](https://github.com/masongcm/permtest). 

### References

- Freedman, D., & Lane, D. (1983). A nonstochastic interpretation of reported significance levels. Journal of Business & Economic Statistics, 1(4), 292-298.
- Katsouris, C. (2021). Treatment effect validation via a permutation test in Stata. arXiv preprint [arXiv:2110.12268](https://arxiv.org/abs/2110.12268).
- Romano, J. P., & Wolf, M. (2016). Efficient computation of adjusted p-values for resampling-based stepdown multiple testing. Statistics & Probability Letters, 113, 38-40.

## Application B2: Panel Data Estimation in Stata

Consider the following econometric specification which tests the electoral competition hypothesis

$$y_{it} = \delta_1 ( CPA_t . D_i) + \delta_2 (Z_{it} . CPA_t) + \delta_3 (CPA_t . D_i . Z_{it}) + \beta^{\prime} X_{it} + \eta_{t} + u_{i} + \epsilon_{it},$$

for i =1,...,N and t = 1,...,n, where y_it = ( T_it, Q_it, e_it ) and Z_it is a given variable in the dataset which measures the level of electoral competition for each council, during each time period. 

```Stata

// Datasets in Stata: 
// To import a dataset into Stata we can use either the .xls or .txt data format.
// Alternatively, we can use the .dta format which is already converted into the Stata data format.

// STEP 1: CONSTRUCTION OF VARIABLES

// Generating the aggregate indicator for the quality dependent variable Q
generate aggexp = (socialexprp + eduexprp + environexprp + corporatexprp)

// Sorting the data by area code i.e., 1,...,172
sort code

// Compute the time average of indicators for each council 
by code: egen avgaggexp        = mean(aggexp)
by code: egen avgsocialexprp   = mean(socialexprp)
by code: egen avgeduexprp      = mean(eduexprp)
by code: egen avgenvironexprp  = mean(environexprp)
by code: egen avgcorporatexprp = mean(corporatexprp)

// Compute the weights for each of the indicator as relative expenditures
generate w_bvpi38  = avgeduexprp / avgaggexp
generate w_bvpi54  = avgsocialexprp / avgaggexp
generate w_bvpi82a = avgenvironexprp / avgaggexp
generate w_bvpi8   = avgcorporatexprp / avgaggexp
generate aggout    = (bvpi38*w_bvpi38)+(bvpi54*w_bvpi54)+(bvpi82a*w_bvpi82a)+(bvpi8*w_bvpi8)
generate aggout_noedu = (bvpi54*w_bvpi54)+(bvpi82a*w_bvpi82a)+(bvpi8*w_bvpi8)

// STEP 2: FIX THE PANEL DATA STRUCTURE 

// Sort the panel by increasing order via code and year 
sort code year
iis code
tis year
xtdes, i(code) t(year)
tsset code year

// STEP 3: PANEL DATA ESTIMATIONS

// The following code implements a Panel Data Model with clustered robust standard errors
xi: xtreg taxreqrp england dummyCPAengland i.year lgrantrp ... lselfemployed, fe cluster(code)

```

### References

- Lockwood, B., & Porcelli, F. (2013). Incentive schemes for local government: Theory and evidence from comprehensive performance assessment in england. American Economic Journal: Economic Policy, 5(3), 254-86.

## Task 1

Based on one of the following studies, prepare a short replication study. Give emphasis on the data structure in each case, the econometric specification as well as suitable robustness checks that allow to examine the economic theory under investigation.    

[1] Egger, P., & Koethenbuerger, M. (2010). Government spending and legislative organization: Quasi-experimental evidence from Germany. American Economic Journal: Applied Economics, 2(4), 200-212.

[2] Gilchrist, D. S. (2016). Patents as a spur to subsequent innovation? Evidence from pharmaceuticals. American Economic Journal: Applied Economics, 8(4), 189-221.

[3] Miguel, E., & Kremer, M. (2004). Worms: identifying impacts on education and health in the presence of treatment externalities. Econometrica, 72(1), 159-217.


# [C]. Applied Time Series Econometrics

In Applied Time Series Econometrics applications before fitting time series models to stock prices we need to estimate the corresponding stock returns which transform the data into stationary sequences in the convetional sence. To do this, we need to apply the rule below 

$$R_t = \frac{ P_{t+1} - P_t }{ P_t }, \ \ \ \text{for} \ t = 1,...,n,$$

where n is the sample size of the time series under examination. 

```Stata

// Time series time is set as the increasing number of observations
generate t=_n
tsset t

// Create the time series sequence for lag 1 
generate SP500_lag = l.SP500

// Create Actural Return time series
generate return_SP500 = ( SP500 - SP500_lag) / ( SP500_lag)

// Plot time return series
twoway (line return_SP500 t)

// Create first difference time series (1st Method)
 generate diff_SP500 = ( SP500 - SP500_lag)
 
 // Create first difference time series (2nd Method)
 generate dSP500=D.SP500
 
 // Create log-return time series
 generate log_returns = log( SP500 / SP500_lag )
 
 // Plots
twoway (line SP500 t), title (Time Series Plot - S&P500 Prices)
twoway (line return_SP500 t), title (Time Series Plot - Actual Returns of S&P500 Prices)

```

## Application C1: Fitting Arch and Garch models in Stata

```Stata

gen time=_n
tsset time

// generate log-return
foreach x in DJIA SP500 Nasdaq {
gen ln_`x'=log(`x')
gen r_`x'=D.ln_`x'
}

foreach x in r_DJIA r_SP500 r_Nasdaq {
qui tsline `x', name(`x', replace)
}

graph combine r_DJIA r_SP500 r_Nasdaq, cols(3) name(log_returns, replace)

// Fitting univariate GARCH models to time series observations

// GARCH(1,1)
foreach x in DJIA SP500 Nasdaq {
arch r_`x' L.r_`x', arch(1) garch(1)
}

// T-GARCH 
foreach x in DJIA SP500 Nasdaq {
arch r_`x' L.r_`x', arch(1) garch(1) tarch(1)
}

// GARCH-M
foreach x in DJIA SP500 Nasdaq {
arch r_`x' L.r_`x', archm arch(1) garch(1)
}

// Fitting multivariate GARCH models to time series observations

// Constant conditional correlation (CCC)
mgarch ccc (r_DJIA r_SP500 r_Nasdaq = L.r_DJIA L.r_SP500 L.r_Nasdaq), arch(1) garch(1) nolog vsquish

// Dynamic conditional correlation (DCC)
mgarch dcc (r_DJIA r_SP500 r_Nasdaq = L.r_DJIA L.r_SP500 L.r_Nasdaq), arch(1) garch(1) nolog vsquish

// Varying conditional correlation (VCC)
mgarch vcc (r_DJIA r_SP500 r_Nasdaq = L.r_DJIA L.r_SP500 L.r_Nasdaq), arch(1) garch(1) nolog vsquish

```

An alternative Econometrics Software to R, Matlab and Stata is [Eviews](https://www.eviews.com/Learning/index.html) which provides an easy way for econometric analysis of time series data. Eviews has a plethora of online tutorials which makes it easy to understand the [estimation procedure](https://www.eviews.com/Learning/timeseries_a.html) in relation to the econometric theory. An an illustrative example, below we consider the [NFCI](https://fred.stlouisfed.org/series/NFCI) series and its components. 

```Stata

// Time series regression models 
// Fitting an AR(1) regression model with an intercept 

equation eq1.ls(cov=hac) series02 c series02(-1) 

// Fit an AR(1) regression model with an intercept to the components of the NFCI series 
equation eq1.ls(cov=hac) nfci c nfci(-1) 
equation eq2.ls(cov=hac) risk c risk(-1) 
equation eq3.ls(cov=hac) credit c credit(-1) 
equation eq4.ls(cov=hac) leverage c leverage(-1) 

// Apply standard retrospective structural break tests 
eq1.breakls(method=glob, maxbreaks=3) nfci c nfci(-1)
eq2.breakls(method=glob, maxbreaks=3) risk c risk(-1) 
eq3.breakls(method=glob, maxbreaks=3) credit c credit(-1) 
eq4.breakls(method=glob, maxbreaks=3) leverage c leverage(-1)

###########################################################

// Fitting a GARCH(1,1) MODEL to the NFCI series 

equation eq1.arch(1,1,h) ciss c ar(1)
equation eq2.arch(1,1,h) money c ar(1)
equation eq3.arch(1,1,h) finance c ar(1)
equation eq4.arch(1,1,h) bond c ar(1)
equation eq5.arch(1,1,h) equity c ar(1)
equation eq6.arch(1,1,h) exchange c ar(1)

###########################################################

// Unit Root testing 

nfci.uroot(adf,trend)
risk.uroot(adf,trend)
credit.uroot(adf,trend)
leverage.uroot(adf,trend)

nfci.buroot(adf,trend)
risk.buroot(adf,trend)
credit.buroot(adf,trend)
leverage.buroot(adf,trend)

```

# Disclaimer

The author declares no conflicts of interest. 

The above sections correspond to various useful coding procedures implemented in Stata that are suitable for related Econometrics and Time Series Econometics courses.  
