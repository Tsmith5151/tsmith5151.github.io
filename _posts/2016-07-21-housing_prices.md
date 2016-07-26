---
layout: post
section-type: post
title: Linear Regression and Regression Trees to Predict Housing Prices
category: technology
tags: [ 'Machine Learning', 'Regression Trees', 'Scikit-Learn','Linear Regression' ]
author: Trace Smith
---

The Boston housing market is highly competitive, and the goal is to be one of the best real estate agents in the area. To compete in the real estate market, a few basic machine learning concepts are applied in order to assist a client with finding the best selling price for their home. The Boston Housing dataset which contains aggregated data on various features for houses in Greater Boston communities, including the median value of homes for each of those areas. The objective here is to build an optimal model based on a statistical analysis with the tools available and then utilize this model to estimate the best selling price for the client's home. Additional information on the Boston Housing dataset can be found here. The source code developed for this project can be found in the ipython notebook boston_housing.ipynb and the complete repo can be found [`here`](https://github.com/Tsmith5151/Boston-Housing-Prices). The source code is discussed throughout this Blog, but keep in mind that not every output generated by the code is shown below; please refer to the repo link. Additional information on the Boston Housing dataset can be found [`here`](https://archive.ics.uci.edu/ml/datasets/Housing). 

Lets begin. First, a list of the python packages that was utilized in the work is shown below. Note, I will be using Scikit-Learn for the Machine Learning algorithms and tools for data mining and data analysis. 

```python
%matplotlib inline
import warnings
warnings.filterwarnings('ignore')
```

```python
import os
import numpy as np
import pylab as pl
import matplotlib.pyplot as pl
import seaborn as sns
import pandas as pd

#Libraries from Sci-kit Learn
from sklearn import datasets
from sklearn.tree import DecisionTreeRegressor
from sklearn.metrics import mean_squared_error, mean_absolute_error, make_scorer
from sklearn.cross_validation import train_test_split
from sklearn.grid_search import GridSearchCV
#Linear Regression
from sklearn.linear_model import LinearRegression
from sklearn import linear_model
#Polynomial Regression:
from sklearn.linear_model import Ridge
from sklearn.preprocessing import PolynomialFeatures
from sklearn.pipeline import make_pipeline
```


### Explore  The Data

First, the Boston housing dataset from Scikit Learn is uploaded and stored in a Pandas DataFrame. Pandas is a powerful Python package commonly utilized in data analysis which provides fast, flexible, and expressive data structures designed to make working with “relational” or “labeled” data both easy and intuitive. Very simular to an SQL table or Excel spreadsheet, a DataFrame can be thought of as a dict-like container for Series objects with heterogeneously-typed columns. 


```python
#Load boston dataset
boston = datasets.load_boston()

#Values
housing_prices = boston.target #target values
housing_features = boston.data #attributes values

#Store in DataFrame
attributes = boston.feature_names #feature names
df_data = pd.DataFrame(housing_features, columns = attributes)
df_target = pd.DataFrame(housing_prices, columns =['MEDV'])
df_boston = pd.concat([df_data, df_target,], axis = 1) #concat data/target

feats = df_boston.shape[1]
obs = df_boston.shape[0]
print "Number of Housing Features: ", feats
print "Number of Houses: ", obs
```

***Number of Housing Features:  14***
***Number of Houses:  506***

```python
df_boston.describe()
```

<img src = "https://tsmith5151.github.io/Blog/img/Boston/data.png">

### Data Exploration

**Which are significant and what do they measure of the available features for a given home?**

To get a better idea of the data, a histogram is generated to show the distribution of median housing prices in the greater Boston area. Nearly 80 homes are valued approximately at $21,000. Next, we can examine any strong correlations between the attributes by observing the correlation heatmap and by creating scatter plots of the median value of homes (`MEDV`) vs several different housing features. Based on the results, `CRIM`, `RM`, and `LSAT` show a strong linear correlation for predicting the `MEDV`. For instance, as the number of rooms per house increases, so does the housing price. Several other plots are generated to explore relationships among the input features. 

