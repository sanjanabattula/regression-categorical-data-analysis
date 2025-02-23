# The Impact of Insurance on Physician Office Visits among Medicare Beneficiaries Aged 66 and Over: A Statistical Analysis Using NMES1988 Data
**GPH-GU 2354: Categorical Data Analysis**  

**Sanjana Battula** 
**Farah Beche**  
**Kajal Gupta**  
 
---

## Introduction
  Healthcare utilization serves as a vital indicator for assessing the accessibility and effectiveness of healthcare systems. For Medicare beneficiaries aged 66 and older, understanding the factors that influence physician office visits is particularly crucial due to their increased healthcare needs. This cross-sectional study explores the impact of private insurance, along with demographic, socioeconomic, and health-related factors, on the frequency of physician office visits. Drawing on data from the 1988 US National Medical Expenditure Survey (NMES), the analysis aims to shed light on how insurance coverage shapes healthcare utilization patterns within this nationally representative cohort of older adults during the 1987-1988 period.

  Research consistently demonstrates the significant role of private insurance in increasing healthcare utilization by lowering out-of-pocket expenses and enhancing access to providers. For example, in 1995 Miller and Luft found that individuals with supplemental insurance were more likely to use preventive services and have routine physician visits than those without.¹ Similarly, in 2021 the National Healthcare Quality and Disparities Report emphasized income and insurance coverage as critical factors influencing healthcare access and utilization among older adults.² Although the NMES1988 data predates these studies, the relationships between insurance, income, and healthcare utilization remain relevant. These factors have consistently shaped healthcare behavior over time, aligning with long-standing health economic principles, such as the influence of financial barriers on access to care.

---

## Research Questions
1. **Model 1:** Does having private insurance significantly influence the number of physician office visits among Medicare beneficiaries aged 66 and over?
2. **Model 2:** Does the effect of private insurance on the number of physician office visits differ by gender?
3. **Model 3:** Does the effect of private insurance on the number of physician office visits vary by health level?

---

## Methods and Data Analysis
  The NMES1988 dataset was analyzed to identify predictors of physician office visits, with independent variables including age, income, gender, insurance status, health level, employment status, and chronic condition status (new.chronic: 0 = no chronic condition, 1 = one chronic condition, 2 = two or more chronic conditions). Descriptive statistics summarized key characteristics, Spearman's rank correlation assessed relationships between continuous variables, and cross-tabulations analyzed categorical variables.

  Given the count nature of the dependent variable, a Poisson regression model was initially used but exhibited overdispersion (variance > mean). A Negative Binomial (NB) regression model, which accounts for overdispersion, was subsequently employed. A Zero-Inflated Negative Binomial (ZINB) model was also considered due to the zero-inflated distribution of the dependent variable. However, minimal improvement in fit (AIC: 24,382.79; BIC: 24,491.43) did not justify its added complexity, leading to the NB model's selection.

  Three regression models were developed: Model 1 included main effects, Model 2 added an interaction between insurance and gender, and Model 3 included an interaction between insurance and health level. Model performance was assessed using AIC, BIC, and likelihood ratio tests. Predicted values from the final model estimated expected physician visits across subgroups, visualizing interaction effects.

  Goodness-of-fit diagnostics, including Q-Q plots of Pearson residuals, checks for overdispersion, and residual pattern analysis, confirmed the NB model's adequacy in capturing data variability and guiding model selection.

---

