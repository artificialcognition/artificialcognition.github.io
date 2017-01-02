---
author: nhuhu
comments: false
date: 2016-01-12 10:36:51+00:00
layout: post
link: https://artificialcognition.wordpress.com/2016/01/12/geometric-genetic-art-part-2/
slug: geometric-genetic-art-part-2
title: 'Project: Creating Visual Art with Genetic Algorithms Part 2' 
wordpress_id: 310
categories:
- Experiments
---

Okay, there've been some changes. First of all, the algorithm has been slightly altered so that changes in the number of mutations per generation is changeable. Secondly, the triangles have been changed into pentagons. This is because triangles do not possess the appropriate number of vertices for a good image to form. Thirdly, I actually changed the target image from a bird to a sunset, because the bird was too complex to draw with only 50 pentagons and limited processing power.

I only realised that after heaps of experimentation. Oh what I'd give for Google's supercomputers. My laptop takes ages to process the algorithm and I've had to leave it running overnight for many nights now. The first few tries, which even though they took so much time, ended in failure. I later realised that my fitness algorithm was not running correctly. So that took a few days. Then, after those failures, I began to see success! :D

I actually wrote a "recreate" program so that the triangles are saved every 1000 generations, and can then be retrieved so that they can be scaled up at a higher resolution. This speeds up the process because I can process it on a lower resolution and make it bigger at the end.

Anyway, here's **final** final code:

{% highlight python %}
from PIL import Image
import sys, pygame
from pygame.locals import *
import random
import math
import pickle

target = Image.open("sun.jpg")
totalx = target.size[0]
totaly = target.size[1]

pygame.init()
DISPLAYSURF = pygame.display.set_mode((totalx,totaly))
pygame.display.set_caption('Art')

pentno = 50
mutation_rate = 0.5
pop_no = 5

WHITE = (255, 255, 255, 255)
BLACK = (0, 0, 0)

DISPLAYSURF.fill(WHITE)

def diff(first, second):
    a = (first[0]-second[0])**2
    b = (first[1]-second[1])**2
    c = (first[2]-second[2])**2
    return a + b + c

class Pentagon:
    """pentagons that make up each Drawing"""
    def __init__(self):
        self.color = (random.randint(0,255),
        random.randint(0,255),
        random.randint(0,255))
        self.alpha = random.randint(0,255)
        self.points = [
        (random.randint(0,totalx),random.randint(0,totaly)),
        (random.randint(0,totalx),random.randint(0,totaly)),
        (random.randint(0,totalx),random.randint(0,totaly)),
        (random.randint(0,totalx),random.randint(0,totaly)),
        (random.randint(0,totalx),random.randint(0,totaly))]        

class Drawing:
    """pieces of art"""

    def __init__(self):
        self.pentagons = []
        for _ in xrange(pentno):
            t = Pentagon()
            self.pentagons.append(t)

    def calcfitness(self):
    # calculates fitness via how different each pixel's color is
        totaldiff = 0
        for x in xrange(target.size[0]):
            for y in xrange(target.size[1]):
                totaldiff += diff(target.getpixel((x,y)),
                DISPLAYSURF.get_at((x,y))[:3])
        self.fitness = totaldiff

    def crossover(self, partner):
        child = Drawing()
        for i in xrange(pentno):
            child.pentagons[i].color = random.choice([self.pentagons[i].color, partner.pentagons[i].color])
            child.pentagons[i].points = random.choice([self.pentagons[i].points, partner.pentagons[i].points])

        return child

    def mutate(self):
        for _ in range(1):
            pick_index = random.randint(0, pentno-1)

            if random.random() < mutation_rate:
                index = random.randint(0, 3)
                change = random.randint(0, 255)
                if index == 3:
                    self.pentagons[pick_index].alpha = change
                else:
                    colors = list(self.pentagons[pick_index].color)
                    colors[index] = change
                    self.pentagons[pick_index].color = tuple(colors)

            else:
                index = random.randint(0, 2)
                p1 = self.pentagons[pick_index].points[0]
                p2 = self.pentagons[pick_index].points[1]
                p3 = self.pentagons[pick_index].points[2]
                p4 = self.pentagons[pick_index].points[3]
                p5 = self.pentagons[pick_index].points[4]
                changex = random.randint(0, totalx)
                changey = random.randint(0, totaly)
                change = [p1, p2, p3, p4, p5]
                change[index] = (changex, changey)
                self.pentagons[pick_index].points = change

population = []
for _ in xrange(pop_no):
    population.append(Drawing())

