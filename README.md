# ALXUdacity-Wrangle-and-Analyze-Data
This project involves wrangling and analyzing the WeRateDogs twitter archive.
For this project data was gathered from three different sources for analysis.

The project contains the following files:
1. wrangle_act (this contains the code for the wrangle process and visualization)
2. wrangle_report (contains a report describing the wrangling process)
3. act_report (this a report about the entire project and its finding aimed to serve as a blog post type document)
4. twitter-archive-enhanced.csv file (WeRateDogs twitter archive data)
5. tweet_json.txt (contains additional data acquired via Twitter API)
6. image_predictions.tsv (contains the tweet image prediction data)
7. twitter_archive_master.csv (master dataset following merge of the 3 datasets)

# Introduction
The WeRateDogs twitter archive data was analyzed for this project. WeRateDogs is a twitter account that rates people's dog often with humorous comments about the dogs. 

There were 3 datasets to be acquired from 3 different sources:
1. Archive data read from a .csv file
2. Image predictions data, which was a .tsv file
3. Additional data queried via twitter API and loaded into a txt file.

The datasets contain the tweetid, user_id, timestamp for a tweet, source, tweet text, retweets, rating numerator and denominator, dog name, dog stage, amongst others.

# Data Gathering
Each dataset was acquired differently:

### WeRateDogs twitter archive
The twitter archive file provided by Udacity was manually downloaded and read into a dataframe.

```
# import required library
import pandas as pd

#read twitter-archive-enhanced.csd into a dataframe
df_twitter_arch = pd.read_csv('twitter-archive-enhanced.csv')

#display few rows of data to confirm upload is done
df_twitter_arch.head()
```

### Image predictions file
This file was programmatically downloaded and read into a dataframe.
```
#import the required library
import requests

#download the image predictions tsv file
res = requests.get('https://d17h27t6h515a5.cloudfront.net/topher/2017/August/599fd2ad_image-predictions/image-predictions.tsv')

#confirm if successful
res.status_code
```
An output of 200 confirms download success.
```
#read the content of the tsv file into a dataframe
with open("image_predictions.tsv", mode='wb') as file:
    file.write(res.content)

df_image_predictions = pd.read_csv('image_predictions.tsv', sep = '\t')

#display a few rows of data in the dataframe
df_image_predictions.head()
```

### Data from Twitter API
In order to query data via API you should have a twitter developer account.
This will provide the API Key and Key secret, Access token and Access token secret.

