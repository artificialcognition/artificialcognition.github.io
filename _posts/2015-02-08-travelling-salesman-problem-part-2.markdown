---
author: nhuhu
comments: false
date: 2015-02-08 03:01:55+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/02/08/travelling-salesman-problem-part-2/
slug: travelling-salesman-problem-part-2
title: 'Project: The Travelling Salesman Problem Part 2'
wordpress_id: 114
---

No matter how I alter the population or the mutation rate, the distance seems to get stuck around a distance of 8000 and a shape that it seems to stay in. I suspected that the mutation function didn't made the "shape" the easiest permutation and the highest fitness in the beginning, so it stayed that way.

I changed the mutate function so that instead of swapping two random genes, it would pick genes and then randomly choose to replace them with the other chosen genes:

{% highlight python %}
    def mutate(self):
        indexs = []
        tomutate = []

        for i in xrange(len(self.route)):
            if random.random() &lt; mutation_rate:
                indexs.append(i)

        for i in indexs:
            tomutate.append(self.route[i])

        for i in indexs:
            choiceindex = random.randint(0, len(tomutate)-1)
            self.route[i] = tomutate[choiceindex]
            del tomutate[choiceindex]
{% endhighlight %}

I then tried it out at 5% mutation and a population of 100. It got stuck at around a distance of 8000. Increasing the mutation rate to 7% got it closer. Increasing it to 10% got it even, even closer, down to 6700. I tried out a couple of different combos and I think this is the best it can do. This was a bit frustrating, but it made me realise that genetic algorithms have their limits, sadly. The initial population's genes is important though, because that affects, I think, if it will reach the final solution.

[![So close!](https://artificialcognition.files.wordpress.com/2015/02/screen-region-2015-02-05-at-17-58-08.png?w=300)](https://artificialcognition.files.wordpress.com/2015/02/screen-region-2015-02-05-at-17-58-08.png)
