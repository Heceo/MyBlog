---
title: "Debt Default Forecast"
#date: "2018-12-04"
categories : ["project"]
comments: true
tags : ["machine learning"]
draft: false
showpagemeta : true 
showcomments : true
slug: ""
description: "SVM, Random Forest, Stacking"
---

# 1.Introduction

We now have desensitization data for a small number of customer accounts of a bank. Some customers have credit defaults. We need to provide some characteristics based on the data (id, age, education level, service age, address, income, liabilities). Rate, credit card liabilities, other liabilities, etc.) to establish a model to predict whether a customer has a risk of loan default, the data style is as follows:

|      | id     | age  | education | lengthofservice | address | income | debt | creditcarddebt | otherdebt | default |
| ---- | ------ | ---- | --------- | --------------- | ------- | ------ | ---- | -------------- | --------- | ------- |
| 0    | 261656 | 41   | 3         | 17              | 12      | 176    | 9.3  | 11.36          | 5.01      | 1       |
| 1    | 878257 | 27   | 1         | 10              | 6       | 31     | 17.3 | 1.36           | 4         | 0       |
| 2    | 241261 | 40   | 1         | 15              | 14      | 55     | 5.5  | 0.86           | 2.17      | 0       |
| 3    | 314472 | 41   | 1         | 15              | 14      | 120    | 2.9  | 2.66           | 0.82      | 0       |
| 4    | 624718 | 24   | 2         | 2               | 0       | 28     | 17.3 | 1.79           | 3.06      | 1       |
| 5    | 477639 | 41   | 2         | 5               | 5       | 25     | 10.2 | 0.39           | 2.16      | 0       |
| 6    | 361223 | 39   | 1         | 20              | 9       | 67     | 30.6 | 3.83           | 16.67     | 0       |

REquirements:
1. Clean and explore the data.
2. Select the appropriate model algorithm to model.
3. Select appropriate evaluation indicators for model evaluation and adjustment.
4. Save the trained model

# 2.Codes

## 1.调参

### Random Forest

