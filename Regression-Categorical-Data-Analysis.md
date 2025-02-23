Regression Categorical Data Analysis
================
Sanjana Battula
2025-02-23

### Loading The Data

``` r
# Load required libraries
library(AER) # For dataset
library(dplyr) # Data manipulation
library(ggplot2) # Visualization
library(corrplot) # Correlation plot
library(MASS) # Negative binomial regression
library(pscl) # Zero-inflated models
library(lmtest) # Likelihood ratio tests

# Load the NMES1988 dataset
data(NMES1988, package = "AER")

# Data preparation
NMES1988 <- NMES1988 %>%
  mutate(new.chronic = case_when(
    chronic == 0 ~ 0,
    chronic == 1 ~ 1,
    chronic >= 2 ~ 2
  ))
```

# ———————- Descriptive Statistics ———————-

``` r
# For continuous variables: age, income, visits
summary(NMES1988$age) # Age
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   6.600   6.900   7.300   7.402   7.800  10.900

``` r
summary(NMES1988$income) # Income
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ## -1.0125  0.9122  1.6982  2.5271  3.1728 54.8351

``` r
summary(NMES1988$visits) # Dependent variable: Visits
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   0.000   1.000   4.000   5.774   8.000  89.000

``` r
# For categorical variables: gender, insurance, health, employment, new.chronic
table(NMES1988$gender) # Gender
```

    ## 
    ## female   male 
    ##   2628   1778

``` r
table(NMES1988$insurance) # Insurance coverage
```

    ## 
    ##   no  yes 
    ##  985 3421

``` r
table(NMES1988$health) # Health status
```

    ## 
    ##      poor   average excellent 
    ##       554      3509       343

``` r
table(NMES1988$new.chronic) # Chronic condition categories
```

    ## 
    ##    0    1    2 
    ## 1025 1498 1883

# ———————- Visualizations ———————-

``` r
# Distribution of visits (number of physician office visits)
ggplot(NMES1988, aes(x = visits)) + 
  geom_histogram(binwidth = 1, fill = "blue", color = "black", alpha = 0.7) +
  labs(title = "Distribution of Physician Visits", x = "Number of Visits", y = "Frequency")
```

![](Regression-Categorical-Data-Analysis_files/figure-gfm/variable%20plots-1.png)<!-- -->

# ———————- Bivariate Analysis ———————-

``` r
# Correlation analysis
cor_data <- NMES1988 %>% dplyr::select(visits, age, income)
cor_matrix <- cor(cor_data, method = "spearman")
corrplot(cor_matrix, method = "square")
```

![](Regression-Categorical-Data-Analysis_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
# Cross-tabulations
table(NMES1988$gender, NMES1988$insurance)
```

    ##         
    ##            no  yes
    ##   female  627 2001
    ##   male    358 1420

``` r
table(NMES1988$health, NMES1988$insurance)
```

    ##            
    ##               no  yes
    ##   poor       204  350
    ##   average    721 2788
    ##   excellent   60  283

# ———————- Regression Models ———————-

``` r
# 1. Poisson Model
poisson_model <- glm(visits ~ insurance + age + income + gender + health + new.chronic, 
                     family = "poisson", data = NMES1988)
summary(poisson_model)
```

    ## 
    ## Call:
    ## glm(formula = visits ~ insurance + age + income + gender + health + 
    ##     new.chronic, family = "poisson", data = NMES1988)
    ## 
    ## Coefficients:
    ##                  Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)      1.458049   0.077562  18.798  < 2e-16 ***
    ## insuranceyes     0.265077   0.016326  16.237  < 2e-16 ***
    ## age             -0.049217   0.010106  -4.870 1.12e-06 ***
    ## income           0.002560   0.002119   1.208    0.227    
    ## gendermale      -0.102399   0.013023  -7.863 3.76e-15 ***
    ## healthpoor       0.351110   0.016744  20.969  < 2e-16 ***
    ## healthexcellent -0.323046   0.030387 -10.631  < 2e-16 ***
    ## new.chronic      0.340655   0.009123  37.341  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for poisson family taken to be 1)
    ## 
    ##     Null deviance: 26943  on 4405  degrees of freedom
    ## Residual deviance: 23865  on 4398  degrees of freedom
    ## AIC: 36657
    ## 
    ## Number of Fisher Scoring iterations: 5

