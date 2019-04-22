---
layout: post
title: Exploring my Last.fm Data
categories: [data analysis, python]
tags: [music,data,python,requests,pandas]
description: Reviewing nearly 12 year of last.fm data.
---

Welcome to my first step on my data analysis journey. There seemed no better place to start than to use a data set that I have been putting togther passively for 12 years, my music listening habits. A lot of this post has been heavily influenced by [Geoff Boeing's](https://geoffboeing.com/2016/05/analyzing-lastfm-history/) own analysis.

The first step is to get the data, [Last.fm kindly provide an easy to use API](https://www.last.fm/api) with a wide selection of methods, you will need an API key and the documentation is pretty straight forward on what you need to do. The method I am interested in for this will be the [getRecentTracks](https://www.last.fm/api/show/user.getRecentTracks). This holds all the tracks I have recorded at a limit of 200 per page, so to collect everything we will have to find out how many pages there are and iterate through each before dumping all the content into a CSV file the API call function can be seen below. Using python, matplotlib and pandas we can see what my musical history looks like.

{% highlight python %}
for page in range(1,total_pages + 1):
    print(f'Getting page {page}')
    api_url = f'https://ws.audioscrobbler.com/2.0/?method=user.getRecentTracks&user={creds["user"]}&api_key={creds["api_key"]}&limit=200&extended=0&page={page}&format=json'
    print(api_url)
    r = requests.get(api_url)
    while 'error' in r.json().keys():
        print(f'Error: {r.json()["message"]}')
        time.sleep(r.elapsed.total_seconds())
        r = requests.get(api_url)
    responses.append(r)
    print(f'Sleeping for {r.elapsed.total_seconds()} seconds')    
    time.sleep(r.elapsed.total_seconds())
{% endhighlight %}

 Over 12 years I have scrobbled 42,022 tracks attaining to 3,274 artists. So first of all lets look at who my favoured 25 artists are. 

![Top 25 Artists](/assets/lastfm-analysis/lastfm-artists-played-most.png)

Arctic Monkeys are massively ahead of anyone else, by some way, but I am not sure that this graph looking at my overall listening history represents much of what my listening habits are like now. Therefore, I want to investigate my overall music history to see if we can shed some light on why I feel my top listens are heavily set with artists from my youth. 

![Cumlative lists of Top 5 Artists](/assets/lastfm-analysis/lastfm-scrobbles-top-artists-years.png)

Here we can see the trends created from my cumulative listens over the last 12 years from my top 5 artists. Clearly it can be seen that two artists have continued to collect plays; Arctic Monkeys and Maximo Park, the latter enjoying a clear uplift in the last few years. While the remaining three artists have seen a stagnation in plays since 2009. Though I might have to blow the digital dust of the Bloc Party albums before I go and see them in July. 

So clearly I am not listening to as many of the artists that still dominate my top 25, but why haven't any of my new listening habits drifted into my top listens? I first of all looked to see if the amount I have been scrobbling has changed much over the years.

![Scrobbles per year](/assets/lastfm-analysis/lastfm-scrobbles-per-year.png) 


2011 was definitely an outlier of a year, with only one song being recorded (Two Door Cinema Club - You're Not Too Stubborn). So I thought I'd use this as a point to view what were my top artists, post and pre 2011 to see if that tells me more about my listening habits. 

My podcast listening has also increased as they have become more popular and assessible and eaten up the share of listening time away from music. Currently, I am subscribed to 49 podcasts and have a back catalog of 6.1GB worth of listening to get through. It can certainly be hard to keep up.

![Scrobbles Pre-2011](/assets/lastfm-analysis/lastfm-top-artists-before-2011.png)

![Scobbles Post-2011](/assets/lastfm-analysis/lastfm-top-artists-after-2011.png)

Post 2011 we have 14 new entries into the top 25 compared to pre 2015, but the threshold to climb up the chart is much lower, it is Blur that sneak the 25th spot with only 95 listens. But 16 of the artists I have listened too after 2011 still appear in the overall list. So really we can't make the assumption that my music taste as strayed too far away from the early days.

What is evident from after 2011 is that that amount of listens the top 25 artists are getting is much lower and distrubted more evenly. 

![Artists listened to per year](/assets/lastfm-analysis/lastfm-unqiue-artsists-per-year.png)

Lets now look at how many artists I am listening to in each year. This shows that I am definitely spreading my listening over more artists each year. The advent of Spotify, with personalised and curated playlists and instant access to music has definitely had an impact and help mature my music habits. This is evident in the current year in which we are only a third of the way through and I am close to listening to more artists than I ever did pre 2011. 

![First time I listened to a new artists](/assets/lastfm-analysis/lastfm-new-artist.png)

However, a more striking representation of the trend of my maturity, comes from how many new artists I have exposed myself to each year. Only 19 individual new artists were logged in 2007, while 2018 I listened to 636 artists I had never listened to before. 

Finally, I took a look into the number of unique tracks I have listened to from the top artists. Trying to investigate if I just listen to the same Arctic Monkeys track over and over again or did I listen to over 2,000 individual tracks the Rock'n'Roll gods? 

![Number of Tracks per artist in top 25](/assets/lastfm-analysis/lastfm-tracks-per-artist-h.png)

I think this could really be the test of some of my favoured artists, Do I like their range? The Beatles have the advantage of having a large back catalog, while the bottom three artists only have a small number of albums in the world. But it was the likes of The Libertines, Kaiser Chiefs and Jamie T who were part of the soundtrack of my formative years, along with the mainstays of Maximo Park, Arctic Monkeys and Bloc Party.

To add a bit of completion, I thought I would also vizulise the spread artist who have the biggest spread of tracks listened too overall.

![Number of tracks per artists from full dataset](/assets/lastfm-analysis/lastfm-tracks-per-all-artist-h.png)

To wrap up, I have done some simple analysis of my Last.fm dataset, this has given an interesting aspect of how my musical taste and behaviour has changed over the years. There is definitely further digging that could be done. What is my genre listening spread like? How many artists do I listen to just the once? Would it be significant to just look at artists that have n amount of listens?

But there is some significant flaws in the data, Last.fm in early on relied on you installing and configuring it's app on each computer you had with iTunes in order to collect the data. This meant that technical issues, changes of computer and the reliance on myself having everything setup in order to collect information. Also Last.fm has only been tracking Spotify data since 2014 and even then it relied on going into app settings of each device and entering log in details, unprovked. Thankfully now Spotify and Last.fm have integrated a connection of accounts once which should make data collected more reliable from now on.

All my code can be found at [GitHub](https://github.com/spacedlevo/lastfm-analysis.git)