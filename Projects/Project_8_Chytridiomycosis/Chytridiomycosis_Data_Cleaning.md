

```python
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline
```

# Data Gathering


```python
df = pd.read_csv('./data/Chytridiomycosis_cleaned.csv')
```

Create species list to look up conservations status in Wikipedia:


```python
pd.DataFrame({'Species': df_clean.Species.unique()}).to_csv('./data/species_list.csv',index=False)
```

Read resulting dataframe with conservation status filled in: 


```python
species_status = pd.read_csv('./data/species.csv')

species_status.Conservation_Status.value_counts()
```




    Least Concern             81
    Critically Endangered      9
    Endangered                 8
    Vulnerable                 5
    Near Threatened            4
    Name: Conservation_Status, dtype: int64



Merge two datsets together 


```python
df = df.merge(species_status, how="left", left_on="Species", right_on="Species")
```

## Overview


```python
df_clean = df.copy()
```


```python
df.columns
```




    Index(['Compiled_by', 'Database_ID', 'Species', 'Sex', 'Site', 'State',
           'Country', 'Year', 'Diagnostic', 'Individuals', 'Indivs_positive',
           'Collector_source', 'Orig_database', 'Disease_status', 'Accuracy',
           'Latitude', 'Longitude', 'Dead_or_sick', 'Conservation_Status',
           'Conservation_Status_Code'],
          dtype='object')




```python
df.describe()
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Database_ID</th>
      <th>Year</th>
      <th>Individuals</th>
      <th>Indivs_positive</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>10183.00000</td>
      <td>9652.000000</td>
      <td>9854.000000</td>
      <td>9917.000000</td>
      <td>8877.000000</td>
      <td>8877.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>5092.00000</td>
      <td>1999.855988</td>
      <td>1.312969</td>
      <td>0.196632</td>
      <td>-23.299867</td>
      <td>142.291619</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2939.72323</td>
      <td>5.324939</td>
      <td>4.242507</td>
      <td>1.202163</td>
      <td>7.490207</td>
      <td>11.993697</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.00000</td>
      <td>1956.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>-43.606230</td>
      <td>114.366670</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2546.50000</td>
      <td>1998.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>-28.574440</td>
      <td>145.210000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>5092.00000</td>
      <td>2000.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>-19.951390</td>
      <td>145.665560</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7637.50000</td>
      <td>2004.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>-17.425560</td>
      <td>148.633060</td>
    </tr>
    <tr>
      <th>max</th>
      <td>10183.00000</td>
      <td>2007.000000</td>
      <td>319.000000</td>
      <td>77.000000</td>
      <td>-13.725830</td>
      <td>153.536980</td>
    </tr>
  </tbody>
</table>



```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 10183 entries, 0 to 10182
    Data columns (total 20 columns):
    Compiled_by                 10183 non-null object
    Database_ID                 10183 non-null int64
    Species                     10183 non-null object
    Sex                         10183 non-null object
    Site                        10183 non-null object
    State                       10183 non-null object
    Country                     10183 non-null object
    Year                        9652 non-null float64
    Diagnostic                  10183 non-null object
    Individuals                 9854 non-null float64
    Indivs_positive             9917 non-null float64
    Collector_source            10183 non-null object
    Orig_database               10183 non-null object
    Disease_status              10183 non-null object
    Accuracy                    10183 non-null object
    Latitude                    8877 non-null float64
    Longitude                   8877 non-null float64
    Dead_or_sick                10183 non-null object
    Conservation_Status         9728 non-null object
    Conservation_Status_Code    9728 non-null object
    dtypes: float64(5), int64(1), object(14)
    memory usage: 1.6+ MB



```python
df.isnull().sum(axis=0)
```




    Compiled_by                    0
    Database_ID                    0
    Species                        0
    Sex                            0
    Site                           0
    State                          0
    Country                        0
    Year                         531
    Diagnostic                     0
    Individuals                  329
    Indivs_positive              266
    Collector_source               0
    Orig_database                  0
    Disease_status                 0
    Accuracy                       0
    Latitude                    1306
    Longitude                   1306
    Dead_or_sick                   0
    Conservation_Status          455
    Conservation_Status_Code     455
    dtype: int64



## 1: Missing year

### Assessment


