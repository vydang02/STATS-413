Vy Dang
2024-10-28

# Regression Diagnostics and Robust Inference: A Simulation Study

## Executive Summary
This analysis investigates the robustness of regression inference methods when key model assumptions are violated. Through comprehensive diagnostic testing and Monte Carlo simulation, I examine how violations of normality and homoscedasticity affect the reliability of standard errors, confidence intervals, and statistical inference. The findings provide practical guidance for choosing appropriate estimation methods in real-world applications where model assumptions may not hold perfectly.

## Research Context
Ordinary Least Squares (OLS) regression relies on several key assumptions for valid statistical inference. In practice, these assumptions are often violated to varying degrees. Understanding how these violations impact our conclusions is crucial for applied data analysis. This study systematically explores two common violations:

1. Non-normality of error terms
2. Heteroscedasticity (non-constant variance)

## Part I: Empirical Diagnostic Analysis

### Initial Model Assessment
I began by analyzing an empirical dataset to assess the validity of standard regression assumptions:

``` r
xy <- read.csv("xy.csv")
```

``` r
model <- lm(y ~ x, data = xy)
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = y ~ x, data = xy)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.5499 -0.9772 -0.3922  0.3329  9.2143 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   1.2945     0.3627   3.569 0.000558 ***
    ## x             4.9667     0.6543   7.591 1.88e-11 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.821 on 98 degrees of freedom
    ## Multiple R-squared:  0.3703, Adjusted R-squared:  0.3638 
    ## F-statistic: 57.62 on 1 and 98 DF,  p-value: 1.875e-11
    
The model achieved significant results (p < 0.001) with an R² of 0.37, suggesting moderate explanatory power. However, comprehensive diagnostic testing revealed several concerns.

### Diagnostic Assessment

``` r
plot(model)
```

![](unnamed-chunk-2-1.png)![](unnamed-chunk-2-2.png)![](unnamed-chunk-2-3.png)![](unnamed-chunk-2-4.png)

#### Key Findings:

- **Linearity:** The residuals vs. fitted plot shows the values spread wider
  on the 2 sides than in the middle, and more condensed in the middle.
  Linearity might not hold.

- **Homoscedasticity:** Similar to the residuals vs fitted plot, the
  Scale-Location plot shows a pattern across fitted values. The values
  spread wider on the 2 sides than in the middle, and more condensed in
  the middle. Homoscedasticity might not hold.

- **Normality:** The Normal Q-Q plot points don’t lie on a straight line
  since there are deviating points at the end on the line, indicating
  residuals might not normally distributed. Normality might not hold.

- Concern: There are clear deviations or trends in these diagnostic
  checks. This necessitate further model refinement or alternate
  approaches like transformations or robust regression methods.

### Confidence Interval Comparison
Given the diagnostic concerns, I compared standard and robust confidence intervals:

``` r
coef_summary <- summary(model)$coefficients
slope_estimate <- coef_summary["x", "Estimate"] 
slope_se <- coef_summary["x", "Std. Error"]
df <- df.residual(model)
conf_level <- 0.8
alpha <- 1 - conf_level
t_crit <- qt(1 - alpha/2, df)
moe <- t_crit * slope_se
ci_lower <- slope_estimate - moe
ci_upper <- slope_estimate + moe
ci_lower
```

    ## [1] 4.122433

``` r
ci_upper
```

    ## [1] 5.810897

**Results:**
- Standard CI: [4.12, 5.81]
- Robust CI: [3.97, 5.96]

Since there were significant based on the previous diagnostic plots, the interval calculated might not have adequate coverage. The robust confidence interval is wider, reflecting greater uncertainty when accounting for heteroscedasticity. This suggests the standard interval may be overly optimistic. 

## Part II: Simulation Study - Non-Normal Errors

### Methodology
To systematically investigate the impact of non-normality, I conducted a Monte Carlo simulation with 10,000 replications:

``` r
library(sandwich)
```

    ## Warning: package 'sandwich' was built under R version 4.2.1

``` r
library(lmtest)
```

    ## Warning: package 'lmtest' was built under R version 4.2.3

    ## Loading required package: zoo

    ## Warning: package 'zoo' was built under R version 4.2.2

    ## 
    ## Attaching package: 'zoo'

    ## The following objects are masked from 'package:base':
    ## 
    ##     as.Date, as.Date.numeric

``` r
robust_se <- sqrt(diag(vcovHC(model, type = "HC1")))
slope_se_robust <- robust_se["x"]
conf_level <- 0.80
alpha <- 1 - conf_level
t_crit <- qt(1 - alpha/2, df.residual(model))
moe_robust <- t_crit * slope_se_robust
ci_lower_robust <- slope_estimate - moe_robust
ci_upper_robust <- slope_estimate + moe_robust
ci_lower_robust
```

    ##        x 
    ## 3.969356

``` r
ci_upper_robust
```

    ##        x 
    ## 5.963975

Given substantial evidence of heteroskedasticity in the previous plot
analysis, this robust confidence intervals might better reflect the true
uncertainty around the slope than those obtained using ordinary least
squares standard errors. So this calculation in part c might have better
coverage than coverage in part b.

### Error Distribution Analysis
The simulated errors exhibit clear right-skewness:

``` r
x <- xy$x
y <- xy$y
n <- length(x)
beta0 <- 1
beta1 <- 5
sigma.error <- sqrt(.75)
SD.b1.skew <- sigma.error/(sqrt(n-1)*sd(x))

