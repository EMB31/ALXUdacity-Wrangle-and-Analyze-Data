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

```
