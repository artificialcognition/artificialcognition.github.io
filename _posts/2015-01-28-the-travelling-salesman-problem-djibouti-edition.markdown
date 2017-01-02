---
author: nhuhu
comments: false
date: 2015-01-28 05:37:43+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/01/28/the-travelling-salesman-problem-djibouti-edition/
slug: the-travelling-salesman-problem-djibouti-edition
title: 'Project: The Travelling Salesman Problem Part 1'
wordpress_id: 87
categories:
- Experiments
---

I've heard about the [Travelling Salesman Problem](http://en.wikipedia.org/wiki/Travelling_salesman_problem) a while ago, but hadn't looked into it till now. I wanted to try solving it using genetic algorithms, to see if I can.

**Basically, here's the problem:**



	
  * You are given a list of cities and the distance between them.

	
  * The salesman has to visit each city, only once, and return to the original city.

	
  * Find the shortest distance required.


![](http://upload.wikimedia.org/wikipedia/commons/thumb/3/34/Flag_of_Djibouti.svg/250px-Flag_of_Djibouti.svg.png)

So, I looked for data sets online for the TSP and found this [site](http://www.math.uwaterloo.ca/tsp/world/countries.html). I chose Djibouti as it only had 38 cities, which I think is more than enough to try out with. With the data set provided, as well as the optimum solution, I went to work...

[![djpoints](https://artificialcognition.files.wordpress.com/2015/01/djpoints.gif?w=279)](https://artificialcognition.files.wordpress.com/2015/01/djpoints.gif)

Shown above, is the graphed version [here](http://www.math.uwaterloo.ca/tsp/world/djpoints.html). The data points are given [here](http://www.math.uwaterloo.ca/tsp/world/dj38.tsp).

Unfortunately, I found the data set hard to use in it's raw form, so I did this to convert it into a form that was usable:

I converted the string of numbers into a list of coordinates, which was then divided into node, x-coordinate, and y-coordinate. The node number was unnecessary, so it was removed. The strings were then converted into floats.

The floats were then manipulated so that they could be used as coordinates by pygame. By subtracting a number from every co-ord, the distance between them would remain unchanged.

{% highlight python %}
data = """1 11003.611100 42102.500000,
2 11108.611100 42373.888900,
3 11133.333300 42885.833300,
4 11155.833300 42712.500000,
5 11183.333300 42933.333300,
6 11297.500000 42853.333300,
7 11310.277800 42929.444400,
8 11416.666700 42983.333300,
9 11423.888900 43000.277800,
10 11438.333300 42057.222200,
11 11461.111100 43252.777800,
12 11485.555600 43187.222200,
13 11503.055600 42855.277800,
14 11511.388900 42106.388900,
15 11522.222200 42841.944400,
16 11569.444400 43136.666700,
17 11583.333300 43150.000000,
18 11595.000000 43148.055600,
19 11600.000000 43150.000000,
20 11690.555600 42686.666700,
21 11715.833300 41836.111100,
22 11751.111100 42814.444400,
23 11770.277800 42651.944400,
24 11785.277800 42884.444400,
25 11822.777800 42673.611100,
26 11846.944400 42660.555600,
27 11963.055600 43290.555600,
28 11973.055600 43026.111100,
29 12058.333300 42195.555600,
30 12149.444400 42477.500000,
31 12286.944400 43355.555600,
32 12300.000000 42433.333300,
33 12355.833300 43156.388900,
34 12363.333300 43189.166700,
35 12372.777800 42711.388900,
36 12386.666700 43334.722200,
37 12421.666700 42895.555600,
38 12645.000000 42973.333300
""".split(",\n")

for i in range(38):
    data[i] = data[i].split()
    del data[i][0]
    data[i][0] = float(data[i][0]) - 10800
    data[i][1] = float(data[i][1]) - 41500
{% endhighlight %}

I further manipulated the points by dividing all co-ordinates by 3, so that they could fit on the screen.

{% highlight python %}
    for d in data:
        pygame.draw.circle(DISPLAYSURF, (0,100,0), (int(d[0])/3,int(d[1])/3), 2)
{% endhighlight %}

Once I rendered the nodes in Pygame, however, they didn't match up with the one in the image. I realised that the difference was that their render used co-ordinates that started at the bottom left, and that it wouldn't make a difference, so I just kept it that way.

I created each route, in the population, with a list that randomly chose from the cities once, including the first city.

{% highlight python %}
    def __init__(self):
        choices = data[1:38]  # make a list of choices
        self.route = [data[0]]
        for _ in range(37): # pick a choice 37 times
            index = random.randint(0,len(choices)-1)
            self.route.append(choices[index])
            del choices[index]
        # self.route contains town 1 and 37 towns in rand order
        print self.route
{% endhighlight %}

The fitness function was then calculated using the distance formula (which I decided to implement this time, instead of importing numpy:

![distance-formula4](https://artificialcognition.files.wordpress.com/2015/01/distance-formula4.png)

Since self.route didn't contain the original city, at the end, it also had to be added.

{% highlight python %}
    def calcfitness(self):
        distance = 0
        for si in range(1, 37): # si = second index
            distance += ((self.route[si-1][0]-self.route[si][0])**2 +
            (self.route[si-1][1]-self.route[si][1])**2)**0.5
        distance += ((self.route[37][0]-self.route[0][0])**2 +
            (self.route[37][1]-self.route[0][1])**2)**0.5
{% endhighlight %}

The fitness function took a little tinkering. But by using the reciprocal of the distance, converting it to a number > 1 and then raising it to the power of 15, I made it so that the lower the distance, the greater the fitness.

{% highlight python %}
        distance = int(distance)
        print ((1.0/distance**2)*1000000000)**15
{% endhighlight %}

Now here was the really hard part. The customary cross-over function wouldn't've worked, since you could only visit each city once. If a midpoint was chosen and then split along it, the different cities would likely result in cities placed more than once in the genes.

Hence, I thought of creating another "used choices" list, which stores the choices already made. And, instead of picking a midpoint, each index would be randomly chosen from the two parents unless it was already on the list of used choices.

I thought this was all good and worked until I ran into a problem:

Parent 1: A B C
Parent 2: B C A
Possible child: A C (unable to continue)

Using this algorithm, it would've chosen randomly on the first index, between A and B. If it chose A, then A would also be placed on the used choices list. The next index would also be randomly chosen. But, because only A was on the list, it would still randomly select from both. Thus, if it chose C, then the next choice can't be made as both C and A are on the used choices.

What to do then? I thought of creating two lists instead, one of chosen choices and one of the previously unchosen choices instead. If one of the choices have already been unchosen, then that choice would be the unchosen one. Then if both choices have not been unchosen, then it would check with the chosen choices.

1st choice: A (randomly chosen)
List of unchosen: B

2nd choice: B (forced)
List of unchosen: B, C

3rd choice: C (forced)
List of unchosen: B, C, A

Maybe that was obvious, or maybe someone's already done it that way, but I'm still proud of it and proud that I figured it out.

And yet, it still didn't work, which I realised was a fault of this algorithm again and could not be solved without completely going back to the basics. So I remedied the few duplicates by setting up a deadlock list that would take in these pairs and then choose randomly assign the choices that weren't chosen, at the end. I realise that this isn't a true cross-over between the two parents, but since the duplicates are quite low in number, it shouldn't make that much of a difference.

{% highlight python %}
    def crossover(self, partner):
        child = DNA()
        unchosen = []
        deadlock = []
        for i in range(len(self.route)):
            choices = [self.route[i], partner.route[i]]
            
            if choices[0] in unchosen:
                child.route[i] = choices[0]
            elif choices[1] in unchosen:
                child.route[i] = choices[1]
            elif (choices[0] in child.route[0:i]) and (choices[1] in child.route[0:i]):
                child.route[i] = child.route[0]
                deadlock.append(i)
            elif choices[0] in child.route[0:i]:
                child.route[i] = choices[1]
            elif choices[1] in child.route[0:i]:
                child.route[i] = choices[0]
            else:
                child.route[i] = random.choice(choices)

            if child.route[i] == choices[0]:   # not choice gets appended to unchosen
                unchosen.append(choices[1])
            elif child.route[i] == choices[1]:
                unchosen.append(choices[0])
        
        notchosen = []
        for d in data:
            if d not in child.route:
                notchosen.append(d)

        for x in deadlock:
            index = random.randint(0, len(notchosen)-1)
            child.route[x] = notchosen[index]
            del notchosen[index]
{% endhighlight %}

Again, the mutation function wouldn't work if I just copied the generic mutation function. If I randomly mutated a city into another, there would be duplicates. This was easily solved, by simply swapping both cities. The mutation rate would have to be lowered, since each mutation would result in two changes.

{% highlight python %}
    def mutate(self):
        for i in range(len(self.route)):
            if random.random() < mutation_rate:
                index = random.randint(0, len(self.route)-1)
                placeholder = self.route[i]
                self.route[i] = self.route[index]
                self.route[index] = placeholder
{% endhighlight %}

Other than the above functions, the rest of the genetic algorithm was fairly simple to implement. One thing of note was that the lines were drawn, using a list comprehension:

{% highlight python %}
    bestpoints = [[int(x/3), int(y/3)] for [x,y] in max(population, key=maxfit).route]
    pygame.draw.lines(DISPLAYSURF, (0,100,0), True, bestpoints)
{% endhighlight %}

When I ran it the first time, I thought it worked and I was so happy. But I realised that as it would slow down, until it got to around the seventh generation and ground to a stop. It took me longer than I would admit to find out the reason. I tried optimising it, little things like moving stuff around, using xrange, but it kept not working. I finally realised that it was the fitness function at fault. The lower the distance, the greater the fitness value. In fact, it worked too well and the fitness value eventually went over the roof, which incredibly increased the length of the mating list, until it was too long. I simply adjusted the fitness function like so, and it worked:

{% highlight python %}
        self.fitness = ((1.0/distance**2)*1000000000)**2
{% endhighlight %}

The program still doesn't go to the optimum distance of 6600 but gets to around 7100, which is pretty close. Here's the full code, and I'll try to see if I can optimise it even further tomorrow.

{% highlight python %}
import pygame, sys, random
from pygame.locals import *

pygame.init()

FPS = 100
fpsClock = pygame.time.Clock()

DISPLAYSURF = pygame.display.set_mode((700, 700))
pygame.display.set_caption('Travelling Salesman Problem: Djibouti Edition')

WHITE = (255, 255, 255)

data = """1 11003.611100 42102.500000,
2 11108.611100 42373.888900,
3 11133.333300 42885.833300,
4 11155.833300 42712.500000,
5 11183.333300 42933.333300,
6 11297.500000 42853.333300,
7 11310.277800 42929.444400,
8 11416.666700 42983.333300,
9 11423.888900 43000.277800,
10 11438.333300 42057.222200,
11 11461.111100 43252.777800,
12 11485.555600 43187.222200,
13 11503.055600 42855.277800,
14 11511.388900 42106.388900,
15 11522.222200 42841.944400,
16 11569.444400 43136.666700,
17 11583.333300 43150.000000,
18 11595.000000 43148.055600,
19 11600.000000 43150.000000,
20 11690.555600 42686.666700,
21 11715.833300 41836.111100,
22 11751.111100 42814.444400,
23 11770.277800 42651.944400,
24 11785.277800 42884.444400,
25 11822.777800 42673.611100,
26 11846.944400 42660.555600,
27 11963.055600 43290.555600,
28 11973.055600 43026.111100,
29 12058.333300 42195.555600,
30 12149.444400 42477.500000,
31 12286.944400 43355.555600,
32 12300.000000 42433.333300,
33 12355.833300 43156.388900,
34 12363.333300 43189.166700,
35 12372.777800 42711.388900,
36 12386.666700 43334.722200,
37 12421.666700 42895.555600,
38 12645.000000 42973.333300
""".split(",\n")

for i in xrange(38):
    data[i] = data[i].split()
    del data[i][0]
    data[i][0] = float(data[i][0]) - 10800
    data[i][1] = float(data[i][1]) - 41500

mutation_rate = 0.03
population_no = 100

class DNA:
    """DNA of a route"""

    def __init__(self):
        choices = data[1:38]  # make a list of choices
        self.route = [data[0]]
        for _ in xrange(37): # pick a choice 37 times
            index = random.randint(0,len(choices)-1)
            self.route.append(choices[index])
            del choices[index]
        # self.route contains town 1 and 37 towns in rand order

    def calcfitness(self):
        distance = 0
        for si in xrange(1, 37): # si = second index
            distance += ((self.route[si-1][0]-self.route[si][0])**2 +
            (self.route[si-1][1]-self.route[si][1])**2)**0.5
        distance += ((self.route[37][0]-self.route[0][0])**2 +
            (self.route[37][1]-self.route[0][1])**2)**0.5
        self.distance = int(distance)
        self.fitness = ((1.0/distance**2)*1000000000)**2

    def crossover(self, partner):
        child = DNA()
        unchosen = []
        deadlock = []
        for i in xrange(len(self.route)):
            choices = [self.route[i], partner.route[i]]
            
            if choices[0] in unchosen:
                child.route[i] = choices[0]
            elif choices[1] in unchosen:
                child.route[i] = choices[1]
            elif (choices[0] in child.route[0:i]) and (choices[1] in child.route[0:i]):
                child.route[i] = child.route[0]
                deadlock.append(i)
            elif choices[0] in child.route[0:i]:
                child.route[i] = choices[1]
            elif choices[1] in child.route[0:i]:
                child.route[i] = choices[0]
            else:
                child.route[i] = random.choice(choices)

            if child.route[i] == choices[0]:   # not choice gets appended to unchosen
                unchosen.append(choices[1])
            elif child.route[i] == choices[1]:
                unchosen.append(choices[0])
        
        notchosen = []
        for d in data:
            if d not in child.route:
                notchosen.append(d)

        for x in deadlock:
            index = random.randint(0, len(notchosen)-1)
            child.route[x] = notchosen[index]
            del notchosen[index]

        return child
    
    def mutate(self):
        for i in xrange(len(self.route)):
            if random.random() < mutation_rate:
                index = random.randint(0, len(self.route)-1)
                placeholder = self.route[i]
                self.route[i] = self.route[index]
                self.route[index] = placeholder

population = []               # Creates the population filled with DNA
for _ in xrange(population_no):
    population.append(DNA())

def maxfit(thing):
    return thing.fitness

def main():
    for i in population:
        i.calcfitness()

    mating_pool = []
    for i in population:
        n = int(i.fitness)
        for x in xrange(n):
            mating_pool.append(i)

    for i in xrange(len(population)):
        a = random.randint(0, len(mating_pool)-1)
        b = random.randint(0, len(mating_pool)-1)

        parent_a = mating_pool[a]
        parent_b = mating_pool[b]
        
        child = parent_a.crossover(parent_b)

        child.mutate()
        population[i] = child
        population[i].calcfitness()

    
while True:
    DISPLAYSURF.fill(WHITE)

    main()

    for d in data:
        pygame.draw.circle(DISPLAYSURF, (0,100,0), (int(d[0])/3,int(d[1])/3), 2)

    print max(population, key=maxfit).distance
    bestpoints = [[int(x/3), int(y/3)] for [x,y] in max(population, key=maxfit).route]
    pygame.draw.lines(DISPLAYSURF, (0,100,0), True, bestpoints)

    for event in pygame.event.get():
        if event.type == QUIT:
            pygame.quit()
            sys.exit()

    pygame.display.update()
    fpsClock.tick(FPS)
{% endhighlight %}

Distance: ~7100:

[![Distance: 7100ish](https://artificialcognition.files.wordpress.com/2015/01/window-2015-01-30-at-22-34-53.png?w=292)](https://artificialcognition.files.wordpress.com/2015/01/window-2015-01-30-at-22-34-53.png) 

Optimum Distance: 6600:

[![Optimum Distance](https://artificialcognition.files.wordpress.com/2015/01/djtour.gif?w=300)](https://artificialcognition.files.wordpress.com/2015/01/djtour.gif) 