nsim <- 10000
b1.skew <- rep(0, nsim)
se.skew<- rep(0, nsim)
se.skew.hc <- rep(0, nsim)
Epsilon.skew <- matrix(0, n, nsim)

set.seed(123)
for(i in 1:nsim)
{
  errors <- exp(rnorm(n, 0, sqrt(log(1.5)))) - exp(log(1.5)/2)
  Y <- beta0 + beta1*x + errors
  lm.temp <- lm(Y~x)
  b1.skew[i] <- lm.temp$coef[2]
  se.skew[i] <- summary(lm.temp)$coef[2,2]
  se.skew.hc[i] <- sqrt(diag(vcovHC(lm.temp, type = "HC2")))[2]
  Epsilon.skew[,i] <- errors
}
epsilon_first_iter <- Epsilon.skew[, 1]
hist(epsilon_first_iter, breaks = 30, xlab = "Error Terms", col = "lightblue", border = "black")
```

![](unnamed-chunk-5-1.png)

``` r
qqnorm(epsilon_first_iter)
qqline(epsilon_first_iter, col = "red")
```

![](unnamed-chunk-5-2.png)

### Sampling Distribution of Slope Estimates
Despite non-normal errors, the sampling distribution of slope estimates remains approximately normal:

``` r
hist(b1.skew, breaks = 50, probability = TRUE,
     xlab = "Sample Slope Estimates", col = "lightblue", border = "black")
```

![](unnamed-chunk-6-1.png)

``` r
mean_b1 <- mean(b1.skew)
sd_b1 <- sd(b1.skew)

qqnorm(b1.skew)
qqline(b1.skew, col = "red", lwd = 2)
```

![](unnamed-chunk-6-2.png)

The histogram of the slope estimates b1.skew aligns well with the normal
curve. The points in the Q-Q plot lie largely on the reference line with
minor deviations. Despite the non-normality of the residuals, the
sampling distribution of β1^ approximates a normal distribution
reasonably well.

This demonstrates the Central Limit Theorem in action - the asymptotic normality of OLS estimators holds even under non-normal errors.

### Standard Error Accuracy
Both standard and robust standard errors provide reasonable estimates:

``` r
hist(se.skew, breaks = 50, probability = TRUE, main = "Histogram of se.skew",
     xlab = "Standard Error (SE)", col = "lightblue", border = "black")
abline(v = 0.31, col = "red", lwd = 2)  
```

![](unnamed-chunk-7-1.png)

``` r
mean_se_skew <- mean(se.skew)
hist(se.skew.hc, breaks = 50, probability = TRUE, main = "Histogram of se.skew.hc",
     xlab = "Robust Standard Error (SE)", col = "lightblue", border = "black")