``` r
exp(cbind(OR = coef(poisson_model), confint(poisson_model, level = 0.95)))
```

    ## Waiting for profiling to be done...

    ##                        OR     2.5 %    97.5 %
    ## (Intercept)     4.2975656 3.6920298 5.0039069
    ## insuranceyes    1.3035319 1.2626035 1.3460489
    ## age             0.9519747 0.9332721 0.9709855
    ## income          1.0025637 0.9983520 1.0066773
    ## gendermale      0.9026690 0.8798965 0.9259838
    ## healthpoor      1.4206442 1.3746596 1.4679179
    ## healthexcellent 0.7239406 0.6817381 0.7679869
    ## new.chronic     1.4058682 1.3809894 1.4312697

``` r
# To check Overdispersion
# Calculate the observed variance of the dependent variable
observed_variance <- var(NMES1988$visits, na.rm = TRUE)

# Predicted values (mean) from the Poisson model
predicted_means <- predict(poisson_model, type = "response")

# The variance predicted by the Poisson model is approximately equal to the mean
predicted_variance <- mean(predicted_means, na.rm = TRUE)

# Print the results
print(paste("Observed Variance: ", observed_variance))
```

    ## [1] "Observed Variance:  45.6871174020766"

``` r
print(paste("Predicted Variance (Poisson): ", predicted_variance))
```

    ## [1] "Predicted Variance (Poisson):  5.77439855170451"

``` r
# 2. Negative Binomial Model
negbinom_model <- glm.nb(visits ~ insurance + age + income + gender + health + new.chronic, 
                          data = NMES1988)
summary(negbinom_model)
```

    ## 
    ## Call:
    ## glm.nb(formula = visits ~ insurance + age + income + gender + 
    ##     health + new.chronic, data = NMES1988, init.theta = 1.1568979, 
    ##     link = log)
    ## 
    ## Coefficients:
    ##                  Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)      1.270494   0.189990   6.687 2.28e-11 ***
    ## insuranceyes     0.300327   0.038567   7.787 6.85e-15 ***
    ## age             -0.029435   0.024813  -1.186 0.235512    
    ## income           0.004385   0.005375   0.816 0.414586    
    ## gendermale      -0.107400   0.031903  -3.366 0.000761 ***
    ## healthpoor       0.368577   0.047553   7.751 9.12e-15 ***
    ## healthexcellent -0.332036   0.062087  -5.348 8.90e-08 ***
    ## new.chronic      0.346998   0.020949  16.564  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for Negative Binomial(1.1569) family taken to be 1)
    ## 
    ##     Null deviance: 5583.4  on 4405  degrees of freedom
    ## Residual deviance: 5040.5  on 4398  degrees of freedom
    ## AIC: 24494
    ## 
    ## Number of Fisher Scoring iterations: 1
    ## 
    ## 
    ##               Theta:  1.1569 
    ##           Std. Err.:  0.0317 
    ## 
    ##  2 x log-likelihood:  -24476.0490

``` r
exp(cbind(OR = coef(negbinom_model), confint(negbinom_model)))
```

    ## Waiting for profiling to be done...

    ##                        OR     2.5 %    97.5 %
    ## (Intercept)     3.5626108 2.4294627 5.2182785
    ## insuranceyes    1.3502997 1.2511110 1.4565675
    ## age             0.9709944 0.9238891 1.0207421
    ## income          1.0043944 0.9938399 1.0155404
    ## gendermale      0.8981661 0.8435520 0.9564602
    ## healthpoor      1.4456757 1.3178729 1.5879457
    ## healthexcellent 0.7174613 0.6360097 0.8108865
    ## new.chronic     1.4148138 1.3580416 1.4738959

