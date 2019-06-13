---
title: Statistical Inference
layout: default
permalink: \SO_statistical_inference
---

Finding quanitative relationships in the data is an important task for any data scientist. Here I look at a few questions about how the students in the 'dropout' and 'nondropout' groups differ. I make use of t-tests, though the degrees of freedom in this case are large enough that it is very nearly a z-test.

```python
#import relevant packages
import numpy as np
import pandas as pd
from scipy import stats
import matplotlib.pyplot as plt
%matplotlib inline
```


```python
#import student data from wrangling stage
filepath = '/Users/chrismalec/DS_Portfolio/CapstoneProjectOne/'
df_BY = pd.read_pickle(filepath + 'df_BY.pkl')
df_weights = pd.read_pickle(filepath + 'df_weights.pkl')
labels = pd.read_pickle(filepath + 'labels.pkl')

import pickle
pickle_in = open(filepath+'number_labels.pkl',"rb")
number_labels = pickle.load(pickle_in)
```

## Statistical Tests

Since the target variable, student outcome as demonstrated by their 2013 transcript, is a categorical variable, I have two types of tests to perform. The first type is the correlation between two discrete variables and the other is correlation between a discrete and continuous variable. In both cases, I will attempt to use a t-test when possible.

Another possibilities for discrete variables is to use a chi-squared statistic. A second possibility for discrete vs continuous variables is a biseral pearson coefficient or a fit to the logit function (logistic regression).

### Discrete variables

The strategy for discrete variables will be to create counts in multiple categories, e.g. students who both dropped out and go to a school with a dropout prevention program and students who dropped out and do not go to a school with a dropout prevention program. If the counts are different, a percent difference in count, and p-value can be calculated. For variables with many categories, a chi-squared test may be more appropriate, as there is no correction needed for multiple hypothesis testing.

### Continuous variables

For continuous variables the means of the two groups (dropped out - did not drop out) can be compared using a t-test. Since the dropped out group is much smaller, it is important to use unequal variances.

### Multiple testing

In this dataset, there are hundreds of variables for each student, and therefore some type of correction needs to be made to account for the multiple tests. If no correction is made, and a p-value of 0.05 or less is used to reject the null hypothesis, out of 1000 variables, on average 50 would show statistical significance by pure chance. The Bonferroni correction is the simplist, and involves dividing the p-value by the number of tests, so that statistical significance has a higher bar. This can be overly conservative and miss many real correlations, so I will also try the Holm-Bonferroni correction which sorts the hypothesis tests by p-value from lowest to highest and dividing the critical value by $1/(N - i + 1)$ where $N$ is the number of tests and $i$ is the $i^{th}$ test.


```python
#Create a mask using the label vector of student outcomes

print(number_labels['X3TOUTCOME']) #Is there a way to print out dicts nice?
mask = (labels == 8)
N_DO = df_weights.loc[mask,'W3STUDENTTR'].sum()
df_weights_DO = df_weights.loc[mask,:]
df_weights_nDO = df_weights.loc[~mask,:]
print('The total number of students who dropped out in the study was',
      np.sum(mask),
      '. Corresponding to a total of',
      round(N_DO,0),
      'nationwide')
df_DO = df_BY.loc[mask,:]
df_nDO = df_BY.loc[~mask,:]
```

    {'desc': 'X3 Transcript indicated outcome', '1': '"Fall 2012-summer 2013 graduate"', '2': '"Post-summer 2013 graduate"', '3': '"Pre-fall 2012 graduate"', '4': '"Graduation date unknown"', '6': '"Certificate of attendance"', '8': '"Dropped out"', '9': '"Transferred"', '10': '"Left other reason"', '11': '"Still enrolled"', '12': '"Status cannot be determined"', '-8': '"Unit non-response"'}
    The total number of students who dropped out in the study was 101 . Corresponding to a total of 21084.0 nationwide



```python
def estimate_prop(column,df_data,weight,df_weight,code_dict):
    replicate_weight_list = [weight +'{:03d}'.format(x) for x in range(1,201)]
    df_agg = pd.concat([df_data[column].astype('int64',errors = 'ignore'),df_weight[[weight]+replicate_weight_list]],axis=1)
    df_agg = df_agg.groupby(column).sum()
    mask = df_agg.index > 0
    df_agg =  df_agg.loc[mask,:]#dropping the non-response values, may decide differently later
    df_estimate_variance = pd.DataFrame(data={'category':[],'estimate':[],'s.e.':[]})
    #For some reason, some numeric variable labels are floats instead of integers, may fix this is cleaning stage.
    for i, row in df_agg.iterrows():
        #double check std estimation here
        N_samp = len(replicate_weight_list)
        N_pop = df_agg[weight].sum()
        estimate = df_agg.loc[i,weight]/N_pop
        variance = ((df_agg.loc[i,replicate_weight_list]/N_pop - estimate)**2).sum()/(N_samp-1)
        df_estimate_variance.loc[i] = [code_dict[str(round(i))],
                                       estimate,
                                       np.sqrt(variance/N_samp)]
    
    return df_estimate_variance
```


