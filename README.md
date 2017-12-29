
# Generating Alpha using Starmine Analytical Models - a Python example

This article will demonstrate how we can use Starmine Analytical Models to derive insight and generate alpha, in this case, for  equity markets. It is intended as a teaser to intice you to further explore this rich vein rather than a rigorous scientific study. I will be using Eikon and more specifically our new Eikon Data API to access all the data I need to conduct this analysis - however Starmine Analytics is available in other products and delivery channels. I will also provide the Jupyter notebook source via gitub. 

Pre-requisites: 

Thomson Reuters Eikon with access to new Eikon Data APIs

Python 2.x/3.x

Required Python Packages: eikon, pandas, numpy, matplotlib, sklearn, scipy

### Starmine Quantitative Analytics

In short - StarMine provides a suite of proprietary alpha-generating analytics and models spanning sectors, regions, and markets. The list of models is really broad-based and includes both quantitative analytics (such as smartEstimates) and quantitative models (such as the Combined Alpha Model I will demo here). 

All of these analytics and models can provide you with new sources of information and subsequently alpha - they are calculated by our Starmine team and delivered to you - saving you many man years of work and research etc. You can find out more about Starmine Analytics by looking at the PDF attached to this article and you can find out more about the Combined Alpha Model by viewing the short 2 min video below.


```python
from IPython.display import HTML
HTML('<iframe width="560" height="315" src="https://www.youtube.com/embed/wNVJEG9Utlo" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>')
```




<iframe width="560" height="315" src="https://www.youtube.com/embed/wNVJEG9Utlo" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>



### How to Access the Starmine and Other Data

