---
author: nhuhu
comments: false
date: 2015-06-01 07:35:45+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/06/01/creating-a-context-based-translator/
slug: creating-a-context-based-translator
title: 'Project: Creating a Context-based Translator Using Data Mining'
wordpress_id: 155
---

In line with parsing languages, I decided for my next project to be to translate stuff. I've been learning French and I've noticed that Google Translate _really_ isn't that good at translating. When I do have to translate a sentence, I use Reverso. It allows me to search my sentence and comes up with pre-translated sentences (from other sources) that best match it. It often only matches a quarter or half of a sentence or even less, so it's not very automatic and requires fine-tuning by me, with other searches.

I thought that I would solve this problem. The first thing I looked at on Reverso was whether or not there was an APA that I could use. Unfortunately, there was not. I vaguely remembered a Python library called BeautifulSoup to parse the HTML. This was my first time doing so and I found it interesting that I was parsing HTML to get the results from a natural language search thing to translate a text.

I had trouble with the library at first, but eventually I (with my google-foo) managed to work out how to get the first result from a search.

{% highlight python %}
import urllib
from bs4 import BeautifulSoup

page = urllib.urlopen('http://context.reverso.net/translation/english-french/i+like+the+green+side+of+the+leaf').read()
soup = BeautifulSoup(page)

print soup.find_all("div", { "class" : "src" })[0].em
print soup.find_all("div", { "class" : "trg" })[0].em
{% endhighlight %}

Now that I had that, I just needed to get an input sentence and work this magic on it. I thought that I would iterate searching the sentence and then the sentence minus a word and so on, until a search term completely matches a pre-done translated text. This will ensure that the translation is as accurate as possible. 

{% highlight python %}
import urllib
from bs4 import BeautifulSoup

text = "Mary was born in Leiden in the Netherlands"
text = text.split(' ')
translated = False

# searches to see if there is a whole match
search = "+".join(text)
url = 'http://context.reverso.net/translation/english-french/' + search
page = urllib.urlopen(url).read()
soup = BeautifulSoup(page)
if ' '.join(text) == soup.find_all("div", { "class" : "src" })[0].em.text:
    translated = True
    print soup.find_all("div", { "class" : "trg" })[0].em.text

twords = ""
originallength = len(text)

# checks until it finds a match
while translated == False:
    if len(text) == 0:
        translated = True

    if len(text) == 1:
        search = "+".join(text)
        url = 'http://context.reverso.net/translation/english-french/' + search
        page = urllib.urlopen(url).read()
        soup = BeautifulSoup(page)
        
        if ' '.join(text) == soup.find_all("div", { "class" : "src" })[0].em.text:
            twords += soup.find_all("div", { "class" : "trg" })[0].em.text
            twords += " " # add a space to the end for further words
            text = []
            break

    for i in range(1,len(text)):
        newtext = text[:-i]
        search = "+".join(newtext)
        url = 'http://context.reverso.net/translation/english-french/' + search
        page = urllib.urlopen(url).read()
        soup = BeautifulSoup(page)

        if ' '.join(newtext) == soup.find_all("div", { "class" : "src" })[0].em.text:
            twords += soup.find_all("div", { "class" : "trg" })[0].em.text
            twords += " " # add a space to the end for further words
            text = text[-i:]
            
            search = "+".join(text)
            url = 'http://context.reverso.net/translation/english-french/' + search
            page = urllib.urlopen(url).read()
            soup = BeautifulSoup(page)
        
            if ' '.join(text) == soup.find_all("div", { "class" : "src" })[0].em.text:
                twords += soup.find_all("div", { "class" : "trg" })[0].em.text
                twords += " " # add a space to the end for further words
                text = []

            print twords
            break
{% endhighlight %}

It took a while, but here is the test output for "the dog plays with the ball":
```
le chien 
le chien se joue de la balle 
```

Surprisingly, the conjugations come out fine most of the time, which I thought was pretty weird. The translator is pretty good, but sometimes the translated text requires a bit of cleaning up. One problem is when the original text is part of an idiom, which messes up the translated text. After cleaning up the code, adding comments and an interface (so you can choose what language you want to translate into), here is the final code:

{% highlight python %}
import urllib
from bs4 import BeautifulSoup

text = raw_input('Translate:\n')
language = raw_input('Into Arabic, German, Spanish, French, Italian, Dutch, Portuguese?\n')
text = text.split(' ')    # puts the words in a list
translated = False

# searches to see if there is a whole match
search = "+".join(text)
url = 'http://context.reverso.net/translation/english-'+language+'/' + search
page = urllib.urlopen(url).read()
soup = BeautifulSoup(page)
# if the search matches the english completely, it is printed and the program terminates
if ' '.join(text) == soup.find_all("div", { "class" : "src" })[0].em.text:
    translated = True
    print soup.find_all("div", { "class" : "trg" })[0].em.text

twords = ""
originallength = len(text)

# checks until it finds a match
while translated == False:
    if len(text) == 0:         # if all of the text is translated, program ends
        translated = True

    if len(text) == 1:         # if there is one word left to be translated
        search = "+".join(text)
        url = 'http://context.reverso.net/translation/english-'+language+'/' + search
        page = urllib.urlopen(url).read()
        soup = BeautifulSoup(page)
        
        if ' '.join(text) == soup.find_all("div", { "class" : "src" })[0].em.text:
            twords += soup.find_all("div", { "class" : "trg" })[0].em.text
            twords += " " # add a space to the end for further words
            text = []
            break

    for i in range(1,len(text)):
        newtext = text[:-i]
        search = "+".join(newtext)
        url = 'http://context.reverso.net/translation/english-'+language+'/' + search
        page = urllib.urlopen(url).read()
        soup = BeautifulSoup(page)
        
        # finds if there is a match for the segment
        if ' '.join(newtext) == soup.find_all("div", { "class" : "src" })[0].em.text:
            twords += soup.find_all("div", { "class" : "trg" })[0].em.text
            twords += " " # add a space to the end for further words
            text = text[-i:]  # removes the already-translated text
            
            search = "+".join(text)
            url = 'http://context.reverso.net/translation/english-'+language+'/' + search
            page = urllib.urlopen(url).read()
            soup = BeautifulSoup(page)
            
            # checks the whole new segment to see if there's a match
            if ' '.join(text) == soup.find_all("div", { "class" : "src" })[0].em.text:
                twords += soup.find_all("div", { "class" : "trg" })[0].em.text
                twords += " " # add a space to the end for further words
                text = []

            print twords # prints the progress
            break
{% endhighlight %}