Data was queried as follows:
```
#import required libraries
import tweepy
import json
from timeit import default_timer as timer

# get twitter api keys
APIKey = ''
APIKeySecret = ''
AccessToken = ''
AccessTokenSecret = ''

#authenticate the keys
authenticate = tweepy.OAuthHandler(APIKey, APIKeySecret)
authenticate.set_access_token(AccessToken, AccessTokenSecret)

twit_api = tweepy.API(authenticate, wait_on_rate_limit=True)

cnt = 0
failed_dict = {}
start = timer()

twit_id = df_twitter_arch['tweet_id'].values
```
Then create and write the api data into a txt file.
```
#create the tweet_json.txt file and write the data from the twitter api into it
with open('tweet_json.txt', mode='w') as outfile:
    for tweet_id in twit_id:
        cnt += 1
        print(str(cnt) + ": " + str(tweet_id))
        try:
            tweets = twit_api.get_status(tweet_id, tweet_mode='extended')
            print("Success")
            json.dump(tweets._json, outfile)
            outfile.write('\n')
        except tweepy.error.TweepError as e:
            print("Fail")
            failed_dict[tweet_id] = e
            pass
end = timer()
print (end - start)
```
![image](https://user-images.githubusercontent.com/113180085/201698576-f077c752-b946-4183-8a0f-d22f428bba75.png)

Read the txt file into a dataframe
```
# read twitter api data into a dataframe
import json
twitterapi_data = []
with open ('tweet_json.txt') as file:
    for line in file:
        tweets = (json.loads(line))
        twt_id = tweets['id']
        retwt_cnt = tweets['retweet_count']
        fav_cnt = tweets['favorite_count']
        
        twitterapi_data.append({'tweet_id': twt_id,
                        'retweet_count': retwt_cnt,
                        'favorite_count': fav_cnt})
    
    df_twt_apidata = pd.DataFrame(twitterapi_data)

#display a few rows of data in the dataframe
df_twt_apidata.head()
```
![image](https://user-images.githubusercontent.com/113180085/201698941-98814c0d-0eef-4ed0-a337-9cc4ea80649a.png)

# Assessing Data
Data assessment was done both programatically and visually.
As part of requirements for the project; at least 8 quality issues and 2 tidiness issues were to be raised. 

The info and describe methods were used to assess data. The dataframes were also loaded and observed.

```
df_twitter_arch.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 2356 entries, 0 to 2355
Data columns (total 17 columns):
tweet_id                      2356 non-null int64
in_reply_to_status_id         78 non-null float64
in_reply_to_user_id           78 non-null float64
timestamp                     2356 non-null object
source                        2356 non-null object
text                          2356 non-null object
retweeted_status_id           181 non-null float64
retweeted_status_user_id      181 non-null float64
retweeted_status_timestamp    181 non-null object
expanded_urls                 2297 non-null object
rating_numerator              2356 non-null int64
rating_denominator            2356 non-null int64
name                          2356 non-null object
doggo                         2356 non-null object
floofer                       2356 non-null object
pupper                        2356 non-null object
puppo                         2356 non-null object
dtypes: float64(4), int64(3), object(10)
memory usage: 313.0+ KB

df_image_predictions.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 2075 entries, 0 to 2074
Data columns (total 12 columns):
tweet_id    2075 non-null int64
jpg_url     2075 non-null object
img_num     2075 non-null int64
p1          2075 non-null object
p1_conf     2075 non-null float64
p1_dog      2075 non-null bool
p2          2075 non-null object
p2_conf     2075 non-null float64
p2_dog      2075 non-null bool
p3          2075 non-null object
p3_conf     2075 non-null float64
p3_dog      2075 non-null bool
dtypes: bool(3), float64(3), int64(2), object(4)
memory usage: 152.1+ KB

df_twt_apidata.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 2327 entries, 0 to 2326
Data columns (total 3 columns):
favorite_count    2327 non-null int64
retweet_count     2327 non-null int64
tweet_id          2327 non-null int64
dtypes: int64(3)
memory usage: 54.6 KB

```

```
#This is to check if there are any negative or unuasual values
df_twitter_arch.describe()
```
![image](https://user-images.githubusercontent.com/113180085/201716104-a9532f1c-2c92-46b8-9afe-df43cf7f791a.png)

```
df_twt_apidata.describe()
```
![image](https://user-images.githubusercontent.com/113180085/201717256-8482c837-aa89-4191-a27f-ce6fafd177a1.png)

Following the data assesment the following  data quality and tidiness issues were discovered:
### Quality Issues
1. Remove all retweets from the df_twitter_arch dataframe
2. In the df_twitter_arch (twitter archive dataset) -in_reply_to_status_id, in_reply_to_user_ column have numerous null values
3. Change datatypes for columns listed below:
     - timestamp column is a string instead of a datetime column in df_twitter_arch dataframe<br>
    - tweet_id column should be a string instead of an int datatype in all 3 dataframes<br>
    - rating_numerator and rating_denominator columns should be of datatype float instead of int in df_twitter_arch dataset<br> 
4. df_twitter_arch (twitter archive dataset) - Remove retweet-related columns
5. df_twitter_arch (twitter archive dataset) - HTML tags in the source column
6. For some of the tweet ids in the twitter archive dataset there are no corresponding data values in the dataset retrieved via the Twitter API or the image prediction dataset.
7. 'None' in the dog stages columns (doggo, floofer, pupper and puppo) is treated as a non-null value instead of being null values.
8. Some column headers in the df_image_predictions dataframe are non-descriptive (p1, p1_conf, p1_dog, p2, p2_conf, etc)

### Tidiness Issues
1. The dog stages in the twitter archive dataset have been split into separate columns instead of being one column with a dog's stage indicated
2. Tweets about the dogs are spread accross 3 different files/dataframes.

# Cleaning Data

The Define, Code and Test method was used for the data cleaning process.

### Issue #1: Timestamp, tweet_id, rating_numerator and rating_denominator columns have incorrect datatypes

Define
- Convert the Timestamp column to datetimecolumn using pandas to_datetime() method.
- Convert tweet_id column to string
- Convert rating_numerator and rating_denomintor columns to float

Code
```
#Convert timestamp column from a string to a datetime column
df_twitter_arch['timestamp'] = pd.to_datetime(df_twitter_arch['timestamp'])

#Convert tweet_id column to string
df_twitter_arch['tweet_id'] = df_twitter_arch['tweet_id'].astype(str)
df_image_predictions['tweet_id'] = df_image_predictions['tweet_id'].astype(str)
df_twt_apidata['tweet_id'] = df_twt_apidata['tweet_id'].astype(str)

#Convert rating_numerato and denominator columns to float
df_twitter_arch['rating_numerator'] = df_twitter_arch['rating_numerator'].astype(float)
df_twitter_arch['rating_denominator'] = df_twitter_arch['rating_denominator'].astype(float)
```

Test
```
#confirm that the datatype for the columns listed above has been changed
df_twitter_arch.info()

<class 'pandas.core.frame.DataFrame'>
RangeIndex: 2356 entries, 0 to 2355
Data columns (total 17 columns):
tweet_id                      2356 non-null object
in_reply_to_status_id         78 non-null float64
in_reply_to_user_id           78 non-null float64
timestamp                     2356 non-null datetime64[ns]
source                        2356 non-null object
text                          2356 non-null object
retweeted_status_id           181 non-null float64
retweeted_status_user_id      181 non-null float64
retweeted_status_timestamp    181 non-null object
expanded_urls                 2297 non-null object
rating_numerator              2356 non-null float64
rating_denominator            2356 non-null float64
name                          2356 non-null object
doggo                         2356 non-null object
floofer                       2356 non-null object
pupper                        2356 non-null object
puppo                         2356 non-null object
dtypes: datetime64[ns](1), float64(6), object(10)
memory usage: 313.0+ KB
```
