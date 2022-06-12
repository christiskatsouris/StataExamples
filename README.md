# StataExamples

In this teaching page we provide some useful examples for implementation in Stata depending on the application of interest. 

### [A]. Introduction to Econometrics 

We begin by considering a small simulation study for the parameter of a linear regression model below

$$ y_i = \beta x_i + e_i, \ \ \ \text{for} \ \ i = 1,...,n.$$

```Stata

set seed 1234
clear all

// We define a Stata program as below
program define mcexample, rclass
drop _all

set obs 50
gen e = rnormal(0,1)
gen x = rnormal(0,1)

gen y = beta*x + e
reg y x 

return scalar beta  = _b[x]
return scalar se = _se[x]

end
// end of Stata program

// Now we can run our Stata program using the build-in Stata function "simulate"
// Each sample is of size n = 50, while we repeat the above for B = 100 times

simulate beta=r(beta) se=r(se), reps(100): mcexample

```

### [B]. Applied Time Series Econometrics

```Stata

// Time series time is set as the increasing number of observations
gen t=_n
tsset t

// Create the time series sequence for lag 1 
gen SP500_lag = l.SP500

// Create Actural Return time series
gen return_SP500 = ( SP500 - SP500_lag) / ( SP500_lag)

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

### [C]. Resampling Techniques