```python
#数据读取与one hot编码：
import pandas as pd  
import numpy as np  
from sklearn.ensemble import RandomForestClassifier  
from sklearn.grid_search import GridSearchCV  
from sklearn import cross_validation, metrics  
import matplotlib.pylab as plt 
from sklearn.svm import SVC
from sklearn import model_selection 
from sklearn.model_selection import GridSearchCV
from sklearn.metrics import classification_report
from sklearn.linear_model import LogisticRegression
from mlxtend.classifier import StackingClassifier 

data = pd.read_csv('/Users/huyifei/Downloads/34-20180502140510/bankloan.csv',index_col=0)
data.drop('id',axis=1, inplace=True)
data.dropna(axis=0, how='any', thresh=None, subset=None, inplace=True)
y_feature = data['default']
x_feature = data.drop('default', axis=1)
x_feature = pd.get_dummies(data, columns=['education', 'age', 'lengthofservice',\
                                          'address', 'income', 'debt', 'creditcarddebt', 'otherdebt']) 
x_feature = x_feature.astype('float64')

rf0 = RandomForestClassifier(oob_score=True, random_state=10)  
rf0.fit(x_feature,y_feature)  
print (rf0.oob_score_)  
y_feature_predprob = rf0.predict_proba(data)[:,1]  
print ("AUC Score (Train): %f" % metrics.roc_auc_score(y_feature,y_feature_predprob))
#得到初始的两个打分为
0.9428571428571428
AUC Score (Train): 1.000000 

#网格搜索n_estimators：
param_test1= {'n_estimators':list(range(10,100,10))}  
gsearch1= GridSearchCV(estimator = RandomForestClassifier(min_samples_split=20,  
                                 min_samples_leaf=20,max_depth=8,max_features='sqrt' ,random_state=10),  
                       param_grid =param_test1, scoring='roc_auc',cv=5)  
gsearch1.fit(x_feature,y_feature)
print(gsearch1.grid_scores_,gsearch1.best_params_, gsearch1.best_score_)
#输出：
[mean: 0.66030, std: 0.15863, params: {'n_estimators': 10}, mean: 0.98751, std: 0.02472, params: {'n_estimators': 20}, mean: 1.00000, std: 0.00000, params: {'n_estimators': 30}, mean: 1.00000, std: 0.00000, params: {'n_estimators': 40}, mean: 1.00000, std: 0.00000, params: {'n_estimators': 50}, mean: 0.99990, std: 0.00021, params: {'n_estimators': 60}, mean: 0.99848, std: 0.00202, params: {'n_estimators': 70}, mean: 0.99990, std: 0.00021, params: {'n_estimators': 80}, mean: 0.99995, std: 0.00010, params: {'n_estimators': 90}] {'n_estimators': 30} 1.0
#则n_estimators为30最好

#网格搜索max_depth和min_samples_split：
param_test2= {'max_depth':list(range(1,20,2)), 'min_samples_split':list(range(2,20,2))}  
gsearch2= GridSearchCV(estimator = RandomForestClassifier(n_estimators= 30,  
                                 min_samples_leaf=20,max_features='sqrt' ,oob_score=True,random_state=10),  
   param_grid = param_test2,scoring='roc_auc',iid=False, cv=5)  
gsearch2.fit(x_feature,y_feature)
print(gsearch2.grid_scores_,gsearch2.best_params_, gsearch2.best_score_)
#输出：
[mean: 0.99964, std: 0.00073, params: {'max_depth': 1, 'min_samples_split': 2}, mean: 0.99964, std: 0.00073, params: {'max_depth': 1, 'min_samples_split': 4}, mean: 0.99964, std: 0.00073, params: {'max_depth': 1, 'min_samples_split': 6}, mean: 0.99964, std: 0.00073, params: {'max_depth': 1, 'min_samples_split': 8}, mean: 0.99964, std: 0.00073, params: {'max_depth': 1, 'min_samples_split': 10}, mean: 0.99964, std: 0.00073, params: {'max_depth': 1, 'min_samples_split': 12}, mean: 0.99964, std: 0.00073, params: {'max_depth': 1, 'min_samples_split': 14}, mean: 0.99964, std: 0.00073, params: {'max_depth': 1, 'min_samples_split': 16}, mean: 0.99964, std: 0.00073, params: {'max_depth': 1, 'min_samples_split': 18}, mean: 1.00000, std: 0.00000, params: {'max_depth': 3, 'min_samples_split': 2}, mean: 1.00000, std: 0.00000, params: {'max_depth': 3, 'min_samples_split': 4}, mean: 1.00000, std: 0.00000, params: {'max_depth': 3, 'min_samples_split': 6}, mean: 1.00000, std: 0.00000, params: {'max_depth': 3, 'min_samples_split': 8}, mean: 1.00000, std: 0.00000, params: {'max_depth': 3, 'min_samples_split': 10}, mean: 1.00000, std: 0.00000, params: {'max_depth': 3, 'min_samples_split': 12}, mean: 1.00000, std: 0.00000, params: {'max_depth': 3, 'min_samples_split': 14}, mean: 1.00000, std: 0.00000, params: {'max_depth': 3, 'min_samples_split': 16}, mean: 1.00000, std: 0.00000, params: {'max_depth': 3, 'min_samples_split': 18}, mean: 1.00000, std: 0.00000, params: {'max_depth': 5, 'min_samples_split': 2}, mean: 1.00000, std: 0.00000, params: {'max_depth': 5, 'min_samples_split': 4}, mean: 1.00000, std: 0.00000, params: {'max_depth': 5, 'min_samples_split': 6}, mean: 1.00000, std: 0.00000, params: {'max_depth': 5, 'min_samples_split': 8}, mean: 1.00000, std: 0.00000, params: {'max_depth': 5, 'min_samples_split': 10}, mean: 1.00000, std: 0.00000, params: {'max_depth': 5, 'min_samples_split': 12}, mean: 1.00000, std: 0.00000, params: {'max_depth': 5, 'min_samples_split': 14}, mean: 1.00000, std: 0.00000, params: {'max_depth': 5, 'min_samples_split': 16}, mean: 1.00000, std: 0.00000, params: {'max_depth': 5, 'min_samples_split': 18}, mean: 1.00000, std: 0.00000, params: {'max_depth': 7, 'min_samples_split': 2}, mean: 1.00000, std: 0.00000, params: {'max_depth': 7, 'min_samples_split': 4}, mean: 1.00000, std: 0.00000, params: {'max_depth': 7, 'min_samples_split': 6}, mean: 1.00000, std: 0.00000, params: {'max_depth': 7, 'min_samples_split': 8}, mean: 1.00000, std: 0.00000, params: {'max_depth': 7, 'min_samples_split': 10}, mean: 1.00000, std: 0.00000, params: {'max_depth': 7, 'min_samples_split': 12}, mean: 1.00000, std: 0.00000, params: {'max_depth': 7, 'min_samples_split': 14}, mean: 1.00000, std: 0.00000, params: {'max_depth': 7, 'min_samples_split': 16}, mean: 1.00000, std: 0.00000, params: {'max_depth': 7, 'min_samples_split': 18}, mean: 1.00000, std: 0.00000, params: {'max_depth': 9, 'min_samples_split': 2}, mean: 1.00000, std: 0.00000, params: {'max_depth': 9, 'min_samples_split': 4}, mean: 1.00000, std: 0.00000, params: {'max_depth': 9, 'min_samples_split': 6}, mean: 1.00000, std: 0.00000, params: {'max_depth': 9, 'min_samples_split': 8}, mean: 1.00000, std: 0.00000, params: {'max_depth': 9, 'min_samples_split': 10}, mean: 1.00000, std: 0.00000, params: {'max_depth': 9, 'min_samples_split': 12}, mean: 1.00000, std: 0.00000, params: {'max_depth': 9, 'min_samples_split': 14}, mean: 1.00000, std: 0.00000, params: {'max_depth': 9, 'min_samples_split': 16}, mean: 1.00000, std: 0.00000, params: {'max_depth': 9, 'min_samples_split': 18}, mean: 1.00000, std: 0.00000, params: {'max_depth': 11, 'min_samples_split': 2}, mean: 1.00000, std: 0.00000, params: {'max_depth': 11, 'min_samples_split': 4}, mean: 1.00000, std: 0.00000, params: {'max_depth': 11, 'min_samples_split': 6}, mean: 1.00000, std: 0.00000, params: {'max_depth': 11, 'min_samples_split': 8}, mean: 1.00000, std: 0.00000, params: {'max_depth': 11, 'min_samples_split': 10}, mean: 1.00000, std: 0.00000, params: {'max_depth': 11, 'min_samples_split': 12}, mean: 1.00000, std: 0.00000, params: {'max_depth': 11, 'min_samples_split': 14}, mean: 1.00000, std: 0.00000, params: {'max_depth': 11, 'min_samples_split': 16}, mean: 1.00000, std: 0.00000, params: {'max_depth': 11, 'min_samples_split': 18}, mean: 1.00000, std: 0.00000, params: {'max_depth': 13, 'min_samples_split': 2}, mean: 1.00000, std: 0.00000, params: {'max_depth': 13, 'min_samples_split': 4}, mean: 1.00000, std: 0.00000, params: {'max_depth': 13, 'min_samples_split': 6}, mean: 1.00000, std: 0.00000, params: {'max_depth': 13, 'min_samples_split': 8}, mean: 1.00000, std: 0.00000, params: {'max_depth': 13, 'min_samples_split': 10}, mean: 1.00000, std: 0.00000, params: {'max_depth': 13, 'min_samples_split': 12}, mean: 1.00000, std: 0.00000, params: {'max_depth': 13, 'min_samples_split': 14}, mean: 1.00000, std: 0.00000, params: {'max_depth': 13, 'min_samples_split': 16}, mean: 1.00000, std: 0.00000, params: {'max_depth': 13, 'min_samples_split': 18}, mean: 1.00000, std: 0.00000, params: {'max_depth': 15, 'min_samples_split': 2}, mean: 1.00000, std: 0.00000, params: {'max_depth': 15, 'min_samples_split': 4}, mean: 1.00000, std: 0.00000, params: {'max_depth': 15, 'min_samples_split': 6}, mean: 1.00000, std: 0.00000, params: {'max_depth': 15, 'min_samples_split': 8}, mean: 1.00000, std: 0.00000, params: {'max_depth': 15, 'min_samples_split': 10}, mean: 1.00000, std: 0.00000, params: {'max_depth': 15, 'min_samples_split': 12}, mean: 1.00000, std: 0.00000, params: {'max_depth': 15, 'min_samples_split': 14}, mean: 1.00000, std: 0.00000, params: {'max_depth': 15, 'min_samples_split': 16}, mean: 1.00000, std: 0.00000, params: {'max_depth': 15, 'min_samples_split': 18}, mean: 1.00000, std: 0.00000, params: {'max_depth': 17, 'min_samples_split': 2}, mean: 1.00000, std: 0.00000, params: {'max_depth': 17, 'min_samples_split': 4}, mean: 1.00000, std: 0.00000, params: {'max_depth': 17, 'min_samples_split': 6}, mean: 1.00000, std: 0.00000, params: {'max_depth': 17, 'min_samples_split': 8}, mean: 1.00000, std: 0.00000, params: {'max_depth': 17, 'min_samples_split': 10}, mean: 1.00000, std: 0.00000, params: {'max_depth': 17, 'min_samples_split': 12}, mean: 1.00000, std: 0.00000, params: {'max_depth': 17, 'min_samples_split': 14}, mean: 1.00000, std: 0.00000, params: {'max_depth': 17, 'min_samples_split': 16}, mean: 1.00000, std: 0.00000, params: {'max_depth': 17, 'min_samples_split': 18}, mean: 1.00000, std: 0.00000, params: {'max_depth': 19, 'min_samples_split': 2}, mean: 1.00000, std: 0.00000, params: {'max_depth': 19, 'min_samples_split': 4}, mean: 1.00000, std: 0.00000, params: {'max_depth': 19, 'min_samples_split': 6}, mean: 1.00000, std: 0.00000, params: {'max_depth': 19, 'min_samples_split': 8}, mean: 1.00000, std: 0.00000, params: {'max_depth': 19, 'min_samples_split': 10}, mean: 1.00000, std: 0.00000, params: {'max_depth': 19, 'min_samples_split': 12}, mean: 1.00000, std: 0.00000, params: {'max_depth': 19, 'min_samples_split': 14}, mean: 1.00000, std: 0.00000, params: {'max_depth': 19, 'min_samples_split': 16}, mean: 1.00000, std: 0.00000, params: {'max_depth': 19, 'min_samples_split': 18}] {'max_depth': 3, 'min_samples_split': 2} 1.0
#则max_depth为3, min_samples_split为2最好

#再对内部节点再划分所需最小样本数min_samples_split和叶子节点最少样本数min_samples_leaf一起调参 ：
param_test3= {'min_samples_split':list(range(2,20,2)), 'min_samples_leaf':list(range(1,10,1))}  
gsearch3= GridSearchCV(estimator = RandomForestClassifier(n_estimators= 30,max_depth=3, 
                                 max_features='sqrt' ,oob_score=True, random_state=10), 
   param_grid = param_test3,scoring='roc_auc',iid=False, cv=5)  
gsearch3.fit(x_feature,y_feature) 
print(gsearch3.grid_scores_, gsearch3.best_params_, gsearch3.best_score_)
#输出
[mean: 0.99905, std: 0.00176, params: {'min_samples_leaf': 1, 'min_samples_split': 2}, mean: 0.99905, std: 0.00176, params: {'min_samples_leaf': 1, 'min_samples_split': 4}, mean: 0.99905, std: 0.00176, params: {'min_samples_leaf': 1, 'min_samples_split': 6}, mean: 0.99905, std: 0.00176, params: {'min_samples_leaf': 1, 'min_samples_split': 8}, mean: 0.99905, std: 0.00176, params: {'min_samples_leaf': 1, 'min_samples_split': 10}, mean: 0.99905, std: 0.00176, params: {'min_samples_leaf': 1, 'min_samples_split': 12}, mean: 0.99905, std: 0.00176, params: {'min_samples_leaf': 1, 'min_samples_split': 14}, mean: 0.99905, std: 0.00176, params: {'min_samples_leaf': 1, 'min_samples_split': 16}, mean: 0.99911, std: 0.00166, params: {'min_samples_leaf': 1, 'min_samples_split': 18}, mean: 0.99890, std: 0.00184, params: {'min_samples_leaf': 2, 'min_samples_split': 2}, mean: 0.99890, std: 0.00184, params: {'min_samples_leaf': 2, 'min_samples_split': 4}, mean: 0.99890, std: 0.00184, params: {'min_samples_leaf': 2, 'min_samples_split': 6}, mean: 0.99890, std: 0.00184, params: {'min_samples_leaf': 2, 'min_samples_split': 8}, mean: 0.99890, std: 0.00184, params: {'min_samples_leaf': 2, 'min_samples_split': 10}, mean: 0.99890, std: 0.00184, params: {'min_samples_leaf': 2, 'min_samples_split': 12}, mean: 0.99890, std: 0.00184, params: {'min_samples_leaf': 2, 'min_samples_split': 14}, mean: 0.99890, std: 0.00184, params: {'min_samples_leaf': 2, 'min_samples_split': 16}, mean: 0.99890, std: 0.00184, params: {'min_samples_leaf': 2, 'min_samples_split': 18}, mean: 1.00000, std: 0.00000, params: {'min_samples_leaf': 3, 'min_samples_split': 2}, mean: 1.00000, std: 0.00000, params: {'min_samples_leaf': 3, 'min_samples_split': 4}, mean: 1.00000, std: 0.00000, params: {'min_samples_leaf': 3, 'min_samples_split': 6}, mean: 1.00000, std: 0.00000, params: {'min_samples_leaf': 3, 'min_samples_split': 8}, mean: 1.00000, std: 0.00000, params: {'min_samples_leaf': 3, 'min_samples_split': 10}, mean: 1.00000, std: 0.00000, params: {'min_samples_leaf': 3, 'min_samples_split': 12}, mean: 1.00000, std: 0.00000, params: {'min_samples_leaf': 3, 'min_samples_split': 14}, mean: 1.00000, std: 0.00000, params: {'min_samples_leaf': 3, 'min_samples_split': 16}, mean: 1.00000, std: 0.00000, params: {'min_samples_leaf': 3, 'min_samples_split': 18}, mean: 0.99990, std: 0.00021, params: {'min_samples_leaf': 4, 'min_samples_split': 2}, mean: 0.99990, std: 0.00021, params: {'min_samples_leaf': 4, 'min_samples_split': 4}, mean: 0.99990, std: 0.00021, params: {'min_samples_leaf': 4, 'min_samples_split': 6}, mean: 0.99990, std: 0.00021, params: {'min_samples_leaf': 4, 'min_samples_split': 8}, mean: 0.99990, std: 0.00021, params: {'min_samples_leaf': 4, 'min_samples_split': 10}, mean: 0.99990, std: 0.00021, params: {'min_samples_leaf': 4, 'min_samples_split': 12}, mean: 0.99990, std: 0.00021, params: {'min_samples_leaf': 4, 'min_samples_split': 14}, mean: 0.99990, std: 0.00021, params: {'min_samples_leaf': 4, 'min_samples_split': 16}, mean: 0.99990, std: 0.00021, params: {'min_samples_leaf': 4, 'min_samples_split': 18}, mean: 0.99957, std: 0.00050, params: {'min_samples_leaf': 5, 'min_samples_split': 2}, mean: 0.99957, std: 0.00050, params: {'min_samples_leaf': 5, 'min_samples_split': 4}, mean: 0.99957, std: 0.00050, params: {'min_samples_leaf': 5, 'min_samples_split': 6}, mean: 0.99957, std: 0.00050, params: {'min_samples_leaf': 5, 'min_samples_split': 8}, mean: 0.99957, std: 0.00050, params: {'min_samples_leaf': 5, 'min_samples_split': 10}, mean: 0.99957, std: 0.00050, params: {'min_samples_leaf': 5, 'min_samples_split': 12}, mean: 0.99957, std: 0.00050, params: {'min_samples_leaf': 5, 'min_samples_split': 14}, mean: 0.99957, std: 0.00050, params: {'min_samples_leaf': 5, 'min_samples_split': 16}, mean: 0.99936, std: 0.00091, params: {'min_samples_leaf': 5, 'min_samples_split': 18}, mean: 0.99925, std: 0.00125, params: {'min_samples_leaf': 6, 'min_samples_split': 2}, mean: 0.99925, std: 0.00125, params: {'min_samples_leaf': 6, 'min_samples_split': 4}, mean: 0.99925, std: 0.00125, params: {'min_samples_leaf': 6, 'min_samples_split': 6}, mean: 0.99925, std: 0.00125, params: {'min_samples_leaf': 6, 'min_samples_split': 8}, mean: 0.99925, std: 0.00125, params: {'min_samples_leaf': 6, 'min_samples_split': 10}, mean: 0.99925, std: 0.00125, params: {'min_samples_leaf': 6, 'min_samples_split': 12}, mean: 0.99925, std: 0.00125, params: {'min_samples_leaf': 6, 'min_samples_split': 14}, mean: 0.99925, std: 0.00125, params: {'min_samples_leaf': 6, 'min_samples_split': 16}, mean: 0.99925, std: 0.00125, params: {'min_samples_leaf': 6, 'min_samples_split': 18}, mean: 0.99925, std: 0.00112, params: {'min_samples_leaf': 7, 'min_samples_split': 2}, mean: 0.99925, std: 0.00112, params: {'min_samples_leaf': 7, 'min_samples_split': 4}, mean: 0.99925, std: 0.00112, params: {'min_samples_leaf': 7, 'min_samples_split': 6}, mean: 0.99925, std: 0.00112, params: {'min_samples_leaf': 7, 'min_samples_split': 8}, mean: 0.99925, std: 0.00112, params: {'min_samples_leaf': 7, 'min_samples_split': 10}, mean: 0.99925, std: 0.00112, params: {'min_samples_leaf': 7, 'min_samples_split': 12}, mean: 0.99925, std: 0.00112, params: {'min_samples_leaf': 7, 'min_samples_split': 14}, mean: 0.99925, std: 0.00112, params: {'min_samples_leaf': 7, 'min_samples_split': 16}, mean: 0.99925, std: 0.00112, params: {'min_samples_leaf': 7, 'min_samples_split': 18}, mean: 0.99903, std: 0.00111, params: {'min_samples_leaf': 8, 'min_samples_split': 2}, mean: 0.99903, std: 0.00111, params: {'min_samples_leaf': 8, 'min_samples_split': 4}, mean: 0.99903, std: 0.00111, params: {'min_samples_leaf': 8, 'min_samples_split': 6}, mean: 0.99903, std: 0.00111, params: {'min_samples_leaf': 8, 'min_samples_split': 8}, mean: 0.99903, std: 0.00111, params: {'min_samples_leaf': 8, 'min_samples_split': 10}, mean: 0.99903, std: 0.00111, params: {'min_samples_leaf': 8, 'min_samples_split': 12}, mean: 0.99903, std: 0.00111, params: {'min_samples_leaf': 8, 'min_samples_split': 14}, mean: 0.99903, std: 0.00111, params: {'min_samples_leaf': 8, 'min_samples_split': 16}, mean: 0.99903, std: 0.00111, params: {'min_samples_leaf': 8, 'min_samples_split': 18}, mean: 0.99919, std: 0.00090, params: {'min_samples_leaf': 9, 'min_samples_split': 2}, mean: 0.99919, std: 0.00090, params: {'min_samples_leaf': 9, 'min_samples_split': 4}, mean: 0.99919, std: 0.00090, params: {'min_samples_leaf': 9, 'min_samples_split': 6}, mean: 0.99919, std: 0.00090, params: {'min_samples_leaf': 9, 'min_samples_split': 8}, mean: 0.99919, std: 0.00090, params: {'min_samples_leaf': 9, 'min_samples_split': 10}, mean: 0.99919, std: 0.00090, params: {'min_samples_leaf': 9, 'min_samples_split': 12}, mean: 0.99919, std: 0.00090, params: {'min_samples_leaf': 9, 'min_samples_split': 14}, mean: 0.99919, std: 0.00090, params: {'min_samples_leaf': 9, 'min_samples_split': 16}, mean: 0.99919, std: 0.00090, params: {'min_samples_leaf': 9, 'min_samples_split': 18}] {'min_samples_leaf': 3, 'min_samples_split': 2} 1.0
#则min_samples_leaf为3, min_samples_split为2最好

```