``` r
# 3. Zero-Inflated Negative Binomial Model
zinb_model <- zeroinfl(visits ~ insurance + age + income + gender + health + new.chronic |
                       insurance + age + income + gender + health + new.chronic, 
                       data = NMES1988, dist = "negbin")
summary(zinb_model)
```

    ## 
    ## Call:
    ## zeroinfl(formula = visits ~ insurance + age + income + gender + health + 
    ##     new.chronic | insurance + age + income + gender + health + new.chronic, 
    ##     data = NMES1988, dist = "negbin")
    ## 
    ## Pearson residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -1.1090 -0.7063 -0.2788  0.3256 15.2809 
    ## 
    ## Count model coefficients (negbin with log link):
    ##                   Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)      1.7124438  0.1952603   8.770  < 2e-16 ***
    ## insuranceyes     0.1839516  0.0416499   4.417 1.00e-05 ***
    ## age             -0.0579428  0.0252853  -2.292   0.0219 *  
    ## income          -0.0007383  0.0055098  -0.134   0.8934    
    ## gendermale      -0.0517594  0.0323172  -1.602   0.1092    
    ## healthpoor       0.3596758  0.0458912   7.838 4.59e-15 ***
    ## healthexcellent -0.3116757  0.0647700  -4.812 1.49e-06 ***
    ## new.chronic      0.2781948  0.0220075  12.641  < 2e-16 ***
    ## Log(theta)       0.3339487  0.0363369   9.190  < 2e-16 ***
    ## 
    ## Zero-inflation model coefficients (binomial with logit link):
    ##                 Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)       3.4265     1.6858   2.033  0.04210 *  
    ## insuranceyes     -1.4583     0.2572  -5.669 1.43e-08 ***
    ## age              -0.5666     0.2308  -2.455  0.01409 *  
    ## income           -0.2069     0.1654  -1.251  0.21080    
    ## gendermale        0.9272     0.2892   3.206  0.00135 ** 
    ## healthpoor        0.1133     0.3893   0.291  0.77093    
    ## healthexcellent   0.1999     0.3915   0.510  0.60973    
    ## new.chronic      -1.2095     0.1597  -7.576 3.57e-14 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
    ## 
    ## Theta = 1.3965 
    ## Number of iterations in BFGS optimization: 28 
    ## Log-likelihood: -1.217e+04 on 17 Df

``` r
exp(cbind(OR = coef(zinb_model), confint(zinb_model, level = 0.95)))
```

    ##                               OR     2.5 %      97.5 %
    ## count_(Intercept)      5.5424897 3.7800627   8.1266355
    ## count_insuranceyes     1.2019577 1.1077372   1.3041923
    ## count_age              0.9437040 0.8980756   0.9916505
    ## count_income           0.9992620 0.9885289   1.0101116
    ## count_gendermale       0.9495573 0.8912771   1.0116485
    ## count_healthpoor       1.4328649 1.3096118   1.5677178
    ## count_healthexcellent  0.7322189 0.6449242   0.8313297
    ## count_new.chronic      1.3207434 1.2649859   1.3789586
    ## zero_(Intercept)      30.7681456 1.1301514 837.6566312
    ## zero_insuranceyes      0.2326239 0.1405067   0.3851336
    ## zero_age               0.5674611 0.3609745   0.8920632
    ## zero_income            0.8130770 0.5879969   1.1243157
    ## zero_gendermale        2.5273456 1.4337370   4.4551238
    ## zero_healthpoor        1.1200193 0.5222325   2.4020782
    ## zero_healthexcellent   1.2212363 0.5669257   2.6307118
    ## zero_new.chronic       0.2983457 0.2181845   0.4079582

# ———————- Model Comparisons ———————-

