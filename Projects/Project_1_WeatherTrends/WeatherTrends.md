
# Project 1: Weather Trends

## Step 1: Data extraction

The first step was to get the data from the database. The city I was closest to at the moment (and that had the data) was Kiev. The following queries were used: 

<b> City level data </b> 

SELECT cd.year, cd.avg_temp as kiev_avg  
FROM city_data cd  
WHERE cd.city = 'Kiev'  

<b>Global data</b>

SELECT gd.year, gd.avg_temp as global_temp  
FROM global_data gd  

### Check the datasets

When working with SQL I noticed already that there were missing values in the dataset with city level measurements. The missing values were for four consequtive years: 1946 - 1949. Also the earliest year with the data for the global averages was 1750.


```python
import pandas as pd
import numpy as np
kiev_temp = pd.read_csv("kiev_temp.csv")
kiev_temp[kiev_temp.kiev_temp.isna()]
```

<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>year</th>
      <th>kiev_temp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>1746</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1747</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1748</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1749</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>




```python
global_temp = pd.read_csv("global_temp.csv")
global_temp.year.min()
```




    1750



Hence, in this case it doesn't make much sense to impute the missing values and it is reasonable to compare temperature starting in 1750. 


```python
df = pd.merge(kiev_temp, global_temp, how='inner', left_on='year', right_on='year')
df = df.set_index('year')
```


```python
df.head()
```




<div>
<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>kiev_temp</th>
      <th>global_temp</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1750</th>
      <td>7.85</td>
      <td>8.72</td>
    </tr>
    <tr>
      <th>1751</th>
      <td>8.11</td>
      <td>7.98</td>
    </tr>
    <tr>
      <th>1752</th>
      <td>1.21</td>
      <td>5.78</td>
    </tr>
    <tr>
      <th>1753</th>
      <td>6.90</td>
      <td>8.39</td>
    </tr>
    <tr>
      <th>1754</th>
      <td>7.02</td>
      <td>8.47</td>
    </tr>
  </tbody>
</table>
</div>



Alternative query to get exactly the same result in csv: 
SELECT gd.year, gd.avg_temp as global_avg, 
cd.avg_temp as kiev_avg
FROM global_data gd
LEFT JOIN city_data cd ON gd.year = cd.year
WHERE cd.city = 'Kiev'

## Step 2: Moving average 

There are 264 observations now both for Kiev and for global averages. So let's average values for 10-year intervals. 


```python
N = 10
```

### Manual calculation example

To calculate the moving average for N-th entry, we need to take N-1 previous observations and the N-th entry itself (so total - N observations). So the first entry filled in will be N-th entry. We can do that with the loop:


```python
ma_entries = []
for i in range(N,len(df)+1):
    ma_entries.append(np.round(df.kiev_temp[i-N:i].mean(),3))
print(ma_entries[:6])
```

    [6.53, 6.336, 6.253, 6.844, 6.766, 6.802]



```python
kiev_ma = pd.DataFrame({'year': df.index[N-1:], 'kiev_avg_man': ma_entries})
kiev_ma = kiev_ma.set_index('year')
```


```python
kiev_ma.head()
```


<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>kiev_avg_man</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1759</th>
      <td>6.530</td>
    </tr>
    <tr>
      <th>1760</th>
      <td>6.336</td>
    </tr>
    <tr>
      <th>1761</th>
      <td>6.253</td>
    </tr>
    <tr>
      <th>1762</th>
      <td>6.844</td>
    </tr>
    <tr>
      <th>1763</th>
      <td>6.766</td>
    </tr>
  </tbody>
</table>

### Pandas way

In Pandas there is a special method for calculating the moving average which is pd.DataFrame.rolling(...). We can use it to produce the same result:


```python
df_ma = df.rolling(N,N).mean()[N-1:]
```


```python
df_ma.head()
```

<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>kiev_temp</th>
      <th>global_temp</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1759</th>
      <td>6.530</td>
      <td>8.030</td>
    </tr>
    <tr>
      <th>1760</th>
      <td>6.336</td>
      <td>7.877</td>
    </tr>
    <tr>
      <th>1761</th>
      <td>6.253</td>
      <td>7.956</td>
    </tr>
    <tr>
      <th>1762</th>
      <td>6.844</td>
      <td>8.239</td>
    </tr>
    <tr>
      <th>1763</th>
      <td>6.766</td>
      <td>8.150</td>
    </tr>
  </tbody>
</table>


Compare manual way and pandas way: 


