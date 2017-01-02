---
author: nhuhu
comments: false
date: 2015-04-15 08:50:40+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/04/15/creatures-part-5/
slug: creatures-part-5
title: 'Project: Autonomous Creatures Part 5'
wordpress_id: 132
---

Ok, so I've added a starvation code to it, so that the creatures now starve if the the absolute value of the average of their last five directional moves is less than 0.2. This ensures that they are removed from the population. I still haven't implemented the genetic algorithm yet, but I've added random enemies that generate at the start of the program. Even without the GAs, I'm often left with one or two that actually avoid the enemies properly (they turn before they hit them!!). I'm thinking that by randomising the enemies every say 5 turns, I'll get even better results.

I then decided to implement the genetic algorithm. I decided that the fitness of each creature would be the amount of time it lasts. I also looked into how the weights are distributed for the neural nets and found [this](http://stats.stackexchange.com/questions/7758/on-connection-weights-in-an-artificial-neural-network). It says that the mean is 0 and the standard distribution is around 0.3. I decided to check it using a simple program:

{% highlight python %}
from pybrain.tools.shortcuts import buildNetwork

maxno = 1
pop = []
nets = 10000
no = nets * 59

for i in range(nets):
    net = buildNetwork(8, 3, 8)
    for i in net.params:
        pop.append(i)

sumx = 0
sumx2 = 0

for x in pop:
    sumx += x
    sumx2 += x**2

print "mean: ", sumx/no
print "std deviation: ", (sumx2/no)**(0.5)
{% endhighlight %}

I can say with pretty good confidence that the mean is indeed 0, but the standard deviation is 1.

Then I implemented the complete genetic algorithm, tweaked a few settings and here's that:

{% highlight python %}
import pygame, sys
from pygame.locals import *
from pybrain.tools.shortcuts import buildNetwork
import numpy
import random

pygame.init()

FPS = 100
fpsClock = pygame.time.Clock()

DISPLAYSURF = pygame.display.set_mode((600, 600))
pygame.display.set_caption('Creature')

WHITE = (255, 255, 255)

directions = [(0,-1),(0.7,-0.7),(1,0),(0.7,0.7),(0,1),(-0.7,0.7),(-1,0),(-0.7,-0.7)]
population_no = 100
mutation_rate = 0.1

class Creature:
    &amp;amp;quot;&amp;amp;quot;&amp;amp;quot;creatures that move around&amp;amp;quot;&amp;amp;quot;&amp;amp;quot;

    def __init__(self):
        self.x = 300
        self.y = 300
        self.net = buildNetwork(8, 3, 8) # bulid a neural net for the crit
        index = numpy.argmax(self.net.activate([0,0,0,0,0,0,0,0]))
        self.direction = directions[index] # use largest output for direction
        self.dead = False
        self.lastmoves = []

    def calcsection(self, cx, cy, ex, ey):
        # radians from north
        a = numpy.arccos(numpy.dot(numpy.array([0,1]), numpy.array([ex-cx, cy-ey])) 
        / (numpy.linalg.norm(numpy.array([0,1]) * numpy.linalg.norm(numpy.array([ex-cx,cy-ey])))))

        if cx &amp;amp;gt;= ex:
            angle = 360 - a*180/numpy.pi
        else:
            angle = a*180/numpy.pi
        
        # sections of input
        sections = [(22.5, 67.5), (67.5, 112.5), (112.5, 157.5), (157.5, 202.5), 
        (202.5, 247.5), (247.5, 292.5), (292.5, 337.5)]
    
        # finds out which section the enemy is in, based on the angle
        if angle &amp;amp;gt; 337.5 or angle &amp;amp;lt; 22.5:
            return 0
        else:
            for i in range(len(sections)):
                (a, b) = sections[i]
                if angle &amp;amp;gt; a and angle &amp;amp;lt; b:
                    return i + 1 

    def calcdirection(self):
        netin = [0,0,0,0,0,0,0,0]

        for e in enemies:
            (enemx, enemy) = e
            inindex = self.calcsection(self.x, self.y, enemx, enemy)
            distance = numpy.sqrt((self.x-enemx)**2+(self.y-enemy)**2)
            netin[inindex] += (1/distance) * 200
        return netin

    def update(self):
        netin = self.calcdirection()
        index = numpy.argmax(self.net.activate(netin))
        self.direction = directions[index]

        # wraps the creature around the map
        if self.x &amp;amp;lt; 0:
            self.x = 600
        if self.x &amp;amp;gt; 600:
            self.x = 0
        if self.y &amp;amp;lt; 0:
            self.y = 600
        if self.y &amp;amp;gt; 600:
            self.y = 0

        #checks for contact with enemy
        for e in enemies:
            (enemx, enemy) = e
            if (enemx-10 &amp;amp;lt; self.x+10 and enemx+10 &amp;amp;gt; self.x-10) and (
            enemy-10 &amp;amp;lt; self.y+10 and enemy+10 &amp;amp;gt; self.y-10):
                self.dead = True
        
        self.lastmoves.append(self.direction)

        #checks for starvation
        if len(self.lastmoves) &amp;amp;gt; 5:
            sumx = 0
            sumy = 0
            for l in self.lastmoves[-5:]:
                (x, y) = l
                sumx += x
                sumy += y
            if (abs(sumx/5) &amp;amp;lt; 0.2) and (abs(sumy/5) &amp;amp;lt; 0.2):
                self.dead = True
            
        (x, y) = self.direction # moves the creature every frame
        self.x += x*5
        self.y += y*5
        
    def calcfitness(self):
        self.fitness = len(self.lastmoves)

    def crossover(self, partner):
        child = Creature()
        for i in range(len(self.net.params)):
            child.net.params[i] = random.choice([self.net.params[i], 
            partner.net.params[i]])
        return child

    def mutate(self):
        for i in range(len(self.net.params)):
            if random.random() &amp;amp;lt; mutation_rate:
                self.net.params[i] = numpy.random.normal(0,1)

population = []
for _ in range(population_no):
    population.append(Creature())

genlen = 500
count = 0
gen = 0
survivors = 0

while True:
    #sets up scoreboard
    fontObj = pygame.font.Font('freesansbold.ttf', 15)
    textSurfaceObj = fontObj.render(&amp;amp;quot;Gen: &amp;amp;quot; + str(gen) + 
    &amp;amp;quot;        Last Gen Survivors:&amp;amp;quot; + str(survivors), True, (0,0,0))
    textRectObj = textSurfaceObj.get_rect()
    textRectObj.bottomright = (600, 600)

    print gen, &amp;amp;quot;:&amp;amp;quot;, survivors

    survivors = 0

    #sets up however many random enemies
    enemies = []
    for x in range(20):
        enemies.append((random.randint(0,600),random.randint(0,600)))

    while count &amp;amp;lt; genlen:

        DISPLAYSURF.fill(WHITE)
        DISPLAYSURF.blit(textSurfaceObj, textRectObj) # shows scoreboard
    
        # if alive, population is drawn up
        for c in population:
            if c.dead == False:
                c.ob = Rect(c.x, c.y, 20, 20)
                c.ob.center = (c.x, c.y)
                pygame.draw.rect(DISPLAYSURF, (0,0,0), c.ob)
        
        # enemies are drawn
        for e in enemies:
            pygame.draw.circle(DISPLAYSURF, (200,0,0), e, 10)
    
        # if alive, updates the population
        for c in population:
            if c.dead == False:
                c.update()
    
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
        
        pygame.display.update()
        fpsClock.tick(FPS)

        count += 1
        
    for c in population:
        c.calcfitness()
        
        if c.dead == False:
            survivors += 1
    
    mating_pool = []
    for i in population:
        n = int(i.fitness)
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

    count = 0
    gen += 1
{% endhighlight %}

I noticed something interesting. Some of the creatures were moving along the centre, up/down or left/right. I realised that my code set the enemies so that they would avoid the centres (because I didn't want them to spawn on an enemy). I didn't realise that the whole column or row would be free. So I fixed that, but it still didn't work; I needed to optimise the net.

I realised that the fault lied in that my creatures moved too crudely, that the net wasn't working well enough. Instead of avoiding enemies, I decided to convert them into food so that it would have a better fitness function and I thought that the net would work better.

{% highlight python %}
import pygame, sys
from pygame.locals import *
from pybrain.tools.shortcuts import buildNetwork
import numpy
import random

pygame.init()

FPS = 100
fpsClock = pygame.time.Clock()

DISPLAYSURF = pygame.display.set_mode((600, 600))
pygame.display.set_caption('Creature')

WHITE = (255, 255, 255)

directions = [(0,-1),(0.7,-0.7),(1,0),(0.7,0.7),(0,1),(-0.7,0.7),(-1,0),(-0.7,-0.7)]
population_no = 10
mutation_rate = 0.05

class Creature:
    &quot;&quot;&quot;creatures that move around&quot;&quot;&quot;

    def __init__(self):
        self.sectionno = 60
        self.x = 300
        self.y = 300
        self.net = buildNetwork(self.sectionno, 16, 20, 20, 8) # build a neural net for the crit
        index = numpy.argmax(self.net.activate([0]*self.sectionno))
        self.direction = directions[index] # use largest output for direction
        self.lastmoves = []
        self.dead = False
        self.score = 0

    def calcsection(self, cx, cy, ex, ey):
        # radians from north
        a = numpy.arccos(numpy.dot(numpy.array([0,1]), numpy.array([ex-cx, cy-ey])) 
        / (numpy.linalg.norm(numpy.array([0,1]) * numpy.linalg.norm(numpy.array([ex-cx,cy-ey])))))

        if cx &gt;= ex:
            angle = 360 - a*180/numpy.pi
        else:
            angle = a*180/numpy.pi
        
        #sections of input
        sections = []
        mini = 0
        maxi = 360/self.sectionno

        for _ in range(self.sectionno):
            sections.append((mini, maxi))
            mini = maxi
            maxi += 360/self.sectionno
    
        for i in range(len(sections)):
            (a, b) = sections[i]
            if angle &gt;= a and angle &lt;= b:
                return i

    def calcdirection(self):
        netin = [0] * self.sectionno

        for f in food:
            (foodx, foody) = f
            inindex = self.calcsection(self.x, self.y, foodx, foody)
            distance = numpy.sqrt((self.x-foodx)**2+(self.y-foody)**2)
            netin[inindex] += ((1/distance) * 300)**(1.5)
        return netin

    def update(self):
        netin = self.calcdirection()
        index = numpy.argmax(self.net.activate(netin))
        self.direction = directions[index]

        # wraps the creature around the map
        if self.x &lt; 0:
            self.x = 600
        if self.x &gt; 600:
            self.x = 0
        if self.y &lt; 0:
            self.y = 600
        if self.y &gt; 600:
            self.y = 0

        #checks for contact with food
        for f in food:
            (foodx, foody) = f
            if (foodx-10 &lt; self.x+10 and foodx+10 &gt; self.x-10) and (
            foody-10 &lt; self.y+10 and foody+10 &gt; self.y-10):
                self.score += 1
                food.remove(f)
        
        self.lastmoves.append(self.direction)

        #checks for starvation
        if len(self.lastmoves) &gt; 20:
            sumx = 0
            sumy = 0
            for l in self.lastmoves[-20:]:
                (x, y) = l
                sumx += x
                sumy += y
            if (abs(sumx/20) &lt; 0) and (abs(sumy/20) &lt; 0):
                self.dead = True
            
        (x, y) = self.direction # moves the creature every frame
        self.x += x*10
        self.y += y*10
        
    def calcfitness(self):
        self.fitness = self.score**2

    def crossover(self, partner):
        child = Creature()
        for i in range(len(self.net.params)):
            child.net.params[i] = random.choice([self.net.params[i], 
            partner.net.params[i]])
        return child

    def mutate(self):
        for i in range(len(self.net.params)):
            if random.random() &lt; mutation_rate:
                self.net.params[i] = numpy.random.normal(0,1)

population = []
for _ in range(population_no):
    population.append(Creature())

genlen = 200
count = 0
gen = 0

while True:
    #sets up scoreboard
    fontObj = pygame.font.Font('freesansbold.ttf', 15)
    textSurfaceObj = fontObj.render(&quot;Gen: &quot; + str(gen), True, (0,0,0))
    textRectObj = textSurfaceObj.get_rect()
    textRectObj.bottomright = (600, 600)

    #sets up however many random foodies
    food = []
    for f in range(50):
        x = random.randint(0,600)
        y = random.randint(0,600)
        if not (x &gt;= 280 and x &lt;= 320 and y &gt;= 280 and y &lt;= 320):
            food.append((x, y))

    while count &lt; genlen:

        DISPLAYSURF.fill(WHITE)
        DISPLAYSURF.blit(textSurfaceObj, textRectObj) # shows scoreboard
    
        # if alive, population is drawn up
        for c in population:
            if c.dead == False:
                c.ob = Rect(c.x, c.y, 20, 20)
                c.ob.center = (c.x, c.y)
                pygame.draw.rect(DISPLAYSURF, (0,0,0), c.ob)
        
        # food is drawn
        for f in food:
            pygame.draw.circle(DISPLAYSURF, (200,0,0), f, 10)
    
        # if alive, updates the population
        for c in population:
            if c.dead == False:
                c.update()
    
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
        
        pygame.display.update()
        fpsClock.tick(FPS)

        count += 1
    
    foodcount = []
    for c in population:
        c.calcfitness()
        foodcount.append(c.score)

    print max(foodcount)
            
    mating_pool = []
    for i in population:
        n = int(i.fitness)
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

    count = 0
    gen += 1
{% endhighlight %}

And it worked. The creatures moved to get the food. I realised that the starvation code was a little bit redundant. But, once some other creature got the morsel of food, the creature got "unstuck". 

What I found really interesting was that the first maximum score was relatively high. But then, as more fit creatures began to emerge, the score decreased. I guess it makes sense, because as the resources decreased because the adept creatures increased, the individual score would also decrease. So that was cool. :) 

I guess this is the final part of this project. It was fun.

[![Window 2015-04-15 at 20.50.07](https://artificialcognition.files.wordpress.com/2015/04/window-2015-04-15-at-20-50-07.png?w=291)](https://artificialcognition.files.wordpress.com/2015/04/window-2015-04-15-at-20-50-07.png)
