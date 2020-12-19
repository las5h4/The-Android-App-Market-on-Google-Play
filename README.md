## 1. Google Play Store apps and reviews
<p>Mobile apps are everywhere. They are easy to create and can be lucrative. Because of these two factors, more and more apps are being developed. In this notebook, we will do a comprehensive analysis of the Android app market by comparing over ten thousand apps in Google Play across different categories. We'll look for insights in the data to devise strategies to drive growth and retention.</p>
<p><img src="https://assets.datacamp.com/production/project_619/img/google_play_store.png" alt="Google Play logo"></p>
<p>Let's take a look at the data, which consists of two files:</p>
<ul>
<li><code>apps.csv</code>: contains all the details of the applications on Google Play. There are 13 features that describe a given app.</li>
<li><code>user_reviews.csv</code>: contains 100 reviews for each app, <a href="https://www.androidpolice.com/2019/01/21/google-play-stores-redesigned-ratings-and-reviews-section-lets-you-easily-filter-by-star-rating/">most helpful first</a>. The text in each review has been pre-processed and attributed with three new features: Sentiment (Positive, Negative or Neutral), Sentiment Polarity and Sentiment Subjectivity.</li>
</ul>


```python
# Read in dataset
import pandas as pd
apps_with_duplicates = pd.read_csv("datasets/apps.csv")

# Drop duplicates
apps = apps_with_duplicates.drop_duplicates()

# Print the total number of apps
print('Total number of apps in the dataset = ', apps["App"].count())

# Print a concise summary of apps dataframe
print(apps.info())

# Have a look at a random sample of n rows
n = 5
print(apps.sample(n=n, axis=0))
```

    Total number of apps in the dataset =  9659
    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 9659 entries, 0 to 9658
    Data columns (total 14 columns):
     #   Column          Non-Null Count  Dtype  
    ---  ------          --------------  -----  
     0   Unnamed: 0      9659 non-null   int64  
     1   App             9659 non-null   object 
     2   Category        9659 non-null   object 
     3   Rating          8196 non-null   float64
     4   Reviews         9659 non-null   int64  
     5   Size            8432 non-null   float64
     6   Installs        9659 non-null   object 
     7   Type            9659 non-null   object 
     8   Price           9659 non-null   object 
     9   Content Rating  9659 non-null   object 
     10  Genres          9659 non-null   object 
     11  Last Updated    9659 non-null   object 
     12  Current Ver     9651 non-null   object 
     13  Android Ver     9657 non-null   object 
    dtypes: float64(2), int64(2), object(10)
    memory usage: 1.1+ MB
    None
          Unnamed: 0                         App            Category  Rating  \
    2585        3313              Age Calculator               TOOLS     4.4   
    1789        2287  Menstrual Calendar Premium             MEDICAL     4.4   
    5707        6738                  BS Chopper                GAME     NaN   
    5513        6526  BN Pro Solid Battery-White  LIBRARIES_AND_DEMO     3.6   
    1509        1860   Love Nikki-Dress UP Queen                GAME     4.4   
    
          Reviews  Size    Installs  Type  Price Content Rating            Genres  \
    2585    24265   3.3  1,000,000+  Free      0       Everyone             Tools   
    1789     4207   NaN     50,000+  Paid  $3.99       Everyone           Medical   
    5707        0  41.0        100+  Free      0       Everyone            Racing   
    5513        7   0.2      1,000+  Free      0       Everyone  Libraries & Demo   
    1509   212524  93.0  5,000,000+  Free      0       Everyone      Role Playing   
    
              Last Updated         Current Ver         Android Ver  
    2585     July 25, 2017               4.0.6        4.0.3 and up  
    1789     July 26, 2013  Varies with device  Varies with device  
    5707   October 5, 2017                   1          4.1 and up  
    5513  February 5, 2017               2.3.2          1.6 and up  
    1509     July 24, 2018               3.0.0        4.0.3 and up  


## 2. Data cleaning
<p>The four features that we will be working with most frequently henceforth are <code>Installs</code>, <code>Size</code>, <code>Rating</code> and <code>Price</code>. The <code>info()</code> function (from the previous task)  told us that <code>Installs</code> and <code>Price</code> columns are of type <code>object</code> and not <code>int64</code> or <code>float64</code> as we would expect. This is because the column contains some characters more than just [0,9] digits. Ideally, we would want these columns to be numeric as their name suggests. <br>
Hence, we now proceed to data cleaning and prepare our data to be consumed in our analyis later. Specifically, the presence of special characters (<code>, $ +</code>) in the <code>Installs</code> and <code>Price</code> columns make their conversion to a numerical data type difficult.</p>


```python
# List of characters to remove
chars_to_remove = ['+', ',', '$']
# List of column names to clean
cols_to_clean = ["Installs", "Price"]

# Loop for each column
for col in cols_to_clean:
    # Replace each character with an empty string
    for char in chars_to_remove:
        apps[col] = apps[col].astype(str).str.replace(char,'')
    # Convert col to numeric
    apps[col] = pd.to_numeric(apps[col]) 
```

## 3. Exploring app categories
<p>With more than 1 billion active users in 190 countries around the world, Google Play continues to be an important distribution platform to build a global audience. For businesses to get their apps in front of users, it's important to make them more quickly and easily discoverable on Google Play. To improve the overall search experience, Google has introduced the concept of grouping apps into categories.</p>
<p>This brings us to the following questions:</p>
<ul>
<li>Which category has the highest share of (active) apps in the market? </li>
<li>Is any specific category dominating the market?</li>
<li>Which categories have the fewest number of apps?</li>
</ul>
<p>We will see that there are <code>33</code> unique app categories present in our dataset. <em>Family</em> and <em>Game</em> apps have the highest market prevalence. Interestingly, <em>Tools</em>, <em>Business</em> and <em>Medical</em> apps are also at the top.</p>


```python
import plotly
plotly.offline.init_notebook_mode(connected=True)
import plotly.graph_objs as go

# Print the total number of unique categories
num_categories = len(apps['Category'].unique())
print('Number of categories = ', num_categories)

# Count the number of apps in each 'Category' and sort them in descending order
num_apps_in_category = apps['Category'].value_counts().sort_values(ascending = False)

data = [go.Bar(
        x = num_apps_in_category.index, # index = category name
        y = num_apps_in_category.values, # value = count
)]

plotly.offline.iplot(data)
```

    Number of categories =  33