```python
#An example of discrete-discrete hypothesis test one t-test
#Null hypothesis is that the count in both categories is the same

print(number_labels['X1SEX']) #Description of column
N_tests = 2 #get this off of number of rows in aggregated data frame
column = 'X1SEX'
weight = 'W1STUDENT'

df_est_std_DO = estimate_prop(column,
                               df_DO,
                               weight,
                               df_weights_DO,
                               number_labels[column]   
                            )

df_est_std_nDO = estimate_prop(column,
                               df_nDO,
                               weight,
                               df_weights_nDO,
                               number_labels[column]
                            )

df_stats = pd.DataFrame(data={'difference':[],'t_statistic':[],'p-value':[]})

for i in df_est_std_DO.index:
    se = np.sqrt(df_est_std_DO.loc[1,'s.e.']**2 + df_est_std_nDO.loc[2,'s.e.']**2)
    diff = df_est_std_DO.loc[i,'estimate'] - df_est_std_nDO.loc[i,'estimate']
    t_statistic = diff/se
    df = 100 #Not sure what to put for df here, weighted or unweighted? Maybe just use normal
    if diff > 0:
        p_value = 1 - stats.t.cdf(t_statistic,df=df)
    else:
        p_value = stats.t.cdf(t_statistic,df=df)
    df_stats.loc[i] = [diff,t_statistic,p_value]
    
print('Drop out:'+'\n',df_est_std_DO,'\n')
print('Non-drop out:'+'\n',df_est_std_nDO,'\n')
print('Stat summary:'+'\n',df_stats,'\n')
print('Significance is indicated by p<'+str(0.05/N_tests))
```

    {'desc': "X1 Student's sex", '1': '"Male"', '2': '"Female"', '-9': '"Missing"'}
    Drop out:
        category  estimate      s.e.
    1    "Male"  0.673684  0.012252
    2  "Female"  0.326316  0.005288 
    
    Non-drop out:
        category  estimate      s.e.
    1    "Male"  0.502636  0.000485
    2  "Female"  0.497364  0.000515 
    
    Stat summary:
        difference  t_statistic       p-value
    1    0.171048    13.949089  0.000000e+00
    2   -0.171048   -13.949089  1.689540e-25 
    
    Significance is indicated by p<0.025


Here, males are shown to be a much larger group than females within the dropout category. Males are 17 percentage points higher in the dropout category while females are 17 percentage points lower.


```python
#An example of discrete-discrete hypothesis test using multiple t-tests
#Null hypothesis is that the count in all non-target categories is the same

print(number_labels['S1SAFE']) #Description of column
N_tests = 4 #get this off of number of rows in aggregated data frame
column = 'S1SAFE'
weight = 'W1STUDENT'

df_est_std_DO = estimate_prop(column,
                               df_DO,
                               weight,
                               df_weights_DO,
                               number_labels[column])

df_est_std_nDO = estimate_prop(column,
                               df_nDO,
                               weight,
                               df_weights_nDO,
                               number_labels[column])

df_stats = pd.DataFrame(data={'difference':[],'t_statistic':[],'p-value':[]})

for i in df_est_std_DO.index:
    se = np.sqrt(df_est_std_DO.loc[i,'s.e.']**2 + df_est_std_nDO.loc[i,'s.e.']**2)
    diff = df_est_std_DO.loc[i,'estimate'] - df_est_std_nDO.loc[i,'estimate']
    t_statistic = diff/se
    df = 100 #Not sure what to put for df here, weighted or unweighted? Maybe just use normal
    if diff > 0:
        p_value = 1 - stats.t.cdf(t_statistic,df=df)
    else:
        p_value = stats.t.cdf(t_statistic,df=df)
    df_stats.loc[i] = [diff,t_statistic,p_value]
    
print('Drop out:'+'\n',df_est_std_DO,'\n')
print('Non-drop out:'+'\n',df_est_std_nDO,'\n')
print('Stat summary:'+'\n',df_stats,'\n')
print('Significance is indicated by p<'+str(0.05/N_tests))
```

    {'desc': 'S1 E01A 9th grader feels safe at school', '1': '"Strongly agree"', '2': '"Agree"', '3': '"Disagree"', '4': '"Strongly disagree"', '-8': '"Unit non-response"', '-9': '"Missing"'}
    Drop out:
                   category  estimate      s.e.
    1     "Strongly agree"  0.163576  0.003473
    2              "Agree"  0.600024  0.011062
    3           "Disagree"  0.221125  0.006343
    4  "Strongly disagree"  0.015275  0.001104 
    
    Non-drop out:
                   category  estimate      s.e.
    1     "Strongly agree"  0.304911  0.000508
    2              "Agree"  0.595092  0.000496
    3           "Disagree"  0.077045  0.000265
    4  "Strongly disagree"  0.022952  0.000141 
    
    Stat summary:
        difference  t_statistic       p-value
    1   -0.141335   -40.269371  6.541278e-64
    2    0.004932     0.445407  3.284943e-01
    3    0.144080    22.696372  0.000000e+00
    4   -0.007677    -6.894834  2.465504e-10 
    
    Significance is indicated by p<0.0125


