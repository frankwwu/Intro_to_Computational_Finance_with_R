# Lab 6: Constant expected return model

https://campus.datacamp.com/courses/computational-finance-and-financial-econometrics-with-r/lab-6-constant-expected-return-model?ex=1

## Download the data and calculate the returns

Do you still remember from the last lab how to download financial data using the <code>get.hist.quote()</code> function from the <code>tseries</code> package? Have a look at the code on the right as a recap:

* It downloads the data and changes the class of the time index of the three price series to "yearmon".
* The three price series are merged into the <code>all_prices</code> variable.
* The continuously compounded returns are computed and assigned to the variable <code>all_returns</code>.
* Finally, all return data is also stored in the <code>return_matrix</code> variable.

### Instructions

* Make sure that you understand the code on the right and click 'Submit Answer' to run the code and start the lab.


```r
library(PerformanceAnalytics)
library(zoo)
library(tseries)

# Download price data
VBLTX_prices <- get.hist.quote(instrument = "vbltx", start = "2005-09-01", end = "2010-09-30", quote = "AdjClose", provider = "yahoo", origin = "1970-01-01", compression = "m", retclass = "zoo", quiet = TRUE)
FMAGX_prices <- get.hist.quote(instrument = "fmagx", start = "2005-09-01", end = "2010-09-30", quote = "AdjClose", provider = "yahoo", origin = "1970-01-01", compression = "m", retclass = "zoo", quiet = TRUE)
SBUX_prices <- get.hist.quote(instrument = "sbux", start = "2005-09-01", end = "2010-09-30", quote = "AdjClose", provider = "yahoo", origin = "1970-01-01", compression = "m", retclass = "zoo", quiet = TRUE)
index(VBLTX_prices) <- as.yearmon(index(VBLTX_prices))
index(FMAGX_prices) <- as.yearmon(index(FMAGX_prices))
index(SBUX_prices) <- as.yearmon(index(SBUX_prices))

# Create merged price data
all_prices <- merge(VBLTX_prices, FMAGX_prices, SBUX_prices)
colnames(all_prices) <- c("VBLTX", "FMAGX", "SBUX")

# Calculate cc returns as difference in log prices
all_returns <- diff(log(all_prices))
return_matrix = all_returns
```

## The standard error of the variances

The variable <code>return_matrix</code> is already loaded into your environment. This variable contains the continuously compounded returns of each <code>VBLTX_prices</code>, <code>FMAGX_prices</code>, and <code>SBUX_prices</code>.

Consider the constant expected return model (CER):

$$R_{it}=\mu_i+\epsilon_{it},$$
$$t=1,...,T$$
$$\epsilon_{it} \sim iid N(0,\sigma_i^2)$$
$$cov(\epsilon_{it},\epsilon_{jt})=\sigma_{ij},$$
where $R_{it}$ denotes the continuously compounded return on asset $i$, with $i$ equal to the Vanguard long term bond index fund (VBLTX), Fidelity Magellan stock mutual fund (FMAGX), Starbucks stock (SBUX). The model thus assumes that the returns of these assets are normally distributed and that the time series of these continuously compounded returns are covariance stationary.

The parameters of the above CER model are of course unknown to us. It is your task now to estimate the model parameters $\sigma_i^2$ for the different assets. The data that you constructed in the previous exercise is given. Remember from the course that an estimator for $\sigma_i^2$ is just the sample variance.

Once you have these estimates, your second task is to investigate the precision of the estimates. More specifically, you should estimate the standard errors in order to get $\hat{SE}(\hat{\sigma_i^2})$.

The formula is given by:

$$\hat{SE}(\hat{\sigma_i^2})=\sigma_i^2/\sqrt{T/2}.$$

### Instructions

* Assign to <code>n_obs</code> the number of observations in the matrix <code>return_matrix</code>.
* Assign to <code>sigma2hat_vals</code> the estimates of $\sigma_i^2$ for the three different assets.
* Assign to <code>se_sigma2hat</code> the estimates of ${SE}(\hat{\sigma_i^2})$ and print the result to the console.


```r
# The variable return_matrix is preloaded in your workspace

# Number of observations
n_obs <- dim(return_matrix)[1]

# Estimates of sigma2hat
sigma2hat_vals <- apply(return_matrix, 2, var)

# Standard Error of sigma2hat
se_sigma2hat <- sigma2hat_vals/sqrt(n_obs/2)
se_sigma2hat 
```

```
##        VBLTX        FMAGX         SBUX 
## 0.0001599251 0.0011589337 0.0019640262
```

## Estimate the standard error of the correlation parameter

Still consider the constant expected return model (CER) introduced in the previous exercise. Correlations indicate the strength of the dependency between two variables. You are now interested in estimates of the correlations $\rho$ between the returns of the different assets $i$ and $j$. Furthermore, you would like to investigate the precision of these estimates by calculating the standard error of $\hat{\rho_{ij}}$, which will be used for inference in later exercises.

