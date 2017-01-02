---
author: nhuhu
comments: false
date: 2015-08-22 07:54:10+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/08/22/the-programmed-poet/
slug: the-programmed-poet
title: 'Project: Utilizing Convolutional Neural Networks to Create Poetry (EDIT: And got Published!)'
wordpress_id: 184
---

A while ago, I came across Zack's post entitled ["Turning test: passed using computer generated poetry"](https://rpiai.wordpress.com/2015/01/24/turing-test-passed-using-computer-generated-poetry/), who had apparently generated a poem that was accepted into a literary journal. While I thought this was amazing at first, there was a problem. Firstly, I don't think anyone would consider the program anything close to a proper AGI. Heck, nothing currently is 'close'. But even so, Zack's program uses "a Context-free grammar using the notation of Backus-Naur Form", which I think is a very algorithmic and formulaic way of generating poetry. It resulted in very fuzzy and unclear poems that seem incomprehensible. Out of an anthology of 26 poems that were submitted, only one was published.

With that said, the feat's quite impressive in itself, and so I decided to attempt to program something that generates AI, with the aim of creating poems that are less vague and more (I don't know) human? I wanted to use what I already knew, and new tools that I had heard about, in an eclectic approach to solve this task.

Hence, I planned to create poetry using a 4 step process:



	
  1. **Data Scraping. **Collect a corpus of themed poetry from poetryfoundation.org, the most prestigious poetry literary magazine.

	
  2. **Recurrent Neural Net. **Train the RNNs on this corpus, to generate letters that form poetic sentences.

	
  3. **Sense Checker. **Check these sentences for if they are grammatically correct and make sense.

	
  4. **Genetic Algorithms. **Use GAs to weed out the worst sentences and keep the good sentences. Combines these sentences in various ways in order to form poetry.


**Data Scrapping:**

How could I generate a successful poem if I didn't have anything to work with? Hence, I decided to use texts from [Poetry](http://www.poetryfoundation.org) magazine, "one of the leading monthly poetry journals in the English-speaking world".

One thing to note is that I only extracted poems that had the topic "Life Choices" and "Growing Old". Most of these poems lamented what could have been and regrets in life. Being a boy of 17 years of age, meant that I had a very limited experience of this topic and as such, I would have no clue about how to write such a poem. That was the reason behind that choice, in order to properly test the skills of my "AI".

As before, I used urllib and [BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/). However, I had a problem: the page would load into an Error:404. I thought the reason behind this was because of the user-agent, so I added that into the code and the problem fixed itself:

{% highlight python %}
import urllib2
from bs4 import BeautifulSoup

urlbase = 'http://www.poetryfoundation.org/browse/#subject=12&preview=1&poet-birthdate=20&page='

link = urlbase + '1'
opener = urllib2.build_opener()
opener.addheaders = [('User-agent', 'Mozilla/5.0')]
{% endhighlight %}

