---
author: nhuhu
comments: false
date: 2015-02-15 05:37:00+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/02/15/creatures-part-2/
slug: creatures-part-2
title: 'Project: Autonomous Creatures Part 2'
wordpress_id: 123
---

Ok, today I integrated neural networks into the whole thing. So now the "creature" can move according to input.

The directions it can move in were done with vectors. And since a square grid doesn't allow it to move diangonally in the same distance it can move vertically/horizontally, a work-around had to be found.

Using some basic trigonometry, we'll use a rough estimate of (0.7, 0.7) as the vector for diagonal movement.

[![blah](https://artificialcognition.files.wordpress.com/2015/02/blah.jpg?w=300)](https://artificialcognition.files.wordpress.com/2015/02/blah.jpg)

I've also added an enemy circle (cos everyone knows circles are completely and utterly evil...), but I'm still unsure how to input it into the neural net. Don't get me started on how I'm gonna use gAs, cos that's still far off.