abline(v = 0.31, col = "red", lwd = 2)  
```

![](unnamed-chunk-7-2.png)

``` r
mean_se_skew_hc <- mean(se.skew.hc)
mean_se_skew
```

    ## [1] 0.3064537

``` r
mean_se_skew_hc
```

    ## [1] 0.3031857

The means are close to 0.31, so the standard errors are reasonably
accurate.

| Method | Mean SE | True SD |
|--------|---------|---------|
| Standard | 0.306 | 0.31 |
| Robust HC | 0.303 | 0.31 |

### Confidence Interval Coverage
I evaluated the actual coverage of nominal 95% confidence intervals:

``` r
alpha <- 0.05
df <- n-2
t_crit <- qt(1 - alpha/2, df)
lower_bounds_se <- rep(0, nsim)
upper_bounds_se <- rep(0, nsim)

lower_bounds_se_hc <- rep(0, nsim)
upper_bounds_se_hc <- rep(0, nsim)

for (i in 1:nsim) {
  lower_bounds_se[i] <- b1.skew[i] - t_crit * se.skew[i]
  upper_bounds_se[i] <- b1.skew[i] + t_crit * se.skew[i]
  
  lower_bounds_se_hc[i] <- b1.skew[i] - t_crit * se.skew.hc[i]
  upper_bounds_se_hc[i] <- b1.skew[i] + t_crit * se.skew.hc[i]
}

head(cbind(lower_bounds_se, upper_bounds_se))
```

    ##      lower_bounds_se upper_bounds_se
    ## [1,]        4.002021        5.106905
    ## [2,]        5.041702        6.448051
    ## [3,]        4.029501        5.277668
    ## [4,]        4.345251        5.580325
    ## [5,]        4.300522        5.419885
    ## [6,]        4.051109        5.088967

``` r
head(cbind(lower_bounds_se_hc, upper_bounds_se_hc))
```

    ##      lower_bounds_se_hc upper_bounds_se_hc
    ## [1,]           4.011034           5.097892
    ## [2,]           5.097078           6.392674
    ## [3,]           4.038835           5.268333
    ## [4,]           4.346425           5.579151
    ## [5,]           4.294420           5.425986
    ## [6,]           4.152039           4.988037

``` r
true_slope <- 5
coverage_se <- mean(lower_bounds_se <= true_slope & upper_bounds_se >= true_slope)
coverage_se_hc <- mean(lower_bounds_se_hc <= true_slope & upper_bounds_se_hc >= true_slope)
coverage_se
```

    ## [1] 0.9528

``` r
coverage_se_hc
```

    ## [1] 0.9533

Both methods (`se.skew` and `se.skew.hc`) produce confidence intervals
that capture the true slope approximately 95% of the time, even with
right-skewed (non-normal) errors, thanks to the Central Limit Theorem
(CLT), which ensures the asymptotic normality of OLS estimators. The
slightly better coverage provided by heteroskedasticity-robust standard
errors (`se.skew.hc`) suggests marginal improvement, although
traditional standard errors (`se.skew`) still perform well in large
samples. These findings highlight the robustness of OLS methods. With a
reasonably large sample size (n=100), the estimates and associated
confidence intervals remain accurate under non-normality and
heteroskedasticity. While robust standard errors are generally more
reliable in the presence of heteroskedasticity, the traditional standard
errors provide close to nominal coverage due to the asymptotic
properties of OLS estimators as demonstrated by the simulations.

**Results:**
- Standard CI coverage: 95.3%
- Robust CI coverage: 95.3%

Both methods achieve approximately nominal coverage, confirming the robustness of OLS inference to non-normality in moderate sample sizes.

## Part III: Simulation Study - Heteroscedasticity

### Methodology
I generated data with heteroscedastic errors where variance depends on x:

``` r
Sigma <- diag((3.6 * sqrt(.75) * abs(x - 1/2)))^2
Xmat <- cbind(rep(1, n), x)

SD.b1.het <- sqrt((solve(t(Xmat) %*% Xmat) %*% t(Xmat) %*% Sigma %*% Xmat %*% solve(t(Xmat) %*% Xmat))[2, 2])

b1.het <- rep(0, nsim)
se.het <- rep(0, nsim)
se.het.hc <- rep(0, nsim)
Epsilon.het <- matrix(0, n, nsim)

