---
author: nhuhu
comments: false
date: 2015-09-25 03:19:15+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/09/25/playing-with-shakespeare-programming-language-spl/
slug: playing-with-shakespeare-programming-language-spl
title: 'Project: Shakespeare Programming Language (SPL)'
wordpress_id: 198
---

[![Shakespeare](https://artificialcognition.files.wordpress.com/2015/09/shakespeare.jpg?w=660)](https://artificialcognition.files.wordpress.com/2015/09/shakespeare.jpg)

So, I've heard about esoteric programming languages and I found a really cool one called Shakespeare Programming Language. It's basically a turing-complete programming language, said by some to be kinda like Assembly. With SPL, code is written like a Shakespearian play.

Something really interesting about the syntax is that "characters" set up **other** character's values. These characters themselves are variables (well, no, they're actually stacks).

Anyway, these characters interact with each other like:

```
Romeo:
 You lying stupid fatherless big smelly half-witted coward!
```

Though this just looks like a piece of dialogue, it actually means that they character (variable) Romeo is talking to is assigned a value. For example, if Romeo and Juliet are in the same scene, then this means that Juliet is assigned a value of 2^6 x (-1). This is because nouns are sorted into positive nouns (+1) and negative ones (-1). Then, the adjectives multiply the noun value by 2. So n adjectives = 2^n.

The numbers can then be outputted into the numerical value using "Open your heart" or the ASCII value using "Speak your mind". Alternatively, input can be gotten using "Listen to your heart" or "Open your mind". If statements have to be split between two characters as per follows (it's actually an interesting structure):

```
Juliet:
 Am I worse than you?

Romeo:
 If so, let us proceed to scene I.
```

Hence, I tried to write a "Hello World" program:

```
The Beginning of All Beginnings.

Romeo, a young man with ambition.
Juliet, a young woman with extraordinary memory.
Hamlet, a young man with a dream.

        Act I: Merciless Rants.

        Scene I: Anger.

[Enter Romeo and Juliet]

Romeo:
 You are the product of a codpiece and the square of a 
 horrible sorry villainous pig.
 Thou art as slow as the difference between a pretty lovely 
 sweet flower and thyself.
 Speak your mind. 

[Exit Romeo and Juliet]

        Scene II: Insults.

[Enter Hamlet and Juliet]

Juliet:
 You are a brave proud mighty prince.
 You are the sum of yourself and a distasteful pig.
 You are the sum of yourself and a coward.

Hamlet:
 You disgusting oozing infected hound.
 You are the square of the difference between a lovely plum
 and yourself.
 Thou art the sum of myself and yourself. 
 Speak your mind.

Juliet:
 You are as cruel as the difference between a beautiful 
 rich brave girl and a stupid pig!
 Speak your mind.

[Exit Hamlet and Juliet]
```

All that... and I get:

```
Hi
```
Not a very efficient language.
