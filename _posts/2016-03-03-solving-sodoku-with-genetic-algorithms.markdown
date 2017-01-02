---
author: nhuhu
comments: false
date: 2016-03-03 13:57:14+00:00
layout: post
link: https://artificialcognition.wordpress.com/2016/03/04/solving-sodoku-with-genetic-algorithms/
slug: solving-sodoku-with-genetic-algorithms
title: 'Project: Completing Sodoku with Genetic Algorithms'
wordpress_id: 418
categories:
- Experiments
---

After using GAs to create art, and evolve creatures, you'd think that Sodoku would be fairly trivial. It's not. In fact, I found it almost impossibly hard. I spent a week working on this problem, mainly because each iteration takes so long and my processing speed isn't very high. So I've had to leave it overnight for many a night.

I tried heaps of different fitness algorithms, crossover algorithms, and mutation algorithms. In the end, I had to reset the population after it stagnated (only if it has produced the same best fitness for 10 generations), knowing that it had reached its local minima (of which in Sodoku there are many).

{% highlight python %}
    # if it has reached a local minima, reset
    if bests == [currentbest] * 10:
        totalbests += currentbest
        population = []
        for _ in xrange(population_no):
            population.append(Solution())
        cycle += 1
        gen = 0
{% endhighlight %}

I also had to reduce the search space by a preliminary elimination of impossible digits by looking at each row/column:

{% highlight python %}
possibles = []
for x in range(9):
    row = []
    for y in range(9):
        row.append([1,2,3,4,5,6,7,8,9])
    possibles.append(row)

for y in range(9):
    for x in range(9):
        if processed[y][x] != 0:
            possibles[y][x] = [processed[y][x]]
        else:
            possibles[y][x] = [1,2,3,4,5,6,7,8,9]

for y in range(9):
    fixed = []
    for x in range(9):
        if len(possibles[y][x]) == 1:
            fixed.append(possibles[y][x][0])
    for x in range(9):
        if len(possibles[y][x]) != 1:
            for f in fixed:
                possibles[y][x].remove(f)

for x in range(9):
    fixed = []
    column = []
    for y in range(9):
        column.append(possibles[y][x])
    for c in column:
        if len(c) == 1:
            fixed.append(c[0])
    for y in range(9):
        if len(possibles[y][x]) != 1:
            for f in fixed:
                try:
                    possibles[y][x].remove(f)
                except ValueError:
                    continue
{% endhighlight %}

I also had to fiddle with the population size and the mutation rate. I decided between a mutation algorithm that randomly switched digits vs one that would swap digits. I also had to choose between a crossover algorithm being either one that would choose between every digit of parents at a 50-50 rate, or to choose a midpoint and split it.

![Screen Region 2016-02-24 at 22.40.52](https://artificialcognition.files.wordpress.com/2016/03/screen-region-2016-02-24-at-22-40-521.png?w=600)

In the end, I managed to find a solution to the above puzzle, that used a population size of 2000, and mutation rate of 1%, and 623 generations. That's 1246000 instances of the Sodoku board! It came up with the following solution:

```
Cycle: 1 Gen 623: 84 Average Gen/Cycle: 0
[84, 79, 77, 77, 79, 79, 79, 77, 76, 74]
1 8 9 2 4 3 5 7 6
2 4 7 6 5 8 3 1 9
3 5 6 9 7 1 2 4 8
9 3 4 8 1 7 6 2 5
5 6 2 3 9 4 1 8 7
7 1 8 5 2 6 9 3 4
4 9 1 7 3 5 8 6 2
6 7 5 1 8 2 4 9 3
8 2 3 4 6 9 7 5 1
```

Which matches up with the given solution! :D

![Screen Region 2016-02-24 at 22.40.57](https://artificialcognition.files.wordpress.com/2016/03/screen-region-2016-02-24-at-22-40-57.png?w=600)

Without further ado, here's the final code:

{% highlight python %}

import random

s = "180003070000008000306070240004000600002000007000000030401700060005082400000069000"

mutation_rate = 0.01
population_no = 2000

class Solution:
    """DNA of solution"""

    def __init__(self):
        self.missing = []
        flatpossibles = []
        for x in possibles:
            for y in x:
                flatpossibles.append(y)

        for x in range(81):
            if s[x] == "0":
                self.missing.append(random.choice(flatpossibles[x]))

    # fills in the missing blocks into an unprocessed whole
    def reconstruct(self):
        y = list(s)
        index = 0
        for x in xrange(81):
            if y[x] == '0':
                y[x] = str(self.missing[index])
                index += 1
        self.unprocessed = map(int, y)

    # transforms the unprocessed whole into a printable puzzle
    def process(self):
        # processes string into lists
        indx = [9, 18, 27, 36, 45, 54, 63, 72, 80]
        p = list(self.unprocessed)
        temp = []
        processed = []
        for x in xrange(81):
            if int(x) in indx:
                processed.append(temp)
                temp = []
                temp.append(int(p[x]))
                continue
            temp.append(int(p[x]))
        processed[8].append(int(p[-1]))
        self.processed = processed

    def printPuzzle(self):
        for p in self.processed:
            print " ".join(map(str,p))

    # fitness = sum of digits - the digits that are repeated
    # to ensure that the use of all digits is positively selected for
    # and the use of duplicates is negatively selected against
    def calcfitness(self):

        self.reconstruct()
        self.process()

        rowfit = 0
        colfit = 0
        squarefit = 0

        for row in self.processed:
            rowfit += len(set(row))

        for indx in xrange(9):
            column = []
            for r in self.processed:
                column.append(r[indx])
            colfit +=  len(set(column))

        for ys in [xrange(3),xrange(3,6),xrange(6,9)]:
            for xs in [xrange(3),xrange(3,6),xrange(6,9)]:
                square = []
                for y in ys:
                    for x in xs:
                        square.append(self.processed[y][x])
                squarefit += len(set(square))
        self.fitness = (rowfit + colfit + squarefit)**5/10000000000

    def crossover(self, partner):
        child = Solution()

        for i in xrange(len(self.missing)):
            child.missing[i] = random.choice([self.missing[i],
            partner.missing[i]])
        return child
        """

        midpoint = random.randint(0, len(self.missing))
        for i in xrange(len(self.missing)):
            if i &amp;gt;= midpoint:
                child.missing[i] = self.missing[i]
            else:
                child.missing[i] = partner.missing[i]
        return child
        """

    def processnofixed(self):
        # create a dictionary of the missing (index : missing-digit)
        unprocessednofixed = {}
        y = list(s)
        index = 0
        for x in xrange(81):
            if y[x] == '0':
                unprocessednofixed[x] = self.missing[index]
                index += 1

        # create a list of missing w/o the fixed
        missingnofixed = [0]*81
        for index in unprocessednofixed:
            missingnofixed[index] = unprocessednofixed[index]

        # processes string into lists
        indx = [9, 18, 27, 36, 45, 54, 63, 72, 80]
        p = list(missingnofixed)
        temp = []
        processed = []
        for x in xrange(81):
            if int(x) in indx:
                processed.append(temp)
                temp = []
                temp.append(int(p[x]))
                continue
            temp.append(int(p[x]))
        processed[8].append(int(p[-1]))

        self.processednofixed = processed

    def mutate(self):
        for row in self.processednofixed:
            if random.random() &amp;lt; mutation_rate:
                # swap only if both are missing numbers
                both = False
                while both == False:
                    a = random.randint(0,8)
                    b = random.randint(0,8)
                    if a != b:
                        c = row[a]
                        d = row[b]
                        # 0s are fixed numbers
                        if (c != 0) and (d != 0):   # IMPORTANT CHANGE IT
                            row[a] = d
                            row[b] = c
                            both = True

        for indx in xrange(9):
            column = []
            for r in self.processednofixed:
                column.append(r[indx])
            if random.random() < mutation_rate:
                both = False
                while both == False:
                    a = random.randint(0,8)
                    b = random.randint(0,8)
                    if a != b:
                        c = column[a]
                        d = column[b]
                        if (c != 0) and (d != 0):
                            column[a] = d
                            column[b] = c
                            both = True

                for r in self.processednofixed:
                    r[indx] = column[0]
                    del column[0]

        for ys in [xrange(3),xrange(3,6),xrange(6,9)]:
            for xs in [xrange(3),xrange(3,6),xrange(6,9)]:
                square = []
                for y in ys:
                    for x in xs:
                        square.append(self.processednofixed[y][x])
                if random.random() < mutation_rate:
                    both = False
                    while both == False:
                        a = random.randint(0,8)
                        b = random.randint(0,8)
                        if a != b:
                            c = square[a]
                            d = square[b]
                            if (c != 0) and (d != 0):
                                square[a] = d
                                square[b] = c
                                both = True

                    # put square back into processsednofixed
                    for y in ys:
                        for x in xs:
                            self.processednofixed[y][x] = square[0]
                            del square[0]

        fill = []
        for x in self.processednofixed:
            for y in x:
                if y != 0:
                    fill.append(y)

        self.missing = fill

indx = [9, 18, 27, 36, 45, 54, 63, 72, 80]
p = list(s)
temp = []
processed = []
for x in xrange(81):
    if int(x) in indx:
        processed.append(temp)
        temp = []
        temp.append(int(p[x]))
        continue
    temp.append(int(p[x]))
processed[8].append(int(p[-1]))

possibles = []
for x in range(9):
    row = []
    for y in range(9):
        row.append([1,2,3,4,5,6,7,8,9])
    possibles.append(row)

for y in range(9):
    for x in range(9):
        if processed[y][x] != 0:
            possibles[y][x] = [processed[y][x]]
        else:
            possibles[y][x] = [1,2,3,4,5,6,7,8,9]

for y in range(9):
    fixed = []
    for x in range(9):
        if len(possibles[y][x]) == 1:
            fixed.append(possibles[y][x][0])
    for x in range(9):
        if len(possibles[y][x]) != 1:
            for f in fixed:
                possibles[y][x].remove(f)

for x in range(9):
    fixed = []
    column = []
    for y in range(9):
        column.append(possibles[y][x])
    for c in column:
        if len(c) == 1:
            fixed.append(c[0])
    for y in range(9):
        if len(possibles[y][x]) != 1:
            for f in fixed:
                try:
                    possibles[y][x].remove(f)
                except ValueError:
                    continue

print possibles

population = []
for _ in xrange(population_no):
    population.append(Solution())

gen = 1
cycle = 1
bests = [0]*10

def maxfit(obj):
    return obj.fitness

solutionfound = False

totalbests = 0 

while solutionfound == False:

    mating_pool = []
    for i in population:
        i.calcfitness()
        n = int(i.fitness)
        for _ in xrange(n):
            mating_pool.append(i)

    if cycle == 1:
        avg = totalbests/1
    else:
        avg = totalbests/(cycle-1)

    print "Cycle: " + str(cycle) + " Gen " + str(gen) + ": " + str(max(population, key=maxfit).fitness) + " Average Gen/Cycle: " + str(avg)
    currentbest = max(population, key=maxfit).fitness
    bests.insert(0,currentbest)
    bests.pop()
    print bests
    max(population, key=maxfit).printPuzzle()

    for i in xrange(len(population)):
        a = random.randint(0, len(mating_pool)-1)
        b = random.randint(0, len(mating_pool)-1)

        parent_a = mating_pool[a]
        parent_b = mating_pool[b]

        child = parent_a.crossover(parent_b)
        child.processnofixed()
        child.mutate()
        population[i] = child

    # if it has reached a local minima, reset
    if bests == [currentbest] * 10:
        totalbests += currentbest
        population = []
        for _ in xrange(population_no):
            population.append(Solution())
        cycle += 1
        gen = 0

    gen += 1

    # stop if solution is found
    if currentbest == 84:
        solutionfound = True

# restart at local minima
# change mutation

{% endhighlight %}