def minfit(drawing):
    return drawing.fitness

gen = 0

record = []

while True:

    print "Generation: " + str(gen)

    for event in pygame.event.get():
        if event.type == QUIT:
            pygame.quit()
            sys.exit()

    # calculating fitness...

    count = 1

    for d in population:

        DISPLAYSURF.fill(WHITE)    

        for x in d.pentagons:
            surface = pygame.Surface((totalx, totaly))
            surface.set_colorkey((0,0,0))
            surface.set_alpha(x.alpha)
            pygame.draw.polygon(surface, x.color, x.points)
            DISPLAYSURF.blit(surface, (0,0))

        d.calcfitness()
        print str(count) + ": " + str(d.fitness)
        count += 1

    # choosing best...

    best = min(population, key=minfit)
    record.append(best.fitness)

    # drawing...    

    DISPLAYSURF.fill(WHITE)  

    for x in best.pentagons:
        surface = pygame.Surface((totalx, totaly))
        surface.set_colorkey((0,0,0))
        surface.set_alpha(x.alpha)
        pygame.draw.polygon(surface, x.color, x.points)
        DISPLAYSURF.blit(surface, (0,0))

    pygame.display.update()

    # creating mating pool... 

    st = best.pentagons

    c1 = Drawing()
    for i in xrange(pentno):
        c1.pentagons[i].color = st[i].color
        c1.pentagons[i].alpha = st[i].alpha
        c1.pentagons[i].points = st[i].points
    c1.mutate()

    c2 = Drawing()
    for i in xrange(pentno):
        c2.pentagons[i].color = st[i].color
        c2.pentagons[i].alpha = st[i].alpha
        c2.pentagons[i].points = st[i].points
    c2.mutate()

    c3 = Drawing()
    for i in xrange(pentno):
        c3.pentagons[i].color = st[i].color
        c3.pentagons[i].alpha = st[i].alpha
        c3.pentagons[i].points = st[i].points
    c3.mutate()

    c4 = Drawing()
    for i in xrange(pentno):
        c4.pentagons[i].color = st[i].color
        c4.pentagons[i].alpha = st[i].alpha
        c4.pentagons[i].points = st[i].points
    c4.mutate()

    population = []
    population.append(best)
    population.append(c1)
    population.append(c2)
    population.append(c3)
    population.append(c4)

    if gen % 5 == 0:
        pygame.image.save(DISPLAYSURF, str(gen) + ".jpeg")

    if gen % 1000 == 0:
        pickle.dump(best.pentagons, open( "save.p", "wb"))

    print record

    gen += 1
{% endhighlight %}

And, here's rewrite.py:

{% highlight python %}
from PIL import Image
import sys, pygame
from pygame.locals import *
import random
import math
import pickle

target = Image.open("sun.jpg")
totalx = target.size[0]
totaly = target.size[1]

pygame.init()
DISPLAYSURF = pygame.display.set_mode((totalx,totaly))
pygame.display.set_caption('Art')

pentno = 50
mutation_rate = 0.5
pop_no = 5

WHITE = (255, 255, 255, 255)
BLACK = (0, 0, 0)

DISPLAYSURF.fill(WHITE)

def diff(first, second):
    a = (first[0]-second[0])**2
    b = (first[1]-second[1])**2
    c = (first[2]-second[2])**2
    return a + b + c

class Pentagon:
    """pentagons that make up each Drawing"""
    def __init__(self):
        self.color = (random.randint(0,255),
        random.randint(0,255),
        random.randint(0,255))
        self.alpha = random.randint(0,255)
        self.points = [
        (random.randint(0,totalx),random.randint(0,totaly)),
        (random.randint(0,totalx),random.randint(0,totaly)),
        (random.randint(0,totalx),random.randint(0,totaly)),
        (random.randint(0,totalx),random.randint(0,totaly)),
        (random.randint(0,totalx),random.randint(0,totaly))]        