``` r
model_comparison <- data.frame(
  Model = c("Poisson", "Negative Binomial", "ZINB"),
  AIC = c(AIC(poisson_model), AIC(negbinom_model), AIC(zinb_model)),
  BIC = c(BIC(poisson_model), BIC(poisson_model), BIC(zinb_model))
)
print(model_comparison)
```

    ##               Model      AIC      BIC
    ## 1           Poisson 36656.62 36707.75
    ## 2 Negative Binomial 24494.05 36707.75
    ## 3              ZINB 24382.79 24491.43

# ———————- Interaction Terms ———————-

``` r
# Model 1: Main Effect Model
negbinom_model1 <- glm.nb(visits ~ insurance + age + income + gender + health + new.chronic, 
                          data = NMES1988)
summary(negbinom_model1)
```

    ## 
    ## Call:
    ## glm.nb(formula = visits ~ insurance + age + income + gender + 
    ##     health + new.chronic, data = NMES1988, init.theta = 1.1568979, 
    ##     link = log)
    ## 
    ## Coefficients:
    ##                  Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)      1.270494   0.189990   6.687 2.28e-11 ***
    ## insuranceyes     0.300327   0.038567   7.787 6.85e-15 ***
    ## age             -0.029435   0.024813  -1.186 0.235512    
    ## income           0.004385   0.005375   0.816 0.414586    
    ## gendermale      -0.107400   0.031903  -3.366 0.000761 ***
    ## healthpoor       0.368577   0.047553   7.751 9.12e-15 ***
    ## healthexcellent -0.332036   0.062087  -5.348 8.90e-08 ***
    ## new.chronic      0.346998   0.020949  16.564  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for Negative Binomial(1.1569) family taken to be 1)
    ## 
    ##     Null deviance: 5583.4  on 4405  degrees of freedom
    ## Residual deviance: 5040.5  on 4398  degrees of freedom
    ## AIC: 24494
    ## 
    ## Number of Fisher Scoring iterations: 1
    ## 
    ## 
    ##               Theta:  1.1569 
    ##           Std. Err.:  0.0317 
    ## 
    ##  2 x log-likelihood:  -24476.0490

``` r
# Model 2: Interaction between insurance and gender
Nb_model2 <- glm.nb(visits ~ insurance * gender + age + income + health + new.chronic, data = NMES1988)
summary(Nb_model2)
```

    ## 
    ## Call:
    ## glm.nb(formula = visits ~ insurance * gender + age + income + 
    ##     health + new.chronic, data = NMES1988, init.theta = 1.157947142, 
    ##     link = log)
    ## 
    ## Coefficients:
    ##                          Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)              1.305684   0.191978   6.801 1.04e-11 ***
    ## insuranceyes             0.254250   0.047927   5.305 1.13e-07 ***
    ## gendermale              -0.209558   0.070122  -2.988   0.0028 ** 
    ## age                     -0.029168   0.024815  -1.175   0.2398    
    ## income                   0.004225   0.005375   0.786   0.4318    
    ## healthpoor               0.366249   0.047542   7.704 1.32e-14 ***
    ## healthexcellent         -0.331991   0.062084  -5.347 8.92e-08 ***
    ## new.chronic              0.345946   0.020945  16.517  < 2e-16 ***
    ## insuranceyes:gendermale  0.128463   0.078569   1.635   0.1020    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for Negative Binomial(1.1579) family taken to be 1)
    ## 
    ##     Null deviance: 5586.9  on 4405  degrees of freedom
    ## Residual deviance: 5040.8  on 4397  degrees of freedom
    ## AIC: 24493
    ## 
    ## Number of Fisher Scoring iterations: 1
    ## 
    ## 
    ##               Theta:  1.1579 
    ##           Std. Err.:  0.0318 
    ## 
    ##  2 x log-likelihood:  -24473.3830

