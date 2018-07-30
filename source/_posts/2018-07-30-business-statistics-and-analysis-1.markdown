---
layout: post
title: "Business Statistics and Analysis-1"
date: 2018-07-30 07:31
comments: true
categories: Tec
---

课件来源于Rice University的[Business Statistics and Analysis](https://www.coursera.org/specializations/business-statistics-analysis)

![image](/images/tec/BA1/summary.png)

###一，Introduction to Data Analysis Using Excel

1. txt文本几种导入方式
2. Excel中公式的使用
	* IF
	* VLOOKUP
	* HLOOKUP
3. Filter的使用
4. Pivot Table
5. Charts
	* Line，Bar Graphs
	* Pie Charts
	* Pivot Charts
	* Scatter Plots
	* Histograms

<!--more-->

###二，Basic Data Descriptors, Statistical Distributions, and Application to Business Decisions

####Week 1 - Basic Data Descriptors

#####1. Descriptive Statistics
	
![image](/images/tec/BA1/desstatis.jpg)

* Mean
* Median
* Mode
* IQR
![image](/images/tec/BA1/./IQR.png)
* Box Plot
![image](/images/tec/BA1/boxplot.png)
* Standard Deviation
* Variance
* Rule of Thumb
	
	Approximately 68% of the data lie within one 	standard 	deviation, and approximately 95% lie 	within 2 	standard deviations from the mean
	
* Chebyshev’s Theorem
	
![image](/images/tec/BA1/Chebyshev.jpg)
	
####Week 2 - Descriptive Measures of Association, Probability, and Statistical Distributions
	
#####1. Descriptive Measures of Association

* Covariance
	
![image](/images/tec/BA1/covariance.jpg)

* Correltion

	The covariance measure is susceptible to the unit 	of measurement, we can arbitrarily inflate or 	deflate 	the covariance by choice of units.
	
	![image](/images/tec/BA1/correlation.jpg)
	
	• Range: 1 to 1
	
	• Not affected by the units of measurement.
	
	Loosely speaking, correlations > +0.5 or < -0.5 	are considered indicative of a strong positive 	or 	strong negative relationship between two 	variables.
	
* Causation

#####2. Probability

Probability is a numerical measure of the frequency of occurrence of an event. It is 	measured on a scale from 0 to 1. An event of probability 0 will definitely not occur. An event with probability 1 will occur with certainty
	
![image](/images/tec/BA1/probability.jpg)
	
Viewing business processes as Random Experiment with an associated Random Variable is helpful in 	characterizing them and making predictions about the outcome

#####3. Statistical Distributions
		
![image](/images/tec/BA1/distype.jpg)

It is common in business applications to use a continuous distribution such as the Normal (the 	Bell curve) for discrete data
	
`Normal distribution `
	
`t - distribution`
		
####Week 3 - The Normal Distribution

#####1. Probability Mass Function VS Probability Density Function

It is a rule that assigns probabilities to various possible values that a random variable takes when it is being approximated by a particular statistical distribution.
	
![image](/images/tec/BA1/pmf.jpg)
	
![image](/images/tec/BA1/pdf.jpg)
	
![image](/images/tec/BA1/prodis.jpg)
	
![image](/images/tec/BA1/prodis2.jpg)
	

#####2. The Normal Distribution

Normal Distribution, aka the Bell Curve

![image](/images/tec/BA1/normdis.jpg)

![image](/images/tec/BA1/pdff.jpg)

![image](/images/tec/BA1/normdist.jpg)

![image](/images/tec/BA1/normdist2.jpg)

![image](/images/tec/BA1/norminv.jpg)

####Week 4 - Working with Distributions (Normal, Binomial, Poisson), Population and Sample Data

#####1. Applications of the Normal Distribution


A fast-food restaurant sells ‘falafel’ sandwiches. On a typical weekday, the demand for these sandwiches can be approximated by a normal distribution with mean 313 sandwiches and standard deviation of 57 sandwiches

*Ques: What is the probability that on a particular day the demand for falafel sandwiches is less than 300 at the restaurant?*

`Demand ~ Normal(313 sandwiches, 57 sandwiches)`

John can take either of two roads to the airport from his home (Road A or Road B). Owing to varying traffic conditions the travel times on the two roads are not fixed, rather on a Friday around midday the travel times across these roads can be well approximated per normal distributions as follows,

Road A: mean =54 minutes, std = 3 minutes 

Road B: mean =60 minutes, std = 10 minutes

*Ques: Which road should he choose if on midday Friday he must be at the airport within 50 minutes to pick up his spouse?*

```
Road A
Prob(Time < 50) = NORM.DIST(50,54,3,TRUE)
= 0.0912

Road B
Prob(Time < 50) = NORM.DIST(50,60,10,TRUE)
= 0.1586

B is better
```

![image](/images/tec/BA1/stdnorm.jpg)

![image](/images/tec/BA1/stdnorm2.jpg)

#####2. Population and a Sample

**Sample**

It is a subset of the relevant population and is used to make inferences about the population.

![image](/images/tec/BA1/sample.jpg)

![image](/images/tec/BA1/population.jpg)

#####3. The Central Limit Theorem

![image](/images/tec/BA1/clt.jpg)

![image](/images/tec/BA1/clt2.jpg)

![image](/images/tec/BA1/clt3.jpg)

![image](/images/tec/BA1/clt4.jpg)

In Plain Language,

Sample averages are normally distributed irrespective of where the sample came from. Not only are they normally distributed but more importantly they are normally distributed with mean equal to the population mean.

#####4. The Binomial Distribution

Two Popular Discrete Distributions

* The Binomial

	![image](/images/tec/BA1/bidis.jpg)

	Consider a situation where there are n independent trials, where the 	probability of success on each trial is p and the probability of 	failure is 1-p.
	Define random variable X to denote number of successes in n trials. 	Then this random variable is said to have a Binomial distribution.
	
	![image](/images/tec/BA1/bidis2.jpg)
	
	![image](/images/tec/BA1/bidis3.jpg)
	
	![image](/images/tec/BA1/bidis4.jpg)
	
	![image](/images/tec/BA1/bidis5.jpg)
	
	![image](/images/tec/BA1/bidis6.jpg)

* The Poisson

	![image](/images/tec/BA1/podis.jpg)
	
	![image](/images/tec/BA1/podis2.jpg)
	
	![image](/images/tec/BA1/podis3.jpg)




























