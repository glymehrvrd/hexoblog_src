---
title: Kaggle Titanic
date: 2017-01-03 22:24:17
tags:
- Machine Learning
- Python
- Kaggle
- Data Science
---

拿Kaggle上的Titanic练了练手，熟悉了`numpy`,`pandas`,`scikit-learn`。训练的模型效果还没有`gender model`，也就是女的全判存活男的全判死的效果好T_T。不过Kaggle上也写了，数据量太小，并不一定能说明问题。先把代码贴上，以后再慢慢改进。

<!-- more -->
```python
# -*- coding: utf-8 -*-
import pandas as pd
import numpy as np
from sklearn.metrics import f1_score

# fix missing items
def fix_missing(d):
    # assign a constant for missing items
    d.Age[d.Age.isnull()]=40
    d.Embarked[d.Embarked.isnull()]='S'
    d.Fare[d.Fare.isnull()]=8

# read training data csv
data=pd.read_csv('train.csv')
# factorize some column
data['Sex']=data['Sex'].astype('category')
data['Embarked']=data['Embarked'].astype('category')
# split into training data(80%) and testing data(20%)
trd=data[0:int(data.shape[0]*0.8)]
ted=data[int(data.shape[0]*0.8):]

# fix missing items
fix_missing(trd)
fix_missing(ted)

#%% create number-encoded data
trdn=trd[['Pclass','Sex','Age','SibSp','Parch','Fare','Embarked']]
trdn.Sex=trdn.Sex.cat.codes
trdn.Embarked=trdn.Embarked.cat.codes

tedn=ted[['Pclass','Sex','Age','SibSp','Parch','Fare','Embarked']]
tedn.Sex=tedn.Sex.cat.codes
tedn.Embarked=tedn.Embarked.cat.codes

#%% find out survival rate between male and female
male=trd[trd.Sex=='male']
female=trd[trd.Sex=='female']

male_survivor=male[male.Survived==1]
female_survivor=female[female.Survived==1]

print 'male survival rate: '+str(male_survivor.shape[0]/float(male.shape[0]))
print 'female survival rate: '+str(female_survivor.shape[0]/float(female.shape[0]))

#%% find out our score if predicting all female to survive and all male to die
ted.loc[:,'prediction']=ted.Sex.map(lambda sex: 1 if sex=='female' else 0)
print "naive f-score:" +str(f1_score(ted.Survived, ted.prediction))

#%% decision tree method
from sklearn import tree
clf=tree.DecisionTreeClassifier()
clf.fit(trdn,trd['Survived'])
ted.tree_prediction=clf.predict(tedn)
print 'decision tree f-score: ' + str(f1_score(ted.Survived, ted.tree_prediction))

#%% random forest method
from sklearn import ensemble
clf=ensemble.RandomForestClassifier(n_estimators=20)
clf.fit(trdn,trd['Survived'])
ted.rf_prediction=clf.predict(tedn)
print 'RF f-score: ' + str(f1_score(ted.Survived, ted.rf_prediction))

#%% logistic regression method
from sklearn import linear_model
clf=linear_model.LogisticRegression()
clf.fit(trdn,trd['Survived'])
ted.lr_prediction=clf.predict(tedn)
print 'LR f-score: ' + str(f1_score(ted.Survived, ted.lr_prediction))

#%% SVM method
from sklearn import svm
clf=svm.SVC(kernel='linear')
clf.fit(trdn,trd['Survived'])
ted.svm_prediction=clf.predict(tedn)
print 'LR f-score: ' + str(f1_score(ted.Survived, ted.svm_prediction))
```

结果如下：
```console
male survival rate: 0.19298245614
female survival rate: 0.7421875
naive f-score:0.704918032787
decision tree f-score: 0.710144927536
RF f-score: 0.755555555556
LR f-score: 0.752136752137
```