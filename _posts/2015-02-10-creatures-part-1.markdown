---
author: nhuhu
comments: false
date: 2015-02-10 08:49:46+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/02/10/creatures-part-1/
slug: creatures-part-1
title: 'Project: Autonomous Creatures Part 1'
wordpress_id: 119
---

So, I decided to try out that idea I had a few posts ago, combining both neural networks and genetic algorithms. This is gonna be pretty complicated, if it works, so I'm gonna break it down into smaller portions to do.

Today, I didn't do much. I basically set up everything, plopped a square thing in the centre as the creature, and thought about how it would move and scan the field for food/enemies.

Using a bit of maths, I think I can divide the playing field into 16 areas which would then be scanned and then the distance for each object could be calculated pretty easily. This info would then be fed into the neural net for it to do whatever it does.

Behold, my artistic masterpiece...

[![My artistic masterpiece... ](https://artificialcognition.files.wordpress.com/2015/02/grid.png?w=300)](https://artificialcognition.files.wordpress.com/2015/02/grid.png)

Anything further on, I still haven't decided on. But here's the code so far:

{% highlight python %}
import pygame, sys
from pygame.locals import *

pygame.init()

FPS = 100
fpsClock = pygame.time.Clock()

DISPLAYSURF = pygame.display.set_mode((600, 600))
pygame.display.set_caption('Ship')

WHITE = (255, 255, 255)

creature = Rect(0, 0, 20, 20)
critx = 300
crity = 300
creature.center = (critx, crity)

while True:
    DISPLAYSURF.fill(WHITE)
    pygame.draw.rect(DISPLAYSURF, (0,0,0), creature)

    for event in pygame.event.get():
        if event.type == QUIT:
            pygame.quit()
            sys.exit()

    pygame.display.update()
    fpsClock.tick(FPS)
{% endhighlight %}

And here's what it looks like right now, a lone square in the centre in the abyss:

[![And here's what it looks like right now, a lone square in the centre in the abyss.](https://artificialcognition.files.wordpress.com/2015/02/window-2015-02-10-at-21-49-07.png?w=291)](https://artificialcognition.files.wordpress.com/2015/02/window-2015-02-10-at-21-49-07.png)
