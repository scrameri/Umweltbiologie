## Analysis of Variance and Linear Models

### Fitting models to data

ANOVA using aov() and (GENERALIZED) LINEAR MODELS using lm() or glm() are closely related to each other and differ maintly in intent of analysis and default presentation of results. 

You can think of them as extensions of a two-sample t-test. Remember that the two-sample t-test allows to test the difference in the response variable (a.k.a. dependent variable, measure variable, 
Y variable) between two levels of one factor (a.k.a. predictor variable, independent variable, X variable). A common example is the difference in human height (Y) between two sexes (X).

Let us start with a one-sample t.test:

```R
t.test(morph$Sepal_length, mu = 0)
```

![logo]
- What is the null and alternative hypothesis? 
- And what is the test result?

This test can also be accomplished by `lm`.
summary(lm(Sepal_length ~ 1, data = morph))

Let us now extend this to a TWO-SAMPLE T-TEST (one sample at low and one at high elevation):

```R
t.test(Sepal_length ~ Elevation, data = morph, mu = 0)
t.test(morph$Sepal_length[morph$Elevation == "hoch"], morph$Sepal_length[morph$Elevation == "tief"], data = morph, mu = 0)
```

![logo]
- What is the null and alternative hypothesis? 
- And what is the test result?


Again, the same test can be accomplished using lm() or aov(), which can however be used for much more complex tests.

You might ask yourself when to use aov() and when to use lm(). This depends on your HYPOTHESIS and on the INTENT of your analysis. The underlying calculations are similar (aov() actually calls lm(), both rely on the same least squares method):

```R
summary(aov(Sepal_length ~ Elevation, data = morph))
summary(lm(Sepal_length ~ Elevation, data = morph))
```

However, lm() will give you effect size estimates

![logo]
- What is the estimated effect size of Elevation?
- How do you interpret the effect estimate?

The ONE-WAY ANOVA allows for comparing more then just two groups (i.e. one treatment factor can have more than two levels). 

Tests with more than 2 levels in an explanatory factor can have different intents: 
a) test for an OVERALL effect of a factor such as Population on calyx length
b) test for an effect of a SPECIFIC treatment relative to a CONTROL, including 
     effect size estimates

If your goal is a), then you can use aov():

```R
summary(aov(Sepal_length ~ Population, data = morph))

```

If your goal is b), then you can use lm():

```R
summary(lm(Sepal_length ~ Infection, data = morph))
summary(lm(Sepal_length ~ Population, data = morph))
```

There are many extensions to the t-test and one-way ANOVA. Here is an overview:

- The FACTORIAL ANOVA extends the one-way ANOVA to test >1 treatment factors (each with two or more levels), including interactions:
```R
summary(aov(Sepal_length ~ Infection * Elevation, data = morph))

```
- ANCOVA allows for the mixed use of numerical and categorical predictors.

