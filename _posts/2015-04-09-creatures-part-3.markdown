---
author: nhuhu
comments: false
date: 2015-04-09 05:05:43+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/04/09/creatures-part-3/
slug: creatures-part-3
title: 'Project: Autonomous Creatures Part 3'
wordpress_id: 126
---

The creature now has enemies! Oh no, how evil of me... >:)

[![>:D](https://artificialcognition.files.wordpress.com/2015/04/window-2015-04-09-at-16-56-05.png?w=291)](https://artificialcognition.files.wordpress.com/2015/04/window-2015-04-09-at-16-56-05.png)

I've also solved the problem of calculating and inputting the co-ordinates of the enemy into the neural net. Basically, the number of radians from north is calculated for each of the enemy, and then this information is converted into sections (like a pie), before going into the neural net of the creature.

A problem I had was that the angle calculated using numpy resulted sometimes (when the creature was to the right of the enemy)  in an angle that was from the north, but from the other side (anti-clockwise instead of clock-wise). This was quite easily solved though.

Here's that piece of code:

{% highlight python %}
def calcsection(cx, cy, ex, ey):
    # radians from north
    a = numpy.arccos(numpy.dot(numpy.array([0,1]), numpy.array([ex-cx, cy-ey])) / (numpy.linalg.norm(numpy.array([0,1]) * numpy.linalg.norm(numpy.array([ex-cx, cy-ey])))))

    if cx >= ex:
        angle = 360 - a*180/numpy.pi
    else:
        angle = a*180/numpy.pi
    
    # sections of input
    sections = [(22.5, 67.5), (67.5, 112.5), (112.5, 157.5), (157.5, 202.5), (202.5, 247.5), (247.5, 292.5), (292.5, 337.5)]
    
    # finds out which section the enemy is in, based on the angle
    if angle > 337.5 or angle < 22.5:
        return 0
    else:
        for i in range(len(sections)):
            (a, b) = sections[i]
            if angle > a and angle < b:
                return i + 1 # inindex will go into the neural net
{% endhighlight %}

So, when the simulation is run, the section for each enemy is calculated before being entered into the neural network with a set input of 100 (I was just picking a relatively large number). However, I noticed that sometimes the creature would just become stuck because every enemy affected it the same way with no regard for proximity. So, I used some basic maths and worked out the distance for each one and used that as the input into the neural network.

{% highlight python %}
    for e in enemies:
        (enemx, enemy) = e
        inindex = calcsection(critx, crity, enemx, enemy)
        distance = numpy.sqrt((critx-enemx)**2+(crity-enemy)**2)
        netin[inindex] += 1/distance * 1000

    index = numpy.argmax(net.activate(netin))
    direction = directions[index]

    (x, y) = direction # moves the creature every frame
    critx += x
    crity += y
{% endhighlight %}

Now, we just need to apply genetic algorithms to it, which should be pretty formulaic. Yay! 