``` r
# Model 3: Interaction between insurance and health
Nb_model3 <- glm.nb(visits ~ insurance * health + age + income + gender + new.chronic, 
                    data = NMES1988)
summary(Nb_model3)
```

    ## 
    ## Call:
    ## glm.nb(formula = visits ~ insurance * health + age + income + 
    ##     gender + new.chronic, data = NMES1988, init.theta = 1.159269435, 
    ##     link = log)
    ## 
    ## Coefficients:
    ##                               Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)                   1.237282   0.191406   6.464 1.02e-10 ***
    ## insuranceyes                  0.322906   0.044364   7.279 3.37e-13 ***
    ## healthpoor                    0.489307   0.080726   6.061 1.35e-09 ***
    ## healthexcellent              -0.571602   0.159395  -3.586 0.000336 ***
    ## age                          -0.027413   0.024825  -1.104 0.269479    
    ## income                        0.004046   0.005373   0.753 0.451404    
    ## gendermale                   -0.104415   0.031889  -3.274 0.001059 ** 
    ## new.chronic                   0.346804   0.020943  16.559  < 2e-16 ***
    ## insuranceyes:healthpoor      -0.188027   0.097815  -1.922 0.054571 .  
    ## insuranceyes:healthexcellent  0.277215   0.172247   1.609 0.107528    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for Negative Binomial(1.1593) family taken to be 1)
    ## 
    ##     Null deviance: 5591.2  on 4405  degrees of freedom
    ## Residual deviance: 5040.2  on 4396  degrees of freedom
    ## AIC: 24491
    ## 
    ## Number of Fisher Scoring iterations: 1
    ## 
    ## 
    ##               Theta:  1.1593 
    ##           Std. Err.:  0.0318 
    ## 
    ##  2 x log-likelihood:  -24469.0090

# Likelihood Ratio Tests

``` r
lrtest(negbinom_model1, Nb_model2) # Between Models 1 and 2
```

    ## Likelihood ratio test
    ## 
    ## Model 1: visits ~ insurance + age + income + gender + health + new.chronic
    ## Model 2: visits ~ insurance * gender + age + income + health + new.chronic
    ##   #Df LogLik Df  Chisq Pr(>Chisq)
    ## 1   9 -12238                     
    ## 2  10 -12237  1 2.6661     0.1025

``` r
lrtest(negbinom_model1, Nb_model3) # Between Models 1 and 3
```

    ## Likelihood ratio test
    ## 
    ## Model 1: visits ~ insurance + age + income + gender + health + new.chronic
    ## Model 2: visits ~ insurance * health + age + income + gender + new.chronic
    ##   #Df LogLik Df  Chisq Pr(>Chisq)  
    ## 1   9 -12238                       
    ## 2  11 -12234  2 7.0397     0.0296 *
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

# ———————- Model Comparisons ———————-

``` r
# Compare AIC and BIC for all three models
# Extract AIC and BIC for Model 1, Model 2, and Model 3
model_comparisons <- data.frame(
  Model = c("Model 1", "Model 2", "Model 3"),
  AIC = c(AIC(negbinom_model1), AIC(Nb_model2), AIC(Nb_model3)),
  BIC = c(BIC(negbinom_model1), BIC(Nb_model2), BIC(Nb_model3))
)

# Print the comparison table
print(model_comparisons)
```

    ##     Model      AIC      BIC
    ## 1 Model 1 24494.05 24551.57
    ## 2 Model 2 24493.38 24557.29
    ## 3 Model 3 24491.01 24561.31

# ———————- Scatterplot for Interaction Term Analysis ———————-

``` r
#Interaction Term : Insurance * Gender
library(ggplot2)
library(dplyr)

# Check if `gender` is a factor and convert if needed
if (!is.factor(NMES1988$gender)) {
  NMES1988$gender <- as.factor(NMES1988$gender)
}

# Filter the data to ensure no missing values
NMES1988 <- NMES1988 %>%
  filter(!is.na(gender) & !is.na(visits) & !is.na(insurance))

# Fit the Poisson model with interaction between gender and insurance
gender_model <- glm(visits ~ gender * insurance, data = NMES1988, family = "poisson")

# Extract the p-values for all interaction terms
interaction_terms <- grep("gender.*:insurance", rownames(summary(gender_model)$coefficients), value = TRUE)
p_values <- summary(gender_model)$coefficients[interaction_terms, "Pr(>|z|)"]

# Print p-values for clarity
print(p_values)
```

    ## [1] 1.871399e-09

