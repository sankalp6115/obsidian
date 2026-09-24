## Detailed Syllabus Notes

# 1. Statistics — 🔴 Highest Priority

### Descriptive Statistics

Mean,Median,Mode,Weighted mean,Variance,Standard deviation,Range,IQR,Percentiles,Quartiles,Z-score,Coefficient of variation

### Distribution & Shape
Normal distribution, Standard normal distribution, Skewness, Kurtosis, Right/left skew, Outliers, Empirical rule / 68-95-99.7 rule

### Probability

Basic probability, Conditional probability, Independence, Bayes' theorem, Expected value, Random variables, Probability distributions

### Important Distributions

Bernoulli, Binomial, Poisson, Normal, Exponential, Uniform

### Sampling

Population vs sample, Sampling methods, Sampling bias, Sampling distribution, Standard error, Central Limit Theorem

### Inferential Statistics

Point estimation, Interval estimation, Confidence intervals, Hypothesis testing, Null hypothesis, Alternative hypothesis, Test statistic, p-value, Significance level, Type I error, Type II error, Statistical power

### Tests

Know **when and why** to use:
Z-test, T-test, Paired t-test, Chi-square test, ANOVA
You don't need to memorize every derivation. You **do need to know assumptions, interpretation, and use cases**.

---

# 2. A/B Testing & Experimentation — 🔴 Highest Priority

This deserves its own section because it's extremely relevant to product/e-commerce analytics.

### Fundamentals

Control vs treatment, Randomization, Experiment hypothesis, Primary metric, Secondary metrics, Guardrail metrics, Baseline

### Statistical Concepts

Statistical significance, Practical/business significance, p-value, Confidence interval, Statistical power, Minimum detectable effect, Sample size, Type I / Type II errors

### Experiment Design

Choosing the right metric, Experiment duration, Sample ratio mismatch, Randomization, Segmentation, Novelty effect, Seasonality, External events

### Common Problems

Selection bias, Survivorship bias, Peeking, Multiple testing, Simpson's paradox, Network/interference effects

### Product A/B scenarios

Be able to reason through:

New checkout page, New recommendation algorithm, New search ranking, New pricing strategy, New loyalty program, New notification system

---

# 3. Product & E-commerce Analytics — 🔴 Highest Priority

For Flipkart, I would spend **a lot of time here**.

## User Funnel

Understand:

**Impression → Click → Product View → Add to Cart → Checkout → Payment → Order → Delivery → Repeat Purchase**

Know metrics at every stage.

### Funnel Metrics

CTR, Product view rate, Add-to-cart rate, Checkout conversion, Payment success rate, Order conversion, Overall conversion rate, Drop-off rate

---

# 4. Business Metrics — 🔴

### Revenue

Revenue, Gross revenue, Net revenue, GMV, NMV, Profit, Contribution margin

### Customer

DAU, WAU, MAU, New users, Active users, Returning users, Retention, Churn, Reactivation

### Transaction

Number of orders, Units sold, AOV, Average items/order, Purchase frequency

### Marketplace

Seller count, Active sellers, Seller conversion, Seller cancellation, Seller fulfillment, Seller quality, Stock availability, Out-of-stock rate

### Logistics

Delivery time, On-time delivery rate, Late delivery rate, P50 delivery time, P90 delivery time, P95 delivery time, Cancellation rate, Return rate

---

# 5. Customer Analytics — 🔴

### Customer Segmentation

Demographic segmentation, Behavioral segmentation, Geographic segmentation, RFM analysis

### RFM

Understand:

Recency, Frequency, Monetary value

And how you would segment customers using them.

### Customer Lifecycle

Understand:

**Acquisition → Activation → Engagement → Purchase → Retention → Churn → Reactivation**

### Retention

Cohort analysis, Day 1 retention, Day 7 retention, Day 30 retention, Monthly retention, Repeat purchase rate

### Churn

Churn definition, Churn rate, Churn prediction conceptually, Early indicators of churn, Retention strategies

---

# 6. Cohort Analysis — 🔴

Know this well.

Understand:

What is a cohort?, Acquisition cohort, Behavioral cohort, Time-based cohort, Retention matrix, Revenue cohort, Purchase cohort

Example:

> Users who first purchased in January — how many came back in February, March, April?

Be able to interpret the resulting matrix.

---

# 7. Business Case / Product Case Frameworks — 🔴

This isn't a "topic" in the traditional sense, but **you need to practice it**.

### Metric Drop

Know how to approach:

> "Orders dropped 15%."

Think:

**Define → Segment → Funnel → Hypothesize → Validate → Recommend**