```python
df[df.Year.isnull()].isnull().sum(axis=0)
```




    Compiled_by                   0
    Database_ID                   0
    Species                       0
    Sex                           0
    Site                          0
    State                         0
    Country                       0
    Year                        531
    Diagnostic                    0
    Individuals                 206
    Indivs_positive             206
    Collector_source              0
    Orig_database                 0
    Disease_status                0
    Accuracy                      0
    Latitude                    382
    Longitude                   382
    Dead_or_sick                  0
    Conservation_Status         267
    Conservation_Status_Code    267
    dtype: int64




```python
df[df.Year.isnull()].Collector_source.value_counts().head()
```




    K. Aplin          369
    D. Driscoll        47
    various            28
    H. Hines           19
    R. Puschendorf     10
    Name: Collector_source, dtype: int64




```python
df[df.Year.isnull()].Accuracy.value_counts()
```




    unacceptable    476
    acceptable       55
    Name: Accuracy, dtype: int64




```python
df[df.Year.isnull()].Orig_database.value_counts()
```




    K. Murray (K. Aplin sorted database)    445
    Mapping Samples - Aug 05.xls             28
    L. Berger                                19
    D. Mendez                                18
    K. Murray                                10
    D. Mendez / R. Speare                     8
    R. Speare / L. Berger                     2
    K. McDonald                               1
    Name: Orig_database, dtype: int64




```python
df[df.Year.isnull()].Latitude.isnull().sum()
```




    382




```python
df[df.Year.isnull()].Accuracy.value_counts()
```




    unacceptable    476
    acceptable       55
    Name: Accuracy, dtype: int64




```python
df[df.Year.isnull()].Latitude.isnull().sum()
```




    382




```python
df[df.Year.isnull()].Site.value_counts().head()
```




    missing data                   317
    3.7Km E Bullant Dve, Bussel     47
    Airport Perth                   19
    Talbot Rd, Perth                 9
    Tims Thicket                     9
    Name: Site, dtype: int64




```python
df[df.Year.isnull()].State.value_counts()
```




    WA              446
    QLD              49
    missing data     21
    NSW              13
    VIC               1
    SA                1
    Name: State, dtype: int64



Conclusion: 

Most observations without year are from K. Murray (K. Aplin sorted database) source. Also, most of them (445/531) have unacceptable accuracy of geographical location (382 out of 531 locations are simply null). Completely removing them looks like a reasonable solution in this case. 

### Cleaning

#### Define

Remove observations with missing year.

#### Code


```python
df_clean = df_clean[df_clean.Year.notnull()]
```

#### Test


```python
df_clean.Year.isnull().sum()
```




    0



#### Define
Convert Year to integer (now possible due to absence of missing values)

#### Code


```python
df_clean['Year'] = df_clean.Year.astype('int')
```

#### Test


```python
df_clean.Year.head()
```




    0    1997
    1    1997
    2    1997
    3    1997
    4    1997
    Name: Year, dtype: int64



## 2: Null values reported as 'missing data'