``` r
# Scatter plot with regression line and dots representing gender
ggplot(NMES1988, aes(x = insurance, y = visits, color = gender)) +
  geom_point(alpha = 0.7, position = position_jitter(width = 0.2)) +
  stat_summary(fun = mean, geom = "line", aes(group = gender), linewidth = 1.2) +
  facet_wrap(~gender) +
  labs(
    title = "Interaction: Insurance, Visits, and Gender",
    x = "Insurance Status",
    y = "Number of Visits"
  ) +
  theme_minimal()
```

![](Regression-Categorical-Data-Analysis_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
library(ggplot2)
library(dplyr)

# Check if `health` is a factor and convert if needed
if (!is.factor(NMES1988$health)) {
  NMES1988$health <- as.factor(NMES1988$health)
}

# Filter the data to ensure no missing values
NMES1988 <- NMES1988 %>%
  filter(!is.na(health) & !is.na(visits) & !is.na(insurance))

# Fit the Poisson model with interaction between health and insurance
health_model <- glm(visits ~ health * insurance, data = NMES1988, family = "poisson")

# Extract the p-values for all interaction terms
interaction_terms <- grep("health.*:insurance", rownames(summary(health_model)$coefficients), value = TRUE)
p_values <- summary(health_model)$coefficients[interaction_terms, "Pr(>|z|)"]

# Print p-values for clarity
print(p_values)
```

    ##      healthpoor:insuranceyes healthexcellent:insuranceyes 
    ##                 1.448095e-05                 1.146500e-04

``` r
# Scatter plot with regression line and dots representing health
ggplot(NMES1988, aes(x = insurance, y = visits, color = health)) +
  geom_point(alpha = 0.7, position = position_jitter(width = 0.2)) +
  stat_summary(fun = mean, geom = "line", aes(group = health), linewidth = 1.2) +
  facet_wrap(~health) +
  labs(
    title = "Interaction: Insurance, Visits, and Health",
    x = "Insurance Status",
    y = "Number of Visits"
  ) +
  theme_minimal()
```

![](Regression-Categorical-Data-Analysis_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

# ———————- Final Model Selection ———————-

``` r
# Choose Model 3 as final model based on AIC/BIC and likelihood ratio tests
final_model <- Nb_model3  # Model 3 with the interaction between insurance and health
```

# ———————- Predicted Visits ———————-

``` r
# Generate predictions based on the final model
predictions <- NMES1988 %>%
  mutate(pred_visits = predict(final_model, type = "response"))

# Plot predicted visits by health status and insurance type
ggplot(predictions, aes(x = health, y = pred_visits, fill = insurance)) +
  geom_bar(stat = "identity", position = "dodge") +
  labs(title = "Predicted Visits by Health and Insurance Status",
       x = "Health Status", y = "Predicted Visits")
```

![](Regression-Categorical-Data-Analysis_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

# ———————- Goodness-of-Fit ———————-

``` r
# Deviance and Pearson residuals
gof <- residuals(final_model, type = "pearson")
print(paste("Goodness-of-Fit (Pearson Residuals):", sum(gof^2)))
```

    ## [1] "Goodness-of-Fit (Pearson Residuals): 5611.61443445113"

``` r
# Generate Pearson residuals from the Negative Binomial model
pearson_residuals <- residuals(final_model, type = "pearson")

# Q-Q plot for Pearson residuals
qqnorm(pearson_residuals, main = "Q-Q Plot of Pearson Residuals" , col= "skyblue")
qqline(pearson_residuals, col = "salmon", lwd = 2)
```

![](Regression-Categorical-Data-Analysis_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->