In doing so, I ran into another problem. It would just display "loading..." in the contents. I realised that the content (i.e, the poems) was loaded using javascript, so I had to install a new library called [Selenium](http://www.seleniumhq.org) that literally runs the browser, in order to scrape this data. The code basically goes through a number of indexes of poetry, extracts their links, opens them and saves the poem text in a  text file. Here is the rest of code, using Selenium:

_extractor.py:_

{% highlight python %}
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import codecs

urlbase = 'http://www.poetryfoundation.org/browse/#subject=12&preview=1&poet-birthdate=20&page='

poemlinks = []

# for pages 1-16, get links to poetry
counter = 1
for i in range(20,25):
    print "getting links... page " + str(counter)
    counter += 1
    driver = webdriver.Chrome()    
    driver.get(urlbase+str(i))
    for x in driver.find_elements_by_xpath("//div[@id='search-results']/p")[2:-1:2]:
        poemlinks.append(x.find_element_by_tag_name('a').get_attribute('href'))
    driver.close()
    print len(poemlinks)

corpus = ""
counter = 1
for p in poemlinks:
    print "downloading poems... poem " + str(counter) + "/" + str(len(poemlinks))
    counter += 1
    driver = webdriver.Chrome()
    driver.get(p)
    for x in driver.find_elements_by_xpath("//div[@id='poem']/div[@class='poem']/*"):
        corpus = corpus + '\n' + x.text
    driver.close()
    corpus += '\n'
    f = codecs.open('input.txt', 'w', 'utf-8')    
    f.write(corpus)
    f.close()
{% endhighlight %}

**Recurrent Neural Net**
I came across this post [The Unreasonable Effectiveness of Recurrent Neural Networks](http://karpathy.github.io/2015/05/21/rnn-effectiveness/) by Andrej Karpathy, a while ago and I was impressed. The whole concept of recurrent neural nets is interesting, because unlike a 'traditional' NN, the input isn't just manipulated in the hidden layers and exit through the output; instead, the input in RNNs can be looped back and used as input. This results in a rather convoluted system that can do amazing things.

In this case, a single letter is inputed into the net and according to its configuration, more letters are generated according to both what was inputted and what had already been outputted. This means that an RNN can keep in 'mind' what it had already generated and if the RNN is large enough, actually make sense. Apparently, Karpathy's RNNs can generate Shakespearian dialogue, Wikipedia entries, LATEX code and even Linux Source Code. Without getting into the details, I had a fairly good impression that this system could also generate poetry, a capability which I found surprising that it hasn't been done. (This was actually where the idea for this post came from!).

In order for me to train the network on my gathered poems, I had to install not only the [library](https://github.com/karpathy/char-rnn) but also [Lua](http://www.lua.org) and [Torch](http://torch.ch) as well as some other things. It took quite a bit of time and frustration! But in the end, it worked!

The text file containing the input poems (I don't exactly recall how many poems there were, but it numbered in the hundreds) had a size of 1.4 MB. This was large enough for the neural net to be used at its default setting.

```
th train.lua -data_dir data/some_folder -gpuid -1
```

Using that command in the terminal, I trained the RNN on the training set and it took almost an hour! I was waiting in anticipation and in the end, it worked, sorta!

The program trained a net that could generate pseudo-poetry. With a length of 50000 characters and a 'temperature' of 0.75, here is a sample of what I got:


```
Where the same from forest numbers
the conversation for my hands
In the bright same childhood sighs rages,

And he seems to do on a hillside rocking
a scate of sunlight
and sparrold, singing his straight and the human heart.

and all spore at the dark wood.
```

Here, I realised that out of the many sentences generated, only a couple made any sense. This is why there needed to be a next step, in order to delete of the sentences that couldn't be used.

**Sense Checker**

This was a fairly basic piece of code that made use of the [Language Check](https://pypi.python.org/pypi/language-check) library, parsed the sentences, and got rid of the sentences that had any spelling/grammar errors. I did, however, let one error go past: it wouldn't get rid of the sentence if it only had one error and that error was an uncapitalised starting.

{% highlight python %}
c = open('words.txt', 'r')
w = open('sentences.txt', 'w')
corpus = c.read()

import language_check
tool = language_check.LanguageTool('en-US')

corpus = corpus.split('\n')

counter = 0

senseful = []

for blah in [x.strip() for x in corpus if x != '']:
    try:
        matches = tool.check(blah.decode('utf-8'))
    except UnicodeDecodeError:
        pass
    
    if (len(matches) == 1 and matches[0].ruleId == 'UPPERCASE_SENTENCE_START') or \
    (len(matches) == 0):
        print counter
        senseful.append(blah+'\n')
        w.write(blah+'\n')
{% endhighlight %}

After running the sentences through this checker, I was left with a number of relatively good sentences that were unstructured.

**Genetic Algorithms**

In order to structure the computer-made sentences, I decided to use GAs. By assigning each sentence it's own fitness and then combining them into two-sentence-parts, it would then be possible to determine the best possible combinations, given enough time.

{% highlight python %}
import random

w = open('sentences.txt', 'r')
sentences = w.read().split('\n')

class DNA:
    """genotype for 2-lined parts of the poem"""
    def __init__(self):
        self.goodness = 3
        self.genes = []
        for _ in range(5):
            self.genes.append(random.randint(0, (len(sentences)-1)))

    def show(self):
        for s in self.genes:
            print sentences[s]

# 0 no sense at all
# 1 one good sentence
# 2 a few good sentences
# 3 all good sentences
# 4 makes sense
# 5 amazing

    def fitness(self):
        self.fitness = (self.goodness)**3+1

    def crossover(self, partner):
        child = DNA()
        midpoint = random.randint(0, len(self.genes))
        for i in range(len(self.genes)):
            if i >= midpoint:
                child.genes[i] = self.genes[i]
            else:
                child.genes[i] = partner.genes[i]
        return child

    def mutate(self):
        for i in range(len(self.genes)):
            if random.random() < 0.3:
                self.genes[i] = random.randint(0, (len(sentences)-1))

population_no = 10
population = []
for _ in range(population_no):
    population.append(DNA())

def cycle():
    for i in population:
        i.fitness()

    mating_pool = []
    for i in population:
        n = i.fitness
        for x in range(n):
            mating_pool.append(i)

    for i in range(len(population)):
        a = random.randint(0, len(mating_pool)-1)
        b = random.randint(0, len(mating_pool)-1)

        parent_a = mating_pool[a]
        parent_b = mating_pool[b]

        child = parent_a.crossover(parent_b)
        child.mutate()
        population[i] = child 

counter = 1
while True:
    print "generation: ", counter
    for p in population:
        p.show()
        rating = raw_input("rate: ")
        p.goodness = int(rating[0])

    cycle()
{% endhighlight %}

This process took AGES, but eventually I got it to work. After saving the good sentence pairs, I then worked on 4-sentence-parts and then more. Some potentially good pairs had to even be edited in order for them to make sense. This is part of the biggest flaw of my whole process: that the content of the poem is directly the result of my own selection of what makes the best poem. However, even if the GA part inherently brings in human opinion (me) into poetry selection, it is quite unavoidable (in my opinion). Unless a program can determine what is "poetic" and how a poem ought to be structured, I do not see any way of avoiding this. Unlike Zack, whose poems are almost randomly done, mine are at least in some ways, more "creative". This is because, they learn from the best poems and use what they learn to generate their own, new and innovative phrases. I think that's the biggest difference, even if my process has some flaws.

Anyway, after all that effort, the end result is interesting. I had the time to make it create 2 poems, shown below (I made up the title though and the white space.

**Publication:**

After the heartening results, I decided to submit the poems into a few literary journals. So far, Forgotten Remembrances, has been accepted by [Calliope Magazine](http://www.calliopemagazine.com). The editor, had however made a few edits (apparently my white spaces were unnecessary!).

[![Screen Region 2015-08-22 at 19.51.11](https://artificialcognition.files.wordpress.com/2015/08/screen-region-2015-08-22-at-19-51-11.png?w=300)](https://artificialcognition.files.wordpress.com/2015/08/screen-region-2015-08-22-at-19-51-11.png)

[![Screen Region 2015-08-22 at 19.52.24](https://artificialcognition.files.wordpress.com/2015/08/screen-region-2015-08-22-at-19-52-24.png?w=300)](https://artificialcognition.files.wordpress.com/2015/08/screen-region-2015-08-22-at-19-52-24.png)

I was eventually also sent a release form, which I signed and now the poem is waiting to be published!

**Edit**: Corvus Review has also decided to publish the other poem entitled Tick Tock. It can be seen in the Fall Issue of 2015; here's a screenshot:

[![Screen Region 2015-10-03 at 12.55.08](https://artificialcognition.files.wordpress.com/2015/08/screen-region-2015-10-03-at-12-55-08.png?w=231)](https://artificialcognition.files.wordpress.com/2015/08/screen-region-2015-10-03-at-12-55-08.png)

So, what does this mean? Well, since the acceptance rate of both of these magazines are both around 50%, I guess I can say that the poems created by this program are of average to above-average quality. You could also say that poem generation isn't also limited to humans, though creativity arguable.

Also, the initials of the pen name. **A** **I**? Get it? Yeah... just a daft inside-joke :P