![](https://github.com/las5h4/The-Android-App-Market-on-Google-Play/blob/master/newplot(1).png)


## 4. Distribution of app ratings
<p>After having witnessed the market share for each category of apps, let's see how all these apps perform on an average. App ratings (on a scale of 1 to 5) impact the discoverability, conversion of apps as well as the company's overall brand image. Ratings are a key performance indicator of an app.</p>
<p>From our research, we found that the average volume of ratings across all app categories is <code>4.17</code>. The histogram plot is skewed to the left indicating that the majority of the apps are highly rated with only a few exceptions in the low-rated apps.</p>


```python
# Average rating of apps
avg_app_rating = apps['Rating'].mean()
print('Average app rating = ', avg_app_rating)

# Distribution of apps according to their ratings
data = [go.Histogram(
        x = apps['Rating']
)]

# Vertical dashed line to indicate the average app rating
layout = {'shapes': [{
              'type' :'line',
              'x0': avg_app_rating,
              'y0': 0,
              'x1': avg_app_rating,
              'y1': 1000,
              'line': { 'dash': 'dashdot'}
          }]
          }

plotly.offline.iplot({'data': data, 'layout': layout})
```

    Average app rating =  4.173243045387994

![](https://github.com/las5h4/The-Android-App-Market-on-Google-Play/blob/master/newplot.png)

## 5. Size and price of an app
<p>Let's now examine app size and app price. For size, if the mobile app is too large, it may be difficult and/or expensive for users to download. Lengthy download times could turn users off before they even experience your mobile app. Plus, each user's device has a finite amount of disk space. For price, some users expect their apps to be free or inexpensive. These problems compound if the developing world is part of your target market; especially due to internet speeds, earning power and exchange rates.</p>
<p>How can we effectively come up with strategies to size and price our app?</p>
<ul>
<li>Does the size of an app affect its rating? </li>
<li>Do users really care about system-heavy apps or do they prefer light-weighted apps? </li>
<li>Does the price of an app affect its rating? </li>
<li>Do users always prefer free apps over paid apps?</li>
</ul>
<p>We find that the majority of top rated apps (rating over 4) range from 2 MB to 20 MB. We also find that the vast majority of apps price themselves under \$10.</p>


```python
%matplotlib inline
import seaborn as sns
sns.set_style("darkgrid")
import warnings
warnings.filterwarnings("ignore")

# Filter rows where both Rating and Size values are not null
apps_with_size_and_rating_present = apps[(~apps['Rating'].isnull()) & (~apps['Size'].isnull())]

# Subset for categories with at least 250 apps
large_categories = apps_with_size_and_rating_present.groupby('Category').filter(lambda x: len(x) >= 250).reset_index()

# Plot size vs. rating
plt1 = sns.jointplot(x = large_categories['Size'], y = large_categories['Rating'], kind = 'hex')

# Subset apps whose 'Type' is 'Paid'
paid_apps = apps_with_size_and_rating_present[apps_with_size_and_rating_present['Type'] == 'Paid']

# Plot price vs. rating
plt2 = sns.jointplot(x = paid_apps['Price'], y = paid_apps['Rating'])
```


    
![png](output_9_0.png)
    



    
![png](output_9_1.png)
    


## 6. Relation between app category and app price
<p>So now comes the hard part. How are companies and developers supposed to make ends meet? What monetization strategies can companies use to maximize profit? The costs of apps are largely based on features, complexity, and platform.</p>
<p>There are many factors to consider when selecting the right pricing strategy for your mobile app. It is important to consider the willingness of your customer to pay for your app. A wrong price could break the deal before the download even happens. Potential customers could be turned off by what they perceive to be a shocking cost, or they might delete an app they’ve downloaded after receiving too many ads or simply not getting their money's worth.</p>
<p>Different categories demand different price ranges. Some apps that are simple and used daily, like the calculator app, should probably be kept free. However, it would make sense to charge for a highly-specialized medical app that diagnoses diabetic patients. Below, we see that <em>Medical and Family</em> apps are the most expensive. Some medical apps extend even up to \$80! All game apps are reasonably priced below \$20.</p>


```python
import matplotlib.pyplot as plt
fig, ax = plt.subplots()
fig.set_size_inches(15, 8)

# Select a few popular app categories
popular_app_cats = apps[apps.Category.isin(['GAME', 'FAMILY', 'PHOTOGRAPHY',
                                            'MEDICAL', 'TOOLS', 'FINANCE',
                                            'LIFESTYLE','BUSINESS'])]

# Examine the price trend by plotting Price vs Category
ax = sns.stripplot(x = popular_app_cats['Price'], y = popular_app_cats['Category'], jitter=True, linewidth=1)
ax.set_title('App pricing trend across categories')

# Apps whose Price is greater than 200
apps_above_200 = popular_app_cats[['Category', 'App', 'Price']][popular_app_cats['Price'] > 200]
apps_above_200
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
      <th>Category</th>
      <th>App</th>
      <th>Price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3327</th>
      <td>FAMILY</td>
      <td>most expensive app (H)</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>3465</th>
      <td>LIFESTYLE</td>
      <td>💎 I'm rich</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>3469</th>
      <td>LIFESTYLE</td>
      <td>I'm Rich - Trump Edition</td>
      <td>400.00</td>
    </tr>
    <tr>
      <th>4396</th>
      <td>LIFESTYLE</td>
      <td>I am rich</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>4398</th>
      <td>FAMILY</td>
      <td>I am Rich Plus</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>4399</th>
      <td>LIFESTYLE</td>
      <td>I am rich VIP</td>
      <td>299.99</td>
    </tr>
    <tr>
      <th>4400</th>
      <td>FINANCE</td>
      <td>I Am Rich Premium</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>4401</th>
      <td>LIFESTYLE</td>
      <td>I am extremely Rich</td>
      <td>379.99</td>
    </tr>
    <tr>
      <th>4402</th>
      <td>FINANCE</td>
      <td>I am Rich!</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>4403</th>
      <td>FINANCE</td>
      <td>I am rich(premium)</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>4406</th>
      <td>FAMILY</td>
      <td>I Am Rich Pro</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>4408</th>
      <td>FINANCE</td>
      <td>I am rich (Most expensive app)</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>4410</th>
      <td>FAMILY</td>
      <td>I Am Rich</td>
      <td>389.99</td>
    </tr>
    <tr>
      <th>4413</th>
      <td>FINANCE</td>
      <td>I am Rich</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>4417</th>
      <td>FINANCE</td>
      <td>I AM RICH PRO PLUS</td>
      <td>399.99</td>
    </tr>
    <tr>
      <th>8763</th>
      <td>FINANCE</td>
      <td>Eu Sou Rico</td>
      <td>394.99</td>
    </tr>
    <tr>
      <th>8780</th>
      <td>LIFESTYLE</td>
      <td>I'm Rich/Eu sou Rico/أنا غني/我很有錢</td>
      <td>399.99</td>
    </tr>
  </tbody>
</table>
</div>




    
![png](output_11_1.png)
    


## 7. Filter out "junk" apps
<p>It looks like a bunch of the really expensive apps are "junk" apps. That is, apps that don't really have a purpose. Some app developer may create an app called <em>I Am Rich Premium</em> or <em>most expensive app (H)</em> just for a joke or to test their app development skills. Some developers even do this with malicious intent and try to make money by hoping people accidentally click purchase on their app in the store.</p>
<p>Let's filter out these junk apps and re-do our visualization.</p>


```python
# Select apps priced below $100
apps_under_100 = popular_app_cats[popular_app_cats['Price'] < 100]

fig, ax = plt.subplots()
fig.set_size_inches(15, 8)

# Examine price vs category with the authentic apps (apps_under_100)
ax = sns.stripplot(x = apps_under_100['Price'], y = apps_under_100['Category'], jitter=True, linewidth=1)
ax.set_title('App pricing trend across categories after filtering for junk apps')
```




    Text(0.5, 1.0, 'App pricing trend across categories after filtering for junk apps')




    
![png](output_13_1.png)
    


## 8. Popularity of paid apps vs free apps
<p>For apps in the Play Store today, there are five types of pricing strategies: free, freemium, paid, paymium, and subscription. Let's focus on free and paid apps only. Some characteristics of free apps are:</p>
<ul>
<li>Free to download.</li>
<li>Main source of income often comes from advertisements.</li>
<li>Often created by companies that have other products and the app serves as an extension of those products.</li>
<li>Can serve as a tool for customer retention, communication, and customer service.</li>
</ul>
<p>Some characteristics of paid apps are:</p>
<ul>
<li>Users are asked to pay once for the app to download and use it.</li>
<li>The user can't really get a feel for the app before buying it.</li>
</ul>
<p>Are paid apps installed as much as free apps? It turns out that paid apps have a relatively lower number of installs than free apps, though the difference is not as stark as I would have expected!</p>


```python
trace0 = go.Box(
    # Data for paid apps
    y=apps[apps['Type'] == 'Paid']['Installs'],
    name = 'Paid'
)

trace1 = go.Box(
    # Data for free apps
    y=apps[apps['Type'] == 'Free']['Installs'],
    name = 'Free'
)

layout = go.Layout(
    title = "Number of downloads of paid apps vs. free apps",
    yaxis = dict(
        type = 'log',
        autorange = True
    )
)

# Add trace0 and trace1 to a list for plotting
data = [trace0, trace1]
plotly.offline.iplot({'data': data, 'layout': layout})
```


<div>                            <div id="fc292cbf-de8a-4d31-b8b9-a0108638e2fe" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("fc292cbf-de8a-4d31-b8b9-a0108638e2fe")) {                    Plotly.newPlot(                        "fc292cbf-de8a-4d31-b8b9-a0108638e2fe",                        [{"name": "Paid", "type": "box", "y": [100000, 100000, 100000, 10000, 1000, 50, 100, 100, 100, 1000, 1000, 500000, 100000, 100000, 100000, 10000, 50000, 100000, 10000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 100000, 5000, 5000, 1000, 500, 100000, 10000, 10000, 1000, 1000, 1000, 100, 100, 1000, 5000, 5000, 1000, 10000, 10000, 5000, 1000, 10000, 10000, 50000, 10000000, 1000000, 100000, 10000, 10000, 10000, 10000, 10000, 5000, 5000, 100000, 50000, 1000, 10000, 50000, 1000, 1000, 10000, 1000, 1000, 1000, 100, 5000, 5000, 500, 1000, 10000, 1000, 100, 1000, 500, 5000, 1000, 500, 1000, 5000, 10000, 1000, 10000, 100, 1000, 5000, 10000, 1000, 1000, 100, 50000, 1000, 1000, 5000, 5000, 1000, 10000, 50, 1000, 10000, 1000, 1000, 500, 500, 1000, 5000, 1000, 100, 1, 50, 10, 1, 1000000, 100000, 100000, 50000, 1000000, 1000000, 10000, 10000, 100000, 100000, 100000, 100, 100, 5000, 1000, 100, 1000, 500, 50000, 10000, 10000, 10000000, 100000, 100000, 100000, 10000, 10, 100000, 500000, 5, 10, 10, 500, 5000, 1000, 1000, 100, 100, 5000, 5000, 500, 5000, 10, 500, 100, 5000, 500, 100, 10000, 50000, 1, 1000000, 10, 10000, 500, 1000, 100, 1000, 10000, 10000, 100000, 1000, 5000, 100000, 10000, 1000, 10000, 1000, 10000, 100000, 1000, 5000, 10000, 10000, 10000, 10000, 100, 100, 5000, 100, 5, 100, 1000, 100, 10, 500, 1, 100, 10000, 10, 0, 100, 100, 1, 100, 5000, 500, 10, 5000, 100, 50, 1000, 50000, 100000, 100000, 50000, 100, 1000, 1000, 100, 10000, 100000, 10000, 5000, 5000, 10000, 1000, 10000, 5000, 10000, 100000, 50, 50000, 50000, 10000, 500, 1000, 10000, 5000, 10000, 1000, 10000, 5000, 50, 10000, 1000, 100000, 10000, 50000, 1000, 1000000, 10000, 100, 10, 50000, 10000, 500000, 1000, 10000, 10000, 50000, 10000, 10000, 100000, 1000, 10000, 5000, 100, 1000, 10000, 1000, 10, 100000, 1000, 5000, 100, 500, 5000, 1, 100, 5000, 1000, 100, 500, 50000, 100, 10, 100, 50, 10, 50, 10, 10, 0, 10000, 100000, 10000, 10000, 50000, 1000, 1000, 5000, 1000, 10000, 5000, 1000, 10000, 10000, 5000, 5000, 1000, 100000, 100000, 1000, 1000, 10, 100, 500, 100, 5, 10, 0, 10, 1000000, 1, 1000, 1000000, 50000, 500000, 1000000, 100000, 100000, 100000, 10000, 1000, 5000, 100, 1000, 100, 10000, 100000, 1000, 10, 1, 1, 10, 1, 1000, 1000, 1000, 500, 1000, 1, 5, 100, 10, 0, 5000, 1000, 1000, 100, 50, 50000, 10, 50, 100000, 100000, 1000, 1000, 1000, 5000, 50000, 500, 5000, 10, 100000, 50000, 10000, 100, 100000, 50000, 100000, 50000, 1, 10, 500, 100, 100, 10000, 50, 5, 10, 100, 50, 100, 500, 10000, 500, 1000, 500, 1000, 1000, 10, 5000, 1000, 100000, 1000, 1000, 100, 10, 1000, 1000, 100, 1000, 50, 1000, 1, 1000000, 100000, 5000, 0, 50000, 10, 50, 1000, 1000, 10000, 10, 5, 500, 100000, 1000, 1000, 100000, 1000, 500000, 10000, 100, 1000, 1000, 1000, 100, 100, 100, 500, 500, 10000, 100, 1000, 10000, 10000, 100, 50, 1000, 1, 1000, 1000, 1000, 10, 5000, 500, 50000, 5, 1000, 50, 10, 10, 1000, 1000000, 100000, 10000, 10000, 50000, 1000, 10000, 10, 10, 5, 1000000, 100, 10, 10, 10, 100000, 10000, 10000, 100, 10000, 100000, 100000, 100000, 10000, 10000, 50000, 10000, 10000, 5000, 100, 100, 1000, 10, 100, 10, 500, 100, 100, 100, 500000, 100, 10, 10, 100, 100, 1000, 50, 100, 100, 50000, 1000, 1000, 100, 100, 5000, 100, 100, 10000, 5000, 5000, 100, 10, 10, 1, 1000, 500, 10000, 1000, 100000, 10, 10, 10, 1000, 10, 1000, 1, 5, 10, 100000, 100000, 10000, 10000, 500, 50, 500, 10, 10, 1, 50000, 50, 5000, 10, 10000, 10, 10, 10, 100, 100, 10000, 50, 1000, 100, 10, 500000, 1000, 1, 50000, 10, 1000, 5, 50, 5000, 10000, 10, 5000, 1000, 50000, 50000, 10000, 10000, 100000, 10000, 100000, 100000, 100000, 1000, 10000, 500, 50000, 100000, 100, 1000, 10000, 50000, 100000, 50000, 1000000, 100, 50, 10000, 10, 50, 1000000, 100000, 1000, 10000, 100000, 5000, 10000, 1000, 100, 10000, 5000, 100000, 100, 10, 500, 50, 50, 50, 10, 100, 50, 500, 10000, 10000, 10, 1, 1, 1000000, 500000, 1000000, 1000000, 5, 1000, 1000, 500, 100, 10000, 1000, 50, 0, 50000, 50, 10000, 1000, 500000, 1000, 100000, 10000, 1000, 10000, 1000, 5000, 1000, 100000, 10000, 1000, 100, 100, 1000000, 500, 50000, 1000, 50, 100, 100000, 50, 500, 0, 100000, 10, 500000, 500, 50000, 1000, 10000, 0, 10, 0, 100, 10000, 0, 1000000, 10000, 100000, 100000, 500000, 10000, 50000, 5000, 5000, 1000, 5000, 5000, 5000, 5000, 50, 100, 500, 100000, 100, 50000, 100, 10000, 10000, 10000, 1000, 1000, 1000, 100, 5000, 100, 100000, 10000, 1000, 1000, 5000, 5000, 100000, 10, 10, 10, 10, 10, 10, 5, 10, 50, 100000, 1000000, 100, 1, 100, 1000, 10000, 10000, 50]}, {"name": "Free", "type": "box", "y": [10000, 500000, 5000000, 50000000, 100000, 50000, 50000, 1000000, 1000000, 10000, 1000000, 1000000, 10000000, 100000, 100000, 5000, 500000, 10000, 5000000, 10000000, 100000, 100000, 500000, 100000, 50000, 10000, 500000, 100000, 10000, 100000, 100000, 50000, 100000, 100000, 10000, 100000, 500000, 5000000, 10000, 500000, 10000, 100000, 10000000, 100000, 10000, 10000000, 100000, 100000, 100000, 100000, 1000000, 100000, 1000000, 100000, 100000, 100000, 50000, 100000, 100000, 100000, 10000, 100000, 1000000, 100000, 100000, 10000, 50000, 5000000, 100000, 5000000, 5000000, 500000, 10000000, 100000, 500000, 50000, 100000, 1000000, 100000, 1000000, 50000, 1000000, 500000, 100000, 1000000, 1000000, 100000, 100000, 1000000, 100000, 100000, 1000000, 1000000, 1000000, 1000000, 500000, 500000, 100000, 500000, 1000000, 100000, 500000, 1000000, 500000, 100000, 1000000, 50000, 1000000, 10000, 1000000, 100000, 10000, 50000, 100000, 10000, 10000, 10000, 10000000, 500000, 1000000, 50000, 10000, 1000000, 50000, 500000, 500000, 100000, 10000, 5000, 10000, 10000, 100000, 5000, 100000, 10000, 10000, 1000000, 50000, 10000, 100000000, 50000, 100000, 10000000, 100000000, 10000000, 10000000, 10000000, 100000, 1000000, 10000000, 500000, 1000000, 1000000000, 5000000, 100000, 10000000, 500000, 10000000, 1000000, 1000000, 500000, 1000000, 500000, 10000, 5000000, 100000, 5000000, 500000, 1000000, 500000, 500000, 1000000, 5000000, 10000000, 100000, 100000, 100000, 50000, 500000, 10000000, 50000, 1000000, 1000000, 1000000, 500000, 100000, 10000000, 10000000, 50000000, 10000000, 5000000, 1000000, 50000000, 5000000, 100000000, 1000000, 1000000, 500000, 10000000, 5000000, 1000000, 50000000, 10000000, 1000000, 10000000, 5000000, 5000000, 5000000, 5000000, 100000, 1000000, 1000000, 1000000, 10000000, 5000000, 1000000, 5000000, 100000, 10000000, 1000000, 1000000, 100000, 5000000, 1000000, 100000, 50000000, 5000000, 100000, 1000000, 1000000, 10000000, 10000000, 1000000, 50000, 5000000, 5000000, 100000, 100000, 1000000, 500000, 100000, 500000, 500000, 1000000, 1000000, 1000000, 100000, 100000, 1000000, 1000000, 10000000, 100000, 100000, 5000000, 50000, 1000000, 500000, 10000000, 10000, 10000000, 500000, 1000000, 500000, 50000, 50000, 50000, 1000000, 10000, 10000, 10000, 10000, 10000, 100000, 5000000, 100000, 1000000, 100000, 1000000, 1000000, 1000000, 1000000, 100000, 5000000, 50000, 100000, 10000, 10000, 5000, 500000, 1000000, 5000, 1000, 5000000, 10000, 50000, 1000000, 100000, 1000000000, 1000000000, 10000000, 1000000000, 100000000, 1000000000, 1000000000, 500000000, 5000000, 100000000, 100000000, 100000000, 500000000, 50000000, 5000000, 5000000, 100000000, 10000000, 10000000, 10000000, 10000000, 100000000, 1000000, 1000000, 10000000, 5000000, 10000000, 10000000, 100000000, 5000000, 100000000, 100000000, 10000000, 1000000, 100000000, 100000000, 500000000, 10000000, 1000000, 100000, 10000000, 10000000, 10000000, 500000000, 5000000, 5000000, 10000000, 10000000, 5000000, 50000000, 1000000000, 100000000, 1000000, 10000000, 10000000, 10000000, 10000000, 10000000, 500000000, 10000000, 1000000, 100000000, 100000000, 1000000, 50000000, 1000000, 50000000, 1000000, 1000000, 1000000, 10000000, 500000, 500000, 10000000, 10000000, 1000000, 1000000, 5000000, 10000000, 10000000, 1000000, 1000000, 10000000, 10000000, 5000000, 10000000, 5000000, 1000000, 1000000, 5000000, 1000000, 100000000, 1000000, 5000000, 10000000, 1000000, 1000000, 1000000, 10000000, 50000000, 10000, 5000000, 1000000, 500000, 1000000, 5000000, 10000000, 10000000, 10000000, 100000, 500000, 5000000, 1000000, 10000000, 1000000, 10000000, 5000000, 5000000, 100000, 500000, 1000000, 1000000, 10000, 1000000, 1000000, 100000, 10000000, 1000000, 100000, 500000, 5000, 500000, 5000000, 1000000, 100000, 1000000, 1000000, 100000, 100000, 500000, 500000, 500000, 100000, 500000, 500000, 100000, 50000, 1000000, 1000000, 100000, 100000, 100000, 10000, 100000, 1000000, 100000, 10000000, 500000, 100000, 1000000, 50000, 5000000, 500000, 100000, 100000, 500000, 100000, 10000, 100000, 1000000, 50000, 10000, 1000000, 100000, 10000, 10000, 500000, 500000, 500000, 10000, 100000, 100000, 500000, 1000000, 10000, 10000000, 10000, 100000, 10000, 1000000, 50000, 10000, 500000, 5000000, 1000000, 1000, 5000, 1000, 500, 5000, 10000, 100, 10000, 1000, 100, 1000, 1000, 1000, 100, 500, 500, 5000, 100, 100, 50, 5000, 50, 500, 10, 500, 500, 100, 1000, 10, 100, 10, 500, 500, 10, 100, 100, 10, 10, 100, 50, 100, 100, 10, 500, 1000, 500, 50, 1, 10, 1000, 1, 50, 100, 10000, 500, 1000, 10, 5, 10, 100, 1000, 500000, 500, 10000, 50000, 10000, 5000, 10000, 1000, 10000, 5000, 100000000, 10000000, 100000, 5000000, 10000000, 100000, 5000000, 1000000, 500000, 500000, 1000000, 1000000, 5000000, 1000000, 500000, 5000000, 500000, 1000000, 10000000, 10000000, 1000000, 5000000, 50000, 1000000, 1000000, 1000000, 1000000, 1000000, 500000, 1000000, 5000000, 1000000, 500000, 500000, 100000, 10000000, 100000, 5000000, 10000000, 10000000, 1000000, 5000000, 1000000, 5000000, 5000000, 10000000, 1000000, 1000000, 1000000, 1000000, 10000, 50000, 1000000, 1000000, 1000000, 1000000, 1000000, 1000000, 1000000, 1000000, 1000000, 1000000, 100000, 1000000, 100000, 10000, 10000, 50000, 500000, 100000, 100000, 100000, 10000, 100000, 100000, 50000, 50000, 100000, 100000, 5000000, 10000000, 1000000, 1000000, 5000000, 5000000, 1000000, 500000, 1000000, 1000000, 10000000, 1000000, 500000, 1000000, 1000000, 50000, 1000000, 100000, 100000, 100000, 100000, 10000000, 100000, 10000, 10000000, 10000000, 500000, 5000000, 500000, 1000000, 1000000, 1000000, 100000, 1000000, 100000, 1000000, 100000000, 1000000, 1000000, 10000000, 50000000, 10000000, 5000000, 5000000, 10000000, 5000000, 1000000000, 100000000, 5000000, 1000000, 5000000, 5000000, 1000000, 1000000, 10000000, 100000000, 5000000, 10000000, 1000000, 10000000, 50000000, 5000000, 10000000, 1000000, 10000000, 1000000, 10000000, 50000000, 1000000, 100000000, 50000000, 1000000, 5000000, 50000000, 100000000, 1000000, 100000, 1000000, 1000000, 10000000, 100000, 100000, 1000000, 500000, 100000, 100000, 1000000, 1000000, 10000000, 1000000, 10000000, 10000000, 10000000, 50000, 1000000, 10000000, 1000000, 5000000, 1000000, 5000000, 1000000, 10000000, 10000000, 1000000, 1000000, 1000000, 1000000, 1000000, 500000, 5000000, 10000000, 500000, 1000000, 1000000, 10000000, 1000000, 10000000, 10000000, 1000000, 10000000, 1000000, 10000000, 100000, 1000000, 10000, 100000, 100000, 5000000, 100000, 1000000, 1000000, 1000000, 1000000, 10000000, 10000000, 10000000, 100000, 5000000, 500000, 50000, 5000000, 1000000, 10000, 1000000, 100000, 10000, 1000000, 100000, 100000, 100000, 50000, 1000, 1000000, 10000, 500, 50000, 100, 100, 100000, 100000, 500, 100000, 100, 100000, 100, 1000, 5000, 100000, 1000, 100000, 100, 1000, 50000, 5000, 100000, 100, 1000, 1000, 100, 500, 10000000, 1000000, 5000000, 5000000, 5000000, 10000000, 500000, 5000000, 10000000, 1000000, 1000000, 10000000, 5000000, 1000000, 1000000, 10000000, 1000000, 5000000, 1000000, 1000000, 5000000, 1000000, 5000000, 1000000, 1000000, 10000000, 10000000, 1000000, 50000000, 10000000, 1000000, 1000000, 1000000, 10000000, 1000000, 100000000, 5000000, 1000000, 5000000, 1000000, 10000000, 10000, 1000000, 100000, 100000, 100000, 1000000, 1000000, 1000000, 1000000, 5000000, 500000, 1000000, 100000, 500000, 1000000, 1000000, 100000, 100000, 1000000, 1000000, 50000, 1000000, 100000, 100000, 1000000, 5000000, 1000000, 100000, 1000000, 100000, 1000000, 1000000, 50000, 1000000, 5000000, 500000, 1000000, 1000000, 1000000, 5000000, 1000000, 100000, 500000, 1000000, 1000000, 500000, 1000000, 5000000, 500000, 1000000, 5000000, 5000000, 50000, 100000, 1000000, 500000, 1000000, 500000, 1000000, 100000, 100000, 1000000, 1000000, 5000000, 50000, 10000000, 10000, 5000000, 1000000, 5000000, 1000000, 10000000, 10000000, 1000000, 10000000, 50000, 10000000, 1000000, 10000, 100000, 1000000, 10000000, 1000000, 500000, 10000000, 500000, 50000, 1000000, 1000000, 1000000, 500000, 5000000, 1000000, 5000000, 1000000, 10000000, 100000, 10000000, 5000000, 5000000, 1000000, 10000000, 5000000, 1000000, 1000000, 1000000, 1000000, 100000, 100000, 1000000, 50000, 500000, 500000, 100000, 1000000, 1000000, 100000, 100000, 5000000, 500000, 100000, 1000000, 10000, 50000, 5000000, 1000000, 1000000, 10000000, 1000000, 1000000, 1000000, 5000000, 10000000, 10000000, 50000, 100000, 1000000, 1000000, 100000, 10000000, 100000, 1000000, 10000000, 1000000, 5000000, 5000000, 10000000, 10000000, 500000, 5000000, 1000000, 10000000, 10000000, 10000000, 10000000, 10000000, 10000000, 1000000, 10000000, 1000000, 100000, 500000, 100000, 500000, 5000000, 1000000, 10000000, 50000, 10000000, 500000, 100000, 1000000, 10000000, 500000, 10000000, 5000000, 1000000, 50000000, 100000, 5000000, 10000000, 1000000, 100000, 10000000, 1000000, 500000, 100000, 10000000, 1000000, 5000000, 1000000, 500000, 1000000, 1000000, 1000000, 10000000, 10000000, 5000000, 1000000, 10000000, 500000, 10000000, 10000000, 1000000, 5000000, 1000000, 10000000, 10000000, 5000000, 1000000, 5000000, 1000000, 5000000, 1000000, 100000, 1000000, 500000, 500000, 100000, 1000000, 500000, 100000, 100000, 1000000, 5000000, 5000000, 500000, 10000000, 100000, 1000000, 1000000, 1000000, 1000000, 5000000, 1000000, 1000000, 1000000, 500000, 10000000, 500000, 1000000, 100000000, 10000000, 5000000, 5000000, 500000, 1000000, 1000000, 1000000, 10000000, 50000, 50000, 5000000, 5000000, 1000000, 1000000, 10000000, 10000000, 1000000, 500000, 1000000, 1000000, 100000, 10000000, 1000000, 100000, 50000, 500000, 1000000, 100000, 5000000, 100000, 500000, 100000, 1000000, 500000, 100000, 500000, 1000000, 500000, 500000, 1000000, 500000, 5000000, 1000000, 1000000, 10000000, 100000, 1000000, 10000000, 5000000, 1000000, 1000000, 1000000, 10000000, 10000, 10000000, 500000, 100000, 1000000, 1000000, 1000000, 1000000, 1000000, 100000, 5000000, 1000000, 1000000, 1000000, 5000, 5000000, 10000, 100000, 500000, 1000000, 100000, 1000000, 100000, 100000, 1000000, 500000, 500000, 5000000, 500000, 5000, 100000, 10000, 50000, 100000, 10000, 100000, 1000000, 100000, 500000, 1000000, 50000, 1000000, 500000, 1000000, 1000000, 500000, 500000, 50000, 50000, 10000, 100000, 1000, 100000, 10000, 100000, 5000000, 50000, 10000000, 1000, 100000, 50000, 100000, 50000, 100000, 10000, 50000, 50000, 100000, 100000, 10000000, 10000, 100000, 10000, 10000, 5000, 500000, 5000000, 100000, 1000000, 10000, 1000000, 1000, 500000, 1000000, 100000, 1000000, 10000, 1000000, 500000, 500000, 100000, 50000, 500000, 500000, 1000000, 100000, 5000000, 10000000, 100000, 10000000, 5000000, 1000000, 5000000, 1000000, 10000000, 1000000, 1000000, 5000000, 10000000, 500000, 5000000, 10000000, 1000000, 1000000, 10000000, 5000000, 1000000, 10000000, 50000000, 1000000, 500000, 500000, 1000000, 5000000, 1000000, 1000000, 100000, 1000000, 1000000, 10000000, 1000000, 1000000, 10000000, 500000, 10000000, 1000000, 1000000, 100000, 100000, 10000000, 500000, 1000000, 1000000, 1000000, 100000, 1000000, 1000000, 10000000, 100000, 500000, 10000, 100000, 5000000, 500000, 500000, 10000, 1000000, 10000000, 100000, 50000, 5000, 100000, 1000000, 100000, 50000, 1000000, 100000, 1000000, 5000000, 500000, 500000, 1000000, 500000, 1000000, 100000, 1000000, 100000, 100000000, 1000000000, 500000000, 10000000, 10000000, 50000000, 100000000, 100000000, 500000000, 500000000, 100000000, 5000000, 100000000, 100000000, 100000000, 100000000, 1000000, 100000000, 50000000, 5000000, 100000000, 1000000, 100000000, 50000000, 50000000, 10000000, 50000000, 100000000, 100000000, 10000000, 10000000, 10000000, 100000000, 5000000, 10000000, 50000000, 100000000, 50000000, 10000000, 10000000, 10000000, 100000000, 100000000, 10000000, 100000000, 100000000, 100000000, 100000000, 10000000, 100000000, 50000000, 10000000, 10000000, 100000000, 50000000, 500000000, 5000000, 10000000, 100000000, 100000000, 10000000, 100000000, 10000000, 100000000, 100000000, 10000000, 50000000, 1000000, 50000000, 100000000, 100000000, 100000000, 10000000, 50000000, 10000000, 1000000, 10000000, 5000000, 5000000, 100000000, 1000000, 1000000, 10000000, 10000000, 500000, 1000000, 100000000, 5000000, 5000000, 10000000, 50000000, 50000000, 10000000, 10000000, 10000000, 50000000, 10000000, 5000000, 10000000, 1000000, 1000000, 10000000, 10000000, 5000000, 10000000, 10000000, 1000000, 5000000, 5000000, 5000000, 10000000, 5000000, 5000000, 1000000, 5000000, 10000000, 1000000, 1000000, 5000000, 1000000, 10000000, 10000000, 10000000, 1000000, 1000000, 5000000, 10000000, 5000000, 10000000, 100000, 1000000, 1000000, 100000, 100000000, 10000000, 10000000, 10000000, 50000000, 5000000, 10000000, 10000000, 10000000, 10000000, 500000, 1000000, 5000000, 50000000, 5000000, 10000000, 500000, 50000000, 5000000, 1000000, 100000000, 1000000, 10000000, 5000000, 50000000, 100000000, 50000000, 50000000, 50000000, 10000000, 100000000, 10000000, 10000000, 50000000, 10000000, 50000000, 50000000, 10000000, 10000000, 10000000, 10000000, 100000, 10000000, 10000000, 100000000, 1000000, 1000000, 5000000, 10000000, 10000000, 10000000, 10000000, 100000000, 5000000, 5000000, 10000000, 100000, 50000000, 5000000, 50000000, 10000000, 10000000, 5000000, 10000000, 10000000, 10000000, 10000000, 5000000, 10000000, 1000000, 10000000, 10000000, 10000000, 5000000, 10000000, 100000000, 50000000, 10000000, 1000000, 5000000, 5000000, 1000000, 1000000, 5000000, 1000000, 1000000, 500000, 500000, 5000000, 1000000, 10000000, 500000, 1000000, 50000000, 10000000, 1000000, 10000000, 1000000, 5000000, 500000, 1000000, 1000000, 10000000, 10000000, 10000000, 10000000, 1000000, 1000000, 10000000, 100000, 5000000, 100000, 10000, 5000000, 100000, 1000000, 5000000, 1000000, 1000000, 1000000, 1000000, 5000000, 500000, 500000, 1000000, 1000000, 50000, 1000000, 1000000, 10000000, 100000, 100000, 500000, 500000, 1000000, 100000, 1000000, 5000000, 500000, 10000000, 1000000, 1000000, 100000, 50000, 50000000, 500000, 10000000, 500000, 1000000, 10000000, 1000000, 5000000, 1000000, 1000000, 10000000, 10000000, 50000000, 10000000, 5000000, 100000, 10000000, 10000000, 1000000, 1000000, 10000000, 1000000, 10000000, 10000000, 5000000, 1000000, 50000000, 1000000, 10000000, 500000, 500000, 500000, 100000, 100000, 1000000, 10000, 10000000, 500000, 10000000, 5000000, 1000000, 1000000, 1000000, 100000, 100000, 50000, 500000, 100000, 5000000, 1000, 100000, 50000, 10000000, 1000000, 1000000, 1000000, 100000, 100000, 5000000, 100000, 10000000, 5000000, 5000000, 10000000, 5000000, 5000000, 10000000, 1000000, 10000000, 5000000, 1000000, 5000000, 1000000, 1000000, 1000000, 1000000, 5000000, 5000000, 10000000, 1000000, 1000000, 1000000, 1000000, 1000000, 500000, 50000, 5000000, 1000000, 1000000, 100000, 500000, 10000, 1000000, 500000, 500000, 5000000, 50000, 500000, 1000000, 500000, 100000, 100000, 100000, 100000, 500000, 100000, 1000000, 1000000, 50000, 1000000, 100000, 100000, 100000, 100000, 1000000, 50000, 100000, 100000, 1000, 500000, 10000, 1000000, 1000, 100000, 500000, 100000, 50000, 500000, 10000, 1000000, 100000, 5000, 100000, 100000, 100000, 5000, 100000, 100000, 100000, 50000, 5000, 10000, 10000, 50000, 10000, 100000, 100000, 50000, 10000, 50000, 10000, 10000, 50000, 100000, 5000, 1000, 100000, 100000, 100000, 5000, 10000, 1000, 5000, 10000, 5000, 1000, 10000, 10000, 5000, 50000, 1000, 100000, 500, 100, 500, 1000, 50, 100, 100, 10, 10, 100, 100, 10, 100, 50, 10, 100, 1000, 10, 100, 10, 500, 50, 100, 10, 5, 10, 50, 10, 1, 50, 100, 10, 10, 5, 10, 10, 1, 100, 5000, 5, 10, 10, 10, 5, 50, 1, 10, 50, 5, 100, 100, 10, 5, 100, 10, 100, 10, 5, 5, 10, 100000, 10000, 1000, 10000, 1000, 100000, 10000, 500, 10000, 50000, 1000, 100000, 5000, 1000000, 10000, 5000, 10000, 10000, 10000, 100, 5000, 10000, 10000, 50000, 1000, 1000, 1000, 50000, 50000, 100, 1000, 50000, 10000, 1000, 10000, 500, 100000, 5000, 1000000000, 1000000000, 500000000, 10000000, 100000000, 1000000, 500000000, 100000, 100000000, 10000000, 1000000000, 1000000, 1000000, 1000000, 5000000, 100000, 1000000, 5000000, 5000000, 1000000, 5000000, 10000000, 5000000, 1000000, 500000, 5000000, 50000, 10000000, 1000000, 5000000, 1000000, 1000000, 10000000, 10000000, 10000000, 10000000, 10000000, 1000000, 100000000, 1000000, 10000000, 5000000, 5000000, 100000000, 10000000, 10000000, 10000000, 100000000, 50000000, 10000000, 50000000, 10000000, 5000000, 10000000, 5000000, 500000, 100000, 1000000, 500000, 10000000, 500000, 5000000, 50000000, 1000000, 10000000, 5000000, 1000000, 10000000, 1000000, 50000000, 10000000, 10000000, 100000, 50000000, 10000000, 10000000, 100000, 10000000, 10000000, 10000000, 100000000, 10000000, 10000000, 10000000, 5000000, 100000000, 10000000, 100000000, 50000000, 100000000, 50000000, 50000000, 1000000, 10000000, 5000000, 5000000, 1000000, 1000000, 10000000, 50000000, 10000000, 10000000, 1000000, 5000000, 10000000, 1000000, 10000000, 1000000, 10000000, 50000000, 10000000, 10000000, 10000000, 10000000, 10000000, 10000000, 10000000, 10000000, 50000000, 10000000, 10000000, 10000000, 10000000, 100000000, 5000000, 1000000, 5000000, 1000000, 1000000, 1000000, 1000000, 10000000, 500000, 500000, 1000000, 10000000, 10000000, 1000000, 10000000, 10000000, 50000000, 5000000, 10000000, 1000000, 10000000, 10000000, 50000, 1000000, 5000000, 1000000, 1000000, 1000000, 100000, 500000, 500000, 500000, 5000000, 100000, 1000000, 5000000, 5000000, 1000000, 1000000, 1000000, 5000000, 500000, 10000000, 10000000, 1000000, 1000000, 10000000, 10000000, 1000000, 500000, 1000000, 10000000, 100000, 1000000, 100000, 5000000, 1000000, 500000, 1000000, 5000000, 1000000, 500000, 50000, 1000000, 1000000, 1000000000, 50000000, 5000000, 10000000, 10000000, 10000000, 50000000, 1000000, 100000000, 1000000, 100000, 5000000, 1000000, 5000000, 1000000, 1000000, 5000000, 50000000, 500000, 10000000, 10000000, 50000000, 500000, 1000000, 1000000, 100000000, 1000000, 100000000, 50000000, 10000000, 1000000, 1000000, 5000000, 10000000, 1000000, 1000000, 5000000, 10000000, 10000000, 100000000, 10000000, 100000000, 10000000, 1000000, 5000000, 5000000, 10000000, 10000000, 10000000, 50000000, 10000000, 5000000, 100000000, 10000000, 1000000, 100000000, 50000000, 1000000, 100000000, 1000000, 1000000, 1000000, 1000000, 5000000, 100000000, 10000000, 1000000, 10000000, 5000000, 5000000, 10000000, 10000000, 10000000, 5000000, 1000000, 10000000, 10000000, 100000000, 10000000, 1000000, 50000000, 5000000, 10000000, 10000000, 100000000, 50000000, 50000000, 50000000, 10000000, 5000000, 10000000, 1000000, 10000000, 1000000, 10000000, 50000000, 50000000, 50000000, 50000000, 10000000, 100000000, 100000000, 10000000, 100000000, 100000000, 5000000, 10000000, 50000000, 10000000, 1000000, 10000000, 5000000, 10000000, 10000000, 10000000, 100000, 10000000, 5000000, 50000000, 10000000, 10000000, 500000, 10000000, 1000000, 5000000, 5000000, 5000000, 10000000, 1000000, 10000000, 5000000, 5000000, 5000000, 5000000, 5000000, 1000000, 1000000, 10000000, 5000000, 10000000, 10000000, 5000000, 100000, 1000000, 1000000, 1000000, 5000000, 100000, 500000, 500000, 500000, 1000000, 10000000, 100000, 5000000, 100000, 100000, 500000, 10000000, 100000, 1000000, 500000, 100000, 10000000, 1000000, 5000000, 100000, 1000000, 1000000, 100000, 1000000, 100000, 1000000, 1000000, 100000, 50000, 500000, 100000, 1000000, 100000, 1000000, 1000000, 1000000, 1000000, 1000000, 1000000, 1000000, 10000, 50000, 100000, 1000000, 100000, 1000000, 10000, 500000, 5000000, 100000, 5000000, 500000, 500000, 500000, 10000, 10000000, 50000000, 100000, 10000000, 5000000, 1000000, 5000000, 10000000, 10000000, 50000000, 5000000, 1000000, 500000, 1000000, 10000000, 10000000, 500000, 10000000, 100000000, 1000000, 5000000, 100000000, 10000000, 1000000000, 5000000, 5000000, 100000000, 10000000, 5000000, 10000000, 50000000, 10000000, 1000000000, 10000000, 10000000, 1000000, 1000000, 1000000, 5000000, 500000, 100000, 5000000, 5000000, 1000000, 1000000, 10000000, 10000000, 10000000, 10000000, 5000000, 5000000, 10000000, 1000000, 50000000, 5000000, 1000000, 5000000, 1000000, 1000000, 5000000, 1000000, 5000000, 10000000, 10000000, 10000000, 1000000, 100000, 1000, 100000, 5000000, 10000000, 1000000, 1000000, 1000000, 100000, 1000000, 5000000, 1000000, 1000000, 1000000, 1000000, 5000000, 1000000, 1000000, 1000000, 10000000, 5000000, 1000000, 1000000, 1000000, 1000000, 100000, 5000000, 1000000, 1000000, 5000000, 5000000, 10000000, 1000000, 100000, 1000000, 10000000, 1000000, 5000000, 1000000, 1000000, 500000, 500000, 1000000, 10000000, 1000000000, 500000000, 10000000, 50000000, 50000000, 100000000, 1000000, 10000000, 100000000, 100000000, 10000000, 50000000, 50000000, 10000000, 10000000, 5000000, 10000000, 1000000, 10000000, 100000, 500000000, 10000000, 10000000, 1000000, 5000000, 10000000, 100000000, 10000000, 10000000, 10000000, 500000000, 100000000, 10000000, 10000000, 10000000, 10000000, 10000000, 100000000, 10000000, 1000000, 500000, 10000000, 10000000, 50000, 100000, 100000, 10000000, 10000000, 1000000, 5000000, 1000000, 10000000, 50000000, 10000000, 100000000, 10000000, 50000000, 10000000, 5000000, 50000000, 10000000, 5000000, 5000000, 50000000, 5000000, 10000000, 5000000, 5000000, 1000000, 1000000, 1000000, 1000000, 10000000, 10000000, 1000000, 5000000, 1000000, 100000, 1000000, 100000, 50000000, 5000000, 1000000, 1000000, 50000000, 10000000, 1000000, 1000000, 5000000, 5000000, 10000000, 10000000, 50000000, 100000000, 10000000, 10000000, 1000000, 100000000, 10000000, 1000000, 1000000, 10000000, 5000000, 1000000, 1000000, 10000000, 10000000, 100000000, 10000000, 1000000, 1000000, 10000000, 100000000, 10000000, 10000000, 1000000, 50000000, 1000000, 100000000, 5000000, 100000, 5000000, 5000000, 1000000, 100000000, 500000, 1000000, 1000000, 1000000, 50000000, 1000000, 10000000, 10000000, 10000000, 10000000, 10000000, 10000000, 10000000, 100000000, 10000000, 10000000, 500000, 10000000, 10000000, 1000000, 1000000, 50000000, 1000000, 10000000, 100000000, 500000, 1000000, 1000000, 10000000, 5000000, 5000000, 500000, 500000, 5000000, 100000, 500000, 100000, 10000000, 50000000, 5000000, 1000000, 1000000, 1000000, 5000000, 10000000, 5000000, 10000000, 5000000, 10000000, 5000000, 1000000, 10000000, 10000000, 10000000, 10000000, 10000000, 10000000, 10000000, 5000000, 100000000, 1000000, 5000000, 50000000, 1000000, 1000000, 1000000, 1000000, 5000000, 1000000, 10000000, 1000000, 10000000, 10000000, 10000000, 10000000, 100000000, 1000000, 10000000, 1000000, 500000000, 10000000, 100000000, 10000000, 1000000000, 10000000, 10000000, 100000000, 10000000, 100000000, 5000000, 1000000, 10000000, 100000000, 50000000, 100000000, 10000000, 100000000, 5000000, 100000000, 10000000, 10000000, 1000000, 500000000, 10000000, 10000000, 500000000, 100000000, 10000000, 5000000, 5000000, 5000000, 100000000, 10000000, 50000000, 10000000, 100000000, 5000000, 50000000, 100000000, 50000000, 50000000, 50000000, 100000000, 1000000, 50000000, 1000000, 500000, 1000000, 1000000, 10000000, 1000000, 10000000, 1000000, 10000000, 1000000, 1000000, 10000000, 5000000, 10000000, 1000000, 10000000, 1000000, 50000000, 100000, 5000000, 10000000, 1000000, 1000000, 100000000, 10000000, 100000000, 100000000, 100000000, 5000000, 1000000, 5000000, 1000000, 10000000, 1000000, 5000000, 10000000, 1000000, 500000, 1000000, 1000000, 1000000, 1000000, 5000000, 5000000, 1000000, 10000000, 5000000, 10000000, 5000000, 100000, 1000000, 1000000, 1000000, 5000000, 500000000, 10000, 5000000, 10000, 1000000, 10000, 10000, 100000, 10000, 5000, 1000000, 50000, 50000, 100000, 1000000, 100000, 10000, 1000000, 1000000, 10000, 50000, 10000, 50000, 1000000, 500000, 100000, 100000, 100000, 100000, 10000, 100000, 1000000, 5000, 500000, 100000, 10000, 500000, 10000, 100000, 1000000, 100000, 100000, 10000, 100000, 1000000, 1000000, 100000, 100000, 1000000, 100000, 10000, 1000000, 50000000, 1000000, 50000000, 10000, 10000000, 1000000, 10000000, 1000000, 1000000, 1000000, 10000000, 5000000, 500000, 10000000, 1000000, 5000000, 5000000, 1000000, 10000000, 5000000, 1000000, 5000000, 1000000, 50000000, 1000000, 100000, 10000000, 1000000, 10000000, 500000, 1000000, 10000000, 1000000, 500000, 500000, 5000000, 100000, 1000000, 1000000000, 1000000, 10000000, 1000000, 1000000, 100000000, 10000000, 100000000, 100000, 500000, 100000000, 10000000, 50000000, 1000000, 1000000, 10000000, 1000000, 5000000, 1000000, 10000000, 10000000, 50000000, 1000000000, 10000000, 1000000, 50000000, 50000000, 1000000, 50000000, 5000000, 1000000, 5000000, 1000000, 1000000, 1000000, 10000000, 10000000, 10000000, 500000000, 1000000, 500, 1000000, 1000000, 1000000, 10000, 100000, 10000000, 1000000, 10000000, 10000000, 1000000, 1000000, 10000000, 1000000, 1000000, 5000000, 1000000, 10000000, 5000000, 10000000, 1000000, 10000000, 50000000, 1000000, 500000, 1000000, 10000000, 10000000, 100000, 1000000000, 10000000, 10000000, 500000000, 10000000, 500000, 500000, 1000000, 1000000, 1000000, 1000000, 5000000, 5000000, 50000, 1000000, 1000000, 1000000, 1000000, 5000000, 500000000, 1000000, 1000000, 5000000, 1000000, 1000000, 500000, 1000000, 10000, 1000000, 5000000, 1000000, 100000, 1000000, 100000, 5000000, 1000000, 1000000, 1000000, 5000000, 10000000, 100000, 1000000, 1000000, 1000000, 1000000, 10000000, 100000, 1000000, 5000000, 500000, 1000000, 1000000, 10000000, 10000000, 10000000, 10000000, 1000000, 1000000, 1000000, 1000000, 10000000, 5000000, 1000000, 100000000, 5000000, 10000000, 10000000, 5000000, 5000000, 1000000, 100000000, 50000000, 500000, 10000000, 1000000, 1000000, 1000000, 10000000, 1000000, 100000, 5000000, 5000000, 5000000, 5000000, 1000000, 1000000, 1000000, 1000000, 100000, 100000, 1000000, 1000000, 5000000, 1000000, 1000000, 5000000, 1000000, 1000000, 5000000, 1000000, 5000000, 1000000, 1000000, 10000000, 100000, 5000000, 1000000, 1000000, 10000000, 50000, 1000000, 1000000, 5000000, 50000000, 1000000, 1000000, 500000, 10000000, 50000000, 5000000, 1000000, 10000, 10000000, 50000000, 500000, 10000, 10000, 500000, 50000000, 500000, 100000000, 1000000, 500000, 10000000, 50000000, 50000, 10000000, 100000000, 100000000, 10000000, 100000000, 500000, 500000, 10000000, 50000000, 10000000, 10000000, 1000000, 5000000, 5000000, 500000, 10000000, 500000, 5000000, 500000, 1000000, 1000000, 100000000, 5000000, 1000000, 100000000, 1000000, 1000000, 10000000, 50000, 100000000, 10000, 10000, 10000000, 50000000, 10000, 500000, 1000000, 50000000, 50000, 50000, 50000000, 100000000, 1000, 5000, 50000000, 100000, 50000, 1000, 500, 50000, 100000, 10000000, 1000000, 50000, 10000000, 50000, 1000, 100000, 500000000, 10000, 1000000, 10000, 100000, 1000, 5000, 50000, 5000, 10000, 100000, 5000, 10000, 100000, 100000, 50000, 1000, 100000, 5000, 5000, 50000, 5000, 10000, 10000, 5000, 100000000, 10000000, 10000000, 100000000, 50000000, 50000000, 100000000, 100000000, 100000, 50000000, 50000000, 10000000, 100000000, 1000000, 100000000, 500000, 100000000, 50000000, 100000000, 1000000, 10000000, 10000000, 5000000, 100000, 1000000, 10000000, 5000000, 10000000, 500000, 500000, 10000000, 100000000, 100000, 500000, 500000, 1000000, 100000, 5000000, 500000, 10000000, 1000000, 50000000, 500000, 1000000, 100000000, 100000, 5000, 50000, 100000, 50000, 50000, 500000, 1000, 50000, 500000, 5000, 50000000, 100000, 5000, 500, 500000, 10000, 50000, 500000, 5000, 10000, 50000000, 5000000, 100000000, 100000, 1000000, 100000, 10000, 50000000, 100000, 500000, 100000, 10000, 50000, 50000, 50000, 50000, 1000, 10000, 5000000, 100000, 10000000, 100000, 100000000, 5000, 1000000, 1000, 500000, 10000, 5000, 1000000, 5000, 1000, 10000, 10000, 10000000, 50000, 1000000, 100, 1000, 10000, 50000, 50000, 10000, 1000000, 100000000, 10000000, 500000, 5000000, 500000, 1000000, 1000000, 1000000, 100000000, 500000, 10000000, 10000000, 1000000, 5000000, 5000000, 5000000, 10000000, 1000000, 1000000, 100000000, 10000000, 100000, 10000, 10000, 10000000, 50000000, 1000, 10000, 100000, 10000000, 1000, 50000, 1000, 10000000, 1000, 500, 1000, 10000, 1000, 100, 1000, 10000, 10000, 10000, 500, 100000000, 50000, 500000, 100000, 50000000, 10000000, 100000, 1000, 100000, 100000, 100000, 100000, 5000, 1000000, 500000, 100000, 1000, 1000, 1000, 100000, 10000, 5000, 100, 5000, 1000, 10000, 5000, 1000000, 100000, 10000, 10000, 10000, 10000, 50000000, 5000000, 50000000, 500000, 100000000, 1000000, 50000, 50000, 100000, 10000000, 1000000, 500000, 1000000, 100000000, 10000000, 100000, 100000, 50000, 10000000, 10000000, 100000, 50000000, 10000000, 1000000, 10000, 1000000, 1000000, 50000000, 100000, 10000, 100000, 10000, 100000, 5000, 50000, 5000, 100000, 10000000, 100000, 10000, 10000, 10000, 10000, 10000, 10000000, 10000, 1000000, 5000000, 1000000, 1000000, 500000, 1000000, 10000, 10000000, 10000, 50000, 1000000, 100000, 100000, 10000, 1000000, 5000000, 5000000, 1000000, 1000000, 500000, 50000, 1000000, 50000, 100000, 100000, 10000, 50000, 10000, 1000000, 100000, 1000000, 10000000, 50000, 100000, 50000, 50000, 1000000, 500000, 1000000, 100000, 100000, 500000, 10000000, 100000, 50000, 10000000, 10000000, 1000, 100000, 1000000, 5000, 10000000, 10000, 50000, 500, 5000, 10000, 100000000, 1000, 100000, 1000, 10000, 100000, 10000, 500, 1000, 10000, 1000, 10000, 10000, 100000, 500000, 1000000, 500000, 500000, 10000, 50000, 1000000, 1000000, 1000000, 10000000, 500, 500000, 100000, 1000, 5000, 10000, 100000, 10000, 500000, 10000, 100000, 100000, 50000000, 10000000, 1000, 1000, 10000, 10000, 5000, 5000, 100000, 100000, 10000, 1000, 1000, 1000, 500000, 100000, 1000000, 5000, 1000, 1000, 10000, 10000, 5, 10000, 1000, 10, 100000, 1000, 50, 1, 10, 5000, 50, 5000, 1000, 5000, 100000, 100, 100000, 500, 10, 1000000, 100000000, 1000000, 1000000, 100000, 10000000, 100000000, 10000000, 1000000, 100000000, 1000000, 50000000, 500000, 50000, 50000, 1000000, 100000000, 10000000, 10000000, 1000000, 1000000, 10000, 100000, 1000000, 1000000, 10000000, 5000000, 10000000, 500000, 50000000, 10000000, 10000, 10000000, 5000000, 10000000, 1000, 5000000, 100000, 1000000, 5000, 50000, 1000000, 1000000, 1000000, 500000, 5000000, 10000000, 10000000, 500000, 10000, 100000, 10000, 50000, 50000, 50000, 100000, 1000, 500000, 10000, 100000, 500000, 5000, 10000, 10000, 100000, 10000, 100000, 50000, 10000000, 10000, 500000, 5000, 10000, 100000, 100000, 100000, 100000, 100000, 500000, 100000, 5000, 10000, 10000000, 1000000, 50000000, 100000, 5000000, 1000000, 100000, 500000, 10000000, 10000000, 50000000, 1000000, 50000000, 100000, 100000000, 500000, 5000, 100000, 1000000, 100000, 100000, 100000000, 1000000, 10000, 10000, 1000, 100000, 500000, 5000, 5000, 10000, 100000, 100000, 5000, 5000, 5000, 50000000, 50000, 50000000, 10000, 5000, 100000, 10000000, 10000, 1000000, 10000, 1000000, 5000, 5000000, 50000, 10000000, 10000000, 5000000, 10000, 50000000, 5000000, 100000, 5000000, 100000, 5000000, 100000, 50000, 10000000, 1000000, 1000000, 100000, 1000000, 5000000, 10000000, 500000, 10000, 1000000, 100000, 1000000, 1000000, 500000, 50000, 100000, 500000, 1000000, 100000, 1000000, 1000000, 1000000, 100000, 100000, 500000, 100000, 100000, 1000000, 500000, 10000000, 100000000, 50000000, 10000000, 10000000, 10000000, 1000000, 5000000, 5000000, 100000000, 10000000, 1000000, 100000000, 10000000, 10000000, 10000000, 5000000, 50000000, 1000000, 1000000, 10000000, 1000000, 5000000, 100000, 10000000, 1000000, 1000000, 5000000, 50000, 10000, 100000, 100000, 500000, 1000000, 10000, 1000000, 10000, 5000000, 10000, 1000000, 500000, 100000, 5000, 1000000, 10000, 5000000, 500000, 10000000, 1000, 1000000, 10000000, 5000000, 10000000, 10000000, 1000000, 100000, 100000, 100000, 10000000, 5000, 10000000, 1000000, 50000000, 1000000, 10000, 100000, 500000, 10000, 100000, 1000, 100000, 500, 50000, 50000000, 10000, 1000000, 1000, 100000000, 10000, 1000000, 1000, 1000, 1000000, 1000000, 10000, 1000000, 100000, 500000, 100000, 1000000, 500000, 100000, 5000, 10000, 1000000, 50000, 100000, 10000, 10000, 10000, 100000, 5000, 50000, 5000, 50000, 1000000, 100000, 10000, 100000, 100000, 50000, 10000, 10000, 10000, 100000, 10000000, 1000000, 100000, 100000, 50000, 1000, 50000, 100000, 10000, 1000000, 500000, 10000000, 500000, 1000000, 1000000, 500000, 10000000, 1000000, 50000, 1000000, 50000, 1000000, 50000, 1000000, 10000, 1000000, 10000, 100000, 100000, 1000, 100000, 1000000, 5000000, 500000, 50000, 50000, 100000, 100000, 100000, 100000, 100000, 1000000, 5000000, 50000, 100000, 5000, 10000, 100000, 1000000, 500000, 50000, 500000, 1000000, 100000, 1000000, 1000000, 50000, 50000, 500000, 100000, 1000000, 1000, 10000, 5000000, 1000, 1000, 100000, 100000, 10000, 10000, 10000, 10000, 10, 10000000, 10000, 10000, 1000000, 500000, 100, 50000, 10000, 1000, 5000000, 1000, 10000, 10000000, 10000, 10000, 100, 1000, 10000, 10000, 1000, 5000000, 10000000, 5000, 100, 500, 10000, 5000, 50, 100000000, 10000000, 100, 500000, 100, 5000000, 1000, 1000000, 1000, 1000000, 100, 5000000, 1000, 10000000, 100, 10000000, 10000000, 5000000, 100000000, 10000000, 1000000, 1000000, 500000, 1000000, 10000, 10000, 1000, 5000, 500, 10000, 1000, 10000, 1000, 1000, 100000, 100, 5000, 1000, 100, 10, 100, 1000, 10000, 1000, 100, 1000, 100, 50, 100, 1000, 10, 50, 50, 50, 100, 500, 10000, 100, 100, 100, 10000, 100, 1000, 5, 500, 10, 50000, 100, 100000, 100, 100, 1000, 50, 100, 50, 10000, 50000, 100, 50, 100, 100, 10000, 100, 100000, 100, 5000, 100, 50, 100, 100, 100, 100, 500, 50, 100000, 50, 1, 50, 100, 100, 100, 100, 10, 100000, 100, 100, 50, 1, 50, 100, 10, 50000, 50, 100, 1000000, 100000, 50000, 1000000, 1000, 50000, 1000000, 10000, 100000, 10000, 50000, 1000000, 10000, 1000000, 10000, 5000, 100, 1000, 1000, 100, 1000, 100000, 100000, 500, 1000000, 1000, 100000, 5000, 10000, 5000000, 10000000, 100, 100000, 100, 100, 5000, 500, 1000, 10000, 10000, 50000, 100000, 10000, 1000000, 1000, 100, 10, 100, 1000000, 10, 500, 10000, 500, 500, 10, 1000, 1000, 100, 100, 10, 10, 10000, 100, 100, 100, 50, 5000, 100, 5, 1000, 50, 100000, 5, 5, 10, 500000, 10, 100, 500, 100000, 100000, 100, 500000, 50000, 1000, 10000, 1000, 5000, 100, 10000, 10000000, 50000, 10000, 500, 1000000, 500, 500, 100, 1000, 100, 10000000, 10000, 10, 5000, 10, 500, 1000, 1000, 10, 100, 5000, 10000, 5000000, 100, 5000000, 1000, 50000, 10000000, 50000, 50000, 1000, 10000, 10000, 10000, 10000000, 50000, 500000, 10000000, 10000000, 10000000, 1000000, 100000, 1000000, 50000, 1000000, 100000, 1000000, 5000000, 100000, 5000000, 500000, 10000, 50000, 500000, 1000000, 1000000, 1000000, 100000, 50000, 10000000, 50000, 50000, 100000, 10000000, 100000, 10000, 1000000, 1000, 10000, 10000, 100000, 100000, 100000, 100000, 500000, 10000, 1000000, 1000000, 100000, 10000000, 10000000, 5000000, 10000000, 10000000, 10000000, 1000000, 5000000, 5000000, 5000000, 10000000, 5000000, 1000000, 10000000, 1000000, 100000, 1000000, 5000, 1000, 1000000, 10000, 100, 10000000, 100000, 1000000, 1000000, 10000000, 100000000, 10000000, 10000000, 5000000, 1000000, 5000000, 5000000, 10000000, 5000000, 100000, 10000000, 1000000, 10000000, 10000000, 100000, 10000000, 5000000, 1000000, 1000000, 10000000, 50000000, 10000, 100000, 100000, 10000000, 5000000, 10000000, 10000, 10000000, 1000000, 5000, 1000, 100000, 10000, 10000, 10000, 10000, 5000, 100, 5000, 1000, 5000, 1000, 10000, 500000, 100000, 100000, 10000, 10000, 5000, 10000, 10000, 10000, 5000, 1000, 50000, 1000000, 1000, 5000, 5000, 1000, 1000, 1000000, 100000, 50000, 500000, 100000, 10, 100, 10, 500, 500, 10000, 1000, 50, 1000, 10, 10, 5, 10, 5000, 10000, 1000000, 10000000, 50000000, 10000000, 5000000, 1000000, 10000000, 5000000, 10000000, 1000000, 5000000, 5000000, 10000, 100000, 1000000, 1000000, 50000, 10000000, 1000000, 5000000, 100000, 1000, 10000000, 100000, 50000, 5000000, 100000, 100000, 50000000, 10000, 100000, 10000, 10000, 10000, 1000000, 5000, 10000, 5000, 10000, 1000000, 1000000, 5000, 10000, 50000, 1000, 100000, 1000, 5000, 1000000, 10000, 50000, 10000, 1000000, 5000000, 50, 100000, 100000, 100000, 10000, 10000000, 10000000, 100000, 50000, 500000, 1000, 5000000, 10000000, 10000000, 5000000, 5000000, 10000000, 500000000, 5000000, 10000000, 5000000, 5000000, 5000000, 1000000, 50000000, 5000000, 1000000, 50000000, 5000000, 500000, 1000000, 1000000, 10000000, 10000000, 1000000, 5000000, 5000000, 1000000, 5000000, 1000000, 100000, 5000000, 1000000, 1000000, 1000000, 100000, 10000000, 100000, 1000000, 500000, 100000, 5000000, 100000, 100000, 500000, 10000, 100000, 50000, 1000000, 100000000, 1000000, 1000000, 10000, 1000, 1000000, 100000, 50000, 100000, 10000, 1000, 100000, 100000, 100000, 1000, 10000, 100, 100000, 10000, 1000, 50000, 10000, 1000, 100, 1000, 1000000, 500000, 1000, 10000, 5000, 5000, 100000, 1000, 10000, 10000, 100, 1000000, 1000, 1000000, 10000, 50000, 10000, 100000000, 100000, 100000, 10000, 5000, 1000, 10000000, 1000, 10000, 100000, 5000, 100000, 1000000, 50000, 1000, 1000, 10000000, 50000, 50000, 10000, 1000, 50000000, 10000000, 50000, 10000, 10000, 5000, 1000, 10000, 1000000, 5000, 50, 10000000, 1000000, 100, 100000, 100000, 1000, 1000000, 5000, 100000, 100000, 500, 100000, 50000, 500, 1000000, 1000, 100000, 50000, 10, 100000, 5, 100, 500, 100, 5000, 50000, 1000000, 5000000, 1000, 1000, 100000, 5000, 10000000, 10000000, 100000, 1000000, 10, 100000, 1000, 100, 1000000, 100000, 10000, 100000, 500000, 10000, 1000000, 10000, 10000, 100000, 100, 5000, 100000, 50000, 100, 1000, 1000, 50, 500, 5000, 10, 5000, 5, 10000, 1000000, 10, 50000, 50000, 50, 10000, 100, 100, 1000, 100000, 10000, 1000, 1000, 50, 500000, 500, 1000, 1000, 100, 500000, 50, 100, 5000, 5000, 100, 5000, 100, 500, 1000, 10, 500000, 1, 1000, 100, 10000, 100, 10000, 10, 10000, 10, 10000, 100000, 10000, 10, 1000, 1000, 10, 5, 1, 10000, 1000000, 10000000, 5000000, 10000000, 10000000, 10000000, 10000000, 5000000, 5000000, 10000000, 100000, 10000, 1000000, 1000000, 100000, 500000, 5000, 10000, 10000, 10000, 10000, 1000, 500, 500, 100000, 10000, 1000, 10000, 500, 10000, 50000, 1000, 10000, 10000, 1000, 100, 1000, 10000, 10000, 1000000, 100000, 1000, 5000, 10000, 100000, 10000, 10000, 50000, 5000, 5000, 500000, 100, 10000, 10000, 1, 10000, 100, 1000, 10000, 100000, 10000, 10000, 10000, 50000, 1000, 100000, 10000, 50000, 50000, 50000, 10000, 100, 1000, 500, 5000, 1000000, 100000, 10000000, 100000, 100000000, 10, 100000, 10000000, 500000, 1000, 1000, 5000, 50000, 100, 500, 100, 500, 1000000, 1000, 5000, 10000, 50, 10000, 500, 1000, 1000, 1000, 100, 1000, 500, 100, 1000, 100, 50, 500, 500, 100, 1000, 100, 5000000, 1000000, 10000, 5000, 1000000, 500000, 500000, 10000, 10000, 5000, 500000, 50000, 10, 1000, 5000, 10000, 100000, 1000, 10000, 1000, 1000, 10000, 500, 500, 100, 1000, 100000, 50000, 5000, 50000, 10000, 500, 10000, 500000, 100000, 5000, 100, 100000, 100000, 5000, 500000, 10000, 1000000, 100000, 500, 5000, 100, 10, 500, 100, 100000, 100000, 10000000, 500000, 100000, 100, 50000, 100000, 10000000, 100000, 1000, 10000, 5000000, 100000, 1000000, 1000, 50000000, 50000000, 100000, 50000000, 5000000, 5000, 5000, 100000, 10000000, 10000, 100000, 10000000, 10000000, 1000000, 5000000, 1000000, 10000000, 100, 10000000, 10000, 10000, 10000, 50000, 1000000, 50000, 100000, 5000000, 10000000, 10000, 1000, 100000, 50000, 5000000, 100000, 1000000, 500000, 1000, 1000, 1000, 1000, 100000, 5000, 1000000, 1000, 10000, 50, 10, 50, 50000, 100, 100000, 1000000, 100000, 1000, 10000, 1000, 500, 1000, 10000, 100000, 1000, 100, 1000, 10000, 5, 1000, 10000, 10000000, 1000, 1000, 500, 1000, 100, 5000, 100, 1000, 10, 100, 100, 10, 100, 1000, 10, 1000, 1000, 100000, 50000, 100, 500, 500, 1000, 1000000, 1000000, 500, 1000, 500, 100, 1000, 100, 10, 1000, 100, 1000000, 5000000, 1000, 10, 10000000, 100000, 100, 5000000, 1000, 10000, 10, 500000, 50, 1000000, 5000000, 100000, 10000, 10, 100000, 100000, 10000000, 50000, 50, 100000, 1000000, 500000, 1000000, 5000000, 10000, 100000, 500, 100000, 1000000, 10000, 1000, 1000, 10000, 10, 500, 1000, 100000, 10, 10, 1000, 10, 100, 500, 1000, 100000, 100, 10, 1, 10, 1000, 1000, 10, 5000, 10000, 5000, 1000, 50000, 50000, 100, 1000, 10000, 500, 5000, 1, 10, 50, 100, 500, 100000, 500000, 100000, 10000, 1000, 5000, 10000, 100000000, 1000, 10000, 100, 10000, 10000, 100, 10000, 1000, 1000, 10000, 10000000, 5000, 5000, 1000000, 1000000, 50, 10000000, 10000000, 10, 10000000, 50000000, 5000000, 5000000, 10000000, 1000000, 10000000, 100000, 500000, 10000, 10000, 50000, 10000, 100, 500, 1000, 5000, 50000, 100, 1000, 100, 10000, 500000, 10000, 5000000, 100, 50000, 10000000, 10, 500, 100, 1000000, 500, 100, 100000, 500, 10, 5, 10000000, 10, 10000000, 10, 1000000, 1000000, 5, 100000, 10000, 100000, 100000, 100000, 500000, 100000, 100000, 5000000, 5000000, 1000, 100000, 5000, 5000, 5000, 50, 100, 50, 100, 100000, 1000, 10000, 500, 100, 100000000, 100000, 1000, 1000, 10, 10000, 1000, 10000, 1000, 1000, 10000, 5000, 10000, 50000, 10000, 10000, 10000, 100, 5000, 100, 100, 10000, 1000, 10000, 50, 5, 500000, 10000, 10000, 100000, 10000, 10000, 100000, 1000000, 1000000, 5000, 10000, 100000, 100000, 50000, 10000, 100000, 100000, 1000, 10000, 1000, 100, 10000, 500000, 50000, 5000, 10, 10000, 100, 1000, 100, 100, 5, 10000, 1000, 1000, 100, 50, 10, 10, 10000, 100000, 50000, 1000000, 50000, 100000, 100, 100, 10000, 100, 10000, 10000, 50000, 100, 100000, 1000, 1000, 100, 1000, 10, 10000, 100000, 5, 500, 100, 100000, 1000, 500, 1, 500, 10, 500, 10, 100, 1000000, 1000000, 1000, 50, 10000, 100, 100, 10, 10, 500, 500000, 5000, 100000, 500000, 100, 100000, 100000, 10000, 100, 10000, 5000, 10000, 1000, 100, 10000, 10000, 5000, 1000, 100, 10, 5000, 50000, 10000, 5000, 5000, 10000, 1000, 5000, 1000, 1000, 50, 5000, 50000, 1000, 10000, 1000, 10000, 10000, 10000, 5000, 1000, 1000, 50, 10, 1000000, 10000, 50000000, 50000000, 100000000, 10000, 500, 100000, 5000000, 10000, 1000000, 1000000, 1000000, 1000, 10000000, 5000000, 10000000, 10000, 5000000, 5000, 50000000, 1000000, 10000, 10000, 10000, 1000000, 1000, 1000, 100000, 10000000, 10000, 10000, 10000, 1000, 500, 1000, 100, 50, 100, 1000, 500000, 5000, 1000, 1000, 1000, 10, 1000, 100000, 50000, 1000, 1000, 1000, 10000, 50, 100, 500000, 1000000, 1000000, 5000, 1000000, 100, 5000, 1000, 100, 1000000, 1000, 1000, 5000, 50, 1000, 1000, 50, 100, 100, 50, 10000, 5000, 50000, 5000, 100000, 50, 1000, 500000, 5000, 1000000, 100, 10000, 10000000, 1000000, 50000, 50, 100, 5000000, 100, 50, 100, 5, 10, 1000, 100000, 1000000, 100000, 50000, 5000, 5000, 100, 10000, 1000, 500, 100000, 100, 10000, 100000, 5000, 10000, 50, 10, 100000, 100, 1000, 5, 100, 50000, 50, 5000, 10000, 1000, 10000000, 100000, 10000, 10, 500, 100000, 5000000, 1000000, 1000000, 10000, 10000000, 10000, 10000000, 100000, 1000000, 1000, 1000000, 500000, 50, 50, 1000, 50000, 5000, 100, 100, 100, 10000, 1000, 100, 100, 5000, 1000, 50000, 500, 100, 50, 100000, 50, 10000, 50000, 10000, 10, 1000, 100000, 100000, 10, 1000000, 100000, 10000, 10000, 1000000, 1000, 5000000, 100000, 1000000, 500000, 5000, 10000, 10000, 10000, 5000, 500000, 50000, 10000, 10000000, 1000000, 1000, 100000, 500, 500, 5000000, 500, 500, 1000000, 1000, 1000000, 500000, 1000, 10000, 10000, 100000, 500, 10000, 100, 5000, 1000000, 50000, 100, 10, 500, 100, 1000, 1000, 50000, 100, 5000, 1000000, 1000, 10000, 10000, 1000, 10, 100, 10, 100, 1000, 10, 10000, 1000, 1000000, 500, 10, 1000, 50, 100, 1000, 10, 50, 100, 1000, 100, 1000, 100, 50, 500, 1000, 10, 100, 10, 1000, 10000, 1000, 100000000, 100, 100, 100, 100, 500, 500, 1000000, 500, 1000, 100, 100000, 100, 100, 500, 100, 50, 100, 100, 100, 10, 10, 100, 10, 10, 500000, 100, 500, 100000, 10, 50, 50, 10000000, 100000, 10000, 5000000, 50000, 1, 1000, 5, 1000000, 1000000, 50, 100000, 5000, 10000, 50000, 100000, 10000, 1000, 5000, 500, 100000, 10000, 100000, 1000, 5000, 5000, 100, 100, 500, 100, 10, 100, 50, 10, 10000, 1000, 10000, 1000, 50000, 500, 5000, 10000, 10, 100, 1000, 1000000, 100, 10, 50000, 10, 50, 5000, 1000000, 10000, 100, 5000, 10000, 100, 100, 100, 50, 100, 1000, 1000, 5000, 5000, 100, 1000, 1000, 1, 10000, 500, 1000, 100000, 100, 5000000, 1000000, 500, 10000000, 5000000, 1000000, 10000, 1000000, 10000000, 10000000, 5000, 1000000, 10000, 500000, 500000, 500000, 1000000, 500000, 50000, 1000000, 500000, 500000, 10000000, 1000000, 100000, 100000, 1000000, 1000000, 10000000, 1000000, 1000000, 100000, 5000000, 1000000, 100000, 500000, 100000, 100000, 500000, 1000000, 1000000, 100000, 100000, 500000, 500000, 5000000, 50000, 500000, 100000, 5000000, 100000, 1000000, 100000, 100000, 1000000, 100000, 100000, 1000000, 1000000, 1000000, 10000, 100000, 100, 100, 1000, 10000, 1000, 1000, 10, 50000, 50, 10, 1000, 10, 100000, 500, 10000, 1000, 5000, 500, 1000, 100, 1000, 1000, 5000, 50000, 500, 1000, 10000, 100, 500000, 100, 1000, 1000, 5000, 1000, 50, 500, 10000, 10000, 1000, 100000, 5000000, 5000, 10000, 1000, 10, 100000, 1000000, 100, 500, 1000, 10000, 10000, 10, 5000, 1000, 1000000, 100000, 5000, 10000, 100, 1000000, 10000, 100000, 100000, 1000000, 1000, 1000, 100000, 10000, 50000, 500, 100000, 10, 100, 500, 1000000, 50, 1000, 10, 50, 100000, 1000, 1000, 100, 500, 10, 100, 100, 1000, 100, 1000, 10, 50, 10000, 100, 10, 10, 100, 50, 5000, 500, 50000, 5, 5000000, 10000, 1000000, 500000, 10, 500, 50000, 100, 100, 100, 1000, 5, 500, 10000, 10, 500, 100, 500, 10, 100, 500, 500, 100, 500, 5000, 10000, 5, 1000, 100000, 100, 1000, 500, 5, 5, 50, 1000, 5000, 50, 50, 5, 10000, 10000, 10000, 100, 1000, 100, 5000, 10000, 1000, 500, 1000, 50000, 50000, 1000, 1000, 100000, 1000, 500, 100, 100, 100, 5000, 50000000, 500, 100, 100, 10000000, 50, 50, 50, 1000, 10000000, 10, 1, 100, 5, 5000000, 100000, 10000, 100, 100000, 1000, 100, 100, 100, 100, 500, 100, 500, 100, 100, 1000, 100, 100, 10, 100, 1000, 10000, 1000, 100, 100, 100, 50, 100, 10000000, 100, 100, 100, 50, 100, 100, 10, 100, 100, 500, 100, 100, 500, 50000, 10000, 5000000, 10000, 5000, 1000, 1000, 10000, 5000, 100000, 100, 10000, 1000, 100, 10000, 100, 1000, 500, 500, 1000, 1000, 5000, 5000, 50000, 100, 100, 500, 100, 100, 10000, 10, 500, 1000, 1000, 10000, 10000, 100, 50, 500, 1000, 10, 50, 1000, 100000, 1000, 5000, 1000, 1000000, 100000, 5000, 1000, 1000000, 500, 10000, 10000, 5000, 1000000, 100, 500000, 100000, 500000, 100000, 1000, 50000, 1000, 1000, 10000, 500000, 100000, 100, 500000, 100, 1, 500, 100, 5000, 5000, 10, 100, 100000, 10, 100000, 500, 100, 1000, 50, 100, 100, 50, 1000, 1000, 1000, 10, 5000, 10000, 100000, 50, 5000000, 10, 500, 10, 1000, 50000000, 1000, 5000, 10000, 5000000, 100, 1000, 100, 10000, 10000, 1000000, 1000000, 5000, 100, 1000, 500, 500000, 100, 100, 100000, 100000, 500000, 100, 100000, 10000000, 50000, 1000, 10, 100, 0, 500, 50000, 50000, 1000000, 500, 10, 10000000, 100, 1000000, 10, 10, 10, 50, 100000, 10000, 5000000, 1000000, 1000000, 500000, 10000, 50000, 100000, 10000000, 500, 10000, 50000, 10000, 100, 5000000, 5000, 1000, 5000, 500, 50, 5000, 1000000, 10, 100, 1000, 100000, 5000, 1000, 10, 100, 50000, 10, 10, 5000000, 1000, 10000, 10000000, 1000000, 10000, 10, 5, 10000, 100000, 100000, 100000, 1000, 10000, 100, 10, 10, 1000, 5000, 500000, 100000, 500000, 1000, 1000, 10000, 5000, 50000000, 10000, 100, 1000, 10000000, 5000000, 1, 1, 100, 500, 10000000, 5, 5000, 10, 10, 10000, 10, 100, 100, 50, 100, 500000000, 50000000, 1000000, 10000000, 5000000, 100000, 100000, 500000, 5000000, 100000, 100000000, 100000, 1000000, 50000000, 1000000, 100000, 100000, 100000, 10000, 100000, 50000, 100000, 100000, 500000, 500, 10000, 500000, 10000, 10000, 10000, 10000, 100000, 100000, 500000, 1000000, 5000, 10, 100000, 10000, 1000000, 10000000, 100000, 500000, 1000000, 10000000, 5000000, 1000000, 5000000, 1000000, 10000000, 5000000, 500000, 1000000, 1000000, 5000000, 10000000, 1000000, 1000000, 1000000, 1000000, 500000, 100000, 5000000, 1000000, 1000000, 1000, 100, 1000000, 10000, 10000, 1000000, 10000, 100000, 1000, 10000, 5000, 1000, 5000, 1000, 10000, 1000000, 10000, 1000, 100000, 10000, 10000, 1000, 50000, 50000, 1000, 500000, 1000, 500, 10000, 1000000, 5000, 1000, 10000, 500, 5000000, 10000, 1000000, 100, 1000, 1000, 1000, 1000000, 100000, 50000, 10000, 100000, 500, 10000, 500000, 10000, 1000, 1000, 1000, 100000, 10, 1000, 5000, 5000, 1000000, 10000, 50000, 10000, 100000, 1000000, 500, 500, 1000000, 10, 1000, 100, 100, 100, 1000, 100000, 10000, 100, 100, 1000000, 500, 100, 500, 1000000, 500, 100, 10000, 5, 5000, 100, 5000000, 10000, 1000, 1000000, 1000, 100, 100000, 1000, 100000, 500, 500, 10, 10, 500, 5000, 5, 5, 100000, 100000, 10, 10000, 1, 100000, 500000, 10000, 1000, 10000, 10000000, 10000, 100, 10, 100000, 50000, 5000000, 5000000, 100000, 50, 1000, 100, 50, 100000, 100, 100000, 1000, 10000, 1000000, 5000000, 10000000, 500000, 100000, 100000, 500, 100000, 10000, 1000000, 500000, 100, 1000000, 5000, 10000, 1000000, 500000, 50000, 10000, 500000, 5000, 10000, 100, 1000, 1000, 5, 5000, 10000, 100000, 50000, 5000, 5000000, 100, 100, 500, 50000, 5000, 1000, 50, 100, 1000000, 100000000, 1000000, 1000000, 10000000, 5000000, 1000, 100000, 10000, 100000, 100000, 50000, 10000, 50000, 5000, 100000, 500000, 10000, 100, 1000, 100000, 10000, 10000, 1000000, 500000, 1000, 1000000, 100, 10000, 5000, 10000, 50000, 1000, 50, 50000, 10000, 500000, 5000, 100000, 10000, 50000, 10000, 5000, 50000, 100, 50000, 50000, 1000, 5000, 10000, 1000, 1000, 1000, 1000, 500, 1000, 5000, 10000, 10000, 100000, 5000, 1000, 100, 100, 5000, 50000, 10000, 100, 1000, 10000, 1000, 1000, 10000, 100, 1000000, 50, 100000, 10000, 100, 5000, 1000, 50, 10000, 50, 100000, 10000, 10000, 1000, 100000, 50000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 10000, 50000, 50000, 50000, 10000, 10000, 10000, 50000, 100000, 50000, 10000, 1000, 50000, 10000, 5000, 100, 5000000, 1000000, 5000000, 10000000, 10000000, 10000000, 1000000, 10000, 100000, 1000000, 1000000, 100000, 10000, 500000, 5000, 100000, 100000, 10000, 10000, 50000, 100000, 100000, 100000, 1000, 1000000, 50000, 1000000, 100000, 5000, 1000000, 10000, 10000, 100000, 5000, 10000, 5000, 100000, 10000, 10000, 10000, 10000, 50000, 1000, 100000, 10000, 5000, 50000, 10000, 100000, 5000, 5000, 5000, 50000, 10000, 100000, 10, 5000000, 100000, 50000, 100, 1000000, 500, 100, 10, 100, 50, 1000000, 500, 1000000, 100, 500000, 100000, 1000, 100000, 1000, 500000, 50000, 50000, 5000, 500000, 1000, 1000000, 5000000, 5000000, 5000, 100000, 1000, 10000, 10000, 1000000, 10000, 10000, 10000, 1000, 50000, 500, 1000, 1000, 1000, 100000, 500, 5000, 500, 10000, 10, 5000, 100, 100, 10000, 500, 50, 500, 10, 100, 50, 50, 100, 500, 1000, 50, 500000, 100000, 50, 1000, 5, 50, 0, 1000000, 50000, 100, 5000, 5000000, 500, 1000, 100, 10000, 100, 1000, 1000, 100, 100, 50000, 1000, 5000, 100, 100, 100, 100, 10, 10, 1000000, 500, 5, 100, 1000000, 100, 10, 10, 10000000, 1000000, 5000, 1000000, 5, 100000, 500000, 10000, 10000, 10000, 1000000, 100000, 10000, 10000, 1000, 1000000, 10000, 10000, 500000, 100000, 100000, 50000, 50000, 50000, 5, 10, 1000, 10000, 100000, 10, 1000, 100000, 100, 100000, 10, 500, 5000, 500, 100, 10000, 500000, 1000000, 1000, 10000, 10000000, 5000000, 50000000, 5000000, 1000000, 5000000, 10000000, 1000000, 10000000, 1000000, 5000000, 10000000, 10000000, 50000, 10000000, 5000000, 10000000, 5000000, 10000000, 10000000, 50000, 50000, 10000000, 5000, 500000, 5000, 1000, 10000, 10000, 10000, 100, 500000, 1000, 10000, 500, 500, 1000, 10000, 10000, 1000, 1000, 10000, 10000, 10000, 5000, 5000, 100, 10000, 10, 10000, 100, 100000, 5000, 1000, 1000, 1000, 100, 1000, 10000, 10000000, 5000000, 10000000, 500000, 10000, 10000000, 1000000, 10000, 1000, 5000000, 1000, 1000000, 5000, 100, 10000000, 500, 5000, 50000, 1000, 1000, 10000, 50000, 10000000, 50000000, 500000, 100, 5000, 100000, 50, 5000, 500, 500, 1000, 10000, 5000, 5000, 10000000, 10000000, 5000000, 10000000, 10000000, 5000000, 1000000, 10000000, 50000000, 1000000, 10000000, 1000000, 10000000, 5000000, 1000000, 50000000, 5000000, 50000000, 10000000, 10000000, 5000000, 10000000, 10000000, 100000, 1000, 100, 500, 1000, 10000, 100, 50000, 1000, 500, 100, 100, 100, 1000, 1000, 10, 100, 100, 500, 500, 10000, 100, 10, 500000, 50, 5, 10, 100, 500000, 1000, 5, 1, 10, 5000000, 500, 5000000, 100000, 10000, 1000, 1000000, 10000, 1000, 1000, 10000000, 500, 10000, 10000, 1000, 1000000, 500, 10000, 1000, 100, 500000, 100, 100, 100, 10000, 100, 10, 100, 10, 10000, 100, 10000, 100, 10000000, 5000000, 10000000, 50000000, 5000000, 100000, 1000, 5000000, 10000000, 1000000, 1000000, 10000000, 100000, 10000, 50000, 100, 500000, 1000, 1000000, 10000, 1000, 5000000, 100000, 500000, 10000000, 10000, 5000000, 50, 1000000, 100000, 10000000, 100000, 100000, 1000, 10000000, 1000, 100, 10000000, 1000000, 1000000, 50000000, 10000000, 500000, 100000000, 100000, 5000000, 10000000, 100000000, 1000000, 50000000, 10000000, 50000000, 10000000, 10000000, 10000000, 5000000, 10000000, 10000000, 1000000, 100000, 100000, 100000, 1000000, 1000, 1000, 1000, 50000, 1000, 5000, 10000, 10000, 100000, 1000, 10000, 50000, 10000, 100000, 500, 1000000, 50000, 100000, 1000, 100, 50000, 10000, 100, 10, 10, 10000, 10, 1000000, 1000, 10000, 500, 1000, 500, 50, 100, 100, 100, 100000, 1, 10, 10000000, 5000000, 50000000, 10000000, 100000, 1000000, 10000000, 1000000, 1000000, 1000, 5000000, 10000, 5000, 5000000, 100000, 10000000, 5000000, 10000, 1000000, 1000000, 1000000, 50000, 10000, 10000000, 1000000, 100000, 1000000, 1000000, 1000000, 10000, 10000, 5000, 5000, 100000, 5000, 1000, 1000, 1000, 5000, 10000, 1000, 10, 1000, 5000, 1000, 1000, 100, 10000, 1000, 5000, 10, 100, 100, 5000, 500000, 500, 100, 100, 100, 50, 5, 10, 10, 100, 100, 10, 10000, 10000, 1000, 100, 50, 50000, 10, 100000, 500, 5000, 100000, 10, 10, 50, 500, 100, 100, 100, 5, 10, 10, 50000, 100000, 100000, 1000000, 1, 1000000, 0, 100000, 500000, 50000, 1000000, 50000, 50000, 10000, 5000000, 5000000, 1000000, 1000000, 1000000, 10000000, 5000000, 1000000, 100000, 5000000, 1000000, 1000000, 100000, 10000000, 500000, 1000000, 100000, 1000000, 1000000, 500000, 100000, 1000000, 500000, 100000, 1000000, 100000, 100000, 50000, 1000000, 100000, 100000, 1000000, 1000000, 100000, 500000, 10000, 100000, 500000, 1000000, 100000, 5000, 500000, 50000, 100000, 100000, 10000, 500000, 100000, 5000, 100000, 100000, 1000, 1000000, 1000, 5000, 1000, 10000, 10000, 100000, 100000, 50000, 100, 5000, 100000, 10000, 10000, 10000, 5000, 5000, 1000, 10000, 500000, 5000, 1000, 5000, 10000, 50000, 100000, 10000, 500000, 100, 1000, 500, 1, 100000, 1000000, 1000000, 10000, 500000, 100000, 100000, 100000, 100000, 10000000, 10000, 100000, 1000000, 1000000, 1000000, 1000, 100, 100000, 50000000, 10000000, 100000000, 5000000, 100000, 500000, 1000000, 1000000, 1000000, 1000000, 10000000, 500000, 1000000, 10000000, 1000000, 10000, 500000, 500000, 10000000, 1000000, 5000000, 100000, 1000000, 1000000, 10000, 100000, 1000000, 100000, 10000, 10000, 10000000, 1000000, 1000000, 1000, 500000, 1000000, 5000, 1000000, 1000000, 10000000, 500000, 1000000, 500000, 5000000, 500000, 1000000, 1000000, 100000, 500000, 50000, 50000, 100000, 500, 1000, 50000, 100, 500, 100000, 50, 1000, 100000, 10, 1000, 100, 500, 1000000, 1000000, 10000000, 1000, 5, 500, 10, 100, 100000, 1000000, 5000000, 10000, 100, 1000, 100, 500, 1000, 500, 10000, 1000, 5000, 500, 500, 1000, 1000000, 10000000, 1000, 100, 1000, 10, 10, 1, 0, 10, 500, 100, 50, 1, 1000, 10000000, 5000000, 1000, 10, 1000, 500, 100, 10000000, 100, 100000, 50000, 10000, 500000, 1000000, 100000000, 10000000, 10000000, 1000000, 100000, 1000000, 1000000, 5000000, 100000, 50000, 500000, 500000, 5000000, 1000000, 50000, 50000, 100000, 100000, 50000, 100000, 50000, 50000, 50000, 5000, 5000000, 100000, 50000, 10000000, 10000000, 10000, 10000000, 1000000, 50000, 10000, 1000, 5000, 10000, 10000, 10000, 1000, 1000, 10000, 50, 1000, 50000, 10000, 1000000, 100000, 100, 5000, 50000, 100, 50000, 10000, 1000, 100, 100000, 1000, 1000, 100, 1000, 10000, 5000, 500000, 100000, 50, 100, 100, 500, 10, 10000, 5000, 1000, 1000, 100000, 100000, 100000, 1000000, 100000, 10000, 10000, 10000, 100000, 1000, 1000000, 10000, 50000, 100, 10, 5000, 1000, 10000, 1000, 1000, 500000, 10000, 1000, 500, 1000, 10000, 100, 100, 1000, 10, 5000, 100000, 10000, 100, 1000, 1000, 100, 10000, 10, 10, 500, 5000, 10000, 100, 500, 10, 500000, 500000, 5000, 500000, 5000, 1000, 5000, 1000000, 1000, 10000, 100000, 1000, 100000, 10000, 50000, 500, 1000, 10000, 5000, 5000, 10000, 5000, 5000, 5000, 10, 100, 500, 500, 100, 5000, 10000, 50000, 100, 1000, 10000, 100, 5000, 100, 10, 50, 1000, 100, 10, 10000, 50000, 50000, 50, 10000, 100000, 10000, 100, 10000000, 50, 1000000, 100000, 100000, 100000, 10000, 10000, 10000, 50000, 100000, 10000, 1000, 50000, 100000, 5000, 50000, 10000, 100, 1000, 1000000, 100000, 10, 100000, 1000, 5000, 10000, 100, 100, 1000, 5000, 500, 1000, 100000, 500, 10000, 10000, 1000, 10000, 10000, 500, 10000, 1000, 10000, 5000, 10, 10, 10000, 1, 50000000, 50000000, 50000000, 10000000, 10000000, 10000000, 10000000, 0, 10000000, 500000, 10000000, 5000000, 1000, 5000000, 1000000, 1000000, 10000000, 5000000, 1000000, 5000000, 10000000, 10000000, 100000000, 1000000, 100000, 100000, 1000000, 1000000, 1000000, 10000000, 100, 10000, 10000, 10, 100, 50, 1000, 500000, 1000, 100000, 100, 1000000, 5000, 50, 5000, 10, 10000000, 100000, 1000, 50, 1000, 100000, 5000, 10, 10, 100000, 1, 10000, 100000, 1000, 100, 1000, 10000, 10, 10, 10000, 5000, 10000, 50000, 100000, 100000, 5, 10000, 1000000, 1000000, 5000, 1000, 100000, 100000, 1000, 10000, 10000, 10, 50000, 100000, 100, 50, 1000, 100, 1000, 1000, 50, 10000, 1000, 100, 10000, 100, 500, 10, 50000, 50, 100, 100, 10, 500, 500, 100, 1000, 10000, 100, 1000, 100, 10, 1, 1000, 1, 10, 100, 100000, 50000, 10000, 10000, 5000, 100000, 50000, 100000, 5000, 10000, 50000, 50000, 1000, 10000, 1000, 100, 1000, 100, 5000, 1000, 500, 10000, 500, 50, 100, 5000, 100, 500, 1000, 50, 100, 500, 100, 50, 1, 50000, 1000, 5, 10000, 5000, 1000, 100, 1000, 100, 1000000, 100, 10000, 50000, 1000, 50, 1000, 1000, 100, 100000, 100, 10000, 500, 1000, 10, 50, 100, 1000, 10, 100000, 1000000, 1000000, 100000, 10000, 100, 10000, 1000000, 50000000, 1000, 100000, 1000000, 500000, 500000, 50000, 100000, 1000000, 1000000, 1000000, 100000, 1000000, 10000, 1000000, 100000, 10000, 100, 1000, 100, 1000, 100000, 50, 100, 500000, 100, 5000, 5000000, 10000, 1000000, 1000000, 10000000, 50000, 10000, 500000, 5000000, 500000, 10000000, 1000000, 500000, 1000000, 100000, 5000000, 10000000, 5000000, 5000000, 500000, 500000, 10000000, 100000, 10000000, 100000, 100000, 10000000, 10000000, 10000, 1000000, 100000, 5000000, 10000, 1000000, 100, 100, 10, 10000, 100, 1000, 10000, 5, 10, 10, 100, 50, 5000, 10000, 500000, 1000000, 5000, 10, 1000000, 1000000, 5000000, 10, 10000000, 10000, 50000, 5000, 1000000, 1000, 5000000, 10000000, 1000000, 1000, 50000, 100, 1000000, 100, 1000000, 500, 1000, 1000000, 1000, 1000000, 1, 10000000, 10, 1000000, 1000000, 50, 100, 5, 1000000, 1000000, 5000000, 1000000, 100, 100000, 1000000, 50000, 1000000, 500000, 1000000, 10000, 1000000, 5000000, 500000, 1000000, 10000, 50000, 5000, 500000, 10000, 100000, 100000, 500000, 1000000, 1000000, 50000, 100000, 10000, 1000, 10000, 500, 10000, 10000, 50000000, 100, 50, 5000, 5000, 5000, 500, 1000, 10, 1000, 10000, 10, 100, 10, 500, 1000, 500, 100, 1000, 10000000, 1000, 50000, 10000000, 1000000, 1000000, 1000, 1000, 5000, 100, 500, 1, 1000000, 1000, 1000000, 100, 5000000, 10000000, 1000000, 100000, 100000, 100000, 100000, 100000, 500000, 100000, 50000, 1000000, 1000000, 1000000, 100000, 10000, 100000, 5000000, 100000, 10000, 100000, 1000000, 10000000, 10000000, 100000, 5000000, 100000, 100000, 1000000, 5000000, 10000000, 10000000, 5000000, 500000, 500000, 10000000, 500000, 1000000, 100000, 500000, 500000, 1000000, 5000000, 500000, 1000000, 500000, 100000, 500000, 100000, 1000000, 500000, 50000000, 1000000, 10000000, 500000, 5000000, 100000, 500000, 500000, 100000, 10000000, 10000000, 10000000, 100000, 10000000, 10000000, 5000000, 10000000, 1000000, 1000000, 10000000, 1000000, 10000000, 5000000, 10000000, 1000000, 500000, 10000000, 10000000, 10000000, 1000000, 5000000, 5000000, 5000000, 1000000, 1000000, 5000000, 10000000, 5000000, 10000, 1000, 10, 1000, 100, 10, 100, 10, 50, 100, 10, 10, 10, 10, 10000, 10, 50, 1, 50, 10, 1000000, 5000000, 100, 10000000, 500000, 100, 1000000, 1000000, 1000000, 10000000, 1000000, 10000000, 1000000, 500000, 1000000, 1000000, 100, 100, 100, 500, 50000, 100, 10, 100, 100, 1000, 100, 50, 5000, 100, 500000, 1000000, 500, 1000, 10, 10, 100, 500000, 10000, 100000, 500, 1000000, 1000, 10000, 100, 100, 10000000, 10000000, 10000, 1000000, 100000, 1000000, 100000, 100000, 10000000, 500, 5000, 1000000, 100000, 100000, 50000, 50000, 500000, 10000, 500000, 10000, 50000, 1000, 10000, 1000, 100000, 1000, 10000, 5000, 1000, 5000, 500000, 1000000, 100000, 500000, 100000, 500000, 10, 50000, 5000, 5000, 1000, 10000, 500, 100000, 100000, 1000, 50000, 1000, 1000, 10000000, 5000, 500, 1000000, 10000000, 10000000, 100000, 100000, 1000000, 1000000, 1000000, 500000, 100000, 1000000, 1000000, 1000000, 500000, 1000, 100000, 5000, 50000, 1000, 10000, 50000, 10000, 500, 5000, 1000000, 1000, 100, 100, 10000, 50000000, 500000, 100, 10000000, 50000, 5000000, 10000, 500, 100000, 100000, 5000000, 1000000, 10000, 5000, 100000, 100000, 100000, 50000000, 50000, 100000, 100000, 50000, 5000000, 50000000, 10000000, 1000000, 5000000, 10000000, 5000000, 50000000, 10000000, 10000000, 5000000, 10000000, 50000000, 5000000, 10000000, 100000, 50000, 1000, 1000, 100000, 50000, 5000, 1000, 1000, 1000, 100, 5000, 10000, 1000, 50000, 10000, 10000, 100, 100, 50, 500, 100000, 10000, 1000000, 1000, 1000, 1000, 500, 500, 1000, 50000, 1000, 1000, 100, 100, 1000, 5000000, 50, 1000, 500, 50, 10, 1000, 1000, 50, 100, 100, 100, 10, 100, 100, 1000, 1000, 10000, 1000, 10, 10, 50000, 50, 10, 10, 100, 5, 100, 100, 100, 10, 1000000, 10000, 100000, 50000, 100000, 50000, 50000, 1000, 10000, 1000, 1000, 1000, 5000, 10000, 1000, 1000, 10, 100, 1000, 100, 1000, 1000, 100, 1000, 5, 5000, 5000, 1000, 1000000, 1000, 500000, 1000, 500, 100000, 100000, 1000000, 1000000, 1000, 5000000, 500, 500000, 1000000, 1000000, 100000, 100000, 100000, 10000000, 500, 10, 100000, 100, 5, 50, 100, 100000, 10, 1000000, 100, 5, 100, 10, 10000, 10, 10, 100000, 10, 50000000, 100000, 10000, 5000000, 500, 10000000, 10000, 50000, 100000, 100000, 10000000, 5000, 100000, 5000000, 50000, 10000000, 1000000, 5000000, 10000000, 1000000, 100000, 5000000, 10000000, 5000000, 500000, 500000, 50000, 5000, 100000, 1000000, 1000, 100000, 1000000, 1000000, 500000, 100000, 100000, 5000000, 5000, 50000, 5000000, 1000, 100, 5000000, 5000, 5000, 50000, 10000, 1000, 50, 500, 1000, 100, 1000, 1000, 100, 5, 10000, 1000, 5000, 100, 5, 500, 1000, 100, 5, 500, 10000, 500, 10000, 10, 10000000, 1, 500, 100, 5, 10, 10, 1000, 50, 10000, 100, 1000000, 1000000, 10000000, 10000, 100000, 100000, 500, 100, 500000, 5000000, 1000000, 1000000, 500000, 1000000, 100000, 10000, 10000, 1000, 50000, 100000, 100000, 100000, 1000000, 10000, 5000, 500000, 1000, 50000, 10000, 10000, 10000, 50000, 1000000, 1000, 100000, 1000000, 5000, 10000, 50000, 10000, 10000, 5000, 100000, 5000, 1000, 10000, 100000, 1000, 50000, 5000, 5000, 1000, 1000, 100, 100000, 100000, 5, 1000000, 5000, 5000, 1000, 500, 1000000, 5000000, 1000, 1000000, 10000000, 1000, 1000, 1000, 10, 10000000, 1000000, 10000000, 10000000, 10000000, 1000000, 50000000, 1000000, 500000, 1000000, 100000, 1000000, 10000, 500000, 1000000, 500000, 100000, 50000, 500000, 10000000, 10000, 500, 100000, 100000, 50000, 500000, 5000, 100000, 100000, 5000, 5000, 1000000, 100000, 100000, 5000, 1000, 10000, 100000, 10000, 50, 5000, 100000, 10000, 500, 10000, 10000, 1000000, 100000, 500000, 10000, 10000, 10000, 10000, 1000000, 100, 50000, 10000, 10000, 50000, 100000, 100000000, 100000, 10000, 500, 1000000, 5000, 10000, 10000, 5000, 10000, 1000, 50000, 500, 100, 10000, 10000, 50000, 100000, 1000, 10000, 100000, 5000000, 500, 50000, 10000, 500, 10000, 100000, 10000, 100000, 5000, 10000, 100000, 1000, 100000, 100000, 10000, 10000, 500, 100000, 1000, 1000000, 10, 1000, 1, 1000000, 1000, 10000, 5000, 500000, 10000, 1000, 10000, 1000, 5000, 10000, 100, 100000, 10000, 50, 10000, 10000, 50000, 1000000, 5000000, 1000, 5000, 1000, 50000, 1000, 10, 500, 100, 10, 500, 100, 100, 1000, 1000, 1000, 1000, 100, 100, 100, 1000, 10000, 1000, 500, 100, 1000, 500, 10, 100, 50, 100, 1000, 100, 10000, 100, 5000, 10, 1000, 100, 500, 5000, 1000, 1000000, 1000, 10, 100, 10, 10, 10000, 50, 1000000, 5000000, 5000000, 1000, 10000000, 10000, 10000000, 10, 50000000, 5000000, 10000, 1000000, 1000000, 10000000, 1000000, 1000000, 10000000, 500000, 100000, 5000000, 1000000, 500000, 500000, 1000000, 1000000, 500000, 5000000, 10000, 5000, 10, 100, 10000, 500, 1000, 500, 1000, 1000, 1000, 50000000, 5000, 1000, 500, 100, 500, 100, 1000, 100, 1, 1000, 1000000, 100, 500, 1000, 5000000, 10000, 100, 5000000, 1000, 10000, 50000000, 50000, 100000, 100, 500, 1000000, 5000000, 100000, 1000000, 500000, 100, 100000, 1000000, 10000, 100000, 1000000, 500000, 1000000, 10000, 1000000, 1000, 500000, 5000, 100000, 50000, 500000, 5000, 100000, 100000, 10000, 10000, 50000, 500000, 10000, 100000, 1000, 50000, 50000, 10000, 1000, 50000, 500000, 500000, 10000, 100, 1000, 100, 10000, 1000, 5000, 1000000, 10000, 500, 1000, 5000000, 100, 100, 100, 50, 10000, 100, 100, 100, 1000000, 10000000, 100000, 10000000, 10000000, 500000, 10000000, 10, 1000000, 100000, 50000000, 1000000, 50000, 500, 1000000, 10, 10000, 10000000, 1000000, 100000, 10, 10, 1000000, 10000, 10, 10, 1, 100, 10, 5, 50, 100, 50, 100, 10, 1, 5, 50, 10, 10, 50, 10, 10, 50000000, 10, 10, 10000, 5, 5000000, 5, 1000, 10, 100, 5, 10, 10, 5, 10, 100, 10, 50000, 5, 5000000, 100000, 10000, 50000, 10000, 100000, 5000, 1000, 10000, 5000, 100000, 50000, 10000, 100000, 5000, 1000, 5000, 1000, 5000, 1000, 500, 1000, 1000, 5000, 50000, 1000, 100000, 500, 10000, 10000, 1000, 10000, 50000, 10, 100, 5000, 5000, 500, 5000, 100, 10, 10000, 5000, 100, 1000000, 5000, 100, 10000, 10000, 100, 50, 100000, 10000, 10, 50, 1000, 1000, 1000, 100000, 1000, 5000, 10000, 1000, 5000, 10000000, 500, 10000, 10000, 1000, 100000, 500000, 10000, 10000, 10000, 10000, 100000, 50, 1, 1000, 100, 5, 10, 50, 10, 500, 10000, 1000, 5000, 10, 1000000, 10, 1000000, 500, 100, 10000, 100000, 100000, 5000000, 1000000, 10000, 10000, 10000000, 100000, 1000, 500, 100, 100, 100, 100, 100, 100, 50, 1000, 1, 500, 50, 100, 100, 500, 100000000, 1, 100, 10000000, 1000000, 1000000, 100000, 5000000, 10000000, 1000000, 5000, 1000, 100, 100000, 10000000, 10000, 1000, 1000000, 50000, 10000000, 10000000, 5000000, 100000, 10000, 100, 100, 1000, 5000, 500000, 10000000, 1000000, 1000, 500, 100, 1, 1000, 5000, 10, 1000, 100000, 1000, 5000, 10000, 10000, 5000, 50000, 1000000, 1000, 1000, 10, 50000, 1, 100000, 100000, 500, 100, 100000, 1000, 100, 10, 10, 1, 10, 100000, 10000, 10000, 1000, 10000000, 50000, 10000000, 50000, 50000, 500, 50000, 50000, 50000, 1000000, 500000, 1000, 100000, 1000000, 1000000, 100000, 5000, 1000, 10000, 1000000, 100000, 100, 500, 100, 50000, 1000000, 100, 100, 1000, 10000, 50000, 500000, 100, 100000, 10000, 5000, 1000, 50, 10, 100, 10000, 100, 5000000, 5000, 10000, 10000, 100000, 5000, 100000, 1000, 500, 10, 5000, 100, 1000, 1000, 10000000]}],                        {"template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "autotypenumbers": "strict", "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Number of downloads of paid apps vs. free apps"}, "yaxis": {"autorange": true, "type": "log"}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('fc292cbf-de8a-4d31-b8b9-a0108638e2fe');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


## 9. Sentiment analysis of user reviews
<p>Mining user review data to determine how people feel about your product, brand, or service can be done using a technique called sentiment analysis. User reviews for apps can be analyzed to identify if the mood is positive, negative or neutral about that app. For example, positive words in an app review might include words such as 'amazing', 'friendly', 'good', 'great', and 'love'. Negative words might be words like 'malware', 'hate', 'problem', 'refund', and 'incompetent'.</p>
<p>By plotting sentiment polarity scores of user reviews for paid and free apps, we observe that free apps receive a lot of harsh comments, as indicated by the outliers on the negative y-axis. Reviews for paid apps appear never to be extremely negative. This may indicate something about app quality, i.e., paid apps being of higher quality than free apps on average. The median polarity score for paid apps is a little higher than free apps, thereby syncing with our previous observation.</p>
<p>In this notebook, we analyzed over ten thousand apps from the Google Play Store. We can use our findings to inform our decisions should we ever wish to create an app ourselves.</p>


```python
# Load user_reviews.csv
reviews_df = pd.read_csv("datasets/user_reviews.csv")

# Join and merge the two dataframe
merged_df = pd.merge(apps, reviews_df, on = 'App', how = "inner")

# Drop NA values from Sentiment and Translated_Review columns
merged_df = merged_df.dropna(subset=['Sentiment', 'Review'])

sns.set_style('ticks')
fig, ax = plt.subplots()
fig.set_size_inches(11, 8)

# User review sentiment polarity for paid vs. free apps
ax = sns.boxplot(x = 'Type', y = 'Sentiment_Polarity', data = merged_df)
ax.set_title('Sentiment Polarity Distribution')
```




    Text(0.5, 1.0, 'Sentiment Polarity Distribution')




    
![png](output_17_1.png)
    



```python

```
