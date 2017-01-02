---
title: 'Project: Using Genetic Algorithms to Create DNA Art'
---

I decided to do a follow-up on [one of my previous projects]({{ site.baseurl }}{% post_url 2016-01-12-geometric-genetic-art-part-2 %}). This time, I planned to apply genetic algorithms onto DNA to create art. It just seemed somehow fitting to me.

Here is the reference image:

![here]({{ site.url }}/assets/reference.jpeg)

My first task was to adapt my previous code to this new image. I also changed the resolution of it so that the algorithms could progress faster without comprimising the final quality.

I changed the base shape from a pentagon to a circle, so that rather than having the colours being filled by the shape, it would instead be circles connected by lines (much like a connect the dots game.)

I also restored the genetic algorithm to its full form so that it properly mutates. I had previously modified it because it didn't suit the problem then.

Here is the bulk of the code:
{% highlight python %}
from PIL import Image
import sys, pygame
from pygame.locals import *
import random
import math
import pickle

target = Image.open("dna.jpg")
totalx = target.size[0]
totaly = target.size[1]
pop_no = 5
circles_no = 1000
mutation_rate = 0.01

pygame.init()
DISPLAYSURF = pygame.display.set_mode((totalx,totaly))
pygame.display.set_caption('DNA')

WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

def maxfit(thing):
    return thing.fitness


def diff(first, second):
    a = abs(first[0]-second[0])
    b = abs(first[1]-second[1])
    c = abs(first[2]-second[2])
    return (a + b + c)


class Circle:
    def __init__(self):
        self.color = random.choice([BLACK, WHITE])
        self.pos = (random.randint(0,totalx), 
                random.randint(0,totaly))
        self.radius = 2

class Drawing:
    def __init__(self):
        self.circles = []
        for _ in xrange(circles_no):
            t = Circle()
            self.circles.append(t)

    def calcfitness(self):
        totaldiff = 0
        for x in xrange(target.size[0]):
            for y in xrange(target.size[1]):
                totaldiff += diff(target.getpixel((x,y)), 
                DISPLAYSURF.get_at((x,y))[:3])
        self.fitness = (20173671.0/totaldiff)**3

    def crossover(self, partner):
        child = Drawing()
        for i in xrange(circles_no):
            child.circles[i].color = random.choice([BLACK, WHITE])
            child.circles[i].pos = random.choice([self.circles[i].pos, 
                partner.circles[i].pos])

        return child
    
    def mutate(self):
        index = random.randint(0, circles_no-1)
            
        for x in self.circles:
            if random.random() < mutation_rate:
                x.color = random.choice([BLACK, WHITE])
                
        for x in self.circles:
            if random.random() < mutation_rate:
                x.pos = (random.randint(0,totalx), 
                random.randint(0,totaly))

population = []
for _ in xrange(pop_no):
    t = Drawing()
    population.append(t)
gen = 0

while True:
    print "Generation: " + str(gen)
     

    # calculate fitness
    for d in population:
        DISPLAYSURF.fill(BLACK)

        for x in d.circles:
            surface = pygame.Surface((totalx, totaly))
            surface.set_colorkey((0,0,0))
            pygame.draw.circle(surface, x.color, x.pos, x.radius)
            DISPLAYSURF.blit(surface, (0,0))
        
        d.calcfitness()
        print d.fitness
    
    best = max(population, key=maxfit)    
    
    DISPLAYSURF.fill(BLACK)
    for x in best.circles:
        surface = pygame.Surface((totalx, totaly))
        surface.set_colorkey((0,0,0))
        pygame.draw.circle(surface, x.color, x.pos, x.radius)
        DISPLAYSURF.blit(surface, (0,0))


    # mate
    st = best.circles
    
    c1 = Drawing()
    for i in xrange(circles_no):
        c1.circles[i].color = st[i].color
        c1.circles[i].radius = st[i].radius
        c1.circles[i].pos = st[i].pos
    c1.mutate()

    c2 = Drawing()
    for i in xrange(circles_no):
        c2.circles[i].color = st[i].color
        c2.circles[i].radius = st[i].radius
        c2.circles[i].pos = st[i].pos
    c2.mutate()

    c3 = Drawing()
    for i in xrange(circles_no):
        c3.circles[i].color = st[i].color
        c3.circles[i].radius = st[i].radius
        c3.circles[i].pos = st[i].pos
    c3.mutate()

    c4 = Drawing()
    for i in xrange(circles_no):
        c4.circles[i].color = st[i].color
        c4.circles[i].radius = st[i].radius
        c4.circles[i].pos = st[i].pos
    c4.mutate()
 
    population = []
    population.append(best)
    population.append(c1)   
    population.append(c2)
    population.append(c3)   
    population.append(c4)

   
    pygame.display.update()
    
    if gen % 5 == 0:
        pygame.image.save(DISPLAYSURF, str(gen) + ".jpeg")

    if gen % 1000 == 0:
        pickle.dump(best.circles, open( "save.p", "wb"))

    gen += 1
{% endhighlight %}