### Metrics Increase

> "Revenue increased 20%."

Ask:

Why?, More users?, Higher conversion?, Higher AOV?, Higher prices?, More orders?, Different product mix?

### Root Cause Analysis

Learn:

Segmentation, Drill-down analysis, Funnel decomposition, Time-series comparison, Cohort comparison, Contribution analysis

---

# 8. Product Metrics Framework — 🔴

For any new Flipkart feature, know how to define:

### North Star Metric

Understand what a North Star metric is and its limitations.

### Input Metrics

Metrics that drive the outcome.

### Output Metrics

The ultimate business outcome.

### Guardrail Metrics

Metrics ensuring you don't improve one thing while damaging another.

Example:

A recommendation system might increase:

**CTR ↑**

but:

**Conversion ↓**

or:

**Returns ↑**

You need to think beyond one metric.

---

# 9. Data Interpretation — 🔴

You may be given a chart/table and asked:

> "What do you see?"

Study how to analyze:

### Time Series

Trend, Seasonality, Cyclicality, Spikes, Drops, Moving averages, Growth rates

### Comparisons

YoY, MoM, WoW, Before/after, Control/treatment

### Segmentation

Geography, Category, Device, Customer type, Seller, Price range

### Contribution

Absolute contribution, Percentage contribution, Pareto principle / 80-20

---

# 10. Data Visualization — 🟡/🔴

Know **when to use what chart**.

### Charts

Bar chart, Histogram, Box plot, Scatter plot, Line chart, Area chart, Pie/donut — understand limitations, Heatmap, Funnel chart, Cohort heatmap, KPI cards

### Understand

Distribution, Outliers, Trends, Correlation, Comparisons, Composition

### Dashboard Design

Know:

KPI selection, Filters, Drill-downs, Time comparison, Segmentation, Avoiding clutter, Avoiding misleading visualizations

---

# 11. Excel — 🟡

If the role mentions Excel, prepare:

### Fundamentals

Relative vs absolute references, Sorting, Filtering, Conditional formatting, Data validation

### Functions

SUM, SUMIF / SUMIFS, COUNT, COUNTIF / COUNTIFS, AVERAGE, AVERAGEIF, IF, IFERROR, AND / OR, XLOOKUP, INDEX + MATCH, TEXT functions, Date functions

### Analysis

Pivot tables, Pivot charts, Slicers, Basic dashboarding

### Advanced-ish

Conditional aggregation, Dynamic ranges, Basic Power Query concepts

Don't spend excessive time on Excel unless it's explicitly in the JD.

---

# 12. Python for Data Analysis — 🔴

Since we're excluding ML, focus purely on **analytics Python**.

### Python Fundamentals

Variables, Data types, Lists, Tuples, Sets, Dictionaries, Loops, Conditions, Functions, List comprehensions, Lambda functions

### NumPy

Arrays, Indexing, Vectorization, Mean, Median, Standard deviation, Percentiles, Basic array operations

### Pandas

Series, DataFrame, Reading data, Filtering, Sorting, Groupby, Aggregation, Merge, Concatenation, Pivot tables, Missing values, Duplicates, Apply, Lambda, Datetime, String operations

### EDA

Shape, Data types, Missing values, Duplicates, Outliers, Distributions, Correlations, Group-level analysis

---

# 13. Data Cleaning — 🔴

Very important for analytics interviews.

Know how to handle:

Missing values, Duplicate records, Incorrect data types, Invalid values, Outliers, Inconsistent categories, Incorrect dates, Data-entry errors, Null vs zero, Duplicate entities

Most importantly:

> **Don't blindly clean data.**

Be able to explain **why** you would remove, replace, transform, or retain something.

---

# 14. Data Quality — 🟡

Understand:

Accuracy, Completeness, Consistency, Validity, Uniqueness, Timeliness

And concepts such as:

Data validation, Data integrity, Duplicate records, Missing data, Anomalies

---

# 15. Correlation & Causal Reasoning — 🔴

Know:

Pearson correlation, Spearman correlation, Positive/negative correlation, Correlation ≠ causation, Confounding variables, Selection bias, Reverse causality

Also learn:

### Simpson's Paradox

This is a good interview-level concept.

---

# 16. Time-Series Analytics — 🟡/🔴

Know:

Trend, Seasonality, Moving average, Rolling average, Growth rate, MoM, WoW, YoY, CAGR, Seasonality adjustment conceptually

For e-commerce:

Festival season, Weekend effects, Payday effects, Sale events, Holiday effects

Think about why raw comparisons can be misleading.

---

# 17. Forecasting Fundamentals — 🟡

