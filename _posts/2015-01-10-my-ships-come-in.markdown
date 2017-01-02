---
author: nhuhu
comments: true
date: 2015-01-10 02:04:02+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/01/10/my-ships-come-in/
slug: my-ships-come-in
title: 'Project: Smart Ships Part 2'
wordpress_id: 59
categories:
- Experiments
---

Success! I'm actually quite surprised that it worked, well not surprised that it worked but surprised that I got it to work with as few errors as I got. It was pretty confusing, but I did manage to logic it out with the help of the [book](http://natureofcode.com/book/chapter-9-the-evolution-of-code/) and my head. Some parts of it were ommited in the book and some parts didn't really make sense, so I had to do it with reference to the [previous genetic algorithm](https://artificialcognition.wordpress.com/2014/07/18/so-far/) that I made.

For example, I did the fitness calculation, the cross-over and mutation in the Ship class instead of creating another class. I also found that the fitness equation which the book gave (1/d)^2 was too small to work with, and had to replace it with (1/d) and alter the way the mating pool was generated. However, some of the code for the gA was remarkably similar to the previous one and could easily be translated, so that helped. I added a text through Pygame that counted the number of generations (or I called it a cycle).

Here is the code (that I'm quite proud of), tidied up with a smattering of comments:

{% highlight python %}
import pygame, sys, numpy, random
from pygame.locals import *

pygame.init()

FPS = 100 # controls how fast each frame goes
fpsClock = pygame.time.Clock()

# set up the window
DISPLAYSURF = pygame.display.set_mode((500, 500))
pygame.display.set_caption('Ship')

# basic setup
WHITE = (255, 255, 255)
shipImg = pygame.image.load('ship.gif')
depot = pygame.image.load('depot.gif')

fontObj = pygame.font.Font('freesansbold.ttf', 15)

class DNA:
    """genotype for the ships"""
    def __init__(self):
        self.genes = [numpy.array([0])]*lifetime # creates a vector for every "frame"
        for i in range(lifetime):
            self.genes[i] = numpy.array([random.randint(-1, 1),
            random.randint(-1,1)])

class Ship:
    """A ship"""
    def __init__(self):
        self.location = numpy.array([0,400]) # initial location at bottom left
        self.velocity = numpy.array([0,0]) # not moving
        self.acceleration = numpy.array([0,0]) # not accelerating
        self.dna = DNA() # each ship has a genotype
        self.ontarget = False 

    def applyForce(self, f):
        self.acceleration += f 

    def update(self):
        self.velocity += self.acceleration # so that velocity adds up
        self.location += self.velocity
        self.acceleration *= 0 # resets the acceleration

    def fitness(self):
        d = numpy.linalg.norm(self.location-numpy.array([247, 96]))
        self.fitness = (1/d) # the smaller the distance, the greater the fitness

    def crossover(self, partner): # creates a child from two parents
        child = Ship()
        midpoint = random.randint(0, len(self.dna.genes))
        for i in range(len(self.dna.genes)):
            if i >= midpoint:
                child.dna.genes[i] = self.dna.genes[i]
            else:
                child.dna.genes[i] = partner.dna.genes[i]
        return child

    def mutate(self): # mutates the genes randomly
        for i in range(len(self.dna.genes)):
            if random.random() < mutationrate:
                self.dna.genes[i] = numpy.array([random.randint(-1, 1),
                random.randint(-1,1)])

lifetime = 50 # how many frames there are, per cycle
lifecounter = 0
cyclecounter = 0

# location of depot
depotx = 200
depoty = 0

population_no = 50 # number of ships
mutationrate = 0.01 # rate of mutation of genes

population = []
for _ in range(population_no): # creates an array of Ships
    population.append(Ship())

def cycle():
    for i in population: # calculates fitness for all population
        i.fitness()

    mating_pool = []
    for i in population:
        n = int(i.fitness * 1000)
        for x in range(n): # adds Ships into mating pool based on fitness
            mating_pool.append(i)

    for i in range(len(population)):
        a = random.randint(0, len(mating_pool)-1)
        b = random.randint(0, len(mating_pool)-1)

        parent_a = mating_pool[a]
        parent_b = mating_pool[b]

        child = parent_a.crossover(parent_b) # creates child based on fitness

        child.mutate()
        population[i] = child # replaces the parent with the child

while True: # the main loop
    DISPLAYSURF.fill(WHITE)
    DISPLAYSURF.blit(depot, (depotx, depoty))
    pygame.draw.rect(DISPLAYSURF, (0,0,0), (depotx, depoty, 96, 96,),1)

    textSurfaceObj = fontObj.render("Cycle: " + str(cyclecounter), True, (0,0,0))
    textRectObj = textSurfaceObj.get_rect()
    textRectObj.bottomright = (500, 500)

    DISPLAYSURF.blit(textSurfaceObj, textRectObj)

    if lifecounter < lifetime:
        for s in population:
            # if any ship is in rectangle, then it's on target and stops moving
            if ((s.location[0] > depotx-32 and s.location[0] < depotx+96-32) and
            (s.location[1] > depoty and s.location[1] < depoty+80)):
                s.ontarget = True

            #if not, then the ships keep accelerating per frame
            if s.ontarget != True:
                s.applyForce(s.dna.genes[lifecounter])
                s.update()
                DISPLAYSURF.blit(shipImg, s.location)
            else:
                DISPLAYSURF.blit(shipImg, s.location)
        lifecounter += 1

    else:
        lifecounter = 0
        cyclecounter += 1
        cycle()

    for event in pygame.event.get():
        if event.type == QUIT:
            pygame.quit()
            sys.exit()

    pygame.display.update()
    fpsClock.tick(FPS)
{% endhighlight %}

Here are the images of the [ship.gif ](http://www.pixeljoint.com/files/icons/avion.gif)and the [depot.gif](http://www.pixeljoint.com/files/icons/dodonpachi_final_trans.gif).

I made a gif (in very low fps and resolution, sorry) of it simulating and in the beginning, the ships are accelerating in random directions with apparently no goal. However by cycle 3-4, the ships are mostly all moving towards the depot, with a few stragglers. By cycle 5-10, the ships are pretty homogenised with no noticeable difference in cycles 10-20, other than random mutations and most reach the depot.

[![ships](https://artificialcognition.files.wordpress.com/2015/01/ships.gif?w=286)](https://artificialcognition.files.wordpress.com/2015/01/ships.gif)
