# Meta-analysis
This is a repo for anyone wanting to use R to perform meta-analysis and these codes have been usd consistently by me to publish in high impact academic journals, in the time before large language models and vibe coding. You can just sub in your column names with really very little experience of coding - this was how I started and now I am keen to help others do the same. I've tried vibe coding recently for meta-analysis and the outputs are so verbose!! It's very simple really, just a few lines. I'll talk about a few things I did in this script, why they are useful and then I'll upload the whole script and test data from a recently published MA https://journals.lww.com/annalsofsurgery/abstract/9900/a_systematic_review_and_meta_analysis_of_the.1300.aspx  

## Meta-analysis of binary outcomes
Also known as metabin, this method allows you to compare outcomes between a treatment group and a control group. Usually this is for randomised controlled trials (RCTs) but it can be used with non-randomised studies of interventions. RCTs really should be used for this to be valid; the whole point is that you are randomising confounding variables between the arms of the study and ay factors which may effect the study outcome e.g. age, weight etc are balanced between the two arms. The effect in meta-analysis is that when you then proceed to compare different studies, the raw event rates in the studies may be different by the overall effect size may still be maintained. The information required to run a meta-analysis of binary outcomes is very simple - you require events (e1) and number of participants (n1) in the intervention arm vs events (e2) and number of participants (n2) in the control arm. 

## Meta-analysis of proportions (single armed meta-analysis) 
Meta-analysis of single armed studies is honestly not that valid by itself, but you can use it to support the findings of a meta-analysis of binary outcomes when there is no head-to-head evidence (as in this study of iliofemoral DVT) or when you need a pooled estimate of an event rate (e.g. sample size calculations for a grant application). This is effectively a biased technique where you are at the mercy of the quality of underlying studies (rubbish in = rubbish out), but sometimes there is nothing else. Importantly, you cannot meta-analyse studies with 0 events without performing some kind of transformation. People tend to use a logit transformation for low incidence outcomes, say less than 15-20%, and double arcisine tukey trasnformation if you need studies with no events to be counted in the analysis. The other option is to add a continuity correction to all studies or just the studies which have 0 events (allincr = X, addincr = X). Both these methods willl attribut a study weight. 

## Fixed-effects vs random-effects
Fixed effects is a term that describes the observed variance within your sample of studies. It assumes that all the studies belong to the same sampling distriubution, and that any variance observed between the studies is simply a reflection of normal expected sampling variance. In meta-analysis, the total observed variance is usally given in terms of I^2. It's a bit of a heuristic but some people go by I2 < 25% fixed effects, 25-50% random effects, >50% consider meta-regression to explain additional variance, >70% consider not performign meta-analysis. I'm not saying taht is the right approach, but i've seen it a lot.

Random-effects assumes each study has its own true effect, and you're estimating the average of these different distributions of effects. With random effects, there should really be an attemtp to characterise the sources of heterogeneity using R^2 and subgroup analysis/ meta-regression. R^2 is the proportion of between study variance explained by a given factor.

The way to decide fixed vs random is to really know your studies. Are the populations, treatments and conduct of the studies sufficiently similar? If so, fixed-effects is the way to go. If not, random-effects and try to explain why the studies are different (maybe they use additional co-interventions that you can regress for - we would have liked to do this in this study but reporting of who received co-intervention and who developed outcomes were not stated.  

Modify fixed vs random effects in the code using comb.fixed = T or comb.random = T

## Dealing with heterogeneity - meta-regression 
Meta-regression sounds complicated but actually it's pretty simple - it explores how study-level characteristics (co-variates) influence the magnitude of treatment effects across studies. Instead of simply pooling effect sizes, meta-regression treats the effect size from each study as the dependent variable and examines whether factors like follow-up duration, sample size, patient age, or intervention type can explain between-study heterogeneity. You are effectively charting the study outcomes and analysing how much of the variability is explained by a given co-variate; it can be performed in a single line of code using rma() as below. Because you are drawing a regression line, you can also pull out teh intercept and the slope $b[1] and $b[2]
m_reg_RCT=rma(yi,vi,data=ies.RCT.PTS,mods=~PTS_fu_length+n_int,method="DL"). 
m_reg_RCT
m_reg_RCT$b[1]
m_reg_RCT$b[2]

## Key plots 
The forest plot is the great bastion of met-analysis. However, in order for the code to work you must make sure you remove rows with NA values for events or controls. Also excel tends to save the column as a string column if you've ever typed anything except a number in there so if you're encountering and error with type arguments try this. 

Forest plots are the standard visualisation for meta-analysis results, displaying individual study effects and the overall pooled estimate in a single, intuitive graphic. Each horizontal line represents one study, with a square (whose size is proportional to the study's weight/precision) marking the point estimate and horizontal lines extending to show the confidence interval - wider lines indicate less precise studies with more uncertainty. The vertical line typically represents "no effect" (e.g., OR = 1.0 or risk difference = 0), allowing you to quickly see which studies favor treatment (squares to one side) versus control (squares to the other side). At the bottom, a diamond shape represents the pooled meta-analysis result, with the diamond's center showing the combined effect estimate and its width representing the confidence interval - if the diamond doesn't cross the line of no effect, the result is statistically significant. 

Square size: Proportional to study weight
Diamond: Pooled effect estimate with confidence interval
Colors: Navy for individual studies, maroon for pooled estimates

Meta-Regression Results
Intercept (b[1]): Baseline effect at time zero
Slope (b[2]): Rate of change over time
Confidence intervals: Available for all estimates

## Usage Instructions

### Setup Environment:
r# Install and load packages
require(pacman)
pacman::p_load(meta, metafor, tidyverse, ggplot2)

### Load Data:
rdat_int <- import("~/Desktop/baseline_characteristics_v7.csv")

pre-process the data yourself first!! Important to ensure type errors not causing issues in columns. 

### Run Primary Analysis:
Execute RCT head-to-head comparisons
Generate forest plots for main outcomes
Calculate pooled proportions

### Publication Bias Assessment:
Use funnel plots and statistical tests
Evaluate small-study effects

### Meta-analysis of proportions 
Generate event rates in single-armed studies and using single-arms of head-to-head studies
Calculate the effect size and variance for each study, so tat meta-regression can be performed. Then plot the forest plot
If you do not include events and n in each study then you will not generate your pooled estimate in teh forest plot 

### Meta-regression/ heterogeneity/ Temporal Analysis:
Run meta-regression models
Generate time-series visualizations
Assess temporal trends

### Notes and Considerations
The analysis uses both fixed and random-effects models for reasons stated above
Continuity correction is set to zero (incr=0) for proportion meta-analysis
Follow-up duration is incorporated as a moderator variable
Publication bias assessment is included for primary outcomes

### Customization
Key parameters that can be modified:
Forest plot limits: Adjust xlim parameters
Scale: Modify pscale for percentage display
Colors: Change col.square and col.diamond values
Statistical methods: Switch between fixed/random effects
Transformations: Modify sm parameter (OR, RD, PR, PLO)

### Troubleshooting
Common issues:
Ensure CSV files are in the correct directory path
Check column names match those referenced in the code
Verify data types (numeric for counts, character for labels)
Handle missing data appropriately before analysis