```python
pd.merge(df_ma[['kiev_temp']], kiev_ma, left_index=True, right_index=True).head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>kiev_temp</th>
      <th>kiev_avg_man</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1759</th>
      <td>6.530</td>
      <td>6.530</td>
    </tr>
    <tr>
      <th>1760</th>
      <td>6.336</td>
      <td>6.336</td>
    </tr>
    <tr>
      <th>1761</th>
      <td>6.253</td>
      <td>6.253</td>
    </tr>
    <tr>
      <th>1762</th>
      <td>6.844</td>
      <td>6.844</td>
    </tr>
    <tr>
      <th>1763</th>
      <td>6.766</td>
      <td>6.766</td>
    </tr>
  </tbody>
</table>

## Step 3: Moving averages plot
My key motivation for visualisation was to present the data in the clearest way possible. I chose Matplotlib because of its flexibility. 


```python
%matplotlib inline
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns
def plot_averages(data, title, cities, colors, col_names):
    with sns.axes_style("whitegrid"):
        fig, ax = plt.subplots()
        fig.set_size_inches(14,6)
        for i in range(len(cities)):
            ax.plot(data.index, data[col_names[i]], colors[i], linewidth=2, label=cities[i])
#            ax.plot(data.index, data.global_temp, color='coral', linewidth=2, label='Global')
        ax.legend(bbox_to_anchor=(1, 1), loc=2)
        fig.suptitle(title, fontsize=18)
        plt.xlabel("Year")
        plt.ylabel("Average Temperature")
plot_averages(df_ma, "Moving Averages", ['Kiev', 'Global'], ['teal','coral'], ['kiev_temp', 'global_temp']) 
```


![png](WeatherTrends_files/WeatherTrends_29_0.png)


## Step 4: Observations (for Kiev and Global temperature)

1. The first observation that catches the eye is that it's definitely getting warmer. We see steady growth both in Ukraine and globally. 
2. There was a significant cooldown somewhere in the first quarter of the 19th century which was also the global trend. 
3. Based on these two graphs we can see that they are definitely correlated and that it might be the case that we will see somewhat similar shape if we take other cities. 
4. We can also notice that the range of fluctuations was bigger in 18th and 19th centuries, and up until 1870 (approximately) there was not a clear trend. So it looks like there was some factor that influenced the temperature all over the world and the first thing that comes to mind is anthropogenic factor. However, we can't state that, because correlation does not imply causation. 
5. Global temperature chart is much more unstable and we can note more fluctuations. However, it is hard to make any inferences since average is not a robust and it barely tells us something specific.

## Step 5: Additional exploration

I also decided to take another city - Moscow - and see how cold it it there in comparison to Kiev. The procedures are the same. The SQL is as follows:

SELECT cd.year, cd.avg_temp as moscow_avg  
FROM city_data cd  
WHERE cd.city = 'Moscow'  

Then we append it to the existing dataframe and re-calculate the moving average: 


```python
moscow_temp = pd.read_csv("moscow_temp.csv")
df = pd.merge(moscow_temp, df, how='inner', left_on='year', right_on='year').set_index('year')
df_ma = df.rolling(N,N).mean()[N-1:]
```


```python
plot_averages(df_ma, "Moving Averages", ['Kiev', 'Global', 'Moscow'], 
              ['teal','coral', 'black'], ['kiev_temp', 'global_temp', 'moscow_temp']) 
```


![png](WeatherTrends_files/WeatherTrends_36_0.png)


As we can see, it is definitely colder in Moscow, even compared to Kiev! Another observation we can make is that Moscow is much more coherent with global trends. At this stage it makes sense to check the correlation coefficients. We can do that with numpy functions:


```python
"Kiev and global: {}".format(np.round(np.corrcoef(df_ma.kiev_temp, df_ma.global_temp)[0,1],4))
```




    'Kiev and global: 0.8806'




```python
"Moscow and global: {}".format(np.round(np.corrcoef(df_ma.moscow_temp, df_ma.global_temp)[0,1],4))
```




    'Moscow and global: 0.8985'



The simplest way to try to estimate loal temperature without creating a regression model is to calculate the average difference between global and local temperature which should work pretty well for Moscow since their graps are of almost identical shape: 


```python
mean_deviation = np.round(np.mean(df_ma.global_temp-df_ma.moscow_temp),4)
"The mean deviation is: {}".format(mean_deviation)
```




    'The mean deviation is: 4.3681'




```python
moscow_predicted = df_ma.global_temp-mean_deviation
```

Now let's calculate the error of this prediction as a root mean squared error - subratact actual from predicte, raise to the power of two, find the average and take the square root


```python
'RMSE: {}'.format(np.round(np.sqrt(np.mean((moscow_predicted-df_ma.moscow_temp)**2)),4))
```




    'RMSE: 0.2293'


