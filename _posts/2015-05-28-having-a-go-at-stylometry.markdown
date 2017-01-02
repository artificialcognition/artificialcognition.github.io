---
author: nhuhu
comments: false
date: 2015-05-28 10:45:49+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/05/28/having-a-go-at-stylometry/
slug: having-a-go-at-stylometry
title: 'Project: Stylometry Part 1' 
wordpress_id: 142
---

I've decided to look into natural language parsing, since I've always been interested in it. Have you seen cleverbot? It's pretty cool. Anyway.. I managed to install [TextBlob](http://textblob.readthedocs.org/en/dev/index.html#) which is a natural language processor and went out and looked for cool things to start off making. I found [this](http://stackoverflow.com/questions/1793516/ideas-for-natural-language-processing-project) and decided to make a program that compares two texts to see if it can figure out if they have the same author.

So, I looked into this field and it turns out that it's called stylometry. I further found out about writer invariant and apparently it's a property that each author has. It's not agreed what this property is exactly but apparently "function words" are promising because they are used unconsciously. I then found [this article](http://www.biostat.jhsph.edu/~rpeng/papers/archive/authorship-tas2-final.pdf) which expanded on it.

The function words used in that article are: a, all, also, an, and, any, are, as, at, be, been, but, by, can, do, down, even, every, for, from, had, has, have, her, his, if, in, into, is, it, its, may, more, must, my, no, not, now, of, on, one, only, or, our, should, so, some, such, than, that, the, their, then, there, things, this, to, up, upon, was, were, what, when, which, who, will, with, would, your. 

Looking at this [document](https://www.google.co.nz/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0CBwQFjAA&url=ftp%3A%2F%2Fftp.cis.upenn.edu%2Fpub%2Ftreebank%2Fdoc%2Ftagguide.ps.gz&ei=8MFmVbaeNM_z8QX78oHgAQ&usg=AFQjCNFypM4svb-9zBORHTQQCj1llAyBbg&sig2=b5hBSrW_cQebsC0GDeAwjA&bvm=bv.93990622,d.dGc), I realised that these words weren't really all of the same category. Hence I needed to know what their "corresponding tags" actually were. So:

{% highlight python %}
from textblob import TextBlob

text = " a, all, also, an, and, any, are, as, at, be, been, but, by, can, do, down, even, every, for, from, had, has, have, her, his, if, in, into, is, it, its, may, more, must, my, no, not, now, of, on, one, only, or, our, should, so, some, such, than, that, the, their, then, there, things, this, to, up, upon, was, were, what, when, which, who, will, with, would, your"

blob = TextBlob(text)

stuff = []
for i in blob.tags:
    stuff.append(i[1])

tags = set(stuff)

for i in tags:
    print i, stuff.count(i)
{% endhighlight %}

It produced:
```
MD 6
VB 1
WRB 1
JJ 1
VBD 3
CC 3
WP 2
VBN 1
JJR 1
TO 1
CD 1
VBP 3
WDT 1
PRP 1
RB 8
IN 15
VBZ 2
DT 9
PRP$ 7
NNS 1
EX 1
```

Therefore, I decided, instead of using all the categories, to look only for IN, DT, PRP$ and MD. These are Preposition or subordinating conjunction, Determiner, Possessive Pronoun, and Modal Verbs respectively.

I started out focusing on just one text, Oliver Twist by Charles Dickens, available free online. I also focused on getting the INs from the text and getting a set of words.

{% highlight python %}
from textblob import TextBlob

ot = open("olivertwist.txt", "r")

ottext = TextBlob(ot.read())

otwc = len(ottext.tags)         # wordcount for oliver twist

targets = ['IN', 'DT', 'PRP$', 'MD']


def f(x): return x[1] == 'IN'

IN = filter(f, ottext.tags)
for i in xrange(len(IN)):
    IN[i] = (IN[i][0].lower(),IN[i][1])

blah = set(IN)
for x in blah:
    print x[0]
{% endhighlight %}

That worked, way faster than I thought it would (considering the novels ~170,000 words). So I decided to expand it and get it working with Great Expectations, also by Charles Dickens.

This is the complete code, and gives a result of an average percentage difference of 68.1%:
{% highlight python %}
from textblob import TextBlob

first = open("olivertwist.txt", "r")
firsttext = TextBlob(first.read())
firstwc = len(firsttext.tags)         # wordcount for oliver twist

second = open("greatexpectations.txt", "r")
secondtext = TextBlob(second.read())
secondwc = len(secondtext.tags)          # wordcount for great expectations

targets = ['IN', 'DT', 'PRP$', 'MD']

firstwords = {}
secondwords = {}

for target in targets:
    def f(x): return x[1] == target
    
    firsttags = filter(f, firsttext.tags)   # gets the targeted tags
    for i in xrange(len(firsttags)):
        firsttags[i] = (firsttags[i][0].lower(),firsttags[i][1])   # reduces to lowercase
    
    secondtags = filter(f, secondtext.tags)   # gets the targeted tags
    for i in xrange(len(secondtags)):
        secondtags[i] = (secondtags[i][0].lower(),secondtags[i][1])   # reduces to lowercase

    firsttags_set = set(firsttags)
    secondtags_set = set(secondtags)

    for x in firsttags_set:
        firstwords[x[0]] = (float(firsttags.count(x))/firstwc)*1000  
        # freq in 1000 words
    
    for x in secondtags_set:
        secondwords[x[0]] = (float(secondtags.count(x))/secondwc)*1000  
        # freq in 1000 words

difference = []
number = 0

for word in firstwords:
    if word in secondwords:
        number += 1
        diff = [firstwords[word], secondwords[word]]
        percentchange = (max(diff)-min(diff))/min(diff)*100
        difference.append(percentchange)
summed = 0
for x in difference:
    summed += x

print summed/number
{% endhighlight %}

I still haven't tested out the cut-off limit for different authors. So I guess I'll try it out tomorrow. Can't wait, that was pretty fun and satisfying to make! 
