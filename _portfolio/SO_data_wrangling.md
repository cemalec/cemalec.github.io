
## Data Wrangling

source of data: https://nces.ed.gov/EDAT/Data/Zip/HSLS_2016_v1_0_CSV_Datasets.zip

source of layout data: https://nces.ed.gov/EDAT/Data/Zip/HSLS_2016_v1_0_CodeBook_Layout.zip


```python
import pandas as pd
import numpy as np
```

The csv file is quite large, but not so large that it can't fit in memory, however, it fits in much more easily if we take away some of the columns that won't be useful to the analysis right away. These include columns that are suppressed for public use due to privacy concerns, columns that include statistical weights used to create accurate aggregates of the data, and columns indicating where values have been imputed.

Though these other columns, particularly the weights, will be used later, here we initially only extract the data columns so that we can read in a table in a reasonable amount of time.


```python
#create variables for file locations
filepath = '/Users/chrismalec/DS_Portfolio/CapstoneProjectOne/'
studentdatafile = 'HSLS_2016_v1_0_CSV_Datasets/hsls_16_student_v1_0.csv'
studentcodebookfile = 'HSLS_2016_v1_0_CodeBook_Layout/Layout_STUDENT.txt'
schooldatafile = 'HSLS_2016_v1_0_CSV_Datasets/hsls_09_school_v1_0.csv'
#import a small sample of the file
with open(filepath+studentdatafile) as file:
    student_df = pd.read_csv(file,nrows = 10)
    file.close()

#Remove columns with '-5' and create list of column names removed.
#Create separate list of columns for weighting and imputation.
#The remaining columns will constitute the database.
student_suppressed_columns = []
student_weight_columns = []
student_imputed_columns = []
student_columns = []
for column in student_df.columns:
    if (student_df[column] == -5).all():
        student_suppressed_columns.append(column)
    elif column[0] == 'W':
        student_weight_columns.append(column)
    elif column[-2:] == 'IM':
        student_imputed_columns.append(column)
    else:
        student_columns.append(column)
```


```python
#Import the selected columns from the file for all rows.
with open(filepath+studentdatafile) as file:
    student_df = pd.read_csv(file,usecols = student_columns)
    file.close()
```

There are several variables that indicate the students' dropout status and whether or not they have ever dropped out. This variable was chosen as the label because it indicated the outcome of the student as stated on their transcript four years after the ninth grade study. Other columns may be removed later due to high (trivial) correlation with the label.


```python
#Create series of labels and remove it from the dataframe.
#labels = student_df.pop('X3TOUTCOME')
```


```python
#Create data frame of weights
with open(filepath+studentdatafile) as file:
    df_weights = pd.read_csv(file,usecols = student_weight_columns)
    file.close()
```

As the project moves forward, it will be convenient to easily move between the numbers that denote various responses to the surveys and the human readable responses that the number represent. For example, the column 'S1ABILITYBA' represents a response to whether the '9th grader thinks he/she has the ability to complete a Bachelor's degree.' The possible answers are coded in the data table as 1,2,3,4,-8, and -9 which correspond to "Definitely not", "Probably not", "Probably", "Definitely", "Unit non-response", and "Missing."

I create a dictionary from the documentation so that I can type number_labels['S1ABILITYBA']['4'] and obtain "Definitely" for use as a label in a graph or figure. Similarly, I include the description of the variable under the key 'desc'.


```python
#create dictionary from the student codebook
#Start creating dictionary.
number_labels = {}
coded_columns = student_columns + student_imputed_columns + [labels.name]
with open(filepath+studentcodebookfile,'r') as file:
    #Reads file until it reaches the descriptions
    for line in file:
        if '/* Variable Names, Locations, and Descriptions */' in line:
            break
    
    #Takes first ASCII column and associates it with a data column
    for line in file:
        if len(line.split()) > 0:
            new_key = line.split()[0].strip()
        
        #places description in dictionary under 'desc'
        #reads until the value labels are reached
        if new_key in coded_columns:
            number_labels[new_key]={}
            desc = ' '.join(line.split()[2:])
            number_labels[new_key]['desc'] = desc
        elif '/* Variable Value Labels */' in line:
            break
        else:
            continue
    
    #key places additional key value pairs to specify the response associated with the number in the data table.       
    for line in file:
        if line.strip() in number_labels.keys():
            key = line.strip()
        #this comes up for some reason, not sure why there is a column or two that has no description but has a value mapping.
        elif line.strip() in coded_columns:
            key = line.strip()
            number_labels[key]={}
        elif line.strip()[:2] == '-5':
            continue
        elif line.strip() in student_suppressed_columns:
            continue
        elif line.strip() in student_weight_columns:
            continue
        #unambiguous end of file
        elif line == '':
            break
        else:
            key_value = line.split('=')
            number_labels[key][key_value[0].strip()] = key_value[1].strip()
file.close()
```

All negative numbers are missing data. The different values denote reasons for that the data is missing, which I can load from the original data if it turns out to be useful. I replace them with an nan value to assist in future data operations.


```python
#replace negative numbers with nan
student_df = student_df.replace([x for x in range(-9,0)],np.nan)

print('There are '+str(student_df.isna().values.sum())+' out of '+str(student_df.size)+' missing values in the dataframe.')
print('The dataframe contains '+str(student_df.shape[1])+' features and '+str(student_df.shape[0])+' observations.')
```

    There are 24183783 out of 63904657 missing values in the dataframe.
    The dataframe contains 2719 features and 23503 observations.


The data in the table is from several sources, and four different collection times. The first two letters in each column name specify the source of the data and the collection time of the data.  So as not to mix time periods, it may be useful to split the dataframe into four dataframes that contain data from the four collection times. After all, I don't want to imply that something that happened in a student's post-secondary career affected their decision to drop out in high school.

First character code:
Composite variables = X
Student = S
Parent = P
Mathematics teacher = M
Science teacher = N
Administrator = A
Counselor = C
Weights = W

Second character code:
Base Year = 1
First Followup = 2
Second Followup = 3
Third Followup = 4


```python
#create logical masks to separate tables based on time
mask_BY = [(x[1]=='1')| (x=='STU_ID') for x in student_df.columns]
mask_F1 = [x[1]=='2' for x in student_df.columns]
mask_F2 = [x[1]=='3' for x in student_df.columns]
mask_F3 = [x[1]=='4' for x in student_df.columns]

#slice original table into four separate tables.
df_BY = student_df.loc[:,mask_BY]
df_F1 = student_df.loc[:,mask_F1]
df_F2 = student_df.loc[:,mask_F2]
df_F3 = student_df.loc[:,mask_F3]
```


```python
#Store pickled versions of the dataframes locally so I don't have to redo this every time.
df_BY.to_pickle(filepath+'df_BY.pkl')
df_F1.to_pickle(filepath+'df_F1.pkl')
df_F2.to_pickle(filepath+'df_F2.pkl')
df_F3.to_pickle(filepath+'df_F3.pkl')
labels.to_pickle(filepath+'labels.pkl')
df_weights.to_pickle(filepath+'df_weights.pkl')
```


```python
import pickle
f = open(filepath+'number_labels.pkl','wb')
pickle.dump(number_labels,f)
f.close()
```


```python

```


```python

```


```python

```


```python

```


```python

```
