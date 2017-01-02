---
author: nhuhu
comments: true
date: 2015-01-09 02:50:41+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/01/09/smart-ships/
slug: smart-ships
title: 'Project: Smart Ships Part 1'
wordpress_id: 54
categories:
- Experiments
---

Again, inspired by [this](http://natureofcode.com/book/chapter-9-the-evolution-of-code/), I decided to see if I could do it in Python. This time was a little bit harder, because the genotype and the phenotype were distinct from each other. I've been learning and playing around with Pygame and I thought I could give it a go. So far, this is what I've come up with:

{% highlight python %}

import pygame, sys, numpy, random
from pygame.locals import *

pygame.init()

FPS = 60 # frames per second setting
fpsClock = pygame.time.Clock()

# set up the window
DISPLAYSURF = pygame.display.set_mode((500, 500), 0, 32)
pygame.display.set_caption('Ship')

WHITE = (255, 255, 255)
shipImg = pygame.image.load('ship.gif')

depot = pygame.image.load('depot.gif')

class DNA:
    def __init__(self):
        maxforce = 0.1
        lifetime = 100
        self.genes = [numpy.array([0])]*lifetime
        for i in range(lifetime):
            self.genes[i] = numpy.array([random.random()*2-1, random.random()*2-1])
            self.genes[i] *= random.uniform(0, maxforce)

class Ship:
    """A ship"""
    def __init__(self):
        self.location = numpy.array([0,400])
        self.velocity = numpy.array([0,0])
        self.acceleration = numpy.array([2,-3])
        self.dna = DNA()

    def applyForce(self, f):
        self.acceleration += f

    def update(self):
        self.velocity += self.acceleration
        self.location += self.velocity
        self.acceleration *= 0

    def fitness():
        d = numpy.linalg.norm(self.loaction-target)
        self.fitness = (1/d)^2

ship1 = Ship()

while True: # the main game loop
    DISPLAYSURF.fill(WHITE)
    DISPLAYSURF.blit(depot, (200, 0))

    ship1.update()
    DISPLAYSURF.blit(shipImg, ship1.location)

    for event in pygame.event.get():
        if event.type == QUIT:
            pygame.quit()
            sys.exit()

    pygame.display.update()
    fpsClock.tick(FPS)
{% endhighlight %}


I tried to follow the code on that site to the best of my ability, but it got a bit confusing, so I tried to do it myself. I haven't really implemented any gAs yet, but I've got the ship and the motion down. I'll try to finish it in the next few days if I can, but it's been really confusing so far.

![Window 2015-01-09 at 15.47.51](https://artificialcognition.files.wordpress.com/2015/01/window-2015-01-09-at-15-47-51.png?w=290)

[![So far, it's moving linearly, in a diagonal direction, which is good. I used images i found off google for the ship and depot.](https://artificialcognition.files.wordpress.com/2015/01/window-2015-01-09-at-15-47-30.png?w=290)](https://artificialcognition.files.wordpress.com/2015/01/window-2015-01-09-at-15-47-30.png) 

So far, it's moving linearly, in a diagonal direction, which is good.