for (i in 1:nsim) {
    errors <- rnorm(n, 0, 3.6 * sqrt(.75) * abs(x - 1/2))
    Y <- beta0 + beta1 * x + errors
    lm.temp <- lm(Y ~ x)
    b1.het[i] <- lm.temp$coef[2]
    se.het[i] <- summary(lm.temp)$coef[2, 2]
    se.het.hc[i] <- sqrt(diag(vcovHC(lm.temp, type = "HC2")))[2]
    Epsilon.het[, i] <- errors
}
epsilon_het_first_iter <- Epsilon.het[,1]

plot(x, epsilon_het_first_iter,
     xlab = "x values", ylab = "Error Terms (ε)",
     col = "blue", pch = 19, cex = 0.5)
abline(h = 0, col = "red", lwd = 2)
```

![](unnamed-chunk-10-1.png)

### Heteroscedasticity Pattern
The error variance exhibits a clear V-shaped pattern: We can see a “funnel-shaped” pattern where errors spread out more widely as x
moves away from the center point (here around x = 0.5).

``` r
hist(b1.het, breaks = 50, probability = TRUE,
     xlab = "Sample Slope Estimates", col = "lightblue", border = "black")
```

![](unnamed-chunk-11-1.png)

``` r
qqnorm(b1.het)
qqline(b1.het, col = "red", lwd = 2)
```

![](unnamed-chunk-11-2.png)

The histogram of b1.het suggests the normal approximation is reasonable. In
the Q-Q plot, the sample quantiles lie along the reference line, it
indicates that the distribution of b1.het is approximately normal. Given
the large sample size (n = 100) and the Central Limit Theorem, we expect
the distribution of the sample slopes to be approximately normal.

### Impact on Standard Errors

``` r
hist(se.het, breaks = 50, probability = TRUE,
     xlab = "Standard Error (SE)", col = "lightblue", border = "black")
abline(v = 0.427, col = "red", lwd = 2)
```

![](unnamed-chunk-12-1.png)

``` r
mean_se_het <- mean(se.het)
hist(se.het.hc, breaks = 50, probability = TRUE,
     xlab = "Robust Standard Error (SE)", col = "lightblue", border = "black")
abline(v = 0.427, col = "red", lwd = 2)
```

![](unnamed-chunk-12-2.png)

``` r
mean_se_het_hc <- mean(se.het.hc)
mean_se_het
```

    ## [1] 0.3093379

``` r
mean_se_het_hc
```

    ## [1] 0.4211014

- Mean of se.het (0.3098): This mean is substantially lower than the
  true standard deviation of 0.427. It suggests that the standard errors
  calculated under the assumption of homoskedasticity are
  underestimating the true variability of the slope estimator. This
  underestimation indicates that the homoskedasticity assumption does
  not hold, making these standard errors inappropriate in the presence
  of heteroskedasticity.

- Mean of se.het.hc (0.4214): This mean is very close to the true
  standard deviation of 0.427. It suggests that the
  heteroskedasticity-consistent standard errors provide a more accurate
  estimate of the true variability of the slope estimator under
  heteroskedastic conditions. This accuracy indicates that the robust
  standard errors are more suitable when heteroskedasticity is present
  in the data.

  | Method | Mean SE | True SD |
|--------|---------|---------|
| Standard | 0.309 | 0.427 |
| Robust HC | 0.421 | 0.427 |

Standard errors assuming homoscedasticity substantially underestimate the true variability, while heteroscedasticity-consistent standard errors provide accurate estimates.

### Confidence Interval Coverage
The underestimation of standard errors leads to inadequate coverage:

``` r
alpha <- 0.05
df <- n - 2 
t_crit <- qt(1 - alpha / 2, df)
lower_bounds_se_het <- rep(0, nsim)
upper_bounds_se_het <- rep(0, nsim)

lower_bounds_se_het_hc <- rep(0, nsim)
upper_bounds_se_het_hc <- rep(0, nsim)

