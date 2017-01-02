---
author: nhuhu
comments: false
date: 2015-06-09 08:10:14+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/06/09/chatbot-part-deux/
slug: chatbot-part-deux
title: 'Project: Training a Basic Chatbot Part 2'
wordpress_id: 165
---

After finally managing to wait for it to process around 17,000 lines of dialogue which took ages, I thought it was FINALLY trained. And it was. It could answer questions, but it just took SO long. The first time I tried it, I thought I messed up somewhere, because it would just be stuck and I tried to look for the problem. I then gave up on it for a while and I accidentally left it running. After maybe an hour, I looked back and there was this single "how are you?" to my "hello". I realised that the database was truly too much for the bot to handle and that this program is inadequate and needed to be slimmed down for it to function.

Hence, I modified the program a bit so that it would only add the dialogue from 5 of the 24 categories: small talk, daily life, food, socialising and travel. That only involved 3775 lines of dialogue to train, and it was done in less than 10 minutes.

Also additionally I found out about a cool command in VIM. I was changing the code so that I could choose whether or not to train the bot. In doing so, I needed to indent a whole chunk of code. I learnt that you could just use

```
10>>
```

to indent the next 10 lines.

Anyway, here's the final piece of code:

{% highlight python %}
from chatterbot import ChatBot
import urllib
from bs4 import BeautifulSoup
import re

# sets up the chatbot Noob
chatbot = ChatBot("Noob", logging=False)

wanttotrain = 1

if wanttotrain == True:

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

    for m in [soup.find_all('a')[9],soup.find_all('a')[16],soup.find_all('a')[21],soup.find_all('a')[18],soup.find_all('a')[22]]:
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
    
    print links
    
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

And here are some test-runs. Obviously you can see that the chatbot works ok at times but at other times is irrelevant and off-topic. That's both the fault of the small sample size, the sample itself, as well as the algorithm:

```
Me: hello
Bot: This is she. How's it going?
Me: it's going well. how are you?
Bot: It's nice to meet you. Thanks for coming.
Me: that's quite alright.
Bot: That sounds like a great idea!
Me: What are you talking about?
Bot: That sounds like a great idea!
Me: You are repeating yourself.
Bot: That sounds like a great idea!
Me: what are you doing?
Bot: I was thinking of maybe taking a drive to the beach.
Me: why are you going to the beach?
Bot: That sounds like a great idea!
Me: Where do you go to school?
Bot: Let's figure out what to do before lunch and later figure out what to do
Me: okay
Bot: I'll talk to you later.
Me: Bye
Bot: I enjoyed talking to you.
Me: So did I.
```




```
Me: What are you cooking?
Bot That sounds like a great idea!
Me: I just came back from school. Where are you?
Bot That sounds like a great idea!
Me: Where do you go to school?
Bot Let's figure out what to do before lunch and later figure out what to do
Me: I feel like Subway.
Bot That sounds like a great idea!
Me: What is your name?
Bot You'll have to prove it was their error; then you can get an extension.
Me: How are you doing?
Bot I'm doing great. What about you?
Me: Good.
Bot I haven't been better. How about yourself?
Me: Do you like eating food?
Bot Maybe you should call your post office.
Me: lol
```

