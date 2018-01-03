---
layout: post
title: Linear Modeling in R
---
# Linear Modeling in R

Method call: `fit <- lm(mpg ~ hp, data = mtcars)`
Associated Methods:
`summary(fit)
plot(fit)
`

## Model Summary

Results:
`Call: lm(formula = mpg ~ hp, data = d)`
This is a replay of the method call

`Residuals:
Min     1Q       Median     3Q    Max
-5.7121 -2.1122 -0.8854 1.5819 8.2360`
These are the residuals. Ideally, they will be similar in Min/1Q/Median/3Q/Max and centered around 0

`Coefficients:
            Estimate Std. Error t value Pr(>|t|)
(Intercept) 30.09886 1.63392    18.421 < 2e-16  ***
hp          -0.06823 0.01012    -6.742 1.79e-07 *** `

Estimate is the estimated value.
Standard Error is the average amount that the coefficient estimates varies from the actual average of the response variable, ideally low compared to the coefficients.
t value is the used to determine the Pr(>|t|), this is a measurement of how many standard deviations the estimate is from zero. Ideally higher absolute value.
Pr(>|t|) is the probability of a relationship between the predictor and response by chance.  Ideally less than 0.05.

---
Signif. codes: 0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

`Residual standard error: 3.863 on 30 degrees of freedom `

Residual Standard Error is a quality of linear regression fit. RSE/Intercept = %Error of any estimate

`Multiple R-squared: 0.6024, Adjusted R-squared: 0.5892`

The R-squared is the amount of the model explained by the variables, DANGER this value automatically increases as more variables are added to the model.
The Adjusted R-Squared is the amount of the model explained by the variables, but with an adjustment to account for the number of variables in the model.
In both cases, higher is sometimes better, however caution should be exercised to ensure the model is not overfit.

`F-statistic: 45.46 on 1 and 30 DF, p-value: 1.788e-07`

The F-statistic is a comparison of the model to random. Ideally the statistic is large and the p-value should be less than 0.05.

## Diagnostic Plots

<<resid vs fitted>>

Residuals vs. Fitted Plot
* The residuals should bounce randomly around the 0 line, which suggests that it is reasonable to assume that the relationship is linear
* The residuals should form a "horizontal band" around the 0 line, which suggests the variance of the error terms are equal
* No one/few residuals should stand out from the basic pattern; one that does might be an outlier

<<normal q-q>>

Normal Q-Q Plot
* The plot should form a straight line, which indicates the sample and model use the same distribution
* A curved plot can mean a skewed data sample
* A nearly horizontal line with extremes at the right and left can indicate that the sample has more extreme values

<<scale location>>

Scale-Location Plot
* Ideally, the line will be horizontal with randomly spread points

<<resid vs lev>>

Residuals vs. Leverage Plot
* Uses Cook's Distance to measure leverage - leverage DOES NOT mean an outlier!
* Look for cases in the upper or lower right, noting Cook's Distance curves

# References
https://feliperego.github.io/blog/2015/10/23/Interpreting-Model-Output-In-R
https://stats.stackexchange.com/questions/59250/how-to-interpret-the-output-of-the-summary-method-for-an-lm-object-in-r
https://onlinecourses.science.psu.edu/stat501/node/36
http://data.library.virginia.edu/diagnostic-plots/
http://data.library.virginia.edu/understanding-q-q-plots/
http://blog.yhat.com/posts/r-lm-summary.html
