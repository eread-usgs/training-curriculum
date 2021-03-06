---
author: Jeffrey W. Hollister, Emily Read, Lindsay Carr
date: 9999-01-06
slug: Analyze
title: E. Analyze
image: img/main/intro-icons-300px/analyze.png
menu: 
  main:
    parent: Introduction to R Course
    weight: 1
---
The focus of this workshop hasn't really been statistics, it's been more about R, the language. But it's pretty much impossible to talk a lot about R without getting into stats, as that is what draws most people to R in the first place. So we will spend a little bit of time on it. In this lesson we will touch on some very simple stats that we can do with base R.

Remember that we are using the NWIS dataset for all of these lessons. If you successfully completed the [Clean](/intro-curriculum/clean) lesson, then you should have the cleaned up version of the data frame. If you did not complete the Clean lesson (or are starting in a new R session), just load in the cleaned csv by downloading it from [here](/intro-curriculum/data), saving it in a folder called "data", and using `read.csv` (see below).

``` r
intro_df <- read.csv("data/course_NWISdata_cleaned.csv", stringsAsFactors = FALSE, 
                     colClasses = c("character", rep(NA, 6)))
```

Quick Links to Exercises and R code
-----------------------------------

-   [Exercise 1](#exercise-1): Compute basic statistics and build a simple model with USGS data

Lesson Goals
------------

-   Conduct basic statistical analyses with base R
-   Get a taste of a wide array of analyses in base R

Base statistics
---------------

The capabilities that come out of the box with R are actually quite good and used to (i.e. before R) cost you quite a bit to access. Now it all comes for free! Some things you can do in R without any additional packages include: logistic regression (and all manner of generalized linear models), correlation, principle components analysis, chi-squared tests, clustering, loess, ANOVA, MANOVA, ... In short, we can do a lot without moving out of base r and the `stats` package.

We will talk about a few analyses just to show the tip of the iceberg of what is available.

### t-tests

A t-test is done simply with `t.test()` and you control the specifics (paired, two-sided, etc.) with options. For the simple case of comparing the difference of two means we can use all of the defaults:

``` r
pop1 <- rnorm(30, mean=3, sd=2)
pop2 <- rnorm(30, mean=10, sd=5)
pop_ttest <- t.test(pop1, pop2)
pop_ttest
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  pop1 and pop2
    ## t = -7.5834, df = 37.108, p-value = 4.714e-09
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -9.847124 -5.694896
    ## sample estimates:
    ## mean of x mean of y 
    ##  3.022597 10.793608

`t.test()` can also take a formula specification as input. For a t-test that is all we need to know. R formulas were described in the "Basic Visualization" section of Analyze. Use a t.test to compare discharge between Erroneous and estimated qualifiers (these serve as your two populations). You cannot use `t.test()` to compare more than two groups.

``` r
#Filter so that there are only two Flow_Inst_cd groups
#You might have to load dplyr
library(dplyr)
err_est_df <- filter(intro_df, Flow_Inst_cd %in% c("X", "E"))
t.test(err_est_df$Flow_Inst ~ err_est_df$Flow_Inst_cd)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  err_est_df$Flow_Inst by err_est_df$Flow_Inst_cd
    ## t = -1.9985, df = 905.04, p-value = 0.04596
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -4707.2882   -42.6546
    ## sample estimates:
    ## mean in group E mean in group X 
    ##       -24.28272      2350.68869

There's a lot more you can do with `t.test()`, but you'll have to rely on `?t.test` for more info.

### Correlation

Next let's take a look at correlations. To handle missing values in `cor`, `na.rm` will not work. If you look at the help for `cor()`, you'll see two main optional arguments. First is the `use` argument which allows you to use the entire dataset or select complete cases which is useful when you have `NA` values. There are several options. Also, the default correlation method is for Pearson's. If you would like to use non-parametric correlations (e.g. rank), you specify that here. See `?cor` for more information. `cor.test` already has a default way of omitting NAs using the `na.action` argument. See `?cor.test`.

``` r
#A simple correlation
cor(intro_df$Wtemp_Inst, intro_df$DO_Inst, use="complete.obs")
```

    ## [1] -0.4216771

``` r
#And a test of that correlation
cor.test(intro_df$Wtemp_Inst, intro_df$DO_Inst)
```

    ## 
    ##  Pearson's product-moment correlation
    ## 
    ## data:  intro_df$Wtemp_Inst and intro_df$DO_Inst
    ## t = -24.696, df = 2820, p-value < 2.2e-16
    ## alternative hypothesis: true correlation is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.4515495 -0.3908605
    ## sample estimates:
    ##        cor 
    ## -0.4216771

``` r
#A data frame as input to cor returns a correlation matrix
#Can't just do cor(intro_df) because intro_df has non-numeric columns:
# cor(intro_df)
# use dplyr to select the numeric columns of intro_df
intro_df_onlynumeric <- select(intro_df, -site_no, -dateTime, -Flow_Inst_cd)  
cor(intro_df_onlynumeric, use="complete.obs")
```

    ##               Flow_Inst  Wtemp_Inst      pH_Inst     DO_Inst
    ## Flow_Inst   1.000000000 -0.01760423 -0.008350327  0.03725997
    ## Wtemp_Inst -0.017604230  1.00000000  0.247765343 -0.42596789
    ## pH_Inst    -0.008350327  0.24776534  1.000000000  0.20347569
    ## DO_Inst     0.037259972 -0.42596789  0.203475686  1.00000000

### Linear Regression

Next let's take a look at linear regression. One of the common ways of fitting linear regressions is with `lm()`. We have already seen the formula object so there isn't too much that is new here. Some of the options are new and useful, though. Let's take a look:

``` r
lm(DO_Inst ~ Wtemp_Inst, data=intro_df)
```

    ## 
    ## Call:
    ## lm(formula = DO_Inst ~ Wtemp_Inst, data = intro_df)
    ## 
    ## Coefficients:
    ## (Intercept)   Wtemp_Inst  
    ##     11.3049      -0.1747

``` r
#Not much info, so save to object and use summary
lm_gwq1 <- lm(DO_Inst ~ Wtemp_Inst, data=intro_df)
summary(lm_gwq1)
```

    ## 
    ## Call:
    ## lm(formula = DO_Inst ~ Wtemp_Inst, data = intro_df)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -4.4055 -0.6783 -0.1055  0.7510  5.4808 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 11.304895   0.148356    76.2   <2e-16 ***
    ## Wtemp_Inst  -0.174729   0.007075   -24.7   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.241 on 2820 degrees of freedom
    ##   (178 observations deleted due to missingness)
    ## Multiple R-squared:  0.1778, Adjusted R-squared:  0.1775 
    ## F-statistic: 609.9 on 1 and 2820 DF,  p-value: < 2.2e-16

``` r
#And now a multiple linear regression
lm_gwq2 <- lm(DO_Inst ~ Wtemp_Inst + Flow_Inst, data=intro_df)
summary(lm_gwq2)
```

    ## 
    ## Call:
    ## lm(formula = DO_Inst ~ Wtemp_Inst + Flow_Inst, data = intro_df)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -4.4067 -0.6817 -0.1067  0.7537  5.4795 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  1.131e+01  1.512e-01  74.787   <2e-16 ***
    ## Wtemp_Inst  -1.747e-01  7.216e-03 -24.211   <2e-16 ***
    ## Flow_Inst    2.033e-06  1.415e-06   1.436    0.151    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.245 on 2733 degrees of freedom
    ##   (264 observations deleted due to missingness)
    ## Multiple R-squared:  0.1773, Adjusted R-squared:  0.1767 
    ## F-statistic: 294.6 on 2 and 2733 DF,  p-value: < 2.2e-16

We can also put a regression line on the plot:

``` r
plot(intro_df$Wtemp_Inst, intro_df$DO_Inst)
#abline accepts a linear model object as input
#linear model is done with lm, and uses a formula as input
abline(lm(DO_Inst ~ Wtemp_Inst, data=intro_df))
```

<img src='../static/Analyze/abline_examp_lm-1.png'/ title='Scatter plot of pH versus flow with regression line'/>

We can also just add a straight line defined by slope and intercept. We do this with `abline()`. This is useful if you have a known value that you want to compare to your data.

``` r
plot(intro_df$Wtemp_Inst, intro_df$DO_Inst)
#horizontal line at specified y value
abline(h=11)
#a vertical line
abline(v=15)
#Line with a slope and intercept
abline(7, 0.5)
```

<img src='../static/Analyze/abline_examp-1.png'/ title='Dissolved oxygen versus water temperature scatter plot with lines using abline'/>

All of your standard modeling approaches (and then some) are available in R, including typical variable selection techniques (e.g. stepwise with AIC) and logistic regression, which is implemented with the rest of the generalized linear models in `glm()`. Interaction terms can be specified directly in the model, but we won't be covering them in this course.. Lastly, if you are interested in more involved or newer approaches these are likely implemented in additional packages, beyond base R and `stats`, which you can find on a repository such as [CRAN](https://cran.rstudio.com), [GRAN](http://owi.usgs.gov/R/gran.html), or [Bioconductor](https://www.bioconductor.org/packages/release/BiocViews.html#___Software). You can also check out task pages such as the [CRAN Environmetrics Task View](https://cran.r-project.org/web/views/Environmetrics.html) for more ideas.

Exercise 1
----------

For this exercise, let's start to look at some of the statistical tests and relationships.

1.  First, let's take a look at the relationship between pH and low or high temperatures. Add a section to your script that tests for a difference in the mean value across these parameters between low temperature observations and high temperature observations (high is greater than or equal to 20). What is the conclusion? Are the means statistically different?

2.  Next, let's build a linear model that predicts pH from discharge at a single site (pick any site). Use discharge as the explanatory variable. Add a line to extract the r-squared value from the linear model.

3.  Challenge: Create a multivariate linear model relating pH to two other explanatory variables. Extract the adjusted r-squared value.