```R
summary(aov(Sepal_length ~ Infection * Elevation + Petal_length, data = morph))
```
- GENERALIZED LINEAR MODEL extends ANCOVA by allowing for non-normal error distribution (logistic regression, poisson regression, ...).
```R
plot(glm(Stalk_count ~ Infection * Elevation, family = "Gamma", data = morph))
RRR


### Check model assumptions via Residual Analysis
'Often (but not always!) a model fit to a non-normally distributed response variable will lead to non-normally
distributed residuals. This is a problem since ANOVA aov() or linear regression models lm() make the 
following assumptions:
  
  - Normality: For any fixed value of X, Y is NORMALLY distributed. This implies that the residuals are normally distributed.
  - Homoscedasticity: The RESIDUAL VARIANCE is CONSTANT (the same for any value of X)
  - Independence: Observations are INDEPENDENT of each other (i.e. one measurement is not influenced by another)

If one or more of these assumptions are violated, the model inference (i.e., test results) can be grossly misleading.
'

### Variable transformations
A transformation of the resonse Y often helps to improve model fit, at the expense of interpretability of the fitted model parameters (estimated effect size). There are the following commonly used first-aid transformations.

- for positive COUNTS:   X -> sqrt(X)
- for positive NUMERICS: X -> log(X)
- for fractions [0,1]:   X <- asin(sqrt(X))

```R
par(mfrow=c(2,2))
summary(mod.2 <- lm(sqrt(Stalk_count) ~ Infection + Elevation + Stalk_Elevation, data = morph))
plot(mod.2)
par(mfrow=c(1,1))
```

The fit now looks better than before, but still not perfect. The fitted estimated effect sizes now have to be interpreted as follows:

- Changing Elevation from <hoch> to <tief> decreases the square-root of stalk count by 0.91
- Changing Infection from <gesund> to <inf> increases the square-root of stalk count by 0.35

![logo]
- Any idea why the residuals still show a violation of the model assumptions?


### Interaction plots
Interactions between explanatory variables can be present, and including them in the model can greatly improve the model fit and hence the reliability of test results.

Interactions in a nutshell: Effect of variable X on the response Y depends on variable W.

Example: 
Y = reaction time (e.g., time needed to start breaking after an obstacle appears on the street)
X = amount of coffee drunk 
W = amount of whisky drunk 

X has a negative effect on reaction time (i.e., more coffee, less reaction time)
W has a positive effect on reaction time (i.e, more whisky, more reaction time)

You can increase X to shorten reaction time, but this effect will depend on the amount of whisky drunk.
That is, the amount of coffee interacts with the amount of whisky.


# Create interaction plots
' R implements some methods to visualize potential interactions. The idea is to visualize the mean (or other
  summary) value of the response Y at different levels of an explanatory variable of interest (x.factor), 
  depending on the level of another and potentially interacting explanatory factor (trace.factor).
'
' Let us assume that you are interested in testing the effect of Infection and Altitude on the Sepal length.
  Before you fit any model, make sure to plot the data to get a better feeling for it.
'

ggplot(data = morph, aes(x = Infection, y = Petal_length)) + 
  geom_boxplot(alpha = 0.6) + 
  facet_wrap(~Elevation) +
  theme_bw()

' Sepal length appears to be lower in some infected individuals, but that does not seem to be the case in 
  low elevation habitats. That is, the effect of Infection on Sepal length appears to depend on elevation.

  The following plot visualizes potential interactions. If the lines cross, the factors strongly interact.
'
interaction.plot(x.factor = morph$Infection, trace.factor = morph$Elevation, response = morph$Petal_length, 
                 fun = mean, xlab = "Infection", ylab = "Petallänge (Mittelwerte)")

' Now lets compare two linear models of Sepal length, once with and once without interaction term:
'
par(mfrow=c(2,2))
summary(mod.3 <- lm(log(Petal_length) ~ Elevation+Infection, data = morph))
plot(mod.3)
par(mfrow=c(1,1))

' Nothing looks significant here, although the model fit does not look bad.'

par(mfrow=c(2,2))
summary(mod.4 <- lm(log(Petal_length) ~ Elevation*Infection, data = morph))
plot(mod.4)
par(mfrow=c(1,1))

' If the interaction term is included, we get slightly better-looking residuals, and 
  both explanatory factors and the interaction turn out significant. This illustrates that interaction terms
  can hugely influence test results and intepretations.
'

### Compare nested models
You can formally test two nested models (i.e. fitted to the same data, one model with one or more additional model terms) using the anova() function. If the F-Test turns out significant, it means that the LARGER / COMPLEX model (in our case, the model including the interaction term) is a BETTER fit to the data, and should thus be preferred over the smaller / simpler model.

```R
anova(mod.3, mod.4)
```

![logo]
- Which model (simple or complex) would you choose if the anova F-test turns out not significant, and why?


[logo]: https://img.icons8.com/flat_round/64/000000/question-mark.png "Question"