```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 10183 entries, 0 to 10182
    Data columns (total 20 columns):
    Compiled_by                 10183 non-null object
    Database_ID                 10183 non-null int64
    Species                     10183 non-null object
    Sex                         10183 non-null object
    Site                        10183 non-null object
    State                       10183 non-null object
    Country                     10183 non-null object
    Year                        9652 non-null float64
    Diagnostic                  10183 non-null object
    Individuals                 9854 non-null float64
    Indivs_positive             9917 non-null float64
    Collector_source            10183 non-null object
    Orig_database               10183 non-null object
    Disease_status              10183 non-null object
    Accuracy                    10183 non-null object
    Latitude                    8877 non-null float64
    Longitude                   8877 non-null float64
    Dead_or_sick                10183 non-null object
    Conservation_Status         9728 non-null object
    Conservation_Status_Code    9728 non-null object
    dtypes: float64(5), int64(1), object(14)
    memory usage: 1.6+ MB



```python
df_clean.Site.str.contains('missing').sum()
```




    23




```python
df_clean.State.str.contains('missing').sum()
```




    14




```python
df_clean.Diagnostic.str.contains('missing').sum()
```




    0




```python
df_clean.Collector_source.str.contains('missing').sum()
```




    0




```python
df_clean.Orig_database.str.contains('missing').sum()
```




    0




```python
df_clean[df_clean.Site.str.contains('missing')].head()
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compiled_by</th>
      <th>Database_ID</th>
      <th>Species</th>
      <th>Sex</th>
      <th>Site</th>
      <th>State</th>
      <th>Country</th>
      <th>Year</th>
      <th>Diagnostic</th>
      <th>Individuals</th>
      <th>Indivs_positive</th>
      <th>Collector_source</th>
      <th>Orig_database</th>
      <th>Disease_status</th>
      <th>Accuracy</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Dead_or_sick</th>
      <th>Conservation_Status</th>
      <th>Conservation_Status_Code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2390</th>
      <td>RichardRetallick</td>
      <td>2391</td>
      <td>Limnodynastes peronii</td>
      <td>not recorded</td>
      <td>missing data</td>
      <td>missing data</td>
      <td>Australia</td>
      <td>1999</td>
      <td>Histology</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>A. Beezley / P. Couper</td>
      <td>L. Berger</td>
      <td>no result</td>
      <td>unacceptable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>not noted</td>
      <td>Least Concern</td>
      <td>LC</td>
    </tr>
    <tr>
      <th>2492</th>
      <td>RichardRetallick</td>
      <td>2493</td>
      <td>Litoria fallax</td>
      <td>Female</td>
      <td>missing data</td>
      <td>missing data</td>
      <td>Australia</td>
      <td>2000</td>
      <td>Histology</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>H. Hines</td>
      <td>L. Berger</td>
      <td>negative</td>
      <td>unacceptable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>not noted</td>
      <td>Least Concern</td>
      <td>LC</td>
    </tr>
    <tr>
      <th>2558</th>
      <td>RichardRetallick</td>
      <td>2559</td>
      <td>Limnodynastes peronii</td>
      <td>Male</td>
      <td>missing data</td>
      <td>missing data</td>
      <td>Australia</td>
      <td>1999</td>
      <td>Histology</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>A. Beezley / P. Couper</td>
      <td>L. Berger</td>
      <td>negative</td>
      <td>unacceptable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>not noted</td>
      <td>Least Concern</td>
      <td>LC</td>
    </tr>
    <tr>
      <th>2563</th>
      <td>RichardRetallick</td>
      <td>2564</td>
      <td>Litoria caerulea</td>
      <td>Male</td>
      <td>missing data</td>
      <td>missing data</td>
      <td>Australia</td>
      <td>1998</td>
      <td>Histology</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>C. Taylor SC Uni</td>
      <td>L. Berger</td>
      <td>negative</td>
      <td>unacceptable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>not noted</td>
      <td>Least Concern</td>
      <td>LC</td>
    </tr>
    <tr>
      <th>2585</th>
      <td>RichardRetallick</td>
      <td>2586</td>
      <td>Litoria caerulea</td>
      <td>Male</td>
      <td>missing data</td>
      <td>missing data</td>
      <td>Australia</td>
      <td>1996</td>
      <td>Histology</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>H. Hines</td>
      <td>L. Berger</td>
      <td>positive</td>
      <td>unacceptable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>not noted</td>
      <td>Least Concern</td>
      <td>LC</td>
    </tr>
  </tbody>
</table>



Although we don't have accurate geographical data about these records, it still wrong to remove them, because at least they add information to the temporal dimension - we still have the number of infected species by year. So in this case we should rather convert 'missing data' to None type.

#### Define
Convert 'missing data' to `None` in `Site` and `State` columns

#### Code 


```python
df_clean.loc[df_clean.Site=='missing data','Site'] = None
```


```python
df_clean.loc[df_clean.State=='missing data','State'] = None
```

#### Test