## Results
  Descriptive analysis revealed a right-skewed, zero-inflated distribution of physician visits, with 15% of participants reporting no visits. A histogram illustrated the predominance of lower visit frequencies. The mean was 5.77 visits, the median 4 visits, and the variance was 45.69, indicating overdispersion.

  Bivariate analyses revealed significant associations between key variables and healthcare utilization, especially for categorical variables. Cross-tabulations showed that females had more visits than males (2,628 vs. 1,778), and insured participants had substantially more visits than uninsured participants (3,421 vs. 985). Health status was strongly associated with utilization, with participants in average health accounting for most visits (3,509), followed by those in poor (554) and excellent health (343). Among participants with average health, 2,788 were insured, compared to 721 who were uninsured. These patterns highlight insurance coverage and health status as key predictors of physician visits.

  The Poisson regression model initially used revealed poor fit (AIC = 36,656.62, BIC = 36,707.75), prompting the use of a Negative Binomial (NB) model. The NB model provided a better fit, with lower AIC (24,494.05) and BIC (24,551.57) values. Key predictors included private insurance (OR = 1.35, 95% CI: 1.25–1.46, p < 0.001), poor health status (OR = 1.45, 95% CI: 1.32–1.59, p < 0.001), and a higher number of chronic conditions (OR = 1.42, 95% CI: 1.36–1.47, p < 0.001). Gender was also significant, with males having fewer visits than females (OR = 0.90, 95% CI: 0.84–0.96, p = 0.001). The Zero-Inflated Negative Binomial (ZINB) model, which accounts for excess zeros, showed slightly better fit (AIC = 24,382.79, BIC = 24,491.43), but the improvement was minimal, and its added complexity was not justified, confirming the NB model as sufficient.

  Two interaction models were developed to explore potential moderating effects. Model 2, which included an interaction between insurance and gender, did not show a significant effect (β = 0.12, p = 0.12). A likelihood ratio test comparing Model 2 with the main-effects NB model confirmed no significant improvement in fit (χ² = 2.67, p = 0.1025). Fit statistics for Model 2 (AIC = 24,493.38; BIC = 24,557.29) were slightly worse than the main-effects NB model. Scatterplots confirmed these findings, showing that insured females had slightly more visits than uninsured females, while males showed minimal differences. Additionally, females consistently reported more visits than males, supporting the significant main effect of gender.

  Model 3, which included an interaction between insurance and health status, revealed a significant interaction for participants in poor health (β = -0.21, p = 0.03), indicating that private insurance had a reduced effect on visits among those with poor health. Model 3 provided the best fit, with the lowest AIC (24,491.01) and BIC (24,561.31). The likelihood ratio test confirmed a significant improvement over the main-effects NB model (χ² = 7.04, p = 0.0296). Scatterplots showed that insured participants in average health had the highest number of visits, while those in poor health reported fewer. Uninsured participants in excellent health had the lowest visits. Predicted values from Model 3 supported these trends, with insured participants in average health having the most visits (7.3), followed by those in poor health (5.4), and uninsured participants in excellent health having the fewest (2.8). Bar plots further illustrated these variations, highlighting the interaction effects.

  Goodness-of-fit diagnostics confirmed Model 3’s adequacy. The Q-Q plot showed slight deviations at the upper tail, indicating minor departures from normality. The sum of squared Pearson residuals aligned with the model’s degrees of freedom, indicating no overdispersion. Residual plots showed no systematic patterns, confirming that Model 3 captured the data variability and provided the best fit for physician visits. It accurately represented healthcare utilization, highlighting the interaction between health status and insurance coverage in influencing visits.

---

## Conclusion and Discussion
This study employed a comprehensive analytical approach to address the dataset's unique characteristics, such as overdispersion and zero inflation in physician visit counts. The Negative Binomial (NB) model sufficiently captured the overall variability in visits, while the Zero-Inflated Negative Binomial (ZINB) model accounted for excess zeros, with Model 3 emerging as the best fit due to its ability to explain significant interactions between insurance coverage and health status. Despite these strengths, the study is limited by its reliance on cross-sectional data from 1987-1988, which may not reflect contemporary healthcare utilization patterns or the evolving role of supplemental insurance. Additionally, while the models effectively addressed the research questions, they did not explore the pathways through which demographic, socioeconomic, and health-related factors interact to shape healthcare utilization.

An alternative approach could be to employ machine learning techniques, such as decision trees or random forests, to model the predictors of physician visits. These methods can potentially handle the complex interactions and nonlinearities in the data more effectively than traditional regression models. Additionally, the use of longitudinal data, if available, could provide a more dynamic perspective on how changes in insurance coverage, health status, and other factors influence healthcare utilization over time.

Overall, this study highlights the critical roles of insurance coverage, health status, and chronic conditions in shaping physician visit frequency among Medicare beneficiaries aged 66 and older. These findings emphasize the importance of supplemental insurance in improving healthcare access and utilization, providing valuable insights for policymakers aiming to address disparities in healthcare delivery.

---

## References
1. Miller RH, Luft HS. Does managed care lead to better or worse quality of care? Health Aff (Millwood). 1995;14(4):7-25.
2. Agency for Healthcare Research and Quality. Access to Healthcare and Disparities in Access. National Healthcare Quality and Disparities Report. 2021.
