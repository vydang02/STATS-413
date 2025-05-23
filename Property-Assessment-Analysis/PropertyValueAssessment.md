# Property Value Assessment: Statistical Analysis of Residential Property Valuations

Vy Dang
2024-09-03

``` r
knitr::opts_chunk$set(fig.path='Figs/')
```

``` r
library(ggplot2)
```

    ## Warning: package 'ggplot2' was built under R version 4.2.3

## Executive Summary
This analysis examines the statistical relationship between property size and assessed value using regression techniques. By analyzing a comprehensive dataset of residential properties, I developed a predictive model that quantifies how livable area contributes to property valuation. This model provides valuable insights for property owners, tax authorities, and real estate professionals.

For historical reasons the US has a system of taxing homeowners to fund
a large fraction of local in- frastructure such as local primary, middle
and high schools, town and county administrations, and town and county
roads. The tax, called “property tax”, is based on an assessment
(estimation, determination) of the value of each residence (home) and
the lot (land) that belongs to it. Because the assessments become
outdated after a few years, towns have to hire assessors and update the
assessments every so often. There are of course several factors that
play a role in assessing the value of a property, such as square feet of
livable area, size of the lot (land), quality and condition of the
building, and desirability of the area. I will only consider square
feet of livable area. The following exercise can be used to check for
any property whether its assessed value is in line or out of line with
other properties of similar size in terms of livable area. This data set
is called Residential Property Assessments.csv.

## Data Overview
The dataset contains assessment records for 1,626 residential properties, with the following key variables:

- Assessment value (in dollars)
- Livable area (in square feet)

I implemented a linear regression model to quantify the relationship between these variables.

## Methodology and Results

``` r
data <- read.csv('Residential_Property_Assessments.csv')
```

``` r
data |> ggplot(aes(x = Livable.Area, y = Assessment)) + geom_point() +
  labs(title = "Assessment vs Livable Area",
       x = "Livable Area (sq ft)",
       y = "Assessment ($)")
```

![](unnamed-chunk-3-1.png)

Visual inspection reveals a strong positive linear relationship between livable area and assessment value, confirming the appropriateness of a linear model.
    
### Regression Analysis and Results
I developed a simple linear regression model with Assessment as the response variable and Livable Area as the predictor:

``` r
model <- lm(Assessment~Livable.Area, data = data)
model
```

    ## 
    ## Call:
    ## lm(formula = Assessment ~ Livable.Area, data = data)
    ## 
    ## Coefficients:
    ##  (Intercept)  Livable.Area  
    ##     168518.6         180.8

``` r
coefficients <- coef(model)
intercept <- coefficients[1]
slope <- coefficients[2]
paste("Assessment =", round(intercept,2), "+", round(slope,2), "* Livable Area" )
```

    ## [1] "Assessment = 168518.56 + 180.82 * Livable Area"



**_Formal interpretation of Equation_:** Assessment = β0 + β1 \* Livable Area

The resulting prediction equation: Assessment = $168,518.56 + $180.82 × Livable Area

Holding all other variables equal (ceteris paribus), if two properties differ in their livable area by 1 square foot, then we expect their assessments to differ by β1 units on average. β1 represents the change in the expected assessment for a one-unit increase in the livable area given that other factors are held constant.

**_Informal interpretation of Equation:_** For every additional square foot of livable area, the property assessment increases by 180.8 dollars on average. This means that a larger house with more livable area will have a higher assessed value. Each additional square foot is valued at approximately 180.8 dollars in terms of property assessment.


**_Formal interpretation of Intercept:_** The intercept β0 in a linear regression model is interpreted as the expected value of the response variable (Assessment) when all predictor variables (Livable Area) are zero. Mathematically it represents the value of (E(Y\|X=0)). The intercept β0 would be the assessment value predicted for a property with zero square feet of livable area. So formally, (β0 = 168518.6) means that the expected assessment value for a property with zero square feet of livable area is 168518.6 dollars

**_Informal interpretation of Intercept:_** Despite the fact that a property with zero square feet of livable area is unrealistic, this value can representthe starting point or base value of the property due to land value or fixed costs that are independent of the livable area.


``` r
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = Assessment ~ Livable.Area, data = data)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -330914  -35605    -958   38267  516292 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  1.685e+05  5.894e+03   28.59   <2e-16 ***
    ## Livable.Area 1.808e+02  2.319e+00   77.99   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 64340 on 1624 degrees of freedom
    ## Multiple R-squared:  0.7893, Adjusted R-squared:  0.7891 
    ## F-statistic:  6082 on 1 and 1624 DF,  p-value: < 2.2e-16

#### Key Findings

Fraction of variation in Assessment accounted for by Livable Area ((R^2)): 0.7893. 78.93% of the variation in the Assessment (property value) is
explained by the Livable Area in this regression model. Fraction of variation in Assessment accounted for by other factors: 1 - R^2 = 1 - 0.7893 = 0.2107. 21.07% of the variation in Assessment is due to other factors besides Livable Area.

- _Value per Square Foot_: Each additional square foot of livable area contributes approximately $180.82 to a property's assessed value
  
- _Base Property Value_: The intercept of $168,518.56 represents the estimated value of land and fixed improvements independent of living space size
  
- _Model Fit_: The model explains 78.9% of variation in property assessments (R² = 0.789)
  
- _Unexplained Variation_: The remaining 21.1% of variation is attributable to factors not included in the model, such as location quality, property condition, and lot characteristics

### Practical Applications

#### Valuation Predictions

The model enables accurate predictions for property valuations:

``` r
new_data <- data.frame(Livable.Area=2500)
predict_2500_sqft <- predict(model, newdata = new_data)
predict_2500_sqft
```

    ##        1 
    ## 620569.1

A 2,500 square foot residence has a predicted assessment of $620,569.10.

#### Comparative Analysis

Here we want to know the difference in assessed value for two residences where one is 200 square feet larger than the other. We will use beta-1 to calculate. The slope can directly be scaled to predict the difference for any unit change in the independent variable.

``` r
difference_in_assessment <- slope * 200
difference_in_assessment
```

    ## Livable.Area 
    ##     36164.05

So two properties differing by 200 square feet in livable area would be expected to differ in assessed value by approximately $36,164.05, holding all other factors constant.

### Model Validation

Statistical validation confirms the model's reliability:

``` r
residuals <- residuals(model)
fitted_values <- fitted(model)
mean_residuals <- mean(residuals)
mean_residuals
```

    ## [1] 4.853204e-12

``` r
correlation_fitted_residuals <- cor(fitted_values, residuals)
correlation_fitted_residuals
```

    ## [1] 3.398222e-17

``` r
ggplot(data, aes(x = fitted_values, y = residuals)) +
  geom_point() +
  geom_hline(yintercept = 0, linetype="dashed", color="red") +
  labs(title = "Residuals vs Fitted Values",
       x = "Fitted Values",
       y = "Residuals")
```

![](unnamed-chunk-10-1.png)

``` r
ggplot(data.frame(residuals = residuals), aes(x = residuals)) +
  geom_histogram(bins = 30, color = "black", fill = "skyblue") +
  geom_vline(aes(xintercept = mean_residuals), color = "red", linetype = "dashed") +
  labs(title = "Histogram of Residuals",
       x = "Residuals",
       y = "Frequency")
```

![](unnamed-chunk-11-1.png)

- _Unbiased Estimation:_ Residuals have mean zero, confirming unbiased estimation

- _Linearity Validation:_ Predicted values and residuals are uncorrelated
  
- _Distribution of Residuals:_ Histogram shows approximately normal distribution with some outliers

### Business Implications

This analysis provides:

- **Assessment Benchmarking:** A tool for homeowners to evaluate whether their property assessment is in line with similar properties
  
- **Valuation Framework:** A simple formula for real estate professionals to estimate property values based on square footage
  
- **Tax Authority Guidance:** Insights for tax assessors on the quantitative relationship between property size and value
  
- **Market Transparency:** Increased transparency in understanding how property characteristics translate to market value