For this one test, all but one category had a difference between the proportion of students answering this question a certain way depending on if they were in the 'drop out' or 'non-drop out' category. The strongest effects were that those who 'strongly agreed' that they feel safe at the school were 14% percentage points less likely to be in the drop out category, and those who 'disagreed' that they feel safe at school were 14% percentage points more likely to be in the drop out category. The remaining categories were less than a 1 percentage point difference. (Maybe need to state this differently)


```python
def estimate_statistic(column,df_data,weight,df_weight,func = np.sum,normalize = True):
    replicate_weight_list = [weight +'{:03d}'.format(x) for x in range(1,201)]
    analytic_weight = df_weight[weight]
    replicate_weights = df_weight[replicate_weight_list]
    sum_analytic_weight = analytic_weight.sum()
    value = df_data[[column]]
    if(normalize == False):
        sum_analytic_weight = 1
        
    estimate = func(df_data[column]*analytic_weight)/sum_analytic_weight
    N_samp = len(replicate_weight_list)
    replicate_estimates = np.empty(len(replicate_weight_list))
    for i,w in enumerate(replicate_weights.columns):
        replicate_estimate = func(df_data[column].multiply(replicate_weights[w]))/sum_analytic_weight
        replicate_estimates[i] = replicate_estimate
    
    variance = ((replicate_estimates - estimate)**2).sum()/(N_samp-1)
    se = np.sqrt(variance/N_samp)
    return(estimate, se)
```


```python
#An example of discrete-continuous hypothesis test
#Null hypothesis is that the mean in the target categories is the same
print(number_labels['X1TXMTSCOR'])
column = 'X1TXMTSCOR'
weight = 'W1MATHTCH'
N_tests = 1
na_mask = df_DO[column] > 0
mean_DO, se_DO = estimate_statistic(column,
                                    df_DO.loc[na_mask,:],
                                    weight,
                                    df_weights_DO)

mean_nDO, se_nDO = estimate_statistic(column,
                                    df_nDO,
                                    weight,
                                    df_weights_nDO)

df_stats = pd.DataFrame(data={'difference':[],'t_statistic':[],'p-value':[]})
se = np.sqrt(se_DO**2 + se_nDO**2)
diff = mean_DO - mean_nDO
t_statistic = diff/se
df = 100 #Not sure what to put for df here, weighted or unweighted? Maybe just use normal
if diff > 0:
    p_value = 1 - stats.t.cdf(t_statistic,df=df)
else:
    p_value = stats.t.cdf(t_statistic,df=df)
df_stats.loc[1] = [diff,t_statistic,p_value]

print('Drop out:',mean_DO, se_DO ,'\n')
print('Non-drop out:',mean_nDO, se_nDO,'\n')
print('Stat summary:'+'\n',df_stats,'\n')
print('Significance is indicated by p<'+str(0.05/N_tests))
```

    {'desc': 'X1 Mathematics standardized theta score'}
    Drop out: 43.04742118032044 0.7304670767613165 
    
    Non-drop out: 50.26713040274542 0.0173492630398546 
    
    Stat summary:
        difference  t_statistic       p-value
    1   -7.219709    -9.880902  9.032530e-17 
    
    Significance is indicated by p<0.05


We can see that there is a statistically significant higher math theta score for non-dropouts. The effect is non-trivial since the range of scores lies between 20 and 80.

## Automation

To automate the process of hypothesis testing, first the type of data must be detected, fortunately, the codebook gives us a convenient label. If the column is a discrete data type, the codebook lists the meaning of each numeric value. The way I have set up the dictionary means that if there is no keys after 'desc', then it is a continuous variable.

A stat summary table could be created, and then statistically significant values that pass the more stringent $\alpha$ value set by the multiple hypothesis testing corrections could be inspected further.