```python
df_clean[df_clean.Site.isnull()].head()
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compiled_by</th>
      <th>Database_ID</th>
      <th>Species</th>
      <th>Sex</th>
      <th>Site</th>
      <th>State</th>
      <th>Country</th>
      <th>Year</th>
      <th>Diagnostic</th>
      <th>Individuals</th>
      <th>Indivs_positive</th>
      <th>Collector_source</th>
      <th>Orig_database</th>
      <th>Disease_status</th>
      <th>Accuracy</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Dead_or_sick</th>
      <th>Conservation_Status</th>
      <th>Conservation_Status_Code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2390</th>
      <td>RichardRetallick</td>
      <td>2391</td>
      <td>Limnodynastes peronii</td>
      <td>not recorded</td>
      <td>None</td>
      <td>None</td>
      <td>Australia</td>
      <td>1999</td>
      <td>Histology</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>A. Beezley / P. Couper</td>
      <td>L. Berger</td>
      <td>no result</td>
      <td>unacceptable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>not noted</td>
      <td>Least Concern</td>
      <td>LC</td>
    </tr>
    <tr>
      <th>2492</th>
      <td>RichardRetallick</td>
      <td>2493</td>
      <td>Litoria fallax</td>
      <td>Female</td>
      <td>None</td>
      <td>None</td>
      <td>Australia</td>
      <td>2000</td>
      <td>Histology</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>H. Hines</td>
      <td>L. Berger</td>
      <td>negative</td>
      <td>unacceptable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>not noted</td>
      <td>Least Concern</td>
      <td>LC</td>
    </tr>
    <tr>
      <th>2558</th>
      <td>RichardRetallick</td>
      <td>2559</td>
      <td>Limnodynastes peronii</td>
      <td>Male</td>
      <td>None</td>
      <td>None</td>
      <td>Australia</td>
      <td>1999</td>
      <td>Histology</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>A. Beezley / P. Couper</td>
      <td>L. Berger</td>
      <td>negative</td>
      <td>unacceptable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>not noted</td>
      <td>Least Concern</td>
      <td>LC</td>
    </tr>
    <tr>
      <th>2563</th>
      <td>RichardRetallick</td>
      <td>2564</td>
      <td>Litoria caerulea</td>
      <td>Male</td>
      <td>None</td>
      <td>None</td>
      <td>Australia</td>
      <td>1998</td>
      <td>Histology</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>C. Taylor SC Uni</td>
      <td>L. Berger</td>
      <td>negative</td>
      <td>unacceptable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>not noted</td>
      <td>Least Concern</td>
      <td>LC</td>
    </tr>
    <tr>
      <th>2585</th>
      <td>RichardRetallick</td>
      <td>2586</td>
      <td>Litoria caerulea</td>
      <td>Male</td>
      <td>None</td>
      <td>None</td>
      <td>Australia</td>
      <td>1996</td>
      <td>Histology</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>H. Hines</td>
      <td>L. Berger</td>
      <td>positive</td>
      <td>unacceptable</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>not noted</td>
      <td>Least Concern</td>
      <td>LC</td>
    </tr>
  </tbody>
</table>



## 3: Missing geographical data

### Assessment


```python
no_geo = (df_clean.Latitude.isnull() & df_clean.Longitude.isnull())
no_geo.sum()
```




    924




```python
df_clean[no_geo].Accuracy.value_counts()
```




    unacceptable    924
    Name: Accuracy, dtype: int64




```python
df_clean[no_geo].State.value_counts()
```




    WA      842
    QLD      38
    NSW      23
    nsw?      5
    TAS       2
    Name: State, dtype: int64



Conclusion

As we know from dataset description, somethimes it was impossible to establish where the species come from. However, it doesn't make thes observations invaluable. We still want them in the dataset (Note: unacceptable accuracy refers to longitude and latitude accuracy).  

Leave as is.

## 4: Invalid state names

### Assesment


```python
df_clean.State.value_counts()
```




    QLD     6648
    WA      2201
    NSW      481
    TAS      146
    ACT       77
    SA        41
    VIC       25
    NT        14
    nsw?       5
    Name: State, dtype: int64



Conclusion: for the purposes of our story we can accept some innacuracy (but note it here in our analysis, so that the user can refer to it.

### Cleaning

#### Define  
Replace 'nsw?' with 'NSW' in `State` column.


```python
df_clean.loc[df_clean.State=='nsw?','State'] = 'NSW'
```

#### Test


```python
df_clean.State.value_counts()
```




    QLD    6648
    WA     2201
    NSW     486
    TAS     146
    ACT      77
    SA       41
    VIC      25
    NT       14
    Name: State, dtype: int64



## 5: Multiple variable in one column: species stage and species sex

### Assessment


```python
df_clean.Sex.value_counts()
```




    not recorded         6197
    Male                 2044
    Female                725
    Subadult              227
    Juvenile              190
    Tadpole                99
    Metamorph              66
    Juvenile ?             35
    Juvenile female        22
    Juvenile female ?      21
    Subadult male          10
    Juvenile male  ?        3
    Male ?                  3
    Metamorph male          3
    Female ?                2
    Juvenile male           2
    Subadult female         2
    Juv                     1
    Name: Sex, dtype: int64



Conclusion: 
As we can see, in most cases the sex is not reported, but we still can get some valuable insights from about 3000+ observations with identified gender. As in the case with the `State` we can accept some uncertainty and remove the questions marks. However, we also an additional column here: 
- `stage` (one of: Adult, Subadult, Juvenile, Tadpole)

For `Sex` column we will reduce the number of values to 4: 
- Female 
- Male 
- Metamorph 
- not recorded

Note: frogs can change gender to due to environmental and even social changes, that's why we have metamorph here. 

### Cleaning

#### Define
Set `Sex` for all values containing 'female' as 'Female', 'male' -> 'Male', 'metamorph' -> 'metamorph'. Set all the remaining values to 'not recorded'. Create `Stage` column and set values to 'juvenie' for all rows, containing 'juvenile' or 'juv', to 'subadult' for all row, containing 'subadult', 'tadpole' for all rows containing 'tadpole', 'adult' for all rows that are not labeled as 'not recored' and are not in the categories mentioned above. 

#### Code


```python
is_tadpole = (df_clean.Sex.str.contains(r'Tadpole.*',case=False))
is_metamorph = (df_clean.Sex.str.contains(r'Metamorph.*',case=False))
is_juvenile = (df_clean.Sex.str.contains(r'juv.*',case=False))
is_subadult = (df_clean.Sex.str.contains(r'Subadult.*',case=False))
is_adult = (~is_juvenile)&(~is_subadult)&(~is_tadpole)&(~is_metamorph)&(~df_clean.Sex.str.contains('not recorded'))
```


```python
df_clean['Stage'] = None
df_clean.loc[is_tadpole, 'Stage'] = 'Tadpole'
df_clean.loc[is_metamorph, 'Stage'] = 'Metamorph'
df_clean.loc[is_juvenile, 'Stage'] = 'Juvenile'
df_clean.loc[is_subadult, 'Stage'] = 'Subadult'
df_clean.loc[is_adult, 'Stage'] = 'Adult'
```


```python
is_female = df_clean.Sex.str.contains(r'female.*', case=False)
is_male = df_clean.Sex.str.contains(r'male.*', case=False)&(~is_female)
not_recorded = (~is_female)&(~is_male)
```


```python
df_clean.loc[is_female,'Sex'] = 'Female'
df_clean.loc[is_male,'Sex'] = 'Male'
df_clean.loc[not_recorded,'Sex'] = 'not recorded'
```

#### Test


```python
df_clean.Stage.value_counts()
```




    Adult        2774
    Juvenile      274
    Subadult      239
    Tadpole        99
    Metamorph      69
    Name: Stage, dtype: int64




```python
df_clean.Sex.value_counts()
```




    not recorded    6815
    Male            2065
    Female           772
    Name: Sex, dtype: int64



## 6: Too narrow categories in `Species` column

### Assessment


```python
df_clean.Species.nunique()
```




    114




```python
df_clean.Species.unique()
```




    array(['Pseudophryne pengilleyi', 'Pseudophryne corroboree',
           'Mixophyes fleayi', 'Litoria pearsoniana', 'Rhinella marina',
           'Adelotus brevis', 'Litoria caerulea', 'Litoria wilcoxii',
           'Crinia parinsignifera', 'Limnodynastes peronii',
           'Litoria brevipalmata', 'Litoria rubella', 'Litoria brevipes',
           'Litoria fallax', 'Litoria chloris', 'Mixophyes iteratus',
           'Mixophyes fasciolatus', 'Philoria kundagungan', 'Litoria dentata',
           'Platyplectrum ornatum', 'Limnodynastes tasmaniensis',
           'Litoria latopalmata', 'Litoria alboguttata', 'Litoria peronii',
           'Neobatrachus sudelli', 'Limnodynastes terraereginae',
           'Litoria rothii', 'Litoria nasuta', 'Litoria pallida',
           'Litoria tornieri', 'sp. Crinia', 'Pseudophryne bibronii',
           'Litoria genimaculata', 'Litoria nannotis', 'Litoria rheocola',
           'Nyctimystes dayi', 'Litoria lesueuri sensu lato',
           'Hylarana daemeli', 'Mixophyes schevilli', 'Litoria infrafrenata',
           'unknown', 'Taudactylus rheophilus', 'Austrochaperina robusta',
           'Litoria andirrmalin', 'Litoria novaehollandiae',
           'Litoria inermis', 'Litoria gracilenta', 'Uperoleia mimula',
           'Crinia deserticola', 'Uperoleia altissima', 'Litoria jungguy',
           'Litoria eucnemis', 'Litoria longirostris', 'Litoria spenceri',
           'Crinia signifera', 'Taudactylus acutirostris',
           'Litoria raniformis', 'Litoria verreauxii', 'Litoria ewingii',
           'Limnodynastes dumerilii', 'Crinia riparia', 'Litoria citropa',
           'Litoria lesueuri', 'Lechriodus fletcheri',
           'Taudactylus eungellensis', 'Litoria aurea',
           'Heleioporus australiacus', 'Litoria serrata', 'Litoria moorei',
           'Crinia georgiana', 'sp. Mixophyes', 'Litoria xanthomera',
           'Litoria revelata', 'Taudactylus liemi', 'Litoria barringtonensis',
           'Litoria phyllochroa', 'Uperoleia laevigata', 'Litoria tyleri',
           'Geocrinia rosea', 'Heleioporus eyrei', 'Litoria adelaidensis',
           'Crinia glauerti', 'Crinia insignifera', 'Limnodynastes dorsalis',
           'Crinia pseudinsignifera', 'Taudactylus pleione',
           'Litoria bicolor', 'Litoria nigrofrenata', 'Litoria microbelos',
           'Limnodynastes convexiusculus', 'Litoria dahlii',
           'Litoria electrica', 'Litoria australis', 'Litoria Brevipes',
           'Geocrinia leai', 'Myobatrachus gouldii', 'Pseudophryne guentheri',
           'Neobatrachus kunapalari', 'Neobatrachus pelobatoides',
           'sp. Litoria', 'Ambystoma mexicanum', 'Heleioporus barycragus',
           'sp. Heleioporus', 'Heleioporus psammophilus',
           'Crinia subinsignifera', 'sp. Neobatrachus',
           'Spicospina flammocaerulea', 'Litoria coplandi', 'Geocrinia alba',
           'Geocrinia vitellina', 'Heleioporus albopunctatus',
           'Limnodynastes tasmaniensis ', 'Limnodynastes fletcheri',
           'Cophixalus ornatus'], dtype=object)




```python
(df_clean.Species=='unknown').sum()
```




    153




```python
df_clean[is_taudactylus].Species.unique()
```




    array(['Taudactylus rheophilus', 'Taudactylus acutirostris',
           'Taudactylus eungellensis', 'Taudactylus liemi',
           'Taudactylus pleione'], dtype=object)



### Cleaning

#### Define 

Group species into categories to aggreagate close species (e.g. various types of Litoria), mare rare species into group 'Other'. Create a separate column for that category.

#### Code


```python
is_litoria = df_clean.Species.str.contains(r'Litoria',case=False)
is_pseudophryne = df_clean.Species.str.contains(r'Pseudophryne',case=False)
is_crinia = df_clean.Species.str.contains(r'Crinia', case=False)
is_limnodynastes = df_clean.Species.str.contains(r'Limnodynastes', case=False)
is_neobatrachus = df_clean.Species.str.contains(r'Neobatrachus', case=False)
is_heleioporus = df_clean.Species.str.contains(r'Heleioporus', case=False)
is_mixophyes = df_clean.Species.str.contains(r'Mixophyes', case=False)
is_taudactylus = df_clean.Species.str.contains(r'Taudactylus', case=False)
is_uperoleia = df_clean.Species.str.contains(r'Uperoleia', case=False)
is_nyctimystes = df_clean.Species.str.contains(r'Nyctimystes', case=False)
is_hylarana_daemeli = df_clean.Species.str.contains(r'Hylarana daemeli',case=False)
other = ((~is_litoria)&(~is_pseudophryne)&(~is_crinia)&(~is_limnodynastes)&
         (~is_neobatrachus)&(~is_heleioporus)&(~is_mixophyes)&(~is_taudactylus)&
         (~is_uperoleia)&(~is_nyctimystes)&(~is_hylarana_daemeli))
