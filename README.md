# StataExamples

In this teaching page we provide some useful examples for implementation in Stata depending on the application of interest. 

### [A]. Introduction to Econometrics 

We begin by considering a small simulation study for the parameter of a linear regression model below

$$ y_i = \beta x_i + e_i, \ \ \ \text{for} \ \ i = 1,...,n.$$

```Stata

set seed 1234
clear all

% We define a Stata program as below
program define mcexample, rclass
drop _all

set obs 100
gen e = rnormal(0,1)
gen x = rnormal(0,1)

gen y = x + e
reg y x 

return scalar beta  = _b[x]
return scalar se = _se[x]

end
% end of Stata program

% Now we can run our Stata program as defined below

simulate beta=r(beta) se=r(se), reps(30): mcexample

```

### [B]. Applied Time Series Econometrics


### [C]. Resampling Techniques