### Support Vector Machine

```python
# 网格搜索查找SVM的核函数和参数，设置gridsearch的参数
tuned_parameters = [{'kernel': ['rbf'], 'gamma': ['auto'],
                    'C': [1, 10, 100, 1000]},
                    {'kernel': ['linear'], 'C': [1, 10, 100, 1000]},
                   {'kernel':['poly'],'coef0':[0,100],'C':[1,10,100,1000]},
                   {'kernel':['sigmoid'],'gamma':['auto'],'coef0':[0,100],'C':[1,10,100,1000]}
                   ]

#构造这个GridSearch的分类器,5-fold
estimator=SVC(probability=True)
clf_svm=GridSearchCV(estimator,tuned_parameters,scoring='roc_auc',cv=5)
clf_svm.fit(x_train, y_train)
print(clf_svm.best_params_) 
# 得到
{'C': 1, 'gamma': 'auto', 'kernel': 'rbf'}
```



## 2.学习

```python
clf_svm_linear=SVC(C = 1, gamma = 'auto', kernel = 'linear')
clf_svm_linear.fit(x_train, y_train)

clf_svm_rbf=SVC(C = 1, gamma = 'auto', kernel = 'rbf')
clf_svm_rbf.fit(x_train, y_train)

#直接用调好参数的随机森林学习器
clf_rf= RandomForestClassifier(n_estimators= 30,  min_samples_split=2, max_depth=3, 
                                 min_samples_leaf=3, max_features='sqrt' ,oob_score=True)  
clf_rf.fit(data,y_feature)  
print("clf_rf.oob_score_ : ", clf_rf.oob_score_ )

#stackingclassifier
lr=LogisticRegression()
sclf=StackingClassifier(classifiers=[clf_svm_linear, clf_rf],meta_classifier=lr)
for clf,label in zip([clf_svm_linear, clf_svm_rbf, clf_rf,sclf],
                     ['SVM_linear', 'SVM_rbf',
                     'RandomForest',
                     'StackingClassifier']):
    scores=model_selection.cross_val_score(clf,x_train,y_train,cv=5,scoring='accuracy')
    print("Accuracy:%0.5f(+/-%0.5f)[%s]"
          %(scores.mean(),scores.std(),label))
   
#得到
{'C': 1, 'gamma': 'auto', 'kernel': 'rbf'}
clf_rf.oob_score_ :  1.0
Accuracy:1.00000(+/-0.00000)[SVM_linear]
Accuracy:0.73338(+/-0.00670)[SVM_rbf]
Accuracy:0.73338(+/-0.00670)[RandomForest]
Accuracy:0.73338(+/-0.00670)[StackingClassifier]
    
```

# 3.Conclusion

虽然用网格调参得到了Random Forest和SVM的最佳参数，但交叉验证的准确率却并不如核函数为线性核的支持向量机（达到了1.0）。这个结果让我也感到很奇怪，和五月份时做的差别挺大，可能是因为选取了所有的特征（一共8个）的原因吧。

## Reference:

Yifei Hu (胡逸飞)

Wen Miao (缪雯)

Changyang Zeng (曾畅扬)