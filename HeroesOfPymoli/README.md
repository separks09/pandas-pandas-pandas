
Heroes Of Pymoli Data Analysis

Observations:
(1) Item 'Final Critic' is listed under two different Item IDs.  In order to be the most accurate, input from the data owner could clarify the nature of this item (by name, it is the most popular purchase).  For this analysis, I have assumed that Item IDs are unique.
(2) Males are overwhelmingly the largest purchasing group by gender, both by number of purchases and amount of purchases.
(3) The age group of 20-24 year-olds spent more than 2.5 times more than the second-highest grossing group (15-19 year-olds).



```python
#Import dependencies
import json
import pandas as pd
data=json.load(open('purchase_data.json'))
purchase_data_df=pd.DataFrame(data)
```


```python
#Define function for formatting currency 
def currency(x):
    return "${:.2f}".format(x)
#Define function for formatting percent
def percent(x):
    return "{:.2f}%".format(x)
```

Player Count


```python
#Calculate total players and display in DataFrame
players = purchase_data_df["SN"].nunique()
players_df = {"Total Players":[players]}
players_df = pd.DataFrame(data=players_df)
players_df.head()
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
      <th>Total Players</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>573</td>
    </tr>
  </tbody>
</table>
</div>



Purchasing Analysis (Total)


```python
#Calculate values based on unique items, format and display in DataFrame

unique_items = len(purchase_data_df["Item ID"].unique())
avg_purchase_price = purchase_data_df["Price"].mean()
total_purchases = purchase_data_df["Item ID"].count()
total_revenue = purchase_data_df["Price"].sum()

purchase_analysis = {
    "Unique Items":[unique_items],
    "Average Purchase Price":[avg_purchase_price],
    "Total Number of Purchases":[total_purchases],
    "Total Revenue":[total_revenue]
}

purchase_analysis_df = pd.DataFrame(data=purchase_analysis)
purchase_analysis_df["Average Purchase Price"] = purchase_analysis_df["Average Purchase Price"].apply(currency)
purchase_analysis_df["Total Revenue"] = purchase_analysis_df["Total Revenue"].apply(currency)
purchase_analysis_df.head()
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
      <th>Average Purchase Price</th>
      <th>Total Number of Purchases</th>
      <th>Total Revenue</th>
      <th>Unique Items</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>$2.93</td>
      <td>780</td>
      <td>$2286.33</td>
      <td>183</td>
    </tr>
  </tbody>
</table>
</div>



Gender Demographics


```python
#Calculate percent of players based on gender, format and display in DataFrame
unique_SN_df = purchase_data_df.drop_duplicates(["SN"])
male_count = unique_SN_df.Gender.value_counts()["Male"]
female_count = unique_SN_df.Gender.value_counts()["Female"]
other_count = unique_SN_df.Gender.value_counts()["Other / Non-Disclosed"]
percent_male = (male_count / players)*100
percent_female = (female_count / players)*100
percent_other = (other_count / players)*100

purchase_by_gender = {"Gender":[male_count,female_count,other_count],
                     "Percent of Players":[percent_male,percent_female,percent_other]}

purchase_by_gender_df = pd.DataFrame(data=purchase_by_gender,index=["Male","Female","Other"])
purchase_by_gender_df["Percent of Players"] = purchase_by_gender_df["Percent of Players"].apply(percent)
purchase_by_gender_df.head()
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
      <th>Gender</th>
      <th>Percent of Players</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Male</th>
      <td>465</td>
      <td>81.15%</td>
    </tr>
    <tr>
      <th>Female</th>
      <td>100</td>
      <td>17.45%</td>
    </tr>
    <tr>
      <th>Other</th>
      <td>8</td>
      <td>1.40%</td>
    </tr>
  </tbody>
</table>
</div>



Purchasing Analysis (Gender)


```python
#Calculate purchase data based on gender, format for display
purchase_count_gender = purchase_data_df[["Gender","Item ID"]]
purchase_count_gender = purchase_count_gender.groupby(["Gender"]).count()
purchase_count_gender = purchase_count_gender.rename(columns={"Item ID":"Purchase Count"})

amount_gender = purchase_data_df[["Gender","Price"]]
amount_gender = amount_gender.groupby(["Gender"]).sum()
amount_gender = amount_gender.rename(columns={"Price":"Total Purchase Value"})

gender_totals_df = purchase_count_gender.join(amount_gender,how="right")

average_price_array = []
normalized_total_array = []
for x in range(len(gender_totals_df)):
    average_purchase_price = gender_totals_df.iloc[x]["Total Purchase Value"]/gender_totals_df.iloc[x]["Purchase Count"]
    average_price_array.append(average_purchase_price)
    normalized_total = (gender_totals_df.iloc[x]["Total Purchase Value"]-gender_totals_df["Total Purchase Value"].min())/(gender_totals_df["Total Purchase Value"].max()-gender_totals_df["Total Purchase Value"].min())                                                        
    normalized_total_array.append(normalized_total)

gender_totals_df["Average Purchase Price"] = average_price_array
gender_totals_df["Normalized Totals"] = normalized_total_array
gender_totals_df["Total Purchase Value"] = gender_totals_df["Total Purchase Value"].apply(currency)
gender_totals_df["Average Purchase Price"] = gender_totals_df["Average Purchase Price"].apply(currency)
gender_totals_df.head()
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
      <th>Purchase Count</th>
      <th>Total Purchase Value</th>
      <th>Average Purchase Price</th>
      <th>Normalized Totals</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Female</th>
      <td>136</td>
      <td>$382.91</td>
      <td>$2.82</td>
      <td>0.189509</td>
    </tr>
    <tr>
      <th>Male</th>
      <td>633</td>
      <td>$1867.68</td>
      <td>$2.95</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>11</td>
      <td>$35.74</td>
      <td>$3.25</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Set parameters for bins based on DataFrame values
max_age = purchase_data_df["Age"].max()
min_age = purchase_data_df["Age"].min()

num_of_bins = (int(((max_age-min_age)/4)))

bins = []
bins.append(min_age)
y = 10
for x in range(num_of_bins):
    bins.append(y-1)
    y = y+5

bin_labels = []
bin_labels.append("<10") 
z = 10
for x in range(num_of_bins): 
    bin_labels.append(""+str(z)+"-"+str(z+4)+"") 
    z = z+5
bin_labels = bin_labels[:-2] 
bin_labels.append(""+str(z-10)+"+") 

#Create DataFrame with bins, calculate correspoding purchase data and format

purchase_data_df["Age Range"] = pd.cut(purchase_data_df["Age"],bins,labels=bin_labels, include_lowest=True)
purchase_by_age = purchase_data_df.groupby(purchase_data_df["Age Range"])
```

Age Demographics


```python
#Find, calculate, format and display purchase data based on age range bins
players_by_age = purchase_by_age[["Age Range","SN"]].count()
players_by_age = players_by_age.drop(columns={"Age Range"})
players_by_age = players_by_age.rename(columns={"SN":"Total Count"})
age_percent_array = []
for x in range(len(players_by_age)):
    age_percent = (players_by_age.iloc[x]["Total Count"]/players)*100
    age_percent_array.append(age_percent)
players_by_age["Percentage of Players"] = age_percent_array
players_by_age["Percentage of Players"] = players_by_age["Percentage of Players"].apply(percent)
players_by_age.head(10)
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
      <th>Total Count</th>
      <th>Percentage of Players</th>
    </tr>
    <tr>
      <th>Age Range</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;10</th>
      <td>28</td>
      <td>4.89%</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>35</td>
      <td>6.11%</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>133</td>
      <td>23.21%</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>336</td>
      <td>58.64%</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>125</td>
      <td>21.82%</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>64</td>
      <td>11.17%</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>42</td>
      <td>7.33%</td>
    </tr>
    <tr>
      <th>40-44</th>
      <td>16</td>
      <td>2.79%</td>
    </tr>
    <tr>
      <th>45+</th>
      <td>1</td>
      <td>0.17%</td>
    </tr>
  </tbody>
</table>
</div>



Purchasing Analysis (Age)


```python
#Find, calculate, format and display purchase data based on age range bins
items_by_age = purchase_by_age.count()
items_by_age = items_by_age.drop(columns={"Age","Gender","Item Name","SN","Price"})
items_by_age = items_by_age.rename(columns={"Item ID":"Item Count"})

amount_by_age = purchase_by_age["Price"].sum()
amount_by_age_df = pd.DataFrame(pd.Series(amount_by_age))
amount_by_age_df = amount_by_age_df.rename(columns={"Price":"Total Purchase Value"})

age_range_df = amount_by_age_df.join(items_by_age)
average_price_array = []
for x in range(len(age_range_df)):
    average_purchase_price = age_range_df.iloc[x]["Total Purchase Value"]/age_range_df.iloc[x]["Item Count"]
    average_price_array.append(average_purchase_price)
age_range_df["Average Purchase Price"] = average_price_array

normalized_total_array = []
for x in range(len(age_range_df)):
    normalized_total = (age_range_df.iloc[x]["Total Purchase Value"]-age_range_df["Total Purchase Value"].min())/(age_range_df["Total Purchase Value"].max()-age_range_df["Total Purchase Value"].min())                                                        
    normalized_total_array.append(normalized_total)
age_range_df["Normalized Totals"] = normalized_total_array
age_range_df["Total Purchase Value"] = age_range_df["Total Purchase Value"].apply(currency)
age_range_df["Average Purchase Price"] = age_range_df["Average Purchase Price"].apply(currency)
age_range_df.head(10)
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
      <th>Total Purchase Value</th>
      <th>Item Count</th>
      <th>Average Purchase Price</th>
      <th>Normalized Totals</th>
    </tr>
    <tr>
      <th>Age Range</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;10</th>
      <td>$83.46</td>
      <td>28</td>
      <td>$2.98</td>
      <td>0.082721</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>$96.95</td>
      <td>35</td>
      <td>$2.77</td>
      <td>0.096542</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>$386.42</td>
      <td>133</td>
      <td>$2.91</td>
      <td>0.393115</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>$978.77</td>
      <td>336</td>
      <td>$2.91</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>$370.33</td>
      <td>125</td>
      <td>$2.96</td>
      <td>0.376630</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>$197.25</td>
      <td>64</td>
      <td>$3.08</td>
      <td>0.199303</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>$119.40</td>
      <td>42</td>
      <td>$2.84</td>
      <td>0.119543</td>
    </tr>
    <tr>
      <th>40-44</th>
      <td>$51.03</td>
      <td>16</td>
      <td>$3.19</td>
      <td>0.049495</td>
    </tr>
    <tr>
      <th>45+</th>
      <td>$2.72</td>
      <td>1</td>
      <td>$2.72</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>



Top Spenders


```python
#Find, calculate, format and display purchase data based on unique users

SN_purchase = purchase_data_df[["SN", "Price"]]
SN_items = purchase_data_df[["SN", "Item ID"]]

SN_items_group = SN_items.groupby(["SN"])
SN_items_count = SN_items_group.count()
SN_items_count = SN_items_count.sort_values(["Item ID"],ascending=False)

SN_purchase_group = SN_purchase.groupby(["SN"])
SN_purchase_sum = SN_purchase_group.sum()
SN_purchase_sum = SN_purchase_sum.sort_values(["Price"],ascending=False)

SN_purchase_totals = SN_purchase_sum.join(SN_items_count,how="right")
SN_purchase_totals = SN_purchase_totals.sort_values(["Price"],ascending=False)

average_price_array = []
for x in range(len(SN_purchase_totals)):
    average_price = SN_purchase_totals.iloc[x]["Price"]/SN_purchase_totals.iloc[x]["Item ID"]
    average_price_array.append(average_price)
SN_purchase_totals["Average Price"] = average_price_array
SN_purchase_totals["Price"]=SN_purchase_totals["Price"].apply(currency)
SN_purchase_totals["Average Price"]=SN_purchase_totals["Average Price"].apply(currency)
SN_purchase_totals = SN_purchase_totals.rename(columns={"Price":"Total Purchase Value","Item ID":"Puchase Count","Average Price":"Average Purchase Price"})
SN_purchase_totals.head()
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
      <th>Total Purchase Value</th>
      <th>Puchase Count</th>
      <th>Average Purchase Price</th>
    </tr>
    <tr>
      <th>SN</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Undirrala66</th>
      <td>$17.06</td>
      <td>5</td>
      <td>$3.41</td>
    </tr>
    <tr>
      <th>Saedue76</th>
      <td>$13.56</td>
      <td>4</td>
      <td>$3.39</td>
    </tr>
    <tr>
      <th>Mindimnya67</th>
      <td>$12.74</td>
      <td>4</td>
      <td>$3.18</td>
    </tr>
    <tr>
      <th>Haellysu29</th>
      <td>$12.73</td>
      <td>3</td>
      <td>$4.24</td>
    </tr>
    <tr>
      <th>Eoda93</th>
      <td>$11.58</td>
      <td>3</td>
      <td>$3.86</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Gather and calculate purchase data based on Item ID volume
