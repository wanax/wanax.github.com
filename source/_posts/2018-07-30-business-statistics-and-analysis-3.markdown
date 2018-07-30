---
layout: post
title: "Business Statistics and Analysis-3"
date: 2018-07-30 07:55
comments: true
categories: Tec
---

课件来源于Rice University的[Business Statistics and Analysis](https://www.coursera.org/specializations/business-statistics-analysis)

###四，Linear Regression for Business Statistics

####Week 1 - Regression Analysis

#####1. Introducing Linear Regression- Building the Model

**What is Linear Regression?**

*Linear regression attempts to fit a linear relation between a variable of interest and a set of variables that may be related to the variable of interest.*

<!--more-->

```
Example
There is a Sales manager of a toys retail company which sells various
kinds of toys in the local market. This sales manager needs to make
some kind of projections about the number of monthly units that the retail company will be able to sell of this particular toy in the
coming half year. In the past she has been making such projections
based on her gut feeling and now wishes to be a little more
scientific about the whole process.

Based on her experience, the manager figures out that the monthly unit 
sales depend on three important variables, the price at which the toy 
is sold, the monthly amount that the company spends on advertising the 
toy and the monthly amount spent on promotions for the toy.
```

**Overview of Regression:**

**Simple Regression:**

A regression with only one explanatory or X variable

**Multiple Regression:**

A regression with two or more explanatory or X variables

1. Modeling Developing a regression model
2. Estimation Using software to estimate the model
3. Inference Interpreting the estimated regression model
4. Prediction Making predictions about the variable of interest

![image](/images/tec/BA3/lrm.jpg)

![image](/images/tec/BA3/lre.jpg)

![image](/images/tec/BA3/lbi.jpg)

![image](/images/tec/BA3/lbi2.jpg)

![image](/images/tec/BA3/lrp.jpg)

#####2. Errors, Residuals and R-square

![image](/images/tec/BA3/lrerr.jpg)

![image](/images/tec/BA3/lrr.jpg)

**Regression is a process that has errors**

* Residuals and Errors
* R-square: A “goodness of fit” measure.

**Why do we have errors in the regression model ?**

* Omitted variables.
* Functional relationship between the Y and X variables.
* The theory of regression analysis is based on certain assumptions about these errors.

![image](/images/tec/BA3/lrerr2.jpg)

![image](/images/tec/BA3/sir.jpg)

* We have differentiated between the β’s and the b’s.
* In common usage people tend to mix these notations.
* However, as long as you understand the conceptual difference, you should be ok.

####Week 2 - Hypothesis Testing and Goodness of Fit

#####1. Hypothesis Testing in a Linear Regression

![image](/images/tec/BA3/htl.jpg)

![image](/images/tec/BA3/hterror.jpg)

![image](/images/tec/BA3/step1.jpg)

![image](/images/tec/BA3/step2.jpg)

![image](/images/tec/BA3/step3.jpg)

![image](/images/tec/BA3/step4.jpg)

#####2. Lesson Hypothesis Testing in a Linear Regression- 'p-values'

![image](/images/tec/BA3/pv.jpg)

![image](/images/tec/BA3/pv2.jpg)

![image](/images/tec/BA3/pv3.jpg)

1. The t-cutoff approach
2. The p-value approach
3. The confidence interval approach

![image](/images/tec/BA3/ci.jpg)

![image](/images/tec/BA3/pv4.jpg)

#####3. 'Goodness of Fit' measures- R-square and Adjusted R-square

![image](/images/tec/BA3/ir.jpg)

![image](/images/tec/BA3/ir2.jpg)

![image](/images/tec/BA3/ir3.jpg)

![image](/images/tec/BA3/ir4.jpg)

#####4. Categorical Variables in a Regression- Dummy Variables

![image](/images/tec/BA3/dv.jpg)

![image](/images/tec/BA3/dv2.jpg)

![image](/images/tec/BA3/dv3.jpg)

```
Example
A parcel delivery service operates in two different regions, region 
“A” and region “B”. Delivery trucks leave the central warehouse and 
travel to region A and deliver parcels in that region. Similarly 
delivery trucks also leave the central warehouse and travel to region 
B and deliver parcels in that region.
```

![image](/images/tec/BA3/dv4.jpg)

####Week 3 - Dummy Variables, Multicollinearity

#####1. Dummy Variable Regression - Extension to Multiple Categories

![image](/images/tec/BA3/mutidv.jpg)

We need one dummy variable less than the number of categories. Incorrect to have three dummies REGA, REGB and REGC.

![image](/images/tec/BA3/mutidv.jpg)

![image](/images/tec/BA3/mutidv2.jpg)

![image](/images/tec/BA3/mutidv3.jpg)
























