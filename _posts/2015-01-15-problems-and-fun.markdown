---
author: nhuhu
comments: false
date: 2015-01-15 12:22:10+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/01/16/problems-and-fun/
slug: problems-and-fun
title: 'Project: Smart Ships Part 3'
wordpress_id: 66
categories:
- Experiments
---

So today I was playing with the ships, because it was so cool. I tried to add some things, by myself, to it such as obstacles and a success scoreboard.

**Problem #1: Somehow, the ships managed to make it through the obstacle.**

[![They could go through walls.... O.o](https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-15-at-21-16-27.png?w=290)](https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-15-at-21-16-27.png)

The ships could go through walls...

For some inexplicable reason, so it seemed, some of the ships were passing through the barrier while others didn't. I tried extending the wall from x > 0 to x > -2000, just to make sure that none of the ships were slipping through that way. This problem still occurred, so I found out that it was the way my code was written, which allowed the location of the ship to change before it could stop the ship. And, when the ship has already passed the wall, it will carry on as if there wasn't one to begin with. Thus, instead of rewriting the code, I decided to just make the wall thicker, so that the ship wouldn't pass through in a single cycle.

**Pro****blem #2: The ships counter was acting weird.**

[![272 successes when there were only 100 ships? ](https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-15-at-21-26-56.png?w=289)](https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-15-at-21-26-56.png) 

272 successes when there were only 50 ships?

I turned off the wall, to save time. But the success counter just went up really weirdly. I realised that it was cos of a stupid mistake. I made the counter increment every time the frame refreshed and it didn't reset until a new cycle, therefore accounting for the massive digit. That was fixed easily.

**Problem #3: The ships were not getting on target. **

[(https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-15-at-22-06-54.png?w=289)](https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-15-at-22-06-54.png) 

They keep hitting the wall.

No matter how many cycles, for some populations, the ships seem to never progress beyond hitting the walls. For other populations, the ships eventually reach the target. I attribute this to the initial genes of the population and the low rate of mutation.

By increasing the population from 50 to 100, I allowed greater chances of success, and by altering the obstacle so that it was a bit lower and shorter allowed the ships to get through it easier. Also, by making the wall a bit lower allowed the ships that hit it, to have a lower fitness and therefore a smaller effect on the mating pool.

I also decreased the mutation rate to 0.005 in order to lower variation.

**Final Simulation (click to view the whole thing):**

[![shipsobstacle](https://artificialcognition.files.wordpress.com/2015/01/shipsobstacle.gif?w=286)](https://artificialcognition.files.wordpress.com/2015/01/shipsobstacle.gif)

As you can see, like before, the ships initially accelerated in random directions. Eventually, a lone ship who has somehow, either by mutation or some combination of genes, gone past the wall. In the next cycle, there are more ships that get closer and more, until eventually, most get to the target. The success counter is still a bit erroneous because it only takes into account the ships that reach the depot in the cycle and not those that would have reached it given just a bit more time. This is why the success rate is a bit low, even though most of the ships can be extrapolated to reach the target.

**Final Code:**

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
shipImg = pygame.image.load('ship.png')
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
        self.stop = False 

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

lifetime = 110 # how many frames there are, per cycle
lifecounter = 0
cyclecounter = 0
shipscounter = 0

# location of depot
depotx = 200
depoty = 0

population_no = 100 # number of ships
mutationrate = 0.005 # rate of mutation of genes

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
    pygame.draw.rect(DISPLAYSURF, (0,0,0), (0, 300, 250, 32))

    textSurfaceObj = fontObj.render("Cycle: " + str(cyclecounter) + "        Last Cycle Successes:" + str(shipscounter), True, (0,0,0))
    textRectObj = textSurfaceObj.get_rect()
    textRectObj.bottomright = (500, 500)

    DISPLAYSURF.blit(textSurfaceObj, textRectObj)

    if lifecounter < lifetime:
        for s in population:
            # if any ship is in rectangle, then it's on target and stops moving
            if ((s.location[0] > depotx-32 and s.location[0] < depotx+96-32) and 
            (s.location[1] > depoty and s.location[1] < depoty+80)):
                s.stop = True

            elif ((s.location[0] > -10000 and s.location[0] < 218) and 
            (s.location[1] > 300 and s.location[1] < 332)):
                s.stop = True
            
            #if not, then the ships keep accelerating per frame
            if s.stop != True:
                s.applyForce(s.dna.genes[lifecounter])
                s.update()
                DISPLAYSURF.blit(shipImg, s.location)
            else:
                DISPLAYSURF.blit(shipImg, s.location)
        lifecounter += 1

    else:
        lifecounter = 0
        cyclecounter += 1
        shipscounter = 0 
                
        for s in population:
            if ((s.location[0] > depotx-32 and s.location[0] < depotx+96-32) and 
            (s.location[1] > depoty and s.location[1] < depoty+80)):
                shipscounter += 1

        cycle()
    
    for event in pygame.event.get():
        if event.type == QUIT:
            pygame.quit()
            sys.exit()

    pygame.display.update()
    fpsClock.tick(FPS)

{% endhighlight %}