print(is_litoria.sum())
print(is_pseudophryne.sum())
print(is_crinia.sum())
print(is_limnodynastes.sum())
print(is_neobatrachus.sum())
print(is_heleioporus.sum())
print(is_mixophyes.sum())
print(is_taudactylus.sum())
print(is_uperoleia.sum())
print(is_nyctimystes.sum())
print(is_hylarana_daemeli.sum())
print(other.sum())
```

    5485
    434
    970
    240
    29
    303
    1019
    398
    64
    351
    65
    294



```python
unclassified = df_clean[other]
unclassified.Species.value_counts()
```




    unknown                      153
    Cophixalus ornatus            37
    Rhinella marina               34
    Adelotus brevis               30
    Ambystoma mexicanum           11
    Platyplectrum ornatum         10
    Austrochaperina robusta        9
    Philoria kundagungan           4
    Myobatrachus gouldii           3
    Spicospina flammocaerulea      2
    Lechriodus fletcheri           1
    Name: Species, dtype: int64




```python
df_clean['Genus'] = None
df_clean.loc[is_litoria, 'Genus'] = 'Litoria'
df_clean.loc[is_pseudophryne, 'Genus'] = 'Pseudophryne'
df_clean.loc[is_crinia, 'Genus'] = 'Crinia'
df_clean.loc[is_limnodynastes, 'Genus'] = 'Limnodynastes'
df_clean.loc[is_limnodynastes, 'Genus'] = 'Limnodynastes'
df_clean.loc[is_neobatrachus, 'Genus'] = 'Neobatrachus'
df_clean.loc[is_heleioporus, 'Genus'] = 'Heleioporos'
df_clean.loc[is_mixophyes, 'Genus'] = 'Mixophyes'
df_clean.loc[is_taudactylus, 'Genus'] = 'Taudactylus'
df_clean.loc[is_uperoleia, 'Genus'] = 'Uperoleia'
df_clean.loc[is_nyctimystes, 'Genus'] = 'Nyctimystes'
df_clean.loc[is_hylarana_daemeli, 'Genus'] = 'Hylarana daemeli'
df_clean.loc[other, 'Genus'] = 'Other'
```


```python
df_clean.Genus.value_counts()
```




    Litoria             5485
    Mixophyes           1019
    Crinia               970
    Pseudophryne         434
    Taudactylus          398
    Nyctimystes          351
    Heleioporos          303
    Other                294
    Limnodynastes        240
    Hylarana daemeli      65
    Uperoleia             64
    Neobatrachus          29
    Name: Genus, dtype: int64




```python
df_clean.columns
```




    Index(['Compiled_by', 'Database_ID', 'Species', 'Sex', 'Site', 'State',
           'Country', 'Year', 'Diagnostic', 'Individuals', 'Indivs_positive',
           'Collector_source', 'Orig_database', 'Disease_status', 'Accuracy',
           'Latitude', 'Longitude', 'Dead_or_sick', 'Conservation_Status',
           'Conservation_Status_Code', 'Stage', 'Genus'],
          dtype='object')



#### Define
Add species family based on metadata: http://www.esapubs.org/archive/ecol/E091/108/metadata.htm

#### Code


```python
is_bufonidae = df_clean.Species.str.contains('Rhinella')
is_hylidae = (df_clean.Genus=='Litoria') | (df_clean.Species.str.contains('Nyctimystes'))
is_limnodynastidae = ((df_clean.Species.str.contains('Adelotus')) |
                     (df_clean.Species.str.contains('Heleioporus')) |
                      (df_clean.Species.str.contains('Lechriodus')) |
                      (df_clean.Species.str.contains('Limnodynastes') |
                      (df_clean.Species.str.contains('Neobatrachus')))
                     )