At first the output is a random distribution of tiny circles as shown here:

![here]({{ site.url }}/assets/0.jpeg)

But then, by the 16725th generation, the tiny circles closely resemble the reference image:

![here]({{ site.url }}/assets/16725.jpeg)

Using another modified and longer version of remake.py from the previous project, I can enlarge the quality of the image, and "connect the dots."

{% highlight python %}
from PIL import Image
import sys, pygame
from pygame.locals import *
import random
import math
import pickle
from collections import Counter

target = Image.open("dna.jpg")
totalx = 3000
totaly = 6000
pop_no = 5
circles_no = 1000
mutation_rate = 0.01

pygame.init()
DISPLAYSURF = pygame.display.set_mode((totalx,totaly))
pygame.display.set_caption('DNA')

WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

def maxfit(thing):
    return thing.fitness


def diff(first, second):
    a = abs(first[0]-second[0])
    b = abs(first[1]-second[1])
    c = abs(first[2]-second[2])
    return (a + b + c)


class Circle:
    def __init__(self):
        self.color = random.choice([BLACK, WHITE])
        self.pos = (random.randint(0,totalx), 
                random.randint(0,totaly))
        self.radius = 2

class Drawing:
    def __init__(self):
        self.circles = []
        for _ in xrange(circles_no):
            t = Circle()
            self.circles.append(t)

    def calcfitness(self):
        totaldiff = 0
        for x in xrange(target.size[0]):
            for y in xrange(target.size[1]):
                totaldiff += diff(target.getpixel((x,y)), 
                DISPLAYSURF.get_at((x,y))[:3])
        self.fitness = (20173671.0/totaldiff)**3

    def crossover(self, partner):
        child = Drawing()
        for i in xrange(circles_no):
            child.circles[i].color = random.choice([BLACK, WHITE])
            child.circles[i].pos = random.choice([self.circles[i].pos, 
                partner.circles[i].pos])

        return child
    
    def mutate(self):
        index = random.randint(0, circles_no-1)
            
        for x in self.circles:
            if random.random() < mutation_rate:
                x.color = random.choice([BLACK, WHITE])
                
        for x in self.circles:
            if random.random() < mutation_rate:
                x.pos = (random.randint(0,totalx), 
                random.randint(0,totaly))


best = pickle.load(open("save.p", "rb"))

DISPLAYSURF.fill(BLACK)  

t = []

RANGE = 2.5

for x in best:
    maxx = x.pos[0]+RANGE
    minx = x.pos[0]-RANGE
    maxy = x.pos[1]+RANGE
    miny = x.pos[1]-RANGE

    for y in best:
        if (y != x):
            if (y.pos[0] < maxx) and (y.pos[0] > minx):
                if (y.pos[1] < maxy) and (y.pos[1] > miny):
                    t.append(y)

best = list(set(t))
print best

def multiply(x):
    return 36*x

for x in best:
    surface = pygame.Surface((totalx, totaly))
    surface.set_colorkey((0,0,0))
    pygame.draw.circle(surface, x.color, tuple(map(multiply, x.pos)), x.radius*7)
    for y in best:
        if (abs(map(multiply, x.pos)[0] - map(multiply, y.pos)[0]) < 300) and (abs(map(multiply, x.pos)[1] - map(multiply, y.pos)[1]) < 300) and (y != x):
            pygame.draw.line(surface, WHITE, max( [tuple(map(multiply, x.pos)),tuple(map(multiply, y.pos))]), min( [tuple(map(multiply, y.pos)) , tuple(map(multiply, y.pos)) ]), 1)     
    DISPLAYSURF.blit(surface, (0,0))

pygame.display.update()
pygame.image.save(DISPLAYSURF, "highres.jpeg")

{% endhighlight %}

In the end, I get the following image. Personally, I think it looks really good :)

![here]({{ site.url }}/assets/submission2.jpg)

I also played around with the hues and saturation, which results in this:

![here]({{ site.url }}/assets/submission.jpg)