item_totals_df = purchase_data_df[["Item ID","Item Name","Price"]]
item_df = purchase_data_df[["Item ID","Item Name"]]

item_ID_group = item_df.groupby(["Item ID"],as_index=False)
item_ID_count = item_ID_group.count()
item_ID_count = item_ID_count.rename(columns={"Item Name":"Item Count"})
item_ID_count = item_ID_count.sort_values(["Item Count"],ascending =False)

item_ID_count = item_ID_count.merge(item_totals_df,on="Item ID").drop_duplicates().reset_index()
item_ID_count = item_ID_count.sort_values(["Item Count"],ascending=False)

amount_spent_array = []
for x in range(len(item_ID_count)):
    amount_spent = item_ID_count.iloc[x]["Price"]*item_ID_count.iloc[x]["Item Count"]
    amount_spent_array.append(amount_spent)
item_ID_count["Total Purchase Value"] = amount_spent_array
item_ID_count = item_ID_count.drop(columns={"index"})
item_ID_count = item_ID_count.rename(columns={"Item Count":"Purchase Count","Price":"Item Price"})
```


```python
#Sort Item ID DataFrame to find same data by greatest purchase amount
most_profit_df = item_ID_count.sort_values(["Total Purchase Value"],ascending=False)
```

Most Popular Items


```python
#Format and display based on Item ID volume
#Display 6 items because there is a tie for 5th
item_ID_count["Item Price"] = item_ID_count["Item Price"].apply(currency)
item_ID_count["Total Purchase Value"] = item_ID_count["Total Purchase Value"].apply(currency)
item_ID_count.head(6)
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
      <th>Item ID</th>
      <th>Purchase Count</th>
      <th>Item Name</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>39</td>
      <td>11</td>
      <td>Betrayal, Whisper of Grieving Widows</td>
      <td>$2.35</td>
      <td>$25.85</td>
    </tr>
    <tr>
      <th>1</th>
      <td>84</td>
      <td>11</td>
      <td>Arcane Gem</td>
      <td>$2.23</td>
      <td>$24.53</td>
    </tr>
    <tr>
      <th>2</th>
      <td>31</td>
      <td>9</td>
      <td>Trickster</td>
      <td>$2.07</td>
      <td>$18.63</td>
    </tr>
    <tr>
      <th>3</th>
      <td>175</td>
      <td>9</td>
      <td>Woeful Adamantite Claymore</td>
      <td>$1.24</td>
      <td>$11.16</td>
    </tr>
    <tr>
      <th>4</th>
      <td>13</td>
      <td>9</td>
      <td>Serenity</td>
      <td>$1.49</td>
      <td>$13.41</td>
    </tr>
    <tr>
      <th>5</th>
      <td>34</td>
      <td>9</td>
      <td>Retribution Axe</td>
      <td>$4.14</td>
      <td>$37.26</td>
    </tr>
  </tbody>
</table>
</div>



Most Profitable Items


```python
#Format and display based on purchase volume
most_profit_df["Item Price"] = most_profit_df["Item Price"].apply(currency)
most_profit_df["Total Purchase Value"] = most_profit_df["Total Purchase Value"].apply(currency)
most_profit_df.head()
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
      <th>Item ID</th>
      <th>Purchase Count</th>
      <th>Item Name</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>34</td>
      <td>9</td>
      <td>Retribution Axe</td>
      <td>$4.14</td>
      <td>$37.26</td>
    </tr>
    <tr>
      <th>13</th>
      <td>115</td>
      <td>7</td>
      <td>Spectral Diamond Doomblade</td>
      <td>$4.25</td>
      <td>$29.75</td>
    </tr>
    <tr>
      <th>39</th>
      <td>32</td>
      <td>6</td>
      <td>Orenmir</td>
      <td>$4.95</td>
      <td>$29.70</td>
    </tr>
    <tr>
      <th>24</th>
      <td>103</td>
      <td>6</td>
      <td>Singed Scalpel</td>
      <td>$4.87</td>
      <td>$29.22</td>
    </tr>
    <tr>
      <th>9</th>
      <td>107</td>
      <td>8</td>
      <td>Splitter, Foe Of Subtlety</td>
      <td>$3.61</td>
      <td>$28.88</td>
    </tr>
  </tbody>
</table>
</div>