```python
def histogram():
    X,y = housing_features, housing_prices
    pl.figure(figsize=(8,6))
    pl.hist(y, bins =20, color = 'blue',alpha=0.5)
    pl.suptitle('Boston Housing Prices', fontsize = 18)
    pl.xlabel('Housing Prices [$10k]', fontsize = 16)
    pl.ylabel('Frequency', fontsize = 16)
    pl.show()
```

```python
def scatter_plots():
    pl.figure()
    fig,axes = pl.subplots(4, 4, figsize=(14,18))
    fig.subplots_adjust(wspace=.4, hspace=.4)
    img_index = 0
    for i in range(boston.feature_names.size):
        row, col = i // 4, i % 4
        axes[row][col].scatter(boston.data[:,i],boston.target)
        axes[row][col].set_title(boston.feature_names[i] + ' and MEDV')
        axes[row][col].set_xlabel(boston.feature_names[i])
        axes[row][col].set_ylabel('MEDV')
    pl.show()
```

```python
def corrplot():
    fig, ax = pl.subplots(figsize=(10,10))
    pl.title("Correlation Plot",fontsize=16)
    sns.corrplot(df_boston)
```


```python
## Calculate some Boston housing statistics:
def explore_city_data():
    number_houses = housing_features.shape[0] # size of data
    number_features = housing_features.shape[1] # number of features
    min_price = np.min(housing_prices) # minimum price
    max_price = np.max(housing_prices) # maximum price
    mean_price = np.mean(housing_prices) # mean price
    median_price = np.median(housing_prices)# median price
    std_price = np.std(housing_prices)# standard deviation
    
    print "Number of houses:", number_houses
    print "Number of features:", number_features
    print "Minimum Housing Price: ${:,.2f}".format(min_price)
    print "Maximum Housing Price: ${:,.2f}".format(max_price)
    print "Mean Housing Price: ${:,.2f}".format(mean_price)
    print "Median Housing Price: ${:,.2f}".format(median_price)
    print "Standard Deviation: ${:,.2f}".format(std_price)
```

```python
def plot1():
    x = 'CRIM'
    y = 'LSTAT'
    x_label, y_label = x,y
    title = x + ' and '+ y
    create_plot(x,y,x_label,y_label,title)
    
def plot2():
    x = 'AGE'
    y = 'RM'
    x_label, y_label = x,y
    title = x + ' and '+ y
    create_plot(x,y,x_label,y_label,title)

def plot3():
    x = 'PTRATIO'
    y = 'TAX'
    x_label, y_label = x,y
    title = x + ' and '+ y
    create_plot(x,y,x_label,y_label,title)

def create_plot(x,y,x_label,y_label,title):
    hue = df_target
    df = df_data
    markerSize = df_target['MEDV']*6
    g = sns.lmplot(x=x,y=y,data=df,
               scatter_kws={'s':markerSize,'alpha':0.5,'linewidths':1,'edgecolor':'w','color':'g'},
                   size=4.5, aspect=1.4)
    g.set_xlabels(x_label, size = 12)
    g.set_ylabels(y_label, size = 12)
    axes = g.axes
    sns.plt.title(title,fontsize=16)
```

```python
if __name__ == "__main__":   
    print "***Statistics***\n"
    explore_city_data()
    histogram()
    corrplot()
    scatter_plots()
    print "\n***Scatter Plots (Features)***"
    #plot1()
    #plot2()
    plot3()
```


<img src = "https://tsmith5151.github.io/Blog/img/Boston/histogram1.png">


<img src = "https://tsmith5151.github.io/Blog/img/Boston/correlation.png">


<img src = "https://tsmith5151.github.io/Blog/img/Boston/scatterplots.png">


<img src = "https://tsmith5151.github.io/Blog/img/Boston/ptratio_tax.png">


### Training & Testing Data

**Why split the dataset into training and testing subsets?**