Recall that the estimated SE values are computed using the analytic formula:
$$\hat{SE}(\hat{\rho_{ij}})=(1-\hat{\rho_{ij}}^2)/\sqrt{T}.$$

### Instructions

* Assign to <code>cor_matrix</code> the correlation matrix of the returns.
* Assign to <code>rhohat_vals</code> the estimates of the correlations between "VBLTX, FMAGX", "VBLTX, SBUX", "FMAGX, SBUX".
* Assign to <code>se_rhohat</code> the estimates of ${SE}(\hat{\rho_{ij}})$ and print the result to the console.


```r
# The variable return_matrix is preloaded in your workspace

# Calculate the correlation matrix
cor_matrix <- cor(return_matrix)

# Get the lower triangular part of that 'cor_matrix'
rhohat_vals <- cor_matrix[lower.tri(cor_matrix)]

# Set the names
names(rhohat_vals) <- c("VBLTX,FMAGX","VBLTX,SBUX","FMAGX,SBUX")

# Compute the estimated standard errors for correlation
se_rhohat <- (1-rhohat_vals^2)/sqrt(dim(return_matrix)[1])
se_rhohat
```

```
## VBLTX,FMAGX  VBLTX,SBUX  FMAGX,SBUX 
##   0.1250522   0.1266690   0.0897495
```

## Hypothesis test for the mean

Still consider the constant expected return model (CER) that was introduced in exercise 2.

You would like to test for each $\mu_i$ ($i=$ VBLTX, FMAGX and SBUX):

$$H_0:\mu_i=0 \quad vs. \quad H_1:\mu_i\neq0,$$

using a 5% significance level. In other words, you would like to investigate whether the mean return is significantly different from zero according to the data. Perform the test using the t-statistic as well as the 95% confidence. You can use the R function <code>t.test()</code> for this problem.

### Instructions

Use the <code>t.test</code> to perform the t-test for $\mu_{VBLTX}$ and print the result to the console. What do you conclude?


```r
# The all_returns zoo object is preloaded in your workspace
t.test(x=all_returns[,"VBLTX"], y=all_returns[,"FMAGX"])
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  all_returns[, "VBLTX"] and all_returns[, "FMAGX"]
## t = 0.61597, df = 74.979, p-value = 0.5398
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -0.01509966  0.02861705
## sample estimates:
##     mean of x     mean of y 
##  0.0059512560 -0.0008074378
```

## Interpretation of the hypothesis test for the mean

Still consider the constant expected return model (CER) that was introduced in exercise 2. Test for $\mu_{FMAGX}$:

$$H_0:\mu_{FMAGX}=0 \quad vs. \quad H_1:\mu_{FMAGX}\neq0$$
using a 5% significance level.

Can you reject the null hypothesis that the mean is equal to zero?

Note that the <code>all_returns</code> zoo object is preloaded in your workspace.

### Possible Answers

Perform the t-test on the FMAGX returns and have a look at the P-values.


```r
t.test(all_returns[,"FMAGX"])
```

```
## 
## 	One Sample t-test
## 
## data:  all_returns[, "FMAGX"]
## t = -0.078501, df = 59, p-value = 0.9377
## alternative hypothesis: true mean is not equal to 0
## 95 percent confidence interval:
##  -0.02138908  0.01977421
## sample estimates:
##     mean of x 
## -0.0008074378
```

```r
# No
```

## Hypothesis test for the correlation

Still consider the constant expected return model (CER) that was introduced in exercise 2.

You would like to test for each $\rho_i$ ($ij=$ "VBLTX, FMAGX", "VBLTX, SBUX" and "FMAGX, SBUX"):
$$H_0:\mu_i=0 \quad vs. \quad H_1:\mu_i\neq0,$$
using a 5% significance level. In other words, you would like to investigate whether the correlation between two return series is significantly different from zero according to the data. Perform the test for correlation between paired samples at the 95% confidence level. You can use the R function <code>cor.test()</code> for this problem.

### Instructions

Use the <code>cor.test()</code> function to test whether the correlation between the returns of "VBLTX" and "FMAGX" is significantly different from zero. What do you conclude?



```r
# The all_returns zoo object is preloaded in your workspace

# Test the correlation between VBLTX, FMAGX
cor.test(x=all_returns[,"VBLTX"], y=all_returns[,"FMAGX"])
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  all_returns[, "VBLTX"] and all_returns[, "FMAGX"]
## t = 1.3701, df = 58, p-value = 0.1759
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.08048534  0.41243960
## sample estimates:
##       cor 
## 0.1770579
```

## Interpretation of the hypothesis test for correlation

Still consider the constant expected return model (CER) that was introduced in exercise 2. Test for $\rho_{VBLTX,SBUX}$:

