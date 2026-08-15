Which measure of central tendency is defined as the balancing point of a dataset and is sensitive to extreme values?

1/1

Median

Mode

Arithmetic Mean

Midrange

What is the midrange of a dataset?

1/1

The middle value when data is sorted in ascending order

The difference between the 75th and 25th percentiles

The average of the largest and smallest values in the dataset

The most frequently occurring observation

For a positively (right-skewed) distribution, what is the typical relationship between the mean, median, and mode?

1/1

Mean < Median < Mode

Mode < Median < Mean

Median < Mean < Mode

Mean = Median = Mode

Why is the median considered a resistant measure of center compared to the mean?

1/1

It uses every single data value in its algebraic calculation.

It is unaffected by the presence of extreme values or outliers.

It can only be calculated for qualitative/categorical data.

It always equals the arithmetic average in skewed distributions.

Which measure of central tendency is most suitable and meaningful for categorical (qualitative) data?

1/1

Arithmetic Mean

Weighted Mean

Mode

Midrange

When a numerical distribution is skewed in either direction, which pair of statistics is recommended to describe its center and spread?

1/1

Mean and Standard Deviation

Mode and Range

Median and Interquartile Range (IQR)

Midrange and Variance

How is the Interquartile Range (IQR) defined?

1/1

The difference between the maximum and minimum values

IQR = Q3 - Q1, representing the middle 50 percent of the data

The square root of the variance divided by the sample size

IQR = Q2 - Q1, representing the lower 25 percent of the data

According to standard boxplot conventions, a data point is classified as an outlier if it falls beyond:

1/1

1.0 times IQR from the mean

1.5 times IQR above Q3 or below Q1

2.0 times IQR from the median

3.0 times IQR above the maximum value

What is the primary focus of Exploratory Data Analysis (EDA) compared to classical quantitative statistical methods?

1/1

Relying strictly on hypothesis testing and p-values

Emphasizing graphical techniques to maximize insights, uncover structure, and detect outliers

Eliminating the need for data cleaning and pre-processing

Replacing all numerical calculations with machine learning models

What type of univariate plot displays ordered data points along a single line using individual box/square representations?

1/1

Scatterplot

Histogram

Strip Chart

Heatmap

The Pearson correlation coefficient (r) measures what type of relationship between two quantitative variables?

1/1

Any non-linear relationship (including quadratic or exponential)

Strictly linear relationship

Categorical dependency

Causal relationship

If two variables have a Pearson correlation coefficient r = 0.73, what does r-squared (approx. 0.53) indicate?

1/1

The variables have a negative correlation 53 percent of the time.

Approximately 53 percent of the variation in one variable is explained by the linear relationship with the other.

The relationship is non-linear with 53 percent certainty.

There is a 73 percent error rate in the measurement.

What happens if you calculate a Pearson correlation coefficient (r) on bivariate data that has a strong curvilinear/quadratic relationship?

1/1

r will be close to +1.0 or -1.0.

r may be near 0, falsely suggesting no relationship because r only measures linear association.

r will automatically transform the data to fit a linear line.

r will become undefined and throw an error.

Which historical insight allowed continuous variables on completely different scales to be directly compared in correlation calculations?

1/1

Converting raw scores into normalized z-scores

Trimming the top and bottom 10 percent of values

Grouping values into categorical bins

Subtracting the mode from each observation

In a scatterplot, an upper-left to lower-right pattern of points indicates:

1/1

Positive correlation

Negative correlation

Zero correlation

Perfect non-linear correlation

In real-world data processing, a record with Salary = -10 is a classic example of:

1/1

Incomplete data

Noisy (incorrect/error) data

Inconsistent data

Schema redundancy

Having a record where Age = 42 but Birthday = 03/07/1997 represents what type of data issue?

1/1

Incomplete data

Noisy data

Inconsistent data

Missing value

What are the two major task categories in Data Preparation as outlined in the course material?

1/1

Data Mining and Data Storage

Data Cleaning and Data Transformation

Data Visualization and Data Scraping

Data Collection and Data Modeling

Which of the following operations is classified under Data Transformation (rather than Data Cleaning)?

1/1

Filling in missing values

Resolving schema inconsistencies

Normalization and Discretization

Removing duplicate records

Which Python pandas snippet is used to remove outliers based on z-scores falling within 3 standard deviations?

1/1

df[df.skew() < 3]

df[(np.abs(stats.zscore(df)) < 3).all(axis=1)]

df.corr().abs() < 3

df.dropna(thresh=3)