
# JSON examples and exercise
****
+ get familiar with packages for dealing with JSON
+ study examples with JSON strings and files 
+ work on exercise to be completed and submitted 
****
+ reference: http://pandas.pydata.org/pandas-docs/stable/io.html#io-json-reader
+ data source: http://jsonstudio.com/resources/
****


```python
import pandas as pd
import numpy as np
```

## imports for Python, Pandas


```python
import json
from pandas.io.json import json_normalize
```

## JSON example, with string

+ demonstrates creation of normalized dataframes (tables) from nested json string
+ source: http://pandas.pydata.org/pandas-docs/stable/io.html#normalization


```python
# define json string
data = [{'state': 'Florida', 
         'shortname': 'FL',
         'info': {'governor': 'Rick Scott'},
         'counties': [{'name': 'Dade', 'population': 12345},
                      {'name': 'Broward', 'population': 40000},
                      {'name': 'Palm Beach', 'population': 60000}]},
        {'state': 'Ohio',
         'shortname': 'OH',
         'info': {'governor': 'John Kasich'},
         'counties': [{'name': 'Summit', 'population': 1234},
                      {'name': 'Cuyahoga', 'population': 1337}]}]
```


```python
# use normalization to create tables from nested element
json_normalize(data, 'counties')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dade</td>
      <td>12345</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Broward</td>
      <td>40000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Palm Beach</td>
      <td>60000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Summit</td>
      <td>1234</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Cuyahoga</td>
      <td>1337</td>
    </tr>
  </tbody>
</table>
</div>




```python
# further populate tables created from nested element
json_normalize(data, 'counties', ['state', 'shortname', ['info', 'governor']])
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>population</th>
      <th>state</th>
      <th>shortname</th>
      <th>info.governor</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dade</td>
      <td>12345</td>
      <td>Florida</td>
      <td>FL</td>
      <td>Rick Scott</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Broward</td>
      <td>40000</td>
      <td>Florida</td>
      <td>FL</td>
      <td>Rick Scott</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Palm Beach</td>
      <td>60000</td>
      <td>Florida</td>
      <td>FL</td>
      <td>Rick Scott</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Summit</td>
      <td>1234</td>
      <td>Ohio</td>
      <td>OH</td>
      <td>John Kasich</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Cuyahoga</td>
      <td>1337</td>
      <td>Ohio</td>
      <td>OH</td>
      <td>John Kasich</td>
    </tr>
  </tbody>
</table>
</div>



****
## JSON example, with file

+ demonstrates reading in a json file as a string and as a table
+ uses small sample file containing data about projects funded by the World Bank 
+ data source: http://jsonstudio.com/resources/


```python
# load json as string
json.load((open('data/world_bank_projects_less.json')))
```




    [{'_id': {'$oid': '52b213b38594d8a2be17c780'},
      'approvalfy': 1999,
      'board_approval_month': 'November',
      'boardapprovaldate': '2013-11-12T00:00:00Z',
      'borrower': 'FEDERAL DEMOCRATIC REPUBLIC OF ETHIOPIA',
      'closingdate': '2018-07-07T00:00:00Z',
      'country_namecode': 'Federal Democratic Republic of Ethiopia!$!ET',
      'countrycode': 'ET',
      'countryname': 'Federal Democratic Republic of Ethiopia',
      'countryshortname': 'Ethiopia',
      'docty': 'Project Information Document,Indigenous Peoples Plan,Project Information Document',
      'envassesmentcategorycode': 'C',
      'grantamt': 0,
      'ibrdcommamt': 0,
      'id': 'P129828',
      'idacommamt': 130000000,
      'impagency': 'MINISTRY OF EDUCATION',
      'lendinginstr': 'Investment Project Financing',
      'lendinginstrtype': 'IN',
      'lendprojectcost': 550000000,
      'majorsector_percent': [{'Name': 'Education', 'Percent': 46},
       {'Name': 'Education', 'Percent': 26},
       {'Name': 'Public Administration, Law, and Justice', 'Percent': 16},
       {'Name': 'Education', 'Percent': 12}],
      'mjsector_namecode': [{'name': 'Education', 'code': 'EX'},
       {'name': 'Education', 'code': 'EX'},
       {'name': 'Public Administration, Law, and Justice', 'code': 'BX'},
       {'name': 'Education', 'code': 'EX'}],
      'mjtheme': ['Human development'],
      'mjtheme_namecode': [{'name': 'Human development', 'code': '8'},
       {'name': '', 'code': '11'}],
      'mjthemecode': '8,11',
      'prodline': 'PE',
      'prodlinetext': 'IBRD/IDA',
      'productlinetype': 'L',
      'project_abstract': {'cdata': 'The development objective of the Second Phase of General Education Quality Improvement Project for Ethiopia is to improve learning conditions in primary and secondary schools and strengthen institutions at different levels of educational administration. The project has six components. The first component is curriculum, textbooks, assessment, examinations, and inspection. This component will support improvement of learning conditions in grades KG-12 by providing increased access to teaching and learning materials and through improvements to the curriculum by assessing the strengths and weaknesses of the current curriculum. This component has following four sub-components: (i) curriculum reform and implementation; (ii) teaching and learning materials; (iii) assessment and examinations; and (iv) inspection. The second component is teacher development program (TDP). This component will support improvements in learning conditions in both primary and secondary schools by advancing the quality of teaching in general education through: (a) enhancing the training of pre-service teachers in teacher education institutions; and (b) improving the quality of in-service teacher training. This component has following three sub-components: (i) pre-service teacher training; (ii) in-service teacher training; and (iii) licensing and relicensing of teachers and school leaders. The third component is school improvement plan. This component will support the strengthening of school planning in order to improve learning outcomes, and to partly fund the school improvement plans through school grants. It has following two sub-components: (i) school improvement plan; and (ii) school grants. The fourth component is management and capacity building, including education management information systems (EMIS). This component will support management and capacity building aspect of the project. This component has following three sub-components: (i) capacity building for education planning and management; (ii) capacity building for school planning and management; and (iii) EMIS. The fifth component is improving the quality of learning and teaching in secondary schools and universities through the use of information and communications technology (ICT). It has following five sub-components: (i) national policy and institution for ICT in general education; (ii) national ICT infrastructure improvement plan for general education; (iii) develop an integrated monitoring, evaluation, and learning system specifically for the ICT component; (iv) teacher professional development in the use of ICT; and (v) provision of limited number of e-Braille display readers with the possibility to scale up to all secondary education schools based on the successful implementation and usage of the readers. The sixth component is program coordination, monitoring and evaluation, and communication. It will support institutional strengthening by developing capacities in all aspects of program coordination, monitoring and evaluation; a new sub-component on communications will support information sharing for better management and accountability. It has following three sub-components: (i) program coordination; (ii) monitoring and evaluation (M and E); and (iii) communication.'},
      'project_name': 'Ethiopia General Education Quality Improvement Project II',
      'projectdocs': [{'DocTypeDesc': 'Project Information Document (PID),  Vol.',
        'DocType': 'PID',
        'EntityID': '090224b081e545fb_1_0',
        'DocURL': 'http://www-wds.worldbank.org/servlet/WDSServlet?pcont=details&eid=090224b081e545fb_1_0',
        'DocDate': '28-AUG-2013'},
       {'DocTypeDesc': 'Indigenous Peoples Plan (IP),  Vol.1 of 1',
        'DocType': 'IP',
        'EntityID': '000442464_20130920111729',
        'DocURL': 'http://www-wds.worldbank.org/servlet/WDSServlet?pcont=details&eid=000442464_20130920111729',
        'DocDate': '01-JUL-2013'},
       {'DocTypeDesc': 'Project Information Document (PID),  Vol.',
        'DocType': 'PID',
        'EntityID': '090224b0817b19e2_1_0',
        'DocURL': 'http://www-wds.worldbank.org/servlet/WDSServlet?pcont=details&eid=090224b0817b19e2_1_0',
        'DocDate': '22-NOV-2012'}],
      'projectfinancialtype': 'IDA',
      'projectstatusdisplay': 'Active',
      'regionname': 'Africa',
      'sector': [{'Name': 'Primary education'},
       {'Name': 'Secondary education'},
       {'Name': 'Public administration- Other social services'},
       {'Name': 'Tertiary education'}],
      'sector1': {'Name': 'Primary education', 'Percent': 46},
      'sector2': {'Name': 'Secondary education', 'Percent': 26},
      'sector3': {'Name': 'Public administration- Other social services',
       'Percent': 16},
      'sector4': {'Name': 'Tertiary education', 'Percent': 12},
      'sector_namecode': [{'name': 'Primary education', 'code': 'EP'},
       {'name': 'Secondary education', 'code': 'ES'},
       {'name': 'Public administration- Other social services', 'code': 'BS'},
       {'name': 'Tertiary education', 'code': 'ET'}],
      'sectorcode': 'ET,BS,ES,EP',
      'source': 'IBRD',
      'status': 'Active',
      'supplementprojectflg': 'N',
      'theme1': {'Name': 'Education for all', 'Percent': 100},
      'theme_namecode': [{'name': 'Education for all', 'code': '65'}],
      'themecode': '65',
      'totalamt': 130000000,
      'totalcommamt': 130000000,
      'url': 'http://www.worldbank.org/projects/P129828/ethiopia-general-education-quality-improvement-project-ii?lang=en'},
     {'_id': {'$oid': '52b213b38594d8a2be17c781'},
      'approvalfy': 2015,
      'board_approval_month': 'November',
      'boardapprovaldate': '2013-11-04T00:00:00Z',
      'borrower': 'GOVERNMENT OF TUNISIA',
      'country_namecode': 'Republic of Tunisia!$!TN',
      'countrycode': 'TN',
      'countryname': 'Republic of Tunisia',
      'countryshortname': 'Tunisia',
      'docty': 'Project Information Document,Integrated Safeguards Data Sheet,Integrated Safeguards Data Sheet,Project Information Document,Integrated Safeguards Data Sheet,Project Information Document',
      'envassesmentcategorycode': 'C',
      'grantamt': 4700000,
      'ibrdcommamt': 0,
      'id': 'P144674',
      'idacommamt': 0,
      'impagency': 'MINISTRY OF FINANCE',
      'lendinginstr': 'Specific Investment Loan',
      'lendinginstrtype': 'IN',
      'lendprojectcost': 5700000,
      'majorsector_percent': [{'Name': 'Public Administration, Law, and Justice',
        'Percent': 70},
       {'Name': 'Public Administration, Law, and Justice', 'Percent': 30}],
      'mjsector_namecode': [{'name': 'Public Administration, Law, and Justice',
        'code': 'BX'},
       {'name': 'Public Administration, Law, and Justice', 'code': 'BX'}],
      'mjtheme': ['Economic management', 'Social protection and risk management'],
      'mjtheme_namecode': [{'name': 'Economic management', 'code': '1'},
       {'name': 'Social protection and risk management', 'code': '6'}],
      'mjthemecode': '1,6',
      'prodline': 'RE',
      'prodlinetext': 'Recipient Executed Activities',
      'productlinetype': 'L',
      'project_name': 'TN: DTF Social Protection Reforms Support',
      'projectdocs': [{'DocTypeDesc': 'Project Information Document (PID),  Vol.1 of 1',
        'DocType': 'PID',
        'EntityID': '000333037_20131024115616',
        'DocURL': 'http://www-wds.worldbank.org/servlet/WDSServlet?pcont=details&eid=000333037_20131024115616',
        'DocDate': '29-MAR-2013'},
       {'DocTypeDesc': 'Integrated Safeguards Data Sheet (ISDS),  Vol.1 of 1',
        'DocType': 'ISDS',
        'EntityID': '000356161_20131024151611',
        'DocURL': 'http://www-wds.worldbank.org/servlet/WDSServlet?pcont=details&eid=000356161_20131024151611',
        'DocDate': '29-MAR-2013'},
       {'DocTypeDesc': 'Integrated Safeguards Data Sheet (ISDS),  Vol.1 of 1',
        'DocType': 'ISDS',
        'EntityID': '000442464_20131031112136',
        'DocURL': 'http://www-wds.worldbank.org/servlet/WDSServlet?pcont=details&eid=000442464_20131031112136',
        'DocDate': '29-MAR-2013'},
       {'DocTypeDesc': 'Project Information Document (PID),  Vol.1 of 1',
        'DocType': 'PID',
        'EntityID': '000333037_20131031105716',
        'DocURL': 'http://www-wds.worldbank.org/servlet/WDSServlet?pcont=details&eid=000333037_20131031105716',
        'DocDate': '29-MAR-2013'},
       {'DocTypeDesc': 'Integrated Safeguards Data Sheet (ISDS),  Vol.1 of 1',
        'DocType': 'ISDS',
        'EntityID': '000356161_20130305113209',
        'DocURL': 'http://www-wds.worldbank.org/servlet/WDSServlet?pcont=details&eid=000356161_20130305113209',
        'DocDate': '16-JAN-2013'},
       {'DocTypeDesc': 'Project Information Document (PID),  Vol.1 of 1',
        'DocType': 'PID',
        'EntityID': '000356161_20130305113716',
        'DocURL': 'http://www-wds.worldbank.org/servlet/WDSServlet?pcont=details&eid=000356161_20130305113716',
        'DocDate': '16-JAN-2013'}],
      'projectfinancialtype': 'OTHER',
      'projectstatusdisplay': 'Active',
      'regionname': 'Middle East and North Africa',
      'sector': [{'Name': 'Public administration- Other social services'},
       {'Name': 'General public administration sector'}],
      'sector1': {'Name': 'Public administration- Other social services',
       'Percent': 70},
      'sector2': {'Name': 'General public administration sector', 'Percent': 30},
      'sector_namecode': [{'name': 'Public administration- Other social services',
        'code': 'BS'},
       {'name': 'General public administration sector', 'code': 'BZ'}],
      'sectorcode': 'BZ,BS',
      'source': 'IBRD',
      'status': 'Active',
      'supplementprojectflg': 'N',
      'theme1': {'Name': 'Other economic management', 'Percent': 30},
      'theme_namecode': [{'name': 'Other economic management', 'code': '24'},
       {'name': 'Social safety nets', 'code': '54'}],
      'themecode': '54,24',
      'totalamt': 0,
      'totalcommamt': 4700000,
      'url': 'http://www.worldbank.org/projects/P144674?lang=en'}]




```python
# load as Pandas dataframe
sample_json_df = pd.read_json('data/world_bank_projects_less.json')
sample_json_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>_id</th>
      <th>approvalfy</th>
      <th>board_approval_month</th>
      <th>boardapprovaldate</th>
      <th>borrower</th>
      <th>closingdate</th>
      <th>country_namecode</th>
      <th>countrycode</th>
      <th>countryname</th>
      <th>countryshortname</th>
      <th>...</th>
      <th>sectorcode</th>
      <th>source</th>
      <th>status</th>
      <th>supplementprojectflg</th>
      <th>theme1</th>
      <th>theme_namecode</th>
      <th>themecode</th>
      <th>totalamt</th>
      <th>totalcommamt</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>{'$oid': '52b213b38594d8a2be17c780'}</td>
      <td>1999</td>
      <td>November</td>
      <td>2013-11-12T00:00:00Z</td>
      <td>FEDERAL DEMOCRATIC REPUBLIC OF ETHIOPIA</td>
      <td>2018-07-07T00:00:00Z</td>
      <td>Federal Democratic Republic of Ethiopia!$!ET</td>
      <td>ET</td>
      <td>Federal Democratic Republic of Ethiopia</td>
      <td>Ethiopia</td>
      <td>...</td>
      <td>ET,BS,ES,EP</td>
      <td>IBRD</td>
      <td>Active</td>
      <td>N</td>
      <td>{'Name': 'Education for all', 'Percent': 100}</td>
      <td>[{'name': 'Education for all', 'code': '65'}]</td>
      <td>65</td>
      <td>130000000</td>
      <td>130000000</td>
      <td>http://www.worldbank.org/projects/P129828/ethi...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>{'$oid': '52b213b38594d8a2be17c781'}</td>
      <td>2015</td>
      <td>November</td>
      <td>2013-11-04T00:00:00Z</td>
      <td>GOVERNMENT OF TUNISIA</td>
      <td>NaN</td>
      <td>Republic of Tunisia!$!TN</td>
      <td>TN</td>
      <td>Republic of Tunisia</td>
      <td>Tunisia</td>
      <td>...</td>
      <td>BZ,BS</td>
      <td>IBRD</td>
      <td>Active</td>
      <td>N</td>
      <td>{'Name': 'Other economic management', 'Percent...</td>
      <td>[{'name': 'Other economic management', 'code':...</td>
      <td>54,24</td>
      <td>0</td>
      <td>4700000</td>
      <td>http://www.worldbank.org/projects/P144674?lang=en</td>
    </tr>
  </tbody>
</table>
<p>2 rows × 50 columns</p>
</div>



****
## JSON exercise

Using data in file 'data/world_bank_projects.json' and the techniques demonstrated above,
1. Find the 10 countries with most projects
2. Find the top 10 major project themes (using column 'mjtheme_namecode')
3. In 2. above you will notice that some entries have only the code and the name is missing. Create a dataframe with the missing names filled in.


```python
#Load in the JSON file as a string
with open('data/world_bank_projects.json') as json_file:
    json_string = json.load(json_file)
    
#Create data frame from nested 'mjtheme_namecode' column with json_normalize
df = json_normalize(json_string,'mjtheme_namecode',
                    ['countryshortname','status'])

#Rename columns to something more memorable
df.columns = ['project_code', 
              'project_theme', 
              'country', 
              'project_status']

#Do some cleaning
mask = (df['project_status'] == 'Active')
df_active = df[mask]
df_active.loc[:,'project_code'] = pd.to_numeric(df_active['project_code'])
df_active = df_active.replace(to_replace = '',value= np.nan)

#Group the data frame by country and count the items in the project_status column
project_counts = df_active.groupby('country')['project_status'].count()

#Sort the project status series and print out the top 10 counts
top10_projects = project_counts.sort_values(ascending=False).head(10)

print(top10_projects)
```

    country
    India                 47
    Indonesia             43
    Bangladesh            41
    China                 40
    Africa                39
    Yemen, Republic of    34
    Vietnam               32
    Nepal                 28
    Brazil                27
    Pakistan              25
    Name: project_status, dtype: int64



```python
#Fill the na values by forward filling, after sorting to insure proper filling
df_active = df_active.sort_values(by = ['project_code','project_theme'])
df_active = df_active.fillna(method='ffill')

```


```python
#Group the data frame by project_theme and count the project_theme column
theme_counts = df_active.groupby('project_theme')['project_theme'].count().sort_values()

#Sort the project theme series and print out the top ten values
top10_themes = theme_counts.sort_values(ascending=False).head(10)
print(top10_themes)

```

    project_theme
    Environment and natural resources management    236
    Rural development                               199
    Human development                               184
    Social protection and risk management           137
    Public sector governance                        136
    Social dev/gender/inclusion                     117
    Financial and private sector development        109
    Trade and integration                            62
    Urban development                                44
    Economic management                              24
    Name: project_theme, dtype: int64



```python
#Validate filling by grouping by project code. The exact same series should be returned
code_counts = df_active.groupby('project_code')['project_code'].count()
assert ((theme_counts.sort_values().values == code_counts.sort_values().values).all())
```


```python

```