Splitting the data into training and testing provides a measurement on how well the model will perform on out-of-sample data. Ideally, you want to perform a test on the dataset in which the algorithm has not seen in order to exclude any memorization. From there, you can then determine whether the algorithm works well with the out-of-sample data. A method of splitting the data using [sklearn.cross_validation.train_test_split()](http://scikit-learn.org/stable/mod ules/generated/sklearn.cross_validation.train_test_split.html) function allows the sample set to be randomly shuffled and then assigned `70%` as training and the remaining `30%` as the testing set. For this project, out of the `506` sampled houses, `354` are randomly selected and classified as the training set and the model is tested on `152` of the houses.

```python
def split_data():
    # Get the features and labels from the Boston housing data
    X, y = boston.data, boston.target
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30,random_state=None)
    #print "X_training:", X_train.shape
    #print "X_test:", X_test.shape 
    return X_train, y_train, X_test, y_test

X_train, y_train, X_test, y_test = split_data()
```

### Performance Metric:

**Which performance metric below is most appropriate for predicting housing prices and analyzing error?**


Accuracy, Precision, Recall, F1 score, Mean Squared Error (MSE), and Mean Absolute Error (MAE) are several performance metrics. When the target variable that is trying to be predicted is continuous, such as this case of the housing data, is termed as a regression problem. On the other hand, when 'Y' can take on only a small number of discrete values (i.e. given the size of the living room and we want to predict whether this is a house or an apartment) this is referred to as a classification problem. So being that this project is a regression problem and not classification, the metric used to evaluate the performance of the predicting variable (housing price) is the [mean_squared_error](http://scikit- earn.org/stable/modules/generated sklearn.metrics.mean_squared_error.html). The MSE takes all of the errors, the vertical distance from the observation to the regession line, squares them, then finds the average. The MSE will heavily penalize outliers as oppose to the [mean absolute error](http://scikit-learn.org /stable/modules/generated sklearn.metrics.mean_absolute_error.html), which takes all the errors, get the absolute values of them, and find the average. When selecting betweeen the metrics, the objective is the minimize the error, which is why it is better to consider the MSE in the model. The two figures below dipicts the difference between the MSE (left) and the MAE (right) at a given depth `5`; it can be observed the error is minimized more for the MSE as opposed to the MAE. Furthermore, the machine learning algorithm for this project is [sklearn.tree.DecisionTreeRegressor](http://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeRegressor.html), therefore MSE is the valid peformance metric input for determining a split when constructing this model.

***Mean Squarred Error***             

<img src = "https://tsmith5151.github.io/Blog/img/Boston/learning_curve_5.png">

***Mean Absolute Error***

<img src = "https://tsmith5151.github.io/Blog/img/Boston/learning_curve_mae_5.png">

```python
def performance_metric(label, prediction):
    """Calculates and returns the appropriate error performance metric."""
    #mae = mean_absolute_error(label, prediction)
    mse = mean_squared_error(label, prediction)
    return mse
```

### Simple Linear Regression

For the initial case, let's consider a one-dimensional vector (X) where X1 is the percentage of crime and we are trying to predict the medium housing price (Y). We can approximate 'Y' as a linear function of X by the following expression:

Regression equation is defined by: ***Y = w0 + w1*X1 + E***

Where w's are the parameters (also called the weights) parameterizing the space of linear functions mapping X to Y. 

***Cost Function***

So when estimating the function f(x) given the data to model, search all possible values of w0 and w1 such that the results is one that minimizes the squared error. The squared error is the summation of taking the squared difference between the actual and predicted data point. 


*Source: CS2299 Lecture Notes on Linear Regresstion*

We can implement Linear Regression to the Boston Housing dataset by fitting the best relationship between X & Y. The relationship between Crime (X) and the MEDV (Y) is explored below:

```python
def linear_reg(x_train,x_test,y_train,y_test,feat_val):
    # Create Linear Regression Object
    reg = linear_model.LinearRegression()

    # Train the model using the training sets 
    x_train = np.array([x_train]).T 
    y_train = np.array([y_train]).T
    
    x_test = np.array([x_test]).T 
    y_test = np.array([y_test]).T
    
    fit = reg.fit(x_train,y_train)
    pred = reg.predict(x_test)

    print "Equation of Best Fit Line:"
    print "Y = %sx + %s" % (reg.coef_,reg.intercept_)
    
    print "\nMean Square Error: %s" % (mean_square_error(pred,y_test))
    print "\nVariance: %s" % (variance(reg,x_test,y_test))
   
    print "\nPredicted Housing Price (1000$):"
    print np.round(reg.coef_*feat_val + reg.intercept_,2)
    
    plot_linreg(x_train,x_test,y_train,pred)
    
def mean_square_error(pred,y_test):
    mse = round(np.mean(pred-y_test)**2,4)
    return mse

def variance(reg,x_test,y_test):
    # Explained variance score: 1 is perfect prediction
    var = round(reg.score(x_test,y_test),4)
    return var

def plot_linreg(x_train,x_test,y_train,pred):
    fig = pl.figure(figsize=(8,6))
    pl.scatter(x_train, y_train, color="red", marker="o")
    pl.plot(x_test,pred,color='blue')
    ax = fig.add_subplot(1,1,1)
    ax.scatter(x_train,y_train, color="red", marker="o")
    ax.set_ylabel('MEDV (1000$)', fontsize=14)
    ax.set_xlabel('Rooms',fontsize=14)
    ax.set_title('Rooms vs MEDV', fontsize=16)
    ax.set_xlim(2,10)
    ax.set_ylim(0,60)
    pl.show()
    
def main():
    #x_train and x_test for feature of interest
    x_train = []
    x_test = []  
    for item in X_train:
        x_train.append(item[5]) #crime (Note: Enter any feature index here)
    for item in X_test:
        x_test.append(item[5]) #crime (Note: Enter any feature index here)
    
    '''Enter Feature Value'''
    feat_val = 4
    linear_reg(x_train,x_test,y_train,y_test,feat_val)


if __name__ == "__main__":
    main()
```

<img src = "https://tsmith5151.github.io/Blog/img/Boston/linreg.png">


### Multiple Linear Regression

```python
def multiple_reg(X_train,X_test,y_train,y_test):
    reg = LinearRegression()
    reg.fit(X_train,y_train)
    df = pd.DataFrame(zip(attributes,reg.coef_),columns=['Features','Coeff'])
    print df
    print '\nR-Squared: %s\n' % (round(reg.score(X_train,y_train),3))
    
    '''Predict Housing Price:'''
    pred = reg.predict(X_test)
    df_pred = pd.DataFrame({'Observed':y_test,'Predicted':pred})
    print df_pred.head()
    mse = performance_metric(y_test, pred)
    print "\nMean Squared Error: %s" % (round(mse,3))

def main():
    multiple_reg(X_train,X_test,y_train,y_test)

if __name__ == "__main__":
    main()
```

<img src = "https://tsmith5151.github.io/Blog/img/Boston/mreg.png">

### Regression Tree

Decision Trees are a non-parametric supervised learning method and can be used for regression. The goal here is to create a model that predicts the value of the Boston housing prices by learning simple decision rules inferred from the data features. The following blocks consists of code utilized to construct the learning curves, model complexity, and fitting the model.

### Learning Curve

```python
depth = 5
def learning_curve(depth, X_train, y_train, X_test, y_test):
    """Calculate the performance of the model after a set of training data."""

    # We will vary the training set size so that we have 50 different sizes
    sizes = np.round(np.linspace(1, len(X_train), 50))
    train_err = np.zeros(len(sizes))
    test_err = np.zeros(len(sizes))

    print "Decision Tree with Max Depth: "
    print (depth)

    for i, s in enumerate(sizes):

        # Create and fit the decision tree regressor model
        regressor = DecisionTreeRegressor(max_depth=depth)
        regressor.fit(X_train[:s], y_train[:s])

        # Find the performance on the training and testing set
        train_err[i] = performance_metric(y_train[:s], regressor.predict(X_train[:s]))
        test_err[i] = performance_metric(y_test, regressor.predict(X_test))


    # Plot learning curve graph
    learning_curve_graph(sizes, train_err, test_err)


def learning_curve_graph(sizes, train_err, test_err):
    """Plot training and test error as a function of the training size."""
    pl.figure()
    pl.figure(figsize=(8,6))
    pl.title('Decision Trees: Performance vs Training Size', fontsize = 20)
    pl.plot(sizes, test_err, lw=2, label = 'test error')
    pl.plot(sizes, train_err, lw=2, label = 'training error')
    pl.legend()
    pl.xlabel('Training Size', fontsize = 14)
    pl.ylabel('Error', fontsize =14)
    pl.show()
    
X_train, y_train, X_test, y_test = split_data()
learning_curve(depth, X_train, y_train, X_test, y_test)
```

**When the model is using the full training set, does it suffer from high bias or high variance when the max depth is 1? What about when the max depth is 10?**

In the case of `max_depth` = `1`, the training error begins to significanlty
increase as the training size increases and likewise, the error for the test
error remains relatively high. Thus this situation where the `max_depth` of the
decision tree is `1`, the model would be underfitting as the learning value is
restricted to one level of the decision tree and does not allow the training set
to learn data adequately. A low complexity decision tree results in high bias.
For a `max_depth` = `10`, this would clearly illustrate a high variance and
overfitting the data as shown below (right). The model has virtually memorized
the training data but will not be expected to perform well with out-of-sample
data.

***Depth = 1***               

<img src = "https://tsmith5151.github.io/Blog/img/Boston/learning_curve_1.png">

***Depth = 10***

<img src = "https://tsmith5151.github.io/Blog/img/Boston/learning_curve_10.png">


**What is the max depth for the model? As the size of the training set increases, what happens to the training error? Describe what happens to the testing error.**

The learning curves for `max_depth` from 3-5 are shown below. As observed form
the figures, the training error is virtually zero as the model has basically
"memorized" the small training set. However, as more data is added to the model,
the training error begins to increase. When the training size increases, error
begins to increase and the high error translates to underfitting the data;
likewise, the low training error would indicate overfitting the data. The
testing error is relatively high initially because it has not seen enough
examples, but begins to decrease as more training examples are given. Based on
the learning curve plots, it is concluded that the maximum depth for this model
is `4`.

***Max Depth = 3*** 

<img src = "https://tsmith5151.github.io/Blog/img/Boston/learning_curve_3.png">

***Max Depth = 4***

<img src = "https://tsmith5151.github.io/Blog/img/Boston/learning_curve_4.png">

***Max Depth = 5***

<img src = "https://tsmith5151.github.io/Blog/img/Boston/learning_curve_5.png">

### Model Complexity

```python
def model_complexity(X_train, y_train, X_test, y_test):
    """Calculate the performance of the model as model complexity increases."""

    print "Model Complexity: "

    # We will vary the depth of decision trees from 2 to 25
    max_depth = np.arange(1, 25)
    train_err = np.zeros(len(max_depth))
    test_err = np.zeros(len(max_depth))

    for i, d in enumerate(max_depth):
        # Setup a Decision Tree Regressor so that it learns a tree with depth d
        regressor = DecisionTreeRegressor(max_depth=d)

        # Fit the learner to the training data
        regressor.fit(X_train, y_train)

        # Find the performance on the training set
        train_err[i] = performance_metric(y_train, regressor.predict(X_train))

        # Find the performance on the testing set
        test_err[i] = performance_metric(y_test, regressor.predict(X_test))

    # Plot the model complexity graph
    model_complexity_graph(max_depth, train_err, test_err)


def model_complexity_graph(max_depth, train_err, test_err):
    """Plot training and test error as a function of the depth of the decision tree learn."""
    pl.figure()
    pl.title('Decision Trees: Performance vs Max Depth', fontsize = 20)
    pl.plot(max_depth, test_err, lw=2, label = 'test error')
    pl.plot(max_depth, train_err, lw=2, label = 'training error')
    pl.legend()
    pl.xlabel('Max Depth',fontsize =14)
    pl.ylabel('Error', fontsize =14)
    pl.show()
```

### Fit Model

**What is the grid search algorithm and when is it applicable?**

A machine learning model can be fine-tuned by using the grid search algorithm,
a hpyerparameter optimization technique. The performance of the model depends on
the hyperparameters provided to the algorithm when training the model. It is
desirable to find the right combination of the hyperparameters during training.
[sklearn.grid_search.GridSearchCV](http://scikit-learn.org/stable/modules/genera
ted/sklearn.grid_search.GridSearchCV.html#sklearn.grid_search.GridSearchCV) is
an exhaustive search that trains and evaluates a model for all possible
combination of hyperparameters that produce the best model. The `1-10`), and a
scoring function so it is able to evaluate the parameter that performed the best
(performance `metric mean_squared_error`).

**What is cross-validation and how is it performed on a model? Why would cross-validation be helpful when using grid search?**

A common occurrence from machine learning algorithms is a degree of bias that
can exist when random sampling the data and splitting the data between training
and testing. To avoid sampling issues, which can cause the training set to be
too optimistic, cross-validation is a statistical approach that computes the
average on multiple test sets. One of the most common iterators that performs
the k-fold cross-validation is `KFolds` and the steps are as follows:


-Splits the data into K equal folds
-Uses one of the fold as the testing set and the remaining as the training set
-Trains and records the test set results
-The second and third steps are repeated using a different fold as the testing set each time
-Calculate the average and standard deviation of all the k-folds
-Documentation can be found here: [sklearn.cross_validation.KFolds()](http
://scikit-learn.org/stable/modules/generated/sklearn.cross_validation.KFold.html)


The `train_test_split()` method can be very simple to perform however the
downside is the training data is reduced by 30% when finding the best regression
method. Using the default `GridSearchCV` parameters, the `cross-validation`
generator is set to 3-fold CV. So for instance, the housing dataset consists of
`506` observations per feature and if k=3, `169` observations would be in each
fold. The initial iteration entails the first fold containing the testing data
(`169 observations`) and the remaining folds (2-3) containing `337`
observations, `168` each. In second iteration, the testing set is now fold #2
while fold #1 and fold #3 are the training set. We train K different models,
with each time leaving out a single subset for measuring the cross-validation
error. The final cross-validation error is calculated by taking the mean or
median of the K models. The advantage of cross-validation provides a more
accurate estimate of the out-of-sample accuracy and is more efficient as every
observation is used for both training and testing as oppose to the
train/test/split method.

```python
def fit_predict_model():

    # Get the features and labels from the Boston housing data
    
    X, y = boston.data, boston.target

    # Setup a Decision Tree Regressor
    
    regressor = DecisionTreeRegressor()
    
    parameters = {'max_depth':(1,2,3,4,5,6,7,8,9,10)}
    
    mse_scoring = make_scorer(mean_squared_error, greater_is_better=False)
    
    #using grid search to fine tune the Decision Tree Regressor and
    #obtain the parameters that generate the best training performance. 

    reg = GridSearchCV(regressor, parameters, scoring = mse_scoring)
    reg.fit(X,y)
    
    # Fit the learner to the training data to obtain the best parameter set
    print "Final Model: "
    print (reg.fit(X, y))    

    # Using the model to predict the output of a particular sample
    x = [11.95, 0.00, 18.100, 0, 0.6590, 5.6090, 90.00, 1.385, 24, 680.0, 20.20, 332.09, 12.13]
    x = np.array(x)
    x = x.reshape(1, -1)
    y = reg.predict(x)

    #Predict Housing Price:
    print "\nHouse: " + str(x)
    print "\nPredicted: " + str(y)
    print "\nBest Score %s:" % (reg.best_score_)
    
    #DataFrame of Client_Features
    #x = [11.95, 0.00, 18.100, 0, 0.6590, 5.6090, 90.00, 1.385, 24, 680.0, 20.20, 332.09, 12.13]
    #pd.DataFrame(zip(boston.feature_names, x), columns = ['Features', 'Client_Features'])
    
```

### Run Regression-Tree Model

```python
def main():
    
    # Learning Curve Graphs
    max_depths = [4,5,6,7]
    for max_depth in max_depths:
        learning_curve(max_depth, X_train, y_train, X_test, y_test)
    
    # Model Complexity Graph
    model_complexity(X_train, y_train, X_test, y_test)

    #Tune and predict Model
    fit_predict_model()


if __name__ == "__main__":
    #main()
```

**From the model complexity graph, describe the training and testing errors as the max depth increases. Based on your interpretation of the graph, which max depth results in a model that best generalizes the dataset? Why?**

As the `max_depth` is increased, the training error begins to exponentially
decline and approaches zero as the model becomes more complex. The testing error
begins to differs from the training error near an `max_depth` = `4`, which best
generalizes the dataset. At the instance where the model begins to behave too
much like the training data then it might be observed that overfitting is
occurring as the model does not perform well when testing and the error rates
will begin to significantly differ. A decrease in the training error implies
that the model is becoming better at fitting the data; when the testing error
plateaus and is no longer decreasing, additional knowledge is not gained on the
out-of-sample data. If the error does not reduce any further during testing,
then the complexity is increased for no reason and therefore overfitting occurs.

<img src = "https://tsmith5151.github.io/Blog/img/Boston/Model.Complexity.png">

**Using grid search, what is the optimal max depth for your model? How does this result compare to your initial intuition?**

Based on the model complexity graph, the model that best generalizes the data
is when `max_depth` = `4`. Calling the `grid.best_params_` from `GridSearchCV`
confirms the maximum level to split the decision tree is `4`.


### Tune the Optimal Model

```python
def iterate_fit_predict():
    """.Make a prediction on housing data."""

    # Get the features and labels from the Boston housing data
    X, y = boston.data, boston.target

    # Setup a Decision Tree Regressor
    regressor = DecisionTreeRegressor()
  
    mse_scoring = make_scorer(mean_squared_error, greater_is_better=False)
    
    parameters = {'max_depth':(1,2,3,4,5,6,7,8,9,10)}
    
    reg = GridSearchCV(regressor, parameters, scoring = mse_scoring, cv=3)

    # Fit the learner to the training data to obtain the best parameter set
    reg.fit(X, y)

    # Use the model to predict the output of a particular sample
    x = [11.95, 0.00, 18.100, 0, 0.6590, 5.6090, 90.00, 1.385, 24, 680.0, 20.20, 332.09, 12.13]
    x = np.array(x)
    x = x.reshape(1, -1)
    y = reg.predict(x)
    
    return (reg.best_params_['max_depth'], y[0])
```

Iteration: Fit and Predict Model
    (GridSearchCV Results)

```python
Grid_Search_Results = []

for i in range(100):
    Grid_Search_Results.append(iterate_fit_predict())

Grid_Search= np.asarray(Grid_Search_Results)
Grid_Search_Results_Depth = np.asarray(Grid_Search.T[0], dtype=int)
Grid_Search_Results_Price = np.asarray(Grid_Search.T[1], dtype=float)
```

Histogram: Maximum Depth (iterations = 100)

```python
pl.hist(Grid_Search_Results_Depth, bins = 10, color = 'blue')
pl.suptitle("GridSearchCV Results: Max_Depth", fontsize = 20)
pl.xlabel("Max_Depth", fontsize = 14)
pl.ylabel("Frequency", fontsize = 14)
pl.xlim(1,10)
pl.grid()
pl.show()
```

**Discuss whether you would use this model or not to predict the selling price of future clients? homes in the Boston area.**

Being there is variability in the predicted price, which is actually expected
since the code randomizes the data and therefore affects fitting the model to
the data, multiple iterations of the model is desired in order to provide a
conclusive prediction. Ideally, you would want to run the model multiple times
and determine a range of values to that has the highest frequency of occurrence
and identify a central tendency based on the distribution for the `MEDV`. To do
this, `1,000` iterations (randomly sampling each iteration, `cross-validation`
=`3` ) were performed by using  `sklearn.GridSearchCV()` to determine the
`max_depth` of the decision tree and the predicted housing price. Both results
validate that the `max_depth` = `4` and the suggested value of the home is
consistent around `$21,500`.

***Max_Depth***          


<img src = "https://tsmith5151.github.io/Blog/img/Boston/Gridsearch_MaxDepth_Hist.png">



***Predicted Prices***

<img src = "https://tsmith5151.github.io/Blog/img/Boston/Gridsearch_Prices_Hist.png">