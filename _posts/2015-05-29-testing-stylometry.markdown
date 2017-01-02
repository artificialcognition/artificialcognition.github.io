---
author: nhuhu
comments: false
date: 2015-05-29 06:26:10+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/05/29/testing-stylometry/
slug: testing-stylometry
title: 'Project: Stylometry (Results) Part 2'
wordpress_id: 148
---

I didn't know if my program worked or not, so I decided to test it out. I decided to choose authors that wrote in English to ensure the use of the words that I looked for, were natural and consistent. 

First, I would test it against different texts by Charles Dickens and then compare the texts to those by other authors around the same time period:
<table style="width:100%;" >
<tbody >
<tr >

<td >
</td>

<td >Oliver Twist
</td>

<td >Great Expectations
</td>

<td >A Tale of Two Cities
</td>
</tr>
<tr >

<td >Oliver Twist
</td>

<td > 0%
</td>

<td >68.1%
</td>

<td > 60.4%
</td>
</tr>
<tr >

<td >Great Expectations
</td>

<td > 68.1%
</td>

<td > 0%
</td>

<td > 52.1%
</td>
</tr>
<tr >

<td >A Tale of Two Cities
</td>

<td > 60.4%
</td>

<td > 52.1%
</td>

<td > 0%
</td>
</tr>
</tbody>
</table>
Now, comparing different authors:
<table style="width:100%;" >
<tbody >
<tr >

<td >
</td>

<td >Pride and Prejudice
</td>

<td >Dracula
</td>
</tr>
<tr >

<td >Oliver Twist
</td>

<td > 218%
</td>

<td >115%
</td>
</tr>
<tr >

<td >Great Expectations
</td>

<td > 227%
</td>

<td > 127%
</td>
</tr>
<tr >

<td >A Tale of Two Cities
</td>

<td > 240%
</td>

<td > 112%
</td>
</tr>
</tbody>
</table>
So, that worked. I found that the same author had a percentage difference of < ~70% whilst a different author would have a difference of > ~110%.I also saw that a lower word count would lead to less accurate results, so I wanted to try the checker on a novella by Charles Dickens, A Christmas Carol.
<table style="width:100%;" >
<tbody >
<tr >

<td >
</td>

<td >A Christmas Carol (29400 words)
</td>
</tr>
<tr >

<td >Oliver Twist (171826 words)
</td>

<td > 63.0%
</td>
</tr>
<tr >

<td >Great Expectations (190198 words)
</td>

<td > 97.8%
</td>
</tr>
<tr >

<td >A Tale of Two Cities (138330 words)
</td>

<td > 71.3%
</td>
</tr>
</tbody>
</table>
Thus, word count does indeed affect the percentage difference because a smaller lengthed novella would contain less of the typical function words used by the same author. However, even though A Christmas Carol is significantly shorter than the rest of the novels, the percentage difference was still smaller than the books by the other authors. And I suspect if I used a few novellas from the other novels, the percentage differnce will be even smaller. I also think that some of the tags I decided to target is a natural part of English and there might be a more personal list of words. Anyway, the longer the texts, the more accurate this program is.
