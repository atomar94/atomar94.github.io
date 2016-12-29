---
layout: post
title: Who Like Long Comments? Analyzing Reddit Comment Length and Popularity
---

Doing some big data analysis on the distribution of Reddit comments by score and length.

# Comment Length and Score by Subreddit

I started by pulling in all the comments from the front page of the 25 most popular subreddits. I plotted the distribution of comment length and score to see where most posts end up. I.e. "What subs have longer comments?" and "Which subs have higher comment scores?"

These were the distributions for the comment length by subreddit:

![commentlength distribution]({{ site.baseurl }}/images/comment_length_distribution.png)

I think the most interesting thing here is that some defaults get significantly more comment traffic than others. Subs like /r/Aww and /r/Videos have generally low comment counts on their posts, while subs like /r/AskReddit and /r/Iama have a few really popular posts but the rest are pretty low-traffic.

And these were the distributions for the comment score, again by subreddit:

![comment score distribution]({{ site.baseurl }}/images/comment_score_distribution.png)

Aside - It was pretty hard to get Seaborn (the plotting library I used) to not clip text and fit everything into one image nicely. I spent many hours tweaking small graphical things to try and get it to "look nice", this is my best effort.

I found it pretty interesting that some subs like /r/Television and /r/AskScience have so many comments with only a handful of upvotes. I figured comments would be skewed like this but I didn't expect it to be so different between subs. I'm not sure what's happening in /r/LifeProTips and /r/ExplainLikeImFive; they have very little comments with a handful of upvotes while subs like /r/EarthPorn have low scores on most of their comments.

# Score vs. Length - The Comment Landscape

So what's the sweet-spot for Reddit comments? How often do we upvote longer comments vs. shorter comments? Is there an optimal length for comments in order to maximize your karma/character?

If we plot a comment's length against its score we should be able to see what types of comments are getting upvoted. Sadly, my computer was having trouble plotting high-resolution graphs with the amount of data I had. That's why these are a bit blocky.

![score and length]({{ site.baseurl }}/images/score_and_length.png)

These are probability density graphs, with the darker green corresponding to a higher number of comments at that length and score.

I tried pooling all the comments together in one big graph. I stripped off the top 15% of most frequent scores to try and flatten out the graph a bit and get a look at the interesting behavior, i.e. not just a bunch of comments with a score of 1 but to see more of the trends in the data.

![score and length total]({{ site.baseurl }}/images/score_and_length_total.png)

Even with the peaks trimmed there is still a single concentration at about 40 characters long getting between 1-10 karma.

If you liked this analysis of the reddit comments or have another idea of things you'd like me to look at leave a comment below! I'd love more ideas of cool stuff to play around with.
