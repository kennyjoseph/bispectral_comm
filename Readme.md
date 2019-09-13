# Introduction

The code in this repository presents tools that can be used to perform bispectral clustering on the full timelines (up to the last 3,200 tweets) of a set of users who have tweeted about a particular event.

The code is a public version of the code used in the paper *Who Says What with Whom: Using Bi-Spectral Clustering to Organize and Analyze Social Media Protest Networks*.  The code used to produce the results for the article contains sensative data and is conducted at a scale that requires large computational resources. For those interested in replicating this work, please contact the owner of this GitHub repository.  We have, however, provided example data from the article for those interested in further examining our results, in the ```cluster_examples``` directory. Details are below, see the section on **Cluster Examples from LDA and Bispectral Clustering**.

# A workflow for bispectral clustering of Twitter user timelines

The workflow to conduct bispectral clustering on user timelines has XXX steps, the first of which are optional.  Note that these steps require at least one set of Application credentials on Twitter. For more details on this, see [here](https://developer.twitter.com/en/apply-for-access.html)

1. **(Optional) Collect a set of tweets relevant to a particular event**- This can be done using a number of open source libraries. We used Twitter's [Hosebird](https://github.com/twitter/hbc) client, another popular option is the python library [tweepy](https://www.tweepy.org/).
2. **(Optional) Identify only users who have been retweeted**- You can use tweepy to carry out this task.  We use [twitter_dm](https://github.com/kennyjoseph/twitter_dm), a library built for collecting data from Twitter. In this repository, the script ```code/data_collection/recollect_tweets.py``` uses the ```twitter_dm``` library to recollect all tweets given a particular set of tweet IDs (extracted from the tweets collected in Step 1). Details about the script are provided below.
3. **Identify a set of users of interest**- If you carried out Step 1, this can be all users who tweeted during the event. If you carried out Step 2, this can be all users who sent at least one tweet that was retweeted at least once.  Otherwise, this can be any set of users of interest.  If possible, it is best to identify these users by their Twitter ID, as Twitter handles can change. However, either will work here.
4. **Collect up to the last 3,200 tweets of the user**- You can use ```tweepy``` or ```twitter_dm``` for this task. In this repository, the script ```code/data_collection/collect_user_data.py``` uses ```twitter_dm``` to collect the last 3,200 tweets from each user in a list provided as a text file. Details are provided below.
5. **Create the user-to-hashtag matrix**-  This matrix is defined in an edgelist format.  A sample of what is expected for bispectral clustering is provided below (see the section on **User-to-Hashtag File Format**)
6. **Run bispectral clustering** - We provide an R implementation of bispectral clustering here in ```code/bispectral/bispectral_cluster.R```. This implementation reads in the user-to-hashtag network in the format specified and produces various outputs to analyze results. Note that if one prefers to remain in python, there is a ```sklearn``` [implementation of the bispectral clustering algorithm](https://scikit-learn.org/stable/modules/biclustering.html#spectral-coclustering).
7. **(Optional) Analyze, filter, and repeat** - One can analyze results as produced by Step 6, or use it as a means to filter out uninteresting data, filtering out hashtags and users and repeating Steps 5-6.

# ```recollect_tweets.py```

This script takes 3 arguments:

- ```partial_path_to_twitter_credentials``` - A path to Twitter API credentials in ```twitter_dm``` format. See the ```twitter_dm``` Readme for details on this format
- ```file with tweet ids to collect``` - A text file with one tweet ID per line
- ```output_directory```- The name of an output directory.  The directory will contain ```.json``` files and ```.txt``` files. The JSON files contain all tweets still available (i.e. public and not deleted) from the API, as one json object per line.  The text files provide information on all tweets and whether or not they were able to be collected.

# ```collect_user_data.py```


This script takes 6 arguments:

- ```partial_path_to_twitter_credentials``` - A path to Twitter API credentials in ```twitter_dm``` format. See the ```twitter_dm``` Readme for details on this format
- ```file with users (ids or sns) to collect``` - A text file with one Twitter user ID or handle per line.  Note that IDs and handles cannot be mixed
- ```output_directory```- The name of an output directory.  The directory will always contain two subdirectories, ```obj``` and ```json```. The former is a pickled ```TwitterUser``` object from the ```twitter_dm``` library. The latter is a gzipped json file containing tweets, one json object per line. Each file within the subdirectory is identified by the ID of a given user.
- ```collect_friends (y/n)```- If ```y```, in the ```obj``` directory, the file relating to this user will have a ```friends``` variable that is populated with whom the user follows. **Note:** The rate limit for collecting friends is 1 user per minute, so this significantly slows data collection.
- ```collect_followers (y/n)``` - If ```y```, in the ```obj``` directory, the file relating to this user will have a ```followers``` variable that is populated with whom the user is followed by. **Note:** The rate limit for collecting followers is also 1 user per minute, so this significantly slows data collection.
- ```gen_tweet_counts_file (y/n)``` - If ```y```, a subdirectory in ```output_directory``` is created with a list of user IDs and the number of tweets collected for them

# User-to-Hashtag File Format

The file can be either a ```.tsv``` file or a ```.csv``` file. It can also be gzipped if desired.  The file should have three columns, ```Source```, representing the User, ```Target```, which specified the hashtags, and ```weight```, which gives the number of times this user used the given hashtag. Note that zero-entries are not required, i.e. values should only be specified where a user used a hashtag at least once.  For example:
```
Source, Target, weight
USER_ID_1, #cat, 4
USER_ID_1, #dog, 1
USER_ID_2, #cat, 10
...
```

# ```bispectral_cluster.R```

# Cluster Examples from LDA and Bispectral Clustering

In the ```cluster_examples``` directory, there are three files that present example clusterings from our paper.  Note that we **do not** include the users in each cluster, due to concerns over privacy.  The three files are described below:

- ```clusters_before_filtering.pdf``` - The top 25 hashtags, by frequency, in each of the clusters discussed in the paper *before* filtering out spam.  Each page represents a different cluster. Note that the plots unfortunately do not render non-English text well.
- ```clusters_for_case_study.pdf``` - The top 25 hashtags, by frequency, in each of the clusters discussed in the paper *after* filtering out spam.  A subset of these topics were used as examples in the case study presented in the article.  Each page represents a different cluster. Note that the plots unfortunately do not render non-English text well.
- ```lda_topics_lowest_perplexity_model.txt``` - The top 25 hashtags, by posterior probability weight, in each of the clusters from the LDA model described in the paper.  Note this is in text format, where each row gives a topic number and then the 25 words associated with that topic.


