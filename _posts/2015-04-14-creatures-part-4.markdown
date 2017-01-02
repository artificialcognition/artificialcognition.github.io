---
author: nhuhu
comments: false
date: 2015-04-14 09:19:13+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/04/14/creatures-part-4/
slug: creatures-part-4
title: 'Project: Autonomous Creatures Part 4'
wordpress_id: 129
---

I have changed it so that the creature wraps around the map if it goes out of the window. Before that, I tried to make it so that it bounced back in the opposite direction it hit the wall at, but there was a problem. The creature would, everytime without fail, get stuck. I realised this was because if it bounced back, the neural net would still tell it the same thing (almost!) and it would get trapped in a loop. So thus, the wrap:

{% highlight python %}
if self.x < 0:
    self.x = 600

if self.x > 600:
    self.x = 0

if self.y < 0:
    self.y = 600
    
if self.y > 600:
    self.y = 0
{% endhighlight %}

I've also made it so that the creature, upon touching the enemy "dies".

Furthermore, I've completely restructured the whole code so that it's primed for implementing the genetic algorithm. I've added a class called Creature and organised my code from there. 

Then, I added a population and currently the creatures spawn and gradually disappear when they hit the enemy. Some of them go in a sorta loop that allows them to avoid the enemy (which isn't pre-coded, it's on-the-spot). Others get trapped in a certain place because of conflicting signals. I plan so that these stagnated ones die out of hunger. That will only leave the ones that survive and avoid the enemies. 

Here's it in action:

[![creatures](https://artificialcognition.files.wordpress.com/2015/04/creatures.gif?w=300)](https://artificialcognition.files.wordpress.com/2015/04/creatures.gif)