is_microhylidae = (df_clean.Species.str.contains('Cophixalus')|df_clean.Species.str.contains('Austrochaperina'))
is_myobatrachidae = ((df_clean.Species.str.contains('Assa')) |
                     (df_clean.Species.str.contains('Crinia')) |
                     (df_clean.Species.str.contains('Geocrinia')) |
                     (df_clean.Species.str.contains('Mixophyes')) |
                     (df_clean.Species.str.contains('Pseudophryne')) |
                     (df_clean.Species.str.contains('Taudactylus')) |
                     (df_clean.Species.str.contains('Uperoleia')) |
                     (df_clean.Species.str.contains('Philoria')) |
                     (df_clean.Species.str.contains('Platyplectrum')) |
                     (df_clean.Species.str.contains('Myobatrachus')) |
                     (df_clean.Species.str.contains('Spicospina'))
                    )
is_ambystomatidae = df_clean.Species.str.contains('Ambystoma')
is_ranidae = df_clean.Species.str.contains('Hylarana')                    
df_clean['Family'] = None
df_clean.loc[is_bufonidae, 'Family'] = 'Bufonidae'
df_clean.loc[is_hylidae, 'Family'] = 'Hylidae'
df_clean.loc[is_limnodynastidae, 'Family'] = 'Limnodynastidae'
df_clean.loc[is_microhylidae, 'Family'] = 'Microhylidae'
df_clean.loc[is_myobatrachidae, 'Family'] = 'Myobatrachidae'
df_clean.loc[is_ambystomatidae, 'Family'] = 'Ambystomatidae'
df_clean.loc[is_ranidae, 'Family'] = 'Ranidae'