Without going deep into ML, understand:

Naive forecasting, Moving-average forecasting, Trend, Seasonality, Forecast horizon, Forecast error, MAE, RMSE, MAPE

And business considerations:

Promotions, Holidays, New products, Stockouts, Price changes

---

# 18. Sampling & Bias — 🔴

Know:

Random sampling, Stratified sampling, Cluster sampling, Convenience sampling

And biases:

Selection bias, Sampling bias, Survivorship bias, Response bias, Non-response bias, Measurement bias

These are very useful in product analytics.

---

# 19. Common Analytical Fallacies — 🟡

Learn to recognize:

Correlation vs causation, Survivorship bias, Simpson's paradox, Selection bias, Confirmation bias, Small sample conclusions, Base-rate neglect, Misleading averages, Aggregation bias, Multiple comparisons

---

# 20. Business/Growth Analytics — 🔴

Understand:

### Acquisition

Traffic, CTR, CAC, Conversion

### Activation

First purchase, First meaningful action

### Engagement

DAU/MAU, Session frequency, Session duration, Feature usage

### Monetization

Revenue, AOV, Purchase frequency, Margin

### Retention

Repeat purchase, Cohort retention, Churn

### Growth

Understand the relationship:

**Users × Conversion × Frequency × AOV**

This mental model is extremely useful.

---

# 21. Unit Economics — 🟡/🔴

Know:

CAC, LTV, AOV, Gross margin, Contribution margin, Variable cost, Fixed cost, Payback period

Understand:

> Revenue ≠ profit.

And:

> Increasing GMV doesn't necessarily mean improving economics.

---

# 22. Marketplace Analytics — 🔴

Particularly relevant to Flipkart.

Understand the interaction between:

**Customer ↔ Platform ↔ Seller ↔ Logistics**

Metrics:

Seller performance, Seller cancellation, Product availability, Inventory, Price competitiveness, Delivery, Returns, Customer satisfaction

Think about **trade-offs**, not isolated metrics.

---

# 23. Search & Recommendation Analytics — 🟡/🔴

Even without studying ML, understand the analytics side.

### Search

Search volume, Search success rate, Search CTR, Zero-result rate, Search-to-purchase conversion, Query abandonment

### Recommendation

Impression, CTR, Add-to-cart rate, Conversion, Revenue per impression, Average order value, Repeat usage

Also understand:

> Optimizing CTR alone can produce undesirable outcomes.

---

# 24. Customer Experience Analytics — 🟡

Know:

Ratings, Reviews, NPS, CSAT, Complaint rate, Contact rate, Resolution time, Return reasons, Cancellation reasons

Understand how qualitative customer feedback can be converted into measurable metrics.

---

# 25. Analytical Communication — 🔴

This is **seriously underrated**.

Practice explaining an analysis in this structure:

### 1. What happened?

> Conversion declined 8%.

### 2. Where?

> Primarily mobile users in the electronics category.

### 3. Why?

> The decline coincided with a payment failure increase.

### 4. How confident are we?

> The change is statistically significant / evidence is insufficient.

### 5. What should we investigate/do next?

> Investigate payment gateway failures and run an experiment.

Learn to distinguish:

**Observation → Interpretation → Hypothesis → Conclusion**

Don't mix them.

---

# 26. Your Resume / Project Deep Dive — 🔴🔴🔴

This deserves **maximum preparation**.

For every project on your resume, know:

Business problem, Objective, Dataset, Data source, Number of records, Important variables, Data cleaning, EDA, Important insights, Methodology, Why you made each decision, Results, Limitations, Business impact, What went wrong, What you would change, What you learned

And assume the interviewer can ask:

> **"Why?"**

after literally every sentence.

---

# 27. Behavioral / HR — 🟡

Prepare:

Tell me about yourself, Walk me through your resume, Why Flipkart?, Why Data Analytics/Data Science?, Why this role?, Biggest project, Difficult problem, Failure, Conflict, Working under pressure, Working with ambiguity, Working with stakeholders, Leadership example, Data-driven decision you made, Something you learned recently

For each behavioral answer, prepare a **STAR** structure:

**Situation → Task → Action → Result**

---

# 28. Flipkart/E-commerce Domain Knowledge — 🔴

Spend some time understanding:

Marketplace model, First-party vs marketplace, Sellers, Inventory, Warehouses, Logistics, Search, Recommendations, Payments, Returns, Cancellations, Pricing, Discounts, Promotions, Customer acquisition, Customer retention

You don't need to memorize company trivia.

You need to understand **how an e-commerce marketplace works**.
