---
layout: post
title: Automating my Super 6 with Requests
categories: [python, programming, automation]
tags: [requests, python, beautiful soup]
description: Using the Requests python library and Beautiful Soup, to login and then submit data to an online form. 
---

Sky Super6 is a football prediction contest, each round Sky Sports select 6 fixtures in which enter for free your predictions on the final result. Winners take home a cash prize of £250k. You can also pit yourself in a seasonal league with your friends. But rounds can come around on any day and if you forget to submit predictions before the deadline, it can put you well behind your opposition. 

So using Python, the Requests module and BeautifulSoup I am going to automate the collection of my scores as well as their sumbission. My aim here really isn't to teach you how to use the Super 6 in particular, but really to use it as an example of how to log into a page and submit some data.

In order to achieve our task let us breakdown our problem into stages we will need to program.
* Get data
* Login to site
* POST data to Super 6 Server
* Tell me when it is complete

## Get Data
I will leave this to you, I don't really want to publish my own way of doing it. But rest assured it isn't anything of a sophisticated model. I scrape what the fixtures will be for the round, do a bit of math and get to a integer, stick it in a dictonary that I then use to submit my goal predictions. 

## Login to site
Using requests we first of all want to iniate a requests session. This can be done via the context manager `with requests.Session() as s` within I hold all my function calls and handle the cookies of the session that will be required to keep you logged in.

{% highlight python %}

with requests.Session() as s:
    super6_login()
    create_scores(data)
    pushover_notification("Super6 Done")

{% endhighlight %}

 Our aim is to repricate what the browser is asking forunder the hood when you are navigating the site. You therefore want to headover to your preferred browser and open up the inspector. 

[![Logging into Super6](/assets/super6-automation/super6-login-post.png)](/assets/super6-automation/super6-login-post.png)

Now you will want to login as normal, but with the network tab of your inspector open, watch the activity when you click the login button. It is likely that the first request made will be that to authenticate. We really want to see where the requests being made and what information is being sent to the server. That information will be found in the 'Params' tab. In this case *pin* which is your password and *username*. I found just sending that data wasn't enough, with a bit of trial and error I figured something particular was required in the headers. So I copied and pasted what my browser requests to get a valid status code. 

It is then important to see what the server is sending back to you, much of the process will be trial and error and watching to see what information is going back and forth. In this case an ssoToken is required, used in order to allow you to use your skyID around several of there services. Once you have this information, to finally authenticate you will have to send another POST request containing that token. All the login process is wrapped up in the function below;

{% highlight python %}

def super6_login():
    paras = {'username': user, 'pin': pwd}
    headers =  {    
                'Host':'www.skybet.com',
                'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:65.0) Gecko/20100101 Firefox/65.0',
                'Accept': 'application/json',
                'Accept-Language': 'en-US,en;q=0.5',
                'Accept-Encoding': 'gzip, deflate, br',
                'Referer': 'https://www.skybet.com/secure/identity/m/login/super6?urlconsumer=https://super6.skysports.com&dl=1',
                'content-type': 'application/json',
                'x-requested-with': 'XMLHttpRequest',
                'Origin': 'https://www.skybet.com',
                'Content-Length': '38',
                'Connection': 'keep-alive'
    }
    token_json = s.post(login_site, json=paras, headers=headers)
    user_token = token_json.json()['user_data']['ssoToken']
    s.post('https://super6.skysports.com/auth/login', data={'token':user_token})
    return None

{% endhighlight %}

## Posting Our Data

Now that the server knows who I am, it is time to tell it what scores I want submitting.

{% highlight python %}

def create_scores(predictions):
    data = {}
    r = s.get(play_page)
    soup = BeautifulSoup(r.content, 'html.parser')
    match_cards = soup.find_all('div', class_='js-challenge flex-item prediction-card prediction-card__prelive')
    ids = [card['data-challenge-id'] for card in match_cards]
    for id_ in ids:
        fixtures = soup.find_all('div', {'data-challenge-id':id_})
        for fixture in fixtures:
            teams = [i['data-shortname'] for i in fixture.find_all('p', class_='team-name flush--bottom js-team-name')]
            for num, team in enumerate(teams, 1):
                data["score[{}][team{}]".format(id_, num)] = predictions[team]
                data["score[{}][team{}]".format(id_, num)] = predictions[team]
    data["confirmed"] = 1
    data["goldengoal[{}]".format(int(ids[-1]) + 1)] = 11
    data["_csrf_token"] = soup.find('input', {'name':'_csrf_token'})['value']
    submit = s.post(play_page, data=data, timeout=3)
    return submit.status_code

{% endhighlight %}

I already have the data that requires submit, I just need to put that into a format that the server will accept. Again it is back to the network tab to review data is being sent to the servers. 

![Super6 Post Request](/assets/super6-automation/super6-play-url.png)

![Super6 Post Request](/assets/super6-automation/super6-play-scores.png)

Here we see that each fixture has an ID and another key for team1 (home team) and team2 (away team). My first task will be to map my data to those IDs in order to submit the right number of goals to the right team. I first of all need to collect all the fixture information on the site using BeautifulSoup. I collect all the ids that are on the page and then use the short-name informaton which I have also scraped from initally reviewing the fixtures to map the ID to the team and as a result find the value of the prediction for that team. 

But we also need to send a CSRF token to the server. This tells the server that we are the same person that started the session stopping someone hijacking our session to send authorised information to the server if they are snooping on our traffic. These tokens are usually produced in the session and hide within the HTML. In this case within the input tag with a name of `_csrf_token`. We will use the BeautifulSoup library to pick out this value, a quick search for 'csrf' will locate the attributes you will need for this element. 

And with that complete I can run the place the script on my Raspberry Pi. Coupling this up with Pushover Notification API and a cronjob each day it will test to see if we require to post, if so it will do so before sending me a notification to inform me all is complete. 

Hopefully this will give you some extra knowledge to help you map the process to your next project that requires you to log in. I can't promise that everything will be as simple, but the most important thing to remember is that whatever your browser is doing should be able to be coded with python. Trial and error, while trying to understand the requests to and throw from a server will help you put something together. 

Code can be found at this [GitHub repo](https://github.com/spacedlevo/submit_super6).