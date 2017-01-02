---
author: nhuhu
comments: false
date: 2016-02-14 00:12:20+00:00
layout: post
link: https://artificialcognition.wordpress.com/2016/02/14/who-owns-the-zebra/
slug: who-owns-the-zebra
title: "Project: Solving Einstein's Puzzle with Python-Constraint"
wordpress_id: 341
categories:
- Experiments
---

I've known about this problem for a very long time now, and I solved it manually years ago, but it took ages. Also known by other names such as Einstein's Puzzle, the [Zebra Puzzle](https://en.wikipedia.org/wiki/Zebra_Puzzle) is a logic puzzle that gives you a set of clues and ask you to thus deduce who owns the zebra.

Here is the original puzzle from Life International in 1962:


```
1. There are five houses.
2. The Englishman lives in the red house.
3. The Spaniard owns the dog.
4. Coffee is drunk in the green house.
5. The Ukrainian drinks tea.
6. The green house is immediately to the right of the ivory house.
7. The Old Gold smoker owns snails.
8. Kools are smoked in the yellow house.
9. Milk is drunk in the middle house.
10. The Norwegian lives in the first house.
11. The man who smokes Chesterfields lives in the house next to the man with the fox.
12. Kools are smoked in the house next to the house where the horse is kept.
13. The Lucky Strike smoker drinks orange juice.
14. The Japanese smokes Parliaments.
15. The Norwegian lives next to the blue house.

Now, who drinks water? Who owns the zebra?

In the interest of clarity, it must be added that each of the five houses is painted a different color, and their inhabitants are of different national extractions, own different pets, drink different beverages and smoke different brands of American cigarettes. One other thing: in statement 6, right means your right.
```


To solve this problem, I remembered a Python library that I had seen a while ago. It was called [python-constraint](https://labix.org/python-constraint) and it's perfect for this type of problem. Basically, you generate a large library of possibilities and add constraints so that it reduces the size of the library.

After following the tutorial, it was fairly easy to write the code to solve the Zebra Puzzle:

{% highlight python %}
# Houses
# 1 2 3 4 5

from constraint import *
problem = Problem()

nationality = ["English", "Spanish", "Ukrainian", "Norwegian", "Japanese"]
pet = ["dog", "snails", "fox", "horse", "zebra"]
cigarette = ["Old Gold", "Kools", 
"Chesterfields", "Lucky Strike", "Parliaments"]
colour = ["red", "green", "yellow", "blue", "ivory"]
beverage = ["coffee", "milk", "orange juice", "water", "tea"]

criteria = nationality + pet + cigarette + colour + beverage
problem.addVariables(criteria,[1,2,3,4,5])

problem.addConstraint(AllDifferentConstraint(), nationality)
problem.addConstraint(AllDifferentConstraint(), pet)
problem.addConstraint(AllDifferentConstraint(), cigarette)
problem.addConstraint(AllDifferentConstraint(), colour)
problem.addConstraint(AllDifferentConstraint(), beverage)

problem.addConstraint(lambda e, r: e == r, ["English","red"])
problem.addConstraint(lambda s, d: s == d, ("Spanish","dog"))
problem.addConstraint(lambda c, g: c == g, ("coffee","green"))
problem.addConstraint(lambda u, t: u == t, ("Ukrainian","tea"))
problem.addConstraint(lambda g, i: g-i == 1, ("green","ivory"))
problem.addConstraint(lambda o, s: o == s, ("Old Gold","snails"))
problem.addConstraint(lambda k, y: k == y, ("Kools","yellow"))
problem.addConstraint(InSetConstraint([3]), ["milk"])
problem.addConstraint(InSetConstraint([1]), ["Norwegian"])
problem.addConstraint(lambda c, f: abs(c-f) == 1, ("Chesterfields","fox"))
problem.addConstraint(lambda k, h: abs(k-h) == 1, ("Kools","horse"))
problem.addConstraint(lambda l, o: l == o, ["Lucky Strike","orange juice"])
problem.addConstraint(lambda j, p: j == p, ["Japanese","Parliaments"])
problem.addConstraint(lambda k, h: abs(k-h) == 1, ("Norwegian","blue"))

solution = problem.getSolutions()[0]

for i in range(1,6):
    for x in solution:
        if solution[x] == i:
            print str(i), x
{% endhighlight %}

Here was what it produced:

{% highlight python %}
1 yellow
1 water
1 Kools
1 fox
1 Norwegian
2 tea
2 blue
2 horse
2 Ukrainian
2 Chesterfields
3 Old Gold
3 English
3 milk
3 snails
3 red
4 ivory
4 dog
4 Lucky Strike
4 orange juice
4 Spanish
5 Parliaments
5 coffee
5 zebra
5 Japanese
5 green
{% endhighlight %}

The number represents each house, from left to right. This matches the solution on the wikipedia page, so thus it works.
