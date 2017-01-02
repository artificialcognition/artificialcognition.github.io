---
author: nhuhu
comments: false
date: 2016-01-06 06:12:22+00:00
layout: post
link: https://artificialcognition.wordpress.com/2016/01/06/geometric-genetic-art-part-1/
slug: geometric-genetic-art-part-1
title: 'Project: Creating Visual Art with Genetic Algorithms Part 1'
wordpress_id: 269
categories:
- Experiments
---

After seeing [this collection of geometric designs](http://www.noupe.com/inspiration/showcases/gorgeous-geometric-designs.html), I remembered something that I was going to do. Genetic art, or rather evolving pieces of art.

The idea came to me when I was looking into what AI could be used to replicate. Something like creativity. I'm quite into geometric art, like the ones in the link above, so I wondered if I could replicate that using genetic algorithms.

I started out with a plan:
  * Initiate population (create drawings, that each contained 50 triangles of random vertices, colours, and transparencies).

	
  * Calculate the fitness of each drawing by testing how much it matched an original picture, pixel-per-pixel.

	
  * Mate the ones that are most fit.

	
  * Mutate the children.

	
  * Repeat.


This was the target drawing:

![bird.jpg](https://artificialcognition.files.wordpress.com/2016/01/bird.jpg)

It took me a whole day to test and debug, and one of the major problems I encountered was that the fitness function was inefficient. I looked online for a bit of info and adapted mine so that it would only change one triangle per mutation and to not use cross-overs.

During the day, I also actually learnt a lot of Python and things you can do with the Pygame library.

For example:

To create transparent triangles in Pygame, you actually need to create another surface and blitz that surface onto the main surface.

{% highlight python %}
    DISPLAYSURF.fill(WHITE)  

    for x in best.triangles:
        surface = pygame.Surface((totalx, totaly))
        surface.set_colorkey((0,0,0))
        surface.set_alpha(x.alpha)
        pygame.draw.polygon(surface, x.color, x.points)
        DISPLAYSURF.blit(surface, (0,0))

    pygame.display.update()
{% endhighlight %}

You can also take a screenshot of the Pygame surface like so:

{% highlight python %}
pygame.image.save(DISPLAYSURF, str(gen) + ".jpeg")
{% endhighlight %}

Anyway, the code takes forever to run. I wish I had more processing power or one of Google's supercomputers (drool.... :o ) I'll post an update showing the progress and what the result looks like. It'll take a while, and I may have to leave the computer on overnight.

Here's the final code:

{% highlight python %}
from PIL import Image
import sys, pygame
from pygame.locals import *
import random
import pygame.gfxdraw as gfx
import math

target = Image.open("bird.jpg")
totalx = target.size[0]
totaly = target.size[1]

pygame.init()
DISPLAYSURF = pygame.display.set_mode((totalx,totaly))
pygame.display.set_caption('Art')

trigno = 50
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

class Triangle:
    """Triangles that make up each Drawing"""
    def __init__(self):
        self.color = (random.randint(0,255),
        random.randint(0,255),
        random.randint(0,255))
        self.alpha = random.randint(0,255)
        self.points = [
        (random.randint(0,totalx),random.randint(0,totaly)),
        (random.randint(0,totalx),random.randint(0,totaly)),
        (random.randint(0,totalx),random.randint(0,totaly))]        

class Drawing:
    """Pieces of art"""

    def __init__(self):
        self.triangles = []
        for _ in xrange(trigno):
            t = Triangle()
            self.triangles.append(t)

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
        for i in xrange(trigno):
            child.triangles[i].color = random.choice([self.triangles[i].color, partner.triangles[i].color])
            child.triangles[i].points = random.choice([self.triangles[i].points, partner.triangles[i].points])

        return child

    def mutate(self):
        pick_index = random.randint(0, trigno-1)

        if random.random() < mutation_rate:
            index = random.randint(0, 3)
            change = random.randint(0, 255)
            if index == 3:
                self.triangles[pick_index].alpha = change
            else:
                colors = list(self.triangles[pick_index].color)
                colors[index] = change
                self.triangles[pick_index].color = tuple(colors)

        else:
            index = random.randint(0, 2)
            p1 = self.triangles[pick_index].points[0]
            p2 = self.triangles[pick_index].points[1]
            p3 = self.triangles[pick_index].points[2]
            changex = random.randint(0, totalx)
            changey = random.randint(0, totaly)
            change = [p1, p2, p3]
            change[index] = (changex, changey)
            self.triangles[pick_index].points = change

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

        for x in d.triangles:
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

    for x in best.triangles:
        surface = pygame.Surface((totalx, totaly))
        surface.set_colorkey((0,0,0))
        surface.set_alpha(x.alpha)
        pygame.draw.polygon(surface, x.color, x.points)
        DISPLAYSURF.blit(surface, (0,0))

    pygame.display.update()

    # creating mating pool... 

    st = best.triangles

    c1 = Drawing()
    for i in xrange(trigno):
        c1.triangles[i].color = st[i].color
        c1.triangles[i].alpha = st[i].alpha
        c1.triangles[i].points = st[i].points
    c1.mutate()

    c2 = Drawing()
    for i in xrange(trigno):
        c2.triangles[i].color = st[i].color
        c2.triangles[i].alpha = st[i].alpha
        c2.triangles[i].points = st[i].points
    c2.mutate()

    c3 = Drawing()
    for i in xrange(trigno):
        c3.triangles[i].color = st[i].color
        c3.triangles[i].alpha = st[i].alpha
        c3.triangles[i].points = st[i].points
    c3.mutate()

    c4 = Drawing()
    for i in xrange(trigno):
        c4.triangles[i].color = st[i].color
        c4.triangles[i].alpha = st[i].alpha
        c4.triangles[i].points = st[i].points
    c4.mutate()

    population = []
    population.append(best)
    population.append(c1)
    population.append(c2)
    population.append(c3)
    population.append(c4)

    if gen % 5 == 0:
        pygame.image.save(DISPLAYSURF, str(gen) + ".jpeg")

    print record

    gen += 1

{% endhighlight %}







