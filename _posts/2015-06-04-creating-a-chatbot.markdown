---
author: nhuhu
comments: false
date: 2015-06-04 10:15:03+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/06/04/creating-a-chatbot/
slug: creating-a-chatbot
title: 'Project: Training a Basic Chatbot Part 1'
wordpress_id: 162
---

I started researching chatbots and how they worked. It was actually slightly disappointing because as far as it's concerned, chatbots basically use statistics and algorithms to determine what to respond (at least the basic ones).

Anyway, I still decided to play around with [this module](https://github.com/gunthercox/ChatterBot). It's pretty cool because you can train it as you speak to it. However... when I tried that feature, it was very VERY bad. It repeated itself and myself and it made no sense at all. So I decided to give it a little bit of pre-training. 

Now, I began looking for corpora of English dialogue and eventually found [this](https://www.agendaweb.org/listening/practical-english-conversations.html'), after a few unusable databases. It's an ESOL site and it was just PERFECT. It gave the dialogue in an easy A: B: format and I figured I could use BeautifulSoup to parse it pretty easily. 

That was when I ran into a problem, because I simply couldn't parse it. Instead, I had to use Regular Expressions, which I learnt way way back and had to look it up again. After a nice stroll down memory lane, I came up with these expressions:

{% highlight python %}
# A regular expression that targets "A: (blah blah) \n" OR "B: (blah blah) \n"
filt = re.compile('[A|B]: (.*?)\n')
# A regular expression that targets the reversed URL to get the base URL
linkfilt = re.compile('.*?(\/.*)')
{% endhighlight %}

The rest of the code was pretty easy actually; it just took a bit of time. The amount of dialogue is absolutely massive and I gave it a go at processing all of it today (i got up to about line 6000 of dialogue), but it took too long and I gave up. I'll do it tomorrow, but here's the functioning code:

{% highlight python %}
from chatterbot import ChatBot
import urllib
from bs4 import BeautifulSoup
import re

# A regular expression that targets "A: (blah blah) \n" OR "B: (blah blah) \n"
filt = re.compile('[A|B]: (.*?)\n')
# A regular expression that targets the reversed URL to get the base URL
linkfilt = re.compile('.*?(\/.*)')
links = []

# looks at the main page and gets a list of links to more links
main = 'https://www.agendaweb.org/listening/practical-english-conversations.html'
page = urllib.urlopen(main).read()
soup = BeautifulSoup(page)
main = []
for m in soup.find_all('a')[9:-20]:
    main.append(m['href'])

# uses those links to find the actual links to the dialogue
for m in main:
    print m
    link = m
    page = urllib.urlopen(link).read()
    soup = BeautifulSoup(page)
    # adds those links onto a list called links
    for l in soup.find_all('a')[:-1]:
        links.append((m, l['href']))

# a function that extracts the conversation into a list
def getconvo(url):
    page = urllib.urlopen(url).read()
    soup = BeautifulSoup(page)
    text = soup.get_text()

    return filt.findall(text)

# sets up the chatbot Noob
chatbot = ChatBot("Noob", logging=False)

counter = 1
for l in links:
    # reverses it so that the Regular Expression can be used
    reversedbase = linkfilt.findall(l[0][::-1])
    # reverses it again
    base = reversedbase[0][::-1]
    result = getconvo(base + l[1])
    for x in result:
        print counter, x
        counter += 1
    chatbot.train(result)

while True:
    response = chatbot.get_response(raw_input())

    print response
{% endhighlight %}
