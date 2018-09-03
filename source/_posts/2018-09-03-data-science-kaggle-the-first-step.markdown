---
layout: post
title: "Data Science - Kaggle - First step"
date: 2018-09-03 01:07
comments: true
categories: Tec
---

> This is my first blog written by English, you know.....

### A. [Kaggle](https://www.kaggle.com/)

Here is the wiki [link](https://en.wikipedia.org/wiki/Kaggle), talks about what Kaggle is and what can we get.

### B. [Titanic: Machine Learning from Disaster](https://www.kaggle.com/c/titanic)

The **Titanic Disaster** is almost every fresh bird's first lesson to unveil the Kaggle's veil (Yes, "Unveil the veil", I get it from google translation).

Basically,  **Titanic** gives us an Excel called `train.csv` which contains passengers' information such as name, sex, age and so on. The `train.csv` has an important column called **Survived**, which is our task to analysis the relationship between **Survived** and the other columns. And there is another file called `test.csv`, it's columns almost as same as `train.csv` except it does not contain **Survived**. As participants, we should build a model from `train.csv` and use this model to predict each passenger which in the `test.csv`  is alive or not. The more accurate you predict, the better grade you will get.

<!-- more -->

### C. The normal process

- Business Understanding

- Data Understanding

- Data Preparation

- Modeling

- Evaluation

- Deployment

**1. Business Understanding**

Just like Ch B I talk about, every competition has its own targets, we should know exactly what the purpose is.

**2.Data Understanding**

Each `train.csv` contains lots of columns, some of them may very meaningful and the others may be just some smoke bomb (Do they know that's mean?). The first step is find out which attribute actually affect the target such as **Survived**.

Take **Titanic ** as an example, first, we load the data:

`data_train = pd.read_csv(r'D:\house\project\python\test\train.csv')`

![image](/images/tec/kaggle01/form01.png)

As you can see, there are lots of passengers and each passenger has many attributes, some of them's **Survived**=0 means they were dead, on the contrary(I always use this phrase in my English writing),   **Survived**=1 means they made alive.

For intuition, I believe the ticket class which means rich or poor, age and sex will strong affect survived rata. Let me check this.

First, the ticket class:
   
``` 
Survived_0 = data_train.Pclass[data_train.Survived == 0].value_counts()
Survived_1 = data_train.Pclass[data_train.Survived == 1].value_counts()

df = pd.DataFrame({'Survived':Survived_1, 'Dead':Survived_0})
df.plot(kind='bar', stacked=True)
```

   ![image](/images/tec/kaggle01/class.png)

   opps, the higher ticket class, the more chance for people to make alive. Wealth indeed is an important attribute.

   Second, sex:

```
Survived_m = data_train.Survived[data_train.Sex == 'male'].value_counts()
Survived_f = data_train.Survived[data_train.Sex == 'female'].value_counts()
df2 = pd.DataFrame({'male':Survived_m, 'female':Survived_f})
df2.plot(kind='bar', stacked=True)
```

   ![image](/images/tec/kaggle01/sex.png)

   Yes, women have more chance than men to make alive.

   Last, age, let us make some different:

```
plt.subplot2grid((2,3),(1,0), colspan=2)
data_train.Age[data_train.Pclass == 1].plot(kind='kde')
data_train.Age[data_train.Pclass == 2].plot(kind='kde')
data_train.Age[data_train.Pclass == 3].plot(kind='kde')
plt.xlabel('Age')
plt.ylabel('Con')
plt.title('Each class age dis')
plt.legend(('1st','2st','3st'),loc='best')
```

   ![image](/images/tec/kaggle01/age.png)

   Above code shows older people live in higher class cabin which also means the relationship between age and wealth.

   Again, Data Understanding is a very important step, my demo code is too simple to show how to figure out these relations, you can get a clearly instruction from  [An Interactive Data Science Tutorial](https://www.kaggle.com/helgejo/an-interactive-data-science-tutorial).

   **3. Data Preparation**

   Before build prediction model, we should rearrange our data. In fact,  we may face many problems about the `train.csv`. Such as missing or the data type is string which most machine learning models in Python can not handle them very well. Here are some way to solve these problems

   * Drop

     If one attribute lack most of data, like only one of ten has value, you can just drop it away.

```
if X_train[X_train.Age.isnull()].value_counts() > max:
 reduced_X_train = X_train.drop(X_train.Age, axis=1)
```

   * Set Yes or No

     If attribute which you believe is important however it has half and half missing value,  you can set **Yes** or **No** for value or null. Like this:

```
df.loc[(df.Cabin.notnull()), 'Cabin'] = "Yes"
df.loc[(df.Cabin.isnull()), 'Cabin'] = "No"
```

   * Impute

       **KSLearn** provides us an easy way to fill vacancy:

```
##Get Model Score from Imputation
from sklearn.preprocessing.imputation import Imputer
my_imputer = Imputer()
imputed_X_train = my_imputer.fit_transform(X_train)
imputed_X_test = my_imputer.transform(X_test)
```

  * Using Categorical Data with One Hot Encoding

    For example, if you people responded to a survey about which what brand of car they owned, the result would be categorical (because the answers would be things like *Honda*, *Toyota*, *Ford*, *None*, etc.). Responses fall into a fixed set of categories.

    You will get an error if you try to plug these variables into most machine learning models in Python without "encoding" them first. Here we'll show the most popular method for encoding categorical variables.

      ![image](/images/tec/kaggle01/one-hot.png)

   It's most common to one-hot encode these "object" columns, since they can't be plugged directly into most models. Pandas offers a convenient function called **get_dummies** to get one-hot encodings. Call it like this:

```
one_hot_encoded_training_predictors = pd.get_dummies(train_predictors)
```

To see detail about this, can view [Using Categorical Data with One Hot Encoding](https://www.kaggle.com/dansbecker/using-categorical-data-with-one-hot-encoding)

   **4. Modeling**

   Finally, we move here....

   After we have cleaned up data, we can choose one machine learning model to predict. I have just learned **Decision Tree** and **Random Forest**, and have heard the **XGBoost** is the leading model for working with standard tabular data **XGBoost** models dominate many **Kaggle** competitions.

   To use one model, the code like this:

```
model = RandomForestRegressor()
model.fit(X_train, y_train)
preds = model.predict(X_test)
```

   So easy, isn't it? The *preds* is our target which should produce an Excel file to submit to **Kaggle**.

   **5. Evaluation**

   No one can make it perfect at once, we should optimize again and again.  The normal benchmark is called **Mean Absolute Error** (also called **MAE**).

   The prediction error for each passenger is: 

```
error=actual−predicted
```

   It's easy to use like this:

```
from sklearn.metrics import mean_absolute_error
predicted_home_prices = melbourne_model.predict(X)
mean_absolute_error(y, predicted_home_prices)
```

   **6. Deployment**

   Bla Bla Bla, finish!

### D. End

OK, this is the simplest way to go through one competition and the result must be awful, lol. There are lot of detail I have not mentioned or learned, like the powerful model **XSBoost**. 

Anyway, this is my first blog about data science, hope it will be a good beginning. 

