#df_clean.loc[((~is_bufonidae)&(~is_hylidae)&
#          (~is_limnodynastidae)&(~is_microhylidae)&
#          (~is_myobatrachidae)&(~is_ranidae)&(~is_ambystomatidae)), 'Family'] = 'unknown'
```

#### Test


```python
df_clean.Family.value_counts()
```




    Hylidae            5836
    Myobatrachidae     2904
    Limnodynastidae     603
    Ranidae              65
    Microhylidae         46
    Bufonidae            34
    Ambystomatidae       11
    Name: Family, dtype: int64



## 7: Missing values for key indicators: number of individula and number of positively diagnosed

### Assessment


```python
df_clean.Individuals.isnull().sum(), df_clean.Indivs_positive.isnull().sum()
```




    (123, 60)




```python
df_clean[df_clean.Individuals.isnull()].Indivs_positive.isnull().sum()
```




    46




```python
df_clean[df_clean.Indivs_positive.isnull()].Individuals.isnull().sum()
```




    46



### Cleaning

#### Define 
Remove observation with no data about the number of infected species

#### Code


```python
no_indicators = df_clean.Individuals.isnull() | df_clean.Indivs_positive.isnull()
no_indicators.sum()
```




    137




```python
df_clean = df_clean[~no_indicators]
```

#### Test


```python
df_clean.Individuals.isnull().sum(), df_clean.Indivs_positive.isnull().sum()
```




    (0, 0)



## 8: Negative and negative recored as different statuses in `Disease_status`

### Assessment


```python
df_clean.Disease_status.value_counts()
```




    negative     7037
    positive     1239
    no result    1114
    Negative      125
    Name: Disease_status, dtype: int64



### Cleaning

#### Define
Unify status name for "negative" status: all negative values should be labelled as 'negative'

#### Code


```python
df_clean.loc[df_clean.Disease_status=='Negative', 'Disease_status'] = 'negative'
```

#### Test


```python
df_clean.Disease_status.value_counts()
```




    negative     7162
    positive     1239
    no result    1114
    Name: Disease_status, dtype: int64




```python
au_states = {
    'NSW': 'New South Wales',
    'QLD': 'Queensland',
    'SA': 'South Australia',
    'TAS': 'Tasmania',
    'VIC': 'Victoria',
    'WA': 'Western Australia',
    'ACT': 'Australian Capital Territory',
    'NT': 'Northern Territory'
}
```


```python
df_clean['State_FullName'] = df_clean.State.apply(lambda x: au_states[x] if x else None)
```

# Store the cleaned dataset


```python
df_clean.to_csv("./data/Chytridiomycosis_enhanced_cleaned.csv", index=False)
```


```python
(df_clean.Indivs_positive>df_clean.Individuals).sum()
```




    0



# EDA

<a href="https://public.tableau.com/profile/anastasia7889#!/vizhome/ChytridiomycosisinAustraliaFinal/Chytridiomycosis">See Tableau Story</a>

# References

- [Chytridiomycosis](http://wildlife.ohiodnr.gov/portals/wildlife/pdfs/species%20and%20habitats/chytrid.pdf) 
- [Chytridiomycosis on Wikipedia](https://en.wikipedia.org/wiki/Chytridiomycosis#cite_note-Di_Rosa-28)  
- [Overview of Chytridiomycosis](https://amphibiaweb.org/chytrid/chytridiomycosis.html) 
- [Original Dataset Metadata](http://www.esapubs.org/archive/ecol/E091/108/metadata.htm)
- [First Documented Exctinction by Infection](https://www.researchgate.net/publication/29463188_The_Decline_of_the_Sharp-Snouted_Day_Frog_Taudactylus_acutirostris_The_First_Documented_Case_of_Extinction_by_Infection_in_a_Free-Ranging_Wildlife_Species)
- [Full Report](https://www.cabi.org/ISC/datasheet/109124)
- [Chytridiomycosis causes catastrophic organism-wide metabolic dysregulation including profound failure of cellular energy pathways](https://www.researchgate.net/publication/325426490_Chytridiomycosis_causes_catastrophic_organism-wide_metabolic_dysregulation_including_profound_failure_of_cellular_energy_pathways)
