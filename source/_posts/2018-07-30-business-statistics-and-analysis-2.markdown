---
layout: post
title: "Business Statistics and Analysis-2"
date: 2018-07-30 07:54
comments: true
categories: Tec
---

课件来源于Rice University的[Business Statistics and Analysis](https://www.coursera.org/specializations/business-statistics-analysis)

###三, Business Applications of Hypothesis Testing and Confidence Interval Estimation

####Week 1 - Confidence Interval - Introduction

#####1. Introducing the t-distribution, the T.DIST Function

![image](/images/tec/BA2/tdis.jpg)

<!--more-->

* Does not have any standalone business application
* Used as an interim tool to calculate confidence intervals and hypothesis testing
* Has an associated probability density function
* Associated Excel functions are the =T.DIST and =T.INV

![image](/images/tec/BA2/df.jpg)

![image](/images/tec/BA2/df2.jpg)

#####2. Introducing Confidence Interval

* It is an ‘interval’ with some ‘confidence’ or probability attached to it
* An interval for some unknown characteristic of the population data

```
Example: US Presidential Election
• Only two candidates, A and B
• Wish to predict vote percentage for candidate A
• A random selection of 500 potential voters surveyed 
• 300 voters will votefor A and 200 for B
• That is, 60% of surveyed voters will vote for A

Will candidate A get 60% of all votes in the actual election?
```

A 95% confidence interval for the vote share of candidate A,

**[55.7%, 64.3%]**

* 95% of similarly constructed confidence intervals likely to have the actual vote share for A

* there is a 0.95 probability that the actual vote share for A will be between 55.7% and 64.3%

![image](/images/tec/BA2/ci.jpg)

![image](/images/tec/BA2/ci2.jpg)

We wish to find out u

![image](/images/tec/BA2/ci3.jpg)

Example:

US Presidential Election, predicting the proportion of votes for a particular candidate

*confidence interval for the ‘population proportion’*

Example:

Average starting salary of all business students who graduated last year in New York city

*confidence interval for the ‘population mean’*

#####3. The z-statistic and the t-statistic

![image](/images/tec/BA2/clt.jpg)

![image](/images/tec/BA2/zsta.jpg)

![image](/images/tec/BA2/tsta.jpg)

![image](/images/tec/BA2/tsta2.jpg)

#####4. Using z- and t- statistics to Construct Confidence Interval

![image](/images/tec/BA2/exp.jpg)

![image](/images/tec/BA2/sol.jpg)

![image](/images/tec/BA2/sol2.jpg)

![image](/images/tec/BA2/sol3.jpg)

![image](/images/tec/BA2/sol4.jpg)

####Week 2 - Confidence Interval - Applications

#####1. Application of Confidence Interval

* Conceptual understanding 
* Examples 
* Stylized problem

![image](/images/tec/BA2/ci4.jpg)

![image](/images/tec/BA2/ci5.jpg)

When the population standard deviation (σ) is not known,

* we replace it by the sample standard deviation (s)
* The z-statistic gets replaced by the t-statistic 

![image](/images/tec/BA2/cit.jpg)

```
Example
A person wishes to explore the size of single family houses that are
 typically available for purchase in a particular neighborhood of 
 Houston in Texas. She manages to get hold of a list containing house 
 sizes of a sample of 100 houses that were sold in the past two years.

The data is provided in the excel data file Home_Sizes.xlsx, and given
 this data she wishes to assess the average size of houses typically 
 available for purchase in this neighborhood.

1.Population standard deviation not known
2.Need to use calculation for confidence interval based on the 
t-statistic

```

**A 95% confidence interval for the average (population mean) house size:
[3048.2, 3428.3] square feet**

```
Example
A Consultancy firm surveyed a randomly selected set of 210 CEOs of 
‘fast growing small companies’ in the US and Europe. Only 51% of these
 executives had a management succession plan in place, the remaining
  did not have one

Use this information to compute a 90% confidence interval to estimate
 the proportion of all ‘fast growing small companies’ that have a
  management succession plan.
```

![image](/images/tec/BA2/cip.jpg)

**Answer:
The 90% confidence interval is:
[0.453, 0.567] or [45.3%, 56.7%]**

*Summary:*

![image](/images/tec/BA2/cis.jpg)

#####2. Sample Size Calculation

* A pollster wanting to make a prediction about aparticular candidate’s vote share in the US presidential election.

* A quality control manager at a battery manufacturer wanting to  estimate the average number of defective batteries contained in a box shipped by the company.

*Different industries may have different rule of thumb strategies for sample size selection.*

* The pollster may want to have a margin of error +/- 3% with a confidence level of 95%

	*simply make some small survey get std = 0.9*

	![image](/images/tec/BA2/number.jpg)

* The quality control manager may want to assess the average number of defectives in a box with a margin of error of plus minus 0.3 batteries and a confidence level of 95%

	![image](/images/tec/BA2/numberp.jpg)
	
	![image](/images/tec/BA2/numberp2.jpg)
	
* Many industries use rule-of-thumb strategies/heuristics

* We provided here some basis for choosing an appropriate sample size

####Week 3 - Hypothesis Testing

Hypothesis tests are an important tool to analyze data and make some inferences from it.

*All Hypothesis tests follow a basic logic...*

1. An assumption or a claim is made.
2. If your data contradicts this assumption or claim then you conclude that the claim or assumption made must be wrong.

```
Example

As I drove to work one day recently, I assumed that the road that I
normally take to school would, as usual, take me to school.
 
There is the assumption, part 1 of the hypothesis.

But I reached a construction barricade, the road was closed.

There is the data.
It contradicts my assumption. So my original assumption or hypothesis 
was wrong.
```

```
Another Example

You are the production manager at a beverage manufacturer and you
receive a bottling unit that has been recently re-adjusted so that it
puts 200 milliliter of beverage in disposable plastic bottles.
  
There is the assumption, You assume that the unit actually puts 200 milliliter of beverage in the bottles.

Next, rather than accepting this assumption, you decide to test it
using data. You fill out 10 bottles using the unit at different times
so as to obtain a random sample and very carefully measure the amount
of beverage inside each bottle.
   
This is your data. A random sample of 10 observations on the amount of
beverage in the bottles.
```

**if the average amount of beverage per bottle across these 10 bottles is 170 milliliter?**

*easy to conclude that the bottling unit is not properly adjusted.*

**if the average amount of beverage per bottle across these 10 bottles is 200 milliliter?**

*again, the conclusion seems easy given this evidence.*

**if the average amount of beverage per bottle across these 10 bottles is 199.9 milliliter or 200.1 milliliter?**

*perhaps, giving benefit of doubt you would conclude that the unit is properly adjusted.*

**if the average amount per bottle in the sample turns out to be 199.1 ml? 198ml? 202 ml? ...**

![image](/images/tec/BA2/hte.jpg)

```
Is a widely used procedure to test a variety of claims...

testing the fuel efficiency claim of a car manufacturer. testing the
claims of efficacy made by a new drug.
 
testing the claim that the defective rate in your production process is 
greater than the acceptable limit.

testing the claim by an educational website that enrolling in its
courses leads to higher school scores.
 
... ...

```

#####1. The Logic of Hypothesis Testing

![image](/images/tec/BA2/ht.jpg)

![image](/images/tec/BA2/htclt.jpg)

![image](/images/tec/BA2/normdis.jpg)

![image](/images/tec/BA2/normdis2.jpg)

![image](/images/tec/BA2/htzsta.jpg)

![image](/images/tec/BA2/httsta.jpg)

#####2. Conducting a Hypothesis Test, the Four Steps

![image](/images/tec/BA2/hts.jpg)

![image](/images/tec/BA2/hts3.jpg)

![image](/images/tec/BA2/hts4.jpg)

#####3. Single Tail and Two Tail Hypothesis Tests

1. Formulate Hypothesis
2. Calculate the t-statistic
3. Cutoff values for the t-statistic
4. Check whether t-statistic falls in the rejection region

*Interpret results of hypothesis test as applied to the particular business application*

`1. Two tailed test`

![image](/images/tec/BA2/httt.jpg)

`2. Single tailed test`

![image](/images/tec/BA2/stht.jpg)

```
Example

A fuel additive manufacturer claims that through the use of its’ fuel
additive, automobiles in the small car category should achieve on 
average an increase of 3 miles or more per gallon of fuel.
```

**Claim made: increase in fuel efficiency is 3 miles per gallon or more**

To test the claim :

1. Random selection of 150 small cars.
2. Their fuel efficiency measured before and after the use of fuel
	 additive. 
3. 150 measurements obtained for the increase in miles per gallon
	 achieved.

![image](/images/tec/BA2/sthte.jpg)

![image](/images/tec/BA2/stht2.jpg)

![image](/images/tec/BA2/sthte2.jpg)

![image](/images/tec/BA2/sthte3.jpg)

#####4. Guidelines, Formulas and an Application of Hypothesis Test

![image](/images/tec/BA2/nhg.jpg)

![image](/images/tec/BA2/htg.jpg)

```
Example
(when the Null hypothesis naturally turns out to have a strict inequality)
We wish to test a claim that the average age of Men MBA students across
various MBA programs in the US is greater than 28 years. For this we
collect data on average ages of men MBA students across a sample of
40 MBA programs in the US.
```

![image](/images/tec/BA2/nht.jpg)

![image](/images/tec/BA2/nht2.jpg)

#####5. Hypothesis Test for a Population Proportion

```
Example
A medium sized university in the US introduces a new lunch facility on 
campus on a trial basis. The university operates the Lunch facility for
a few months and then decides to survey the student body. Based on the
survey, university would make this facility a permanent fixture or do
away with it. Specifically, if more than or equal to 70% of the
student body approves of it then the facility would be made 
permanent else it would shut down.

The university conducts a survey with 750 randomly selected students on
campus and finds that 510 of these students (or 68% of the sampled
students) approve of the new facility and the remaining 240 students
or 32% students do not approve of it.
   
Based on the criteria set by University should the facility be made
permanent?
```

![image](/images/tec/BA2/htp1.jpg)

![image](/images/tec/BA2/htp2.jpg)

![image](/images/tec/BA2/htp3.jpg)

#####6. Type I and Type II Errors in a Hypothesis Test

```
Example
Your friend Sam claims that he can shoot 40 or more baskets in an hour
from the 3-point line in a Basketball court. So, Sam is making a claim 
about a population parameter, in this case it is his true shooting 
ability from the 3-point line in a Basketball court. This can be 
likened to the population mean mu. Thus Sam is claiming that the
population mean mu of his shooting ability is greater than or equal 
to 40 baskets in an hour from the 3-point line in a Basketball court.

Next, you decide to test this claim. For that, you take Sam to the 
Basketball court everyday for 10 days and make him shoot baskets from
the 3 point line for an hour every day. You end up with 10 data points 
which are the number of baskets Sam shot in those 10 days. You can 
calculate the sample mean and the sample standard deviation from these
ten observations.
```

![image](/images/tec/BA2/hterror.jpg)

Suppose that Sam’s true ability is indeed ≥ 40.

However the 10 days were not good for Sam.

He gave a low sample average.

You reject the Null hypothesis.

**Type I error: Rejecting the Null Hypothesis when it is true.**

**Type II error: Not rejecting the Null Hypothesis when it is false**

Sam’s true ability is NOT ≥ 40.

However the 10 days were lucky for Sam. He gave a high sample average.

You DID NOT reject the Null hypothesis.

*Reducing the probability of Type I and Type II errors*

* Probability of Type I error is set by our choice of α
* Typically α = 0.05 or 0.01

* Probability of Type II error can be reduced by taking a larger sample size.

####Week 4 - Hypothesis Test - Differences in Mean

#####1. Introducing the Difference-In-Means Hypothesis Test

```
An empirical study using data on heights of people claimed that the 
average height of men aged 18 years to 45 years across the world was
173 cm.
 
This study included men not only from the sports fraternity but across 
a wide spectrum of professions and walks of life.
```

![image](/images/tec/BA2/dfht.jpg)

![image](/images/tec/BA2/dfht2.jpg)

![image](/images/tec/BA2/dfht3.jpg)

* Equal variance t-test for differences in means.
* Unequal variance t-test for differences in means.
* Paired t-test for differences in means.

#####2. The Paired t-Test for Means 

* Used to test claims around difference in two population means.
* Requires a sense of ‘pairing’ across individual observations in the two samples.

```
Paired t-test (Example)

You are the training manager for a large organization. It is your jo to
ensure that training needs of employees are met. One such training need 
is that of computer skills. You regularly send out batches of employees
to such computer skills training organized by a third party vendor. 
Since the training costs money you wish to evaluate whether the 
training is indeed effective in improving the computer skills of 
employees. You decide to give a standardized computer skills test to 
employees before they go for such training and you again administer
similar test when they come back from training. You also set a bench 
mark that the training would be considered effective if on average 
the ‘after scores’ are at least 10 points greater than the ‘before 
scores’.
```
![image](/images/tec/BA2/pht.jpg)

**Paired t-test, some important considerations**

1. Sense of pairing across individual observations of the two samples.
2. The two samples should have equal number of observations.
3. You may have your conclusion reversed in using a paired t-test versus either the equal or unequal variances t-test.

`Three kinds:`

1. Paired t-test for differences in means.
	* Used when there is a sense of ‘pairing’ in the data
2. t-test for differences in means ‘assuming equal variance’.
3. t-test for differences in means ‘assuming unequal variance’.

```
Example
The data can be thought of as a sample from a larger population.

We wish to test whether the monthly dollar sales across the provinces
of Alberta and British Columbia are the same.
 
In other words we wish to check if there is any difference in monthly
dollar sales across these two provinces.
 
We will be making this inference using the 48 months of sample sales 
data we have.
```

**Difference in means test**
