As mentioned above I will be using Eikon and specifically our new Eikon Data API - which is a really easy to use yet performant web API. At the time of writing this article it is still in Beta - but all Eikon users can access it by simply upgrading their Eikon to v4.0.36+ and going to the developer portal [here](https://developers.thomsonreuters.com/eikon-data-apis) and follow instructions. If you have any questions around the API there are also monitored Q&A forums [here](https://developers.thomsonreuters.com/eikon-data-apis/qa).

Once we have access to the Eikon Data API its very straightforward to get the data we need. First lets import some packages we will need to conduct this analysis and also set our App ID (this is available from the App ID generator - see the quick start guide for further details):


```python
import eikon as ek
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
ek.set_app_id('Your App ID here')
```

Next we need to formulate our API call. In this case, I will be requesting data for all Dow Jones Industrial Average index constituents and I will be requesting a mixture of both Reference, Starmine Analytics and Price performance data for 8 quarters - I have made it more generic here - but essentially this is one line of code! One thing to note here is that the 3 Month Total Return is forward looking so its great for this type of analysis ie it does not need to be offset/shifted. And as you will see the API returns the data in a pandas dataframe.


```python
RICS = ['0#.dji']
fields =['TR.TRBCIndustryGroup','TR.CombinedAlphaCountryRank(SDate=0,EDate=-7,Frq=FQ)','TR.CombinedAlphaCountryRank(SDate=0,EDate=-7,Frq=FQ).Date',
         'TR.TotalReturn3Mo(SDate=0,EDate=-7,Frq=FQ)']

ids,err=ek.get_data(RICS,fields=fields)
ids.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Instrument</th>
      <th>TRBC Industry Group Name</th>
      <th>Combined Alpha Model Country Rank</th>
      <th>Date</th>
      <th>3 Month Total Return</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>57.0</td>
      <td>2017-09-30T00:00:00Z</td>
      <td>1.40588724845</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MMM.N</td>
      <td></td>
      <td>74.0</td>
      <td>2017-06-30T00:00:00Z</td>
      <td>9.49681533207</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MMM.N</td>
      <td></td>
      <td>65.0</td>
      <td>2017-03-31T00:00:00Z</td>
      <td>7.83853634636</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MMM.N</td>
      <td></td>
      <td>65.0</td>
      <td>2016-12-31T00:00:00Z</td>
      <td>1.98169008175</td>
    </tr>
    <tr>
      <th>4</th>
      <td>MMM.N</td>
      <td></td>
      <td>58.0</td>
      <td>2016-09-30T00:00:00Z</td>
      <td>1.2548750062</td>
    </tr>
    <tr>
      <th>5</th>
      <td>MMM.N</td>
      <td></td>
      <td>54.0</td>
      <td>2016-06-30T00:00:00Z</td>
      <td>5.71827717603</td>
    </tr>
    <tr>
      <th>6</th>
      <td>MMM.N</td>
      <td></td>
      <td>46.0</td>
      <td>2016-03-31T00:00:00Z</td>
      <td>11.4201046525</td>
    </tr>
    <tr>
      <th>7</th>
      <td>MMM.N</td>
      <td></td>
      <td>37.0</td>
      <td>2015-12-31T00:00:00Z</td>
      <td>6.94860740857</td>
    </tr>
    <tr>
      <th>8</th>
      <td>AXP.N</td>
      <td>Banking Services</td>
      <td>66.0</td>
      <td>2017-09-30T00:00:00Z</td>
      <td>7.78845748316</td>
    </tr>
    <tr>
      <th>9</th>
      <td>AXP.N</td>
      <td></td>
      <td>77.0</td>
      <td>2017-06-30T00:00:00Z</td>
      <td>6.78786348296</td>
    </tr>
  </tbody>
</table>
</div>



Now that we have our data in a dataframe we may need to do some wrangling to get the types correctly set. The get_data call is the most flexible of calls so this is to be expected. We can easily check the types of data in the dataframe by column:


```python
ids.dtypes
```




    Instrument                            object
    TRBC Industry Group Name              object
    Combined Alpha Model Country Rank    float64
    Date                                  object
    3 Month Total Return                  object
    dtype: object



First we want to turn the Date object (a string) into a datetime type, then we want to set that datetime field as the index for the frame. Secondly, we want to make sure any numeric fields are numeric and any text fields are strings:


```python
ids['Date']=pd.to_datetime(ids['Date'])
ads=ids.set_index('Date')[['Instrument','TRBC Industry Group Name','Combined Alpha Model Country Rank','3 Month Total Return']]
ads['3 Month Total Return'] = ads['3 Month Total Return'].astype(str).astype(np.float64)
ads['TRBC Industry Group Name'] = ads['TRBC Industry Group Name'].astype(str)
ads.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Instrument</th>
      <th>TRBC Industry Group Name</th>
      <th>Combined Alpha Model Country Rank</th>
      <th>3 Month Total Return</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-09-30</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>57.0</td>
      <td>1.405887</td>
    </tr>
    <tr>
      <th>2017-06-30</th>
      <td>MMM.N</td>
      <td></td>
      <td>74.0</td>
      <td>9.496815</td>
    </tr>
    <tr>
      <th>2017-03-31</th>
      <td>MMM.N</td>
      <td></td>
      <td>65.0</td>
      <td>7.838536</td>
    </tr>
    <tr>
      <th>2016-12-31</th>
      <td>MMM.N</td>
      <td></td>
      <td>65.0</td>
      <td>1.981690</td>
    </tr>
    <tr>
      <th>2016-09-30</th>
      <td>MMM.N</td>
      <td></td>
      <td>58.0</td>
      <td>1.254875</td>
    </tr>
    <tr>
      <th>2016-06-30</th>
      <td>MMM.N</td>
      <td></td>
      <td>54.0</td>
      <td>5.718277</td>
    </tr>
    <tr>
      <th>2016-03-31</th>
      <td>MMM.N</td>
      <td></td>
      <td>46.0</td>
      <td>11.420105</td>
    </tr>
    <tr>
      <th>2015-12-31</th>
      <td>MMM.N</td>
      <td></td>
      <td>37.0</td>
      <td>6.948607</td>
    </tr>
    <tr>
      <th>2017-09-30</th>
      <td>AXP.N</td>
      <td>Banking Services</td>
      <td>66.0</td>
      <td>7.788457</td>
    </tr>
    <tr>
      <th>2017-06-30</th>
      <td>AXP.N</td>
      <td></td>
      <td>77.0</td>
      <td>6.787863</td>
    </tr>
  </tbody>
</table>
</div>



A bit more wrangling is required here as we can see for example that the API has returned the TRBC Industry Group only for the most current instance of each instrument - not for the historical quarters. We can recitfy this easily in 2 lines of code. First we will replace blanks with nan (not a number). We can then use the excellent and surgical fillna dataframe function to fill the Sector name to the historic quarters: 


```python
ads1 = ads.replace('', np.nan, regex=True)
```


```python
ads1['TRBC Industry Group Name'].fillna(method='ffill',limit=7, inplace=True)
ads1.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Instrument</th>
      <th>TRBC Industry Group Name</th>
      <th>Combined Alpha Model Country Rank</th>
      <th>3 Month Total Return</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-09-30</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>57.0</td>
      <td>1.405887</td>
    </tr>
    <tr>
      <th>2017-06-30</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>74.0</td>
      <td>9.496815</td>
    </tr>
    <tr>
      <th>2017-03-31</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>65.0</td>
      <td>7.838536</td>
    </tr>
    <tr>
      <th>2016-12-31</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>65.0</td>
      <td>1.981690</td>
    </tr>
    <tr>
      <th>2016-09-30</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>58.0</td>
      <td>1.254875</td>
    </tr>
    <tr>
      <th>2016-06-30</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>54.0</td>
      <td>5.718277</td>
    </tr>
    <tr>
      <th>2016-03-31</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>46.0</td>
      <td>11.420105</td>
    </tr>
    <tr>
      <th>2015-12-31</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>37.0</td>
      <td>6.948607</td>
    </tr>
    <tr>
      <th>2017-09-30</th>
      <td>AXP.N</td>
      <td>Banking Services</td>
      <td>66.0</td>
      <td>7.788457</td>
    </tr>
    <tr>
      <th>2017-06-30</th>
      <td>AXP.N</td>
      <td>Banking Services</td>
      <td>77.0</td>
      <td>6.787863</td>
    </tr>
  </tbody>
</table>
</div>



So now our Dataframe is seemingly in good shape - what are we going to do next? We can start with the question do higher CAM score leads to better performance? Or put more generically, are CAM scores RELATED to performance? Remember these CAM scores are dynamic and change overtime depending on changes in their underlying components AND relative to other companies in the universe (in this case companies in the same country). 

So lets just see what this looks like in terms of a scatterplot for one sector of the Dow. Here we are just filtering the frame using a text filter and aggregating using a groupby statement. In this case we have 2 companies.


```python
%matplotlib inline
adsl = ads1[ads1['TRBC Industry Group Name'] =='Industrial Conglomerates']
adsl = adsl.groupby('Instrument')
ax = adsl.plot(x='Combined Alpha Model Country Rank', y='3 Month Total Return', kind='scatter')
```


![png](output_14_0.png)



![png](output_14_1.png)


So lets be clear about what we are looking at. Each dot represents a pair of observations for CAM rank and 3M total return for a quarter - we should have 8 quarters of observations for each instrument. So we could implement a simple linear regression and that would have different parameters such as intercept, slope. 

As I am in exploratory mode - I just am interested in the slope of the best fit linear line (of the form y = ax + b), where a is the slope coefficient. This should offer me some indication of how 3 Month total returns change for an increase in CAM rank. So our slope coefficient can tell us for example whether our variables are positively related (positive a) or negatively related (negative a) or really not related at all (near zero a). 

We can also more formally calculate Spearman's Rank Correlation Coefficient which will give us a more rigorous measure of relatedness and also a probability that the null hypothesis (the two variables are not related) is true.

So rather than create plots for everything we can just create a model to provide the linear best fit solution for each instrument and then store that value in a new column as well as calculating the Spearman's Rank Correlation Coefficient and p-value. 

First, I want to check for if any data is null as this may result in errors downstream.


```python
ads1.isnull().any().count()
```




    4



Here we can see there are 4 null values in our dataset - we can deal with these by replacing them with either mean values or most recent values. In our case, I choose the latter and will use a forward fill function to replace nulls: 


```python
adsNN = ads1.fillna(method='ffill')
```


```python
adsNN.isnull().any()
```




    Instrument                           False
    TRBC Industry Group Name             False
    Combined Alpha Model Country Rank    False
    3 Month Total Return                 False
    dtype: bool



Now we have confirmed we have no null values we can move on. Next we will use the Linear Regression model from the Linear Model tools from Scikit Learn package. We want to solve for 8 quarters of data for each Instrument. So we iterate over each intrument then use the model.fit method to generate the best fit linear solution (OLS) and then store the 'coef_' parameter of the model as a new column in the adsNN dataframe called 'slope'. 

Whilst we are here we will also calculate a Spearman's Rank Correlation Coefficient using a routine from scipy package. The routine returns 2 values, the first is the Coefficient (Rho) and the second is the p-value. I just store these 2 elements in 2 seperate columns.


```python
import sklearn
import scipy
from sklearn import linear_model
model = linear_model.LinearRegression()
for (group, adsNN_gp) in adsNN.groupby('Instrument'):
    X=adsNN_gp[['Combined Alpha Model Country Rank']]
    y=adsNN_gp[['3 Month Total Return']]
    model.fit(X,y)
    spearmans = scipy.stats.spearmanr(X,y)
    adsNN.loc[adsNN.Instrument == adsNN_gp.iloc[0].Instrument, 'slope'] = model.coef_
    adsNN.loc[adsNN.Instrument == adsNN_gp.iloc[0].Instrument, 'Rho'] = spearmans[0]
    adsNN.loc[adsNN.Instrument == adsNN_gp.iloc[0].Instrument, 'p'] = spearmans[1]

adsNN.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Instrument</th>
      <th>TRBC Industry Group Name</th>
      <th>Combined Alpha Model Country Rank</th>
      <th>3 Month Total Return</th>
      <th>slope</th>
      <th>Rho</th>
      <th>p</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-09-30</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>57.0</td>
      <td>1.405887</td>
      <td>-0.042706</td>
      <td>-0.035929</td>
      <td>0.932691</td>
    </tr>
    <tr>
      <th>2017-06-30</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>74.0</td>
      <td>9.496815</td>
      <td>-0.042706</td>
      <td>-0.035929</td>
      <td>0.932691</td>
    </tr>
    <tr>
      <th>2017-03-31</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>65.0</td>
      <td>7.838536</td>
      <td>-0.042706</td>
      <td>-0.035929</td>
      <td>0.932691</td>
    </tr>
    <tr>
      <th>2016-12-31</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>65.0</td>
      <td>1.981690</td>
      <td>-0.042706</td>
      <td>-0.035929</td>
      <td>0.932691</td>
    </tr>
    <tr>
      <th>2016-09-30</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>58.0</td>
      <td>1.254875</td>
      <td>-0.042706</td>
      <td>-0.035929</td>
      <td>0.932691</td>
    </tr>
    <tr>
      <th>2016-06-30</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>54.0</td>
      <td>5.718277</td>
      <td>-0.042706</td>
      <td>-0.035929</td>
      <td>0.932691</td>
    </tr>
    <tr>
      <th>2016-03-31</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>46.0</td>
      <td>11.420105</td>
      <td>-0.042706</td>
      <td>-0.035929</td>
      <td>0.932691</td>
    </tr>
    <tr>
      <th>2015-12-31</th>
      <td>MMM.N</td>
      <td>Industrial Conglomerates</td>
      <td>37.0</td>
      <td>6.948607</td>
      <td>-0.042706</td>
      <td>-0.035929</td>
      <td>0.932691</td>
    </tr>
    <tr>
      <th>2017-09-30</th>
      <td>AXP.N</td>
      <td>Banking Services</td>
      <td>66.0</td>
      <td>7.788457</td>
      <td>0.580752</td>
      <td>0.666667</td>
      <td>0.070988</td>
    </tr>
    <tr>
      <th>2017-06-30</th>
      <td>AXP.N</td>
      <td>Banking Services</td>
      <td>77.0</td>
      <td>6.787863</td>
      <td>0.580752</td>
      <td>0.666667</td>
      <td>0.070988</td>
    </tr>
  </tbody>
</table>
</div>



Voila - I have all the calculations I requested and I think I just want to average these by Instrument so I can get a summary view. (note averaging the slope, Rho and p values does not change them as they were calculated once for the 8 periods and just copied 8 times). 


```python
Averages = adsNN.groupby(['Instrument']).mean()
Averages
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Combined Alpha Model Country Rank</th>
      <th>3 Month Total Return</th>
      <th>slope</th>
      <th>Rho</th>
      <th>p</th>
    </tr>
    <tr>
      <th>Instrument</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>AAPL.OQ</th>
      <td>73.875</td>
      <td>5.195261</td>
      <td>-0.415002</td>
      <td>-0.238095</td>
      <td>0.570156</td>
    </tr>
    <tr>
      <th>AXP.N</th>
      <td>65.375</td>
      <td>3.509887</td>
      <td>0.580752</td>
      <td>0.666667</td>
      <td>0.070988</td>
    </tr>
    <tr>
      <th>BA.N</th>
      <td>80.000</td>
      <td>9.847959</td>
      <td>0.774764</td>
      <td>0.706599</td>
      <td>0.050063</td>
    </tr>
    <tr>
      <th>CAT.N</th>
      <td>55.375</td>
      <td>9.602497</td>
      <td>0.074619</td>
      <td>0.380952</td>
      <td>0.351813</td>
    </tr>
    <tr>
      <th>CSCO.OQ</th>
      <td>91.250</td>
      <td>4.271595</td>
      <td>0.101271</td>
      <td>0.036147</td>
      <td>0.932283</td>
    </tr>
    <tr>
      <th>CVX.N</th>
      <td>61.000</td>
      <td>6.544678</td>
      <td>0.167824</td>
      <td>0.299407</td>
      <td>0.471261</td>
    </tr>
    <tr>
      <th>DIS.N</th>
      <td>41.250</td>
      <td>0.215785</td>
      <td>0.175213</td>
      <td>0.047619</td>
      <td>0.910849</td>
    </tr>
    <tr>
      <th>DWDP.N</th>
      <td>19.000</td>
      <td>3.051503</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>GE.N</th>
      <td>74.625</td>
      <td>0.738084</td>
      <td>-0.185066</td>
      <td>0.309524</td>
      <td>0.455645</td>
    </tr>
    <tr>
      <th>GS.N</th>
      <td>49.250</td>
      <td>5.564424</td>
      <td>0.619657</td>
      <td>0.610789</td>
      <td>0.107721</td>
    </tr>
    <tr>
      <th>HD.N</th>
      <td>81.125</td>
      <td>5.141574</td>
      <td>-0.559756</td>
      <td>-0.595238</td>
      <td>0.119530</td>
    </tr>
    <tr>
      <th>IBM.N</th>
      <td>78.625</td>
      <td>1.454870</td>
      <td>0.000606</td>
      <td>-0.289178</td>
      <td>0.487261</td>
    </tr>
    <tr>
      <th>INTC.OQ</th>
      <td>85.375</td>
      <td>4.088071</td>
      <td>0.352581</td>
      <td>0.395217</td>
      <td>0.332517</td>
    </tr>
    <tr>
      <th>JNJ.N</th>
      <td>86.625</td>
      <td>4.975873</td>
      <td>0.160626</td>
      <td>0.059881</td>
      <td>0.887991</td>
    </tr>
    <tr>
      <th>JPM.N</th>
      <td>88.875</td>
      <td>6.677507</td>
      <td>0.511293</td>
      <td>0.047619</td>
      <td>0.910849</td>
    </tr>
    <tr>
      <th>KO.N</th>
      <td>60.250</td>
      <td>2.287097</td>
      <td>0.011978</td>
      <td>-0.119048</td>
      <td>0.778886</td>
    </tr>
    <tr>
      <th>MCD.N</th>
      <td>76.250</td>
      <td>7.080045</td>
      <td>0.596103</td>
      <td>0.574861</td>
      <td>0.136058</td>
    </tr>
    <tr>
      <th>MMM.N</th>
      <td>57.000</td>
      <td>5.758099</td>
      <td>-0.042706</td>
      <td>-0.035929</td>
      <td>0.932691</td>
    </tr>
    <tr>
      <th>MRK.N</th>
      <td>87.750</td>
      <td>4.167913</td>
      <td>0.764098</td>
      <td>0.838338</td>
      <td>0.009323</td>
    </tr>
    <tr>
      <th>MSFT.OQ</th>
      <td>90.500</td>
      <td>7.836693</td>
      <td>0.040697</td>
      <td>0.059881</td>
      <td>0.887991</td>
    </tr>
    <tr>
      <th>NKE.N</th>
      <td>29.625</td>
      <td>-1.740508</td>
      <td>0.213786</td>
      <td>0.730552</td>
      <td>0.039556</td>
    </tr>
    <tr>
      <th>PFE.N</th>
      <td>84.375</td>
      <td>2.602840</td>
      <td>0.288559</td>
      <td>0.357143</td>
      <td>0.385121</td>
    </tr>
    <tr>
      <th>PG.N</th>
      <td>68.000</td>
      <td>3.837856</td>
      <td>0.251476</td>
      <td>0.035929</td>
      <td>0.932691</td>
    </tr>
    <tr>
      <th>TRV.N</th>
      <td>77.250</td>
      <td>3.225188</td>
      <td>0.335607</td>
      <td>0.706599</td>
      <td>0.050063</td>
    </tr>
    <tr>
      <th>UNH.N</th>
      <td>89.375</td>
      <td>7.168842</td>
      <td>0.444293</td>
      <td>0.313276</td>
      <td>0.449908</td>
    </tr>
    <tr>
      <th>UTX.N</th>
      <td>84.500</td>
      <td>4.074656</td>
      <td>-0.212942</td>
      <td>-0.359288</td>
      <td>0.382065</td>
    </tr>
    <tr>
      <th>V.N</th>
      <td>27.625</td>
      <td>5.676034</td>
      <td>-0.060229</td>
      <td>-0.089392</td>
      <td>0.833281</td>
    </tr>
    <tr>
      <th>VZ.N</th>
      <td>75.250</td>
      <td>3.136394</td>
      <td>0.414491</td>
      <td>0.714286</td>
      <td>0.046528</td>
    </tr>
    <tr>
      <th>WMT.N</th>
      <td>85.375</td>
      <td>3.266748</td>
      <td>-0.078561</td>
      <td>-0.047619</td>
      <td>0.910849</td>
    </tr>
    <tr>
      <th>XOM.N</th>
      <td>35.625</td>
      <td>1.958404</td>
      <td>0.082098</td>
      <td>0.011976</td>
      <td>0.977547</td>
    </tr>
  </tbody>
</table>
</div>



### Conclusion

So now we have a summarised view of our study - can we answer the question posited earlier? 'Are CAM scores RELATED to performance?' Let us have a look at average CAM score versus average 3 month total return (over 8 quarters). This seems to indicate that there is a positive relation between the two variables.


```python
ax = Averages.plot(x='Combined Alpha Model Country Rank', y='3 Month Total Return', kind='scatter')
```


![png](output_25_0.png)


We can check this further by running the same Spearman's Rank Correlation Coefficient test against these summary results. Here we can see Rho of 31.26 which is positive and with p of 0.092 the null hypothesis can be safetly rejected. 


```python
spearmans = scipy.stats.spearmanr(Averages['Combined Alpha Model Country Rank'],Averages['3 Month Total Return'])
Rho = spearmans[0]
p = spearmans[1]
print Rho,p
```

    0.312604296072 0.0925882700203
    

Looking at the distribution of Rho and p we can see that there are a range of relations between CAM score and performance from negative to positive with some showing no relation at all. There appear to be groups of companies displaying differing characteristics, maybe some of these could be exploitable - though you would need to conduct more research to confirm this.

This was not meant to be a conclusive study, more a taster of what is possible using our data in the python ecosystem. This analysis was done on one Equity index the Dow - but you could easily try this for any equity index - maybe some markets are more responsive than others, maybe looking at a longer term return say 12 months might be interesting. There is plenty of scope. I hope to have shown you that it can be straightforward to produce relatively complex studies from a small amount of code. 


```python
bx = Averages.plot(x='Rho', y='p', kind='scatter')
```


![png](output_29_0.png)


Author: jason.ramchandani@tr.com