$$H_0:\rho_{VBLTX,SBUX}=0 \quad vs. \quad H_1:\rho_{VBLTX,SBUX} \neq 0,$$
using a 5% significance level.

Can you reject the null hypothesis that the correlation between the returns of VBLTX and SBUX is equal to zero?

Note that the <code>all_returns</code> zoo object is preloaded in your workspace.

### Possible Answers

Perform the test for correlation between paired samples at the 95% confidence level and have a look at the P-value.


```r
cor.test(x=all_returns[,"VBLTX"], y=all_returns[,"FMAGX"])
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  all_returns[, "VBLTX"] and all_returns[, "FMAGX"]
## t = 1.3701, df = 58, p-value = 0.1759
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.08048534  0.41243960
## sample estimates:
##       cor 
## 0.1770579
```

```r
# No
```

## Normality of the asset returns

Remember that the Constant Expected Return model assumes that returns are normally distributed. Is that a reasonable assumption? The Jarque Bera test provides a way to answer that question. You can easily perform the Jarque Bera test for normality in R with the <code>jarque.bera.test</code> function.

Let us say that you want to investigate whether it is reasonable that the returns of VBLTX are normally distributed. More formally, you would like to test the null hypothesis:

$$H_0:r_{it}\sim normal \quad vs. \quad H_1:r_{it}\sim not normal$$
using a 5% significance level, with $i=VBLTX$.

### Instructions
Use the <code>jarque.bera.test</code> to test the normality of the "VBLTX" returns and print the output to the console. What do you conclude?



```r
# The all_returns zoo object is preloaded in your workspace

# Test the normality of the returns of VBLTX
jarque.bera.test(all_returns[,"VBLTX"])
```

```
## 
## 	Jarque Bera Test
## 
## data:  all_returns[, "VBLTX"]
## X-squared = 21.63, df = 2, p-value = 2.009e-05
```

## Interpretation of the normality test

Is it reasonable to assume that the returns of $FMAGX$ were drawn from a normal distribution?

Or more formally, do you reject the null hypothesis:

$$H_0:r_{it}\sim normal \quad vs. \quad H_1:r_{it}\sim not normal$$
using a 5% significance level, with $i=FMAGX$?

Note that the <code>all_returns</code> zoo object is preloaded in your workspace.

### Possible Answers


```r
# Yes
```

## Bootstrapping

In the previous exercise you rejected the null hypothesis that the returns were drawn from a normal distribution. In this exercise we will investigate the distribution of the estimator for the mean return, without making assumptions about the distribution of the returns and by using the technique called "bootstrapping"" that was introduced during the course.

In the code on the right, the function <code>mean_boot</code> is defined. It estimates the mean of the bootstrapped samples. It takes as first element the data (in this case the return series). As a second argument, it takes a vector that is used to scramble the data before estimating the mean. The output of the function is then the mean value that is computed by using resampled data.

In R, the <code>boot</code> package offers the <code>boot()</code> function for convenient bootstrapping. As a first argument, you should supply the data. As a second argument, the function used to estimate the parameter of interest (the <code>mean_boot()</code> function in this case) and as a third argument, the number of bootstrap samples that you would like to create. The output of the function should be an object of the class "boot".

This is the final exercise of this lab on DataCamp. If you would like to explore this topic more in depth, you can certainly have a look at "lab7.R" under the resources tab on Coursera.

### Instructions

Use the <code>boot()</code> function to generate <code>999</code> bootstrap samples based on the returns of VBLTX and assign the result to <code>VBLTX_mean_boot</code>.
Plot the bootstrapped distribution of the mean and a QQ-plot. This can be done by applying the <code>plot()</code> function on the <code>VBLTX_mean_boot</code> that you have created in the previous step.


```r
library("boot")

# Function for bootstrapping sample mean: 
mean_boot <- function(x, idx) {
  ans <- mean(x[idx])
  ans 
} 

# Construct VBLTX_mean_boot:
VBLTX_mean_boot <- boot(return_matrix[,"VBLTX"], statistic = mean_boot, R=999)

# Print the class of VBLTX_mean_boot
class(VBLTX_mean_boot)
```

```
## [1] "boot"
```

```r
# Print VBLTX_mean_boot
VBLTX_mean_boot
```

```
## 
## ORDINARY NONPARAMETRIC BOOTSTRAP
## 
## 
## Call:
## boot(data = return_matrix[, "VBLTX"], statistic = mean_boot, 
##     R = 999)
## 
## 
## Bootstrap Statistics :
##        original        bias    std. error
## t1* 0.005951256 -9.076374e-05 0.003741296
```

```r
# Plot bootstrap distribution and qq-plot against normal
plot(VBLTX_mean_boot)
```

![](6_Constant_expected_return_model_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

