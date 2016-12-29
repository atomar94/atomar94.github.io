---
layout: post
title: Playing with Reddit Using Python, PRAW, and Pandas
---

This is part 1 of a 3 part series about using Python to gather comments from Reddit and analyze them using Seaborn, Pandas, and Matplotlib.

In this part I am going to give an overview of using [PRAW](https://praw.readthedocs.io/en/latest/), a Python Reddit API Wrapper, to collect Reddit comments and load it into a Pandas Dataframe.

# Setup and Installation

## Python & Dependencies

For this project I am using Anaconda Python 3.5 interpreter, which you can download [here.](https://www.continuum.io/downloads) A few of the other libraries, such as Pandas and Numpy, come preinstalled with Anaconda and I highly recommend it.

To grab Reddit comments I am using PRAW, which I linked above. To download the PRAW package for Anaconda open the Anaconda Prompt and run

```shell
pip install PRAW
```

## Registering Your Reddit Bot

In order to use PRAW, or any other Reddit API, we first need to register our bot through Reddit. You must have a Reddit account in order to create a bot.

1. Go to the Reddit Application Preferences page [here.](https://ssl.reddit.com/prefs/apps)

2. Create a new Application of type "Web App" and in the Description field put a small description of what your bot does.
..* The About URL field can be left blank. The Redirect URL field put "http://example.com/redirect".

![Reddit Bot Config]({{ site.baseurl }}/images/reddit-bot-fields.png)

# Connecting to Reddit With PRAW

To connect PRAW to Reddit we need to supply our client ID, client secret, and user agent.

```python
import praw

r = praw.Reddit(client_id = "02......",
    client_secret = "Ts.....",
    user_agent = "A Reddit Scraping bot made by /u/<your-reddit-username>")
```

This returns a Reddit object *r* that we can use to collect all kinds of Reddit data.

*Note - The user_agent field must be populated with a small description of what you are doing and your reddit username.*

## Credential Handling

It's nice to have all your code on Github but if you put your client secret in your code then you probably shouldn't upload it. After all it's supposed to be, well, secret! So how can we get around this?

Luckily for us Python has some great built-in libraries to help deal with configuration files and we can use those to help us. The [configparser](https://docs.python.org/3/library/configparser.html) library is a really simple and easy way to create, read, and write config files.

Because my program only needs to read from cfg files I didn't implement file creation support. Instead I created a config file with the code below and then changed the filler values in a text editor.

```python
import configparser

cfg = configparser.ConfigParser()
cfg['CREDENTIALS'] =  {"Username": "filler",
                       "Password": "filler"}
cfg['AUTH'] = {"ClientID": "filler",
               "Secret": "filler",
               "UserAgent": "Playing with the Reddit API by <filler>"}

with open("credentials.ini", 'w') as cfgfile:
    cfg.write(cfgfile)
```

This code will produce a file with the following contents. You can replace the contents with your reddit username, password, and authentication information. I don't end up using the username and password fields in this tutorial but it's nice to have in case you choose to start using PRAW to interact with Reddit instead of just pulling content. I recommend adding "credentials.ini" to your .gitignore file so that you don't accidentally upload your secret info to the internet.

```
[CREDENTIALS]
username = filler
password = filler

[AUTH]
clientid = filler

useragent = Playing with the Reddit API by <filler>
secret = filler
```

You can load the credentials from this file using the code below.

```python
# configparser.py
# https://github.com/atomar94/Reddit_Bot/blob/master/configloader.py

import configparser

class ConfigLoader(configparser.ConfigParser):

	def __init__(self, cfgfile=""):
		super(configparser.ConfigParser, self).__init__()

		self.username = ""
		self.password = ""
		self.client_id = ""
		self.client_secret = ""
		self.user_agent = ""

		if cfgfile:
			self.read(cfgfile)

	def read(self, filename):
		super(configparser.ConfigParser, self).read(filename)
		self.username = self["CREDENTIALS"]["Username"]
		self.password = self["CREDENTIALS"]["Password"]
		self.client_id = self["AUTH"]["ClientID"]
		self.client_secret = self["AUTH"]["Secret"]
		self.user_agent = self["AUTH"]["UserAgent"]
```

I made ConfigLoader a subclass of ConfigParser because I wanted to use all of the underlying functionality that ConfigParser provides but I am only using this class to load Reddit credentials so I overrode the read() method to also load the values from the credential file into memory.

We can use this code to simplify our PRAW object creation.

```python
# main.py
# https://github.com/atomar94/Reddit_Bot/blob/master/main.py

import praw
import configloader

if __name__ == "__main__":
	cfg = configloader.ConfigLoader("credentials.ini")

	r = praw.Reddit(client_id = cfg.client_id,
	 client_secret = cfg.client_secret,
   user_agent = cfg.user_agent)
```

## Collecting Reddit Comments

Let's start by getting the comments from the top 25 submissions in /r/AskReddit

```python

comments = []
for submission in r.subreddit("askreddit").hot(limit=25):
  submission.comments.replace_more(limit=32)
  comments.append(submission.comments.list())
```

Here I'm asking for the top 25 posts in "askreddit" and iterating through each of the submission objects. When returning Reddit comments sometimes Reddit will return to you a chunk of comments, analagous to a "Load More Comments" button on the website. To load and return all the comments we use *submission.comments.replace_more(limit=0)*. For more info see [here](https://praw.readthedocs.io/en/latest/tutorials/comments.html#extracting-comments)

Then I append the full list of comments into the *comments* list.

# Loading Everything into Pandas

>pandas is a Python package providing fast, flexible, and expressive data structures designed to make working with “relational” or “labeled” data both easy and intuitive.

 - pandas.pydata.org

Pandas is a great way to store large data sets in Python. If you've never used Pandas I highly recommend the [10 Minutes to Pandas](http://pandas.pydata.org/pandas-docs/stable/10min.html) tutorial on the Pandas website.

For the rest of the tutorial I am going to analyze Reddit comments based on their length and score so we can load that into a Pandas Dataframe.

```python
import pandas as pd

df = pd.DataFrame({"Length": [len(x.body) for x in comments],
                  "Score":   [x.score for x in comments]
                  })
```

I jumped around a bit in this tutorial so at the end your code should roughly look like this. The complete configloader.py file is shown above.

```python
# main.py

import pandas as pd
import praw
import configloader

if __name__ == "__main__":
	cfg = configloader.ConfigLoader("credentials.ini")

	r = praw.Reddit(client_id = cfg.client_id,
	 client_secret = cfg.client_secret,
   user_agent = cfg.user_agent)

   comments = []
   for submission in r.subreddit("askreddit").hot(limit=25):
     submission.comments.replace_more(limit=32)
     comments.append(submission.comments.list())

   df = pd.DataFrame({"Length": [len(x.body) for x in comments],
                     "Score":   [x.score for x in comments]
                     })
```