class Drawing:
    """pieces of art"""

    def __init__(self):
        self.pentagons = []
        for _ in xrange(pentno):
            t = Pentagon()
            self.pentagons.append(t)

    def calcfitness(self):
    # calculates fitness via how different each pixel's color is
        totaldiff = 0
        for x in xrange(target.size[0]):
            for y in xrange(target.size[1]):
                totaldiff += diff(target.getpixel((x,y)),
                DISPLAYSURF.get_at((x,y))[:3])
        self.fitness = totaldiff

    def crossover(self, partner):
        child = Drawing()
        for i in xrange(pentno):
            child.pentagons[i].color = random.choice([self.pentagons[i].color, partner.pentagons[i].color])
            child.pentagons[i].points = random.choice([self.pentagons[i].points, partner.pentagons[i].points])

        return child

    def mutate(self):
        for _ in range(1):
            pick_index = random.randint(0, pentno-1)

            if random.random() < mutation_rate:
                index = random.randint(0, 3)
                change = random.randint(0, 255)
                if index == 3:
                    self.pentagons[pick_index].alpha = change
                else:
                    colors = list(self.pentagons[pick_index].color)
                    colors[index] = change
                    self.pentagons[pick_index].color = tuple(colors)

            else:
                index = random.randint(0, 2)
                p1 = self.pentagons[pick_index].points[0]
                p2 = self.pentagons[pick_index].points[1]
                p3 = self.pentagons[pick_index].points[2]
                p4 = self.pentagons[pick_index].points[3]
                p5 = self.pentagons[pick_index].points[4]
                changex = random.randint(0, totalx)
                changey = random.randint(0, totaly)
                change = [p1, p2, p3, p4, p5]
                change[index] = (changex, changey)
                self.pentagons[pick_index].points = change

population = []
for _ in xrange(pop_no):
    population.append(Drawing())

def minfit(drawing):
    return drawing.fitness

gen = 0

record = []

while True:

    print "Generation: " + str(gen)

    for event in pygame.event.get():
        if event.type == QUIT:
            pygame.quit()
            sys.exit()

    # calculating fitness...

    count = 1

    for d in population:

        DISPLAYSURF.fill(WHITE)    

        for x in d.pentagons:
            surface = pygame.Surface((totalx, totaly))
            surface.set_colorkey((0,0,0))
            surface.set_alpha(x.alpha)
            pygame.draw.polygon(surface, x.color, x.points)
            DISPLAYSURF.blit(surface, (0,0))

        d.calcfitness()
        print str(count) + ": " + str(d.fitness)
        count += 1

    # choosing best...

    best = min(population, key=minfit)
    record.append(best.fitness)

    # drawing...    

    DISPLAYSURF.fill(WHITE)  

    for x in best.pentagons:
        surface = pygame.Surface((totalx, totaly))
        surface.set_colorkey((0,0,0))
        surface.set_alpha(x.alpha)
        pygame.draw.polygon(surface, x.color, x.points)
        DISPLAYSURF.blit(surface, (0,0))

    pygame.display.update()

    # creating mating pool... 

    st = best.pentagons

    c1 = Drawing()
    for i in xrange(pentno):
        c1.pentagons[i].color = st[i].color
        c1.pentagons[i].alpha = st[i].alpha
        c1.pentagons[i].points = st[i].points
    c1.mutate()

    c2 = Drawing()
    for i in xrange(pentno):
        c2.pentagons[i].color = st[i].color
        c2.pentagons[i].alpha = st[i].alpha
        c2.pentagons[i].points = st[i].points
    c2.mutate()

    c3 = Drawing()
    for i in xrange(pentno):
        c3.pentagons[i].color = st[i].color
        c3.pentagons[i].alpha = st[i].alpha
        c3.pentagons[i].points = st[i].points
    c3.mutate()

    c4 = Drawing()
    for i in xrange(pentno):
        c4.pentagons[i].color = st[i].color
        c4.pentagons[i].alpha = st[i].alpha
        c4.pentagons[i].points = st[i].points
    c4.mutate()

    population = []
    population.append(best)
    population.append(c1)
    population.append(c2)
    population.append(c3)
    population.append(c4)

    if gen % 5 == 0:
        pygame.image.save(DISPLAYSURF, str(gen) + ".jpeg")

    if gen % 1000 == 0:
        pickle.dump(best.pentagons, open( "save.p", "wb"))

    print record

    gen += 1
{% endhighlight %}

This is the result:
![Screen Region 2016-01-25 at 11.12.40.png](https://artificialcognition.files.wordpress.com/2016/01/screen-region-2016-01-25-at-11-12-40.png)

Here it is in action:

![animated](https://artificialcognition.files.wordpress.com/2016/01/animated.gif)

And.. using recreate.py, here is the final piece of art in a higher resolution, cropped:

![croppedsunset.jpeg](https://artificialcognition.files.wordpress.com/2016/01/croppedsunset.jpeg)

I think it turned out rather well, all things considered. It actually looks like a sunset. I wonder what would happen if I posted it on deviantart.com... :D