for (i in 1:nsim) {
  lower_bounds_se_het[i] <- b1.het[i] - t_crit * se.het[i]
  upper_bounds_se_het[i] <- b1.het[i] + t_crit * se.het[i]
  
  lower_bounds_se_het_hc[i] <- b1.het[i] - t_crit * se.het.hc[i]
  upper_bounds_se_het_hc[i] <- b1.het[i] + t_crit * se.het.hc[i]
}
head(cbind(lower_bounds_se_het, upper_bounds_se_het))
```

    ##      lower_bounds_se_het upper_bounds_se_het
    ## [1,]            4.858713            6.183820
    ## [2,]            4.472823            5.583438
    ## [3,]            4.193444            5.429362
    ## [4,]            4.184004            5.309525
    ## [5,]            4.415113            5.446253
    ## [6,]            4.249189            5.337182

``` r
head(cbind(lower_bounds_se_het_hc, upper_bounds_se_het_hc))
```

    ##      lower_bounds_se_het_hc upper_bounds_se_het_hc
    ## [1,]               4.610051               6.432482
    ## [2,]               4.296830               5.759431
    ## [3,]               4.004872               5.617934
    ## [4,]               3.961129               5.532400
    ## [5,]               4.244126               5.617241
    ## [6,]               4.172416               5.413956

``` r
library(sandwich)
true_slope <- 5
coverage_se_het <- mean(lower_bounds_se_het <= true_slope & upper_bounds_se_het >= true_slope)
coverage_se_het_hc <- mean(lower_bounds_se_het_hc <= true_slope & upper_bounds_se_het_hc >= true_slope)
coverage_se_het
```

    ## [1] 0.8396

``` r
coverage_se_het_hc
```

    ## [1] 0.94

- 0.845: The coverage probability is significantly lower than the
  nominal level of 0.95. This suggests that se.het are not capturing the
  true variability in the slope estimates. The confidence intervals can
  be too narrow and fail to contain the true population slope (β1=5) as
  often as they should. This under coverage highlights the inadequacy of
  using homoskedastic standard errors when there is heteroskedasticity.

- 0.9438: The coverage probability is very close to the nominal level of
  0.95. This indicates that the heteroskedasticity-consistent standard
  errors provide a much better approximation of the true variability.
  The confidence intervals are more accurate and reliable in containing
  the true population slope. This shows how
  heteroskedasticity-consistent standard errors suitable for inference
  under heteroskedastic conditions.

**Results:**
- Standard CI coverage: 84.0% (nominal 95%)
- Robust CI coverage: 94.4% (nominal 95%)

Standard confidence intervals fail to achieve nominal coverage, while robust methods maintain proper coverage rates.

## Key Findings and Recommendations

### 1. Robustness to Non-Normality
- OLS estimators maintain good properties under non-normal errors
- Standard inference procedures remain valid with moderate sample sizes
- The Central Limit Theorem provides protection against non-normality

### 2. Sensitivity to Heteroscedasticity
- Heteroscedasticity severely impacts standard error accuracy
- Standard confidence intervals suffer from undercoverage
- Robust standard errors are essential when variance is non-constant

### 3. Practical Guidelines
1. **Always perform diagnostic checks** before relying on regression results
2. **Use robust standard errors** when heteroscedasticity is suspected
3. **Consider larger sample sizes** when dealing with non-normal errors
4. **Be cautious with prediction intervals** under violated assumptions

## Business Applications

### Risk Management
Understanding the limitations of standard regression methods is crucial for:
- Financial modeling and forecasting
- Quality control in manufacturing
- Economic policy analysis

### Decision Support
These findings help analysts:
- Choose appropriate inference methods
- Communicate uncertainty accurately
- Make more reliable predictions

### Model Validation
The diagnostic approach demonstrated here provides a framework for:
- Assessing model reliability
- Identifying potential issues
- Selecting robust alternatives

## Conclusion
This analysis demonstrates that while OLS regression is remarkably robust to non-normality, heteroscedasticity poses serious challenges to standard inference procedures. The simulation results strongly support the use of heteroscedasticity-consistent standard errors in practice, particularly when diagnostic tests suggest non-constant variance. These findings underscore the importance of thorough model diagnostics and the value of robust inference methods in applied statistical work.
