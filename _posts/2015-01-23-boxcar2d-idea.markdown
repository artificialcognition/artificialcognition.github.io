---
author: nhuhu
comments: false
date: 2015-01-23 04:19:40+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/01/23/boxcar2d-idea/
slug: boxcar2d-idea
title: 'Thoughts: BoxCar2D'
wordpress_id: 80
categories:
- Experiments
---
At the start, the car didn't really move anywhere.

[![At the start, the car didn't really move anywhere.](https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-23-at-17-12-18.png?w=300)](https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-23-at-17-12-18.png) 

But, by the later generation, progress was made.

[![By later generation, progress was made.](https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-23-at-17-08-27.png?w=300)](https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-23-at-17-08-27.png)

Today I played around with this website, that basically used genetic algorithms on cars. I read about how it was implemented [here](http://boxcar2d.com/about.html) and it was interesting, because although there were differences, the fundamental principles were the same and I could understand it.

I've been thinking of combining neural networks and genetic algorithms to create a sorta "game of life", with prey, predators and food. The thing I've been thinking about is how I would implement the fitness algorithm. Would I make it so that the prey that avoid the most predators and get the most food, are given more of a chance in the mating pool (as I have done previously with my other programs)? Or, should I go with a more organic approach whereby the prey who get "eaten" by predators are removed completely from the mating pool, as well as dying from hunger if they don't reach food? I don't know, but it's interesting to think about.
