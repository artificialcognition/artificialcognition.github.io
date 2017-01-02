---
author: nhuhu
comments: false
date: 2016-03-17 00:57:08+00:00
layout: post
link: https://artificialcognition.wordpress.com/2016/03/17/artificial-mozart-using-ai-to-compose-music/
slug: artificial-mozart-using-ai-to-compose-music
title: 'Project: Using Neural Networks to Compose Simple Melodies'
wordpress_id: 474
categories:
- Experiments
---

Similar to using genetic algorithms to generate art, and recurrent neural networks to generate poetry, is an experiment I decided to undertake on whether or not recurrent neural networks can compose music.

Again, I had to use this recurrent neural network library, but I also had to adapt it so that it could generate songs. To do this, I had to research methods on music notation and how songs could be stored and played in Python. After much research, I came across [PySynth](https://mdoege.github.io/PySynth/), which was a library that could play songs in [ABC notation](https://en.wikipedia.org/wiki/ABC_notation).

For example, Deck the Halls would be written in this notation as:

```
X: 1
T: Deck the Halls
R: reel
M: 4/4
L: 1/8
K: C
|:  "C"g3f "(G)"e2d2 | "(Am)"c2d2   e2c2 | "G7"defd  "C"e3d   | "G7"c2B2 "C"c2 :|
[| "G7"d3e      f2d2 |  "C"  e3f "G"g2d2 | "C"e^fg2 "Am"abc'2 | "D7"b2a2 "G"g4 ||
||  "C"g3f "(G)"e2d2 | "(Am)"c2d2   e2c2 | "F"aaaa   "C"g3f   | "G7"e2d2 "C"c2 |]
```

The song number (X:), title (T:), type of tune (R:), time signature (M:), default note length (L:) and key (K:) are all included in ABC notation.

Entering this into PySynth results in this tune being returned:

<iframe width="100%" height="166" scrolling="no" frameborder="no" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/252253384%3Fsecret_token%3Ds-pG0sK&amp;color=ff5500&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false"></iframe>

As it also turns out, there are awesome people out there who have transcribed heaps of songs into ABC notation and have placed them in collection. I decided to use a collection of folk songs as the input. These songs come from around the world, but a majority of them is Germanic.

Using a text file (~1.7 MB) of training data, I set up and trained a recurrent neural network in lua, with a dropout rate of 0.5, and with a rnn_size of 360, giving a parameter size of 1739259. As you can imagine, this process took a fair bit of time and I had to leave it overnight, so it probably trained for 15+ hours for a total of 53.24 epochs.

Finally, I had a recurrent neural network that could generate hopefully passable folk music! I'm not a music expert and I've certainly never done music theory, but I think that these songs aren't too shabby. They sound 'authentic' enough.

Here's a small sampling at a temperature of 0.9 (with 0 being conservative and 1 being risky):

```
X:64
T: Die Schlangenkoechin
N: Q2075S
O: Mitteleuropa, Aehtenland
N: Eingfulgertfimhengekenstertes
R: Maerchen - Lied
M: 3/4
L: 1/8
K: G
D2 | G3BB2 | A2A2
A2 | B2E2G2 | A2z2
d2 | G2B2d2 | d2c2
B2 | c3AA2 | G4
A2 | B3AG2 | A3Bc2 | d6- | d4
B2 | A3GF2 | G3ABG | A2G2D2 | G4

X:25
T: Graf und Nonne (Die Nonne)
N: Q01552
O: Mitteleuropa, Deutschland (DDR) , Sachsen, Gruch arlod,
R: Ballade, Tod, Brautterbung, Tod
M: 6/4
L: 1/8
K: G
d2 | B2G2G2B4G2 | A2F2G4z2
B2 | G2A2B2c2A2G2 | G2
G2G2B2d2B2 | A4G2G2z2
B2 | A2A2F2ABA2G2 | B2A2A2G4

X:42
T: Die Altzierschei Schuerstat
N: Q0100
O: Mitteleuropa, Deutschland (BRD) , Hessen, Wetserau
R: Ballade, Menschenhandel Grab, Frauenreiburg, Fust; schoefkelige
M: 3/8
L: 1/8
K: G
D | GGG | DEc | BBB |
AAG | DFA | G2z |
GGF | GAG | A2d | B2

X:138
T: Jiedes Haucht uns Habe 2Iem lasse Outsscheen Hirten
N: D0000
O: Europa, Mitteleuropa, Deutschland
R: Jahreszeiten - Lied, Fruehling, cettit
M: 6/8
L: 1/8
K: A
A | A2AA2B | e2cA2
c | E2GFEE | E4z2

X:320
T: HIEM MIR MUSS WALSEN DURBACHT
N: B0160
O: Europa, Mitteleuropa, Deutschland, Schlesien
R: Kinder -, Wiegen - Lied
M: 4/4
L: 1/8
K: G
DD | G4G2B2 | A3GF2
E2 | G2B2A2B2 | G4z2

X:72
T: Wiedersieden Matku
N: A0196B
O: Europa, Mitteleuropa, Deutschland
N: Persan zu 1539 S. 27, Ams Jigrvolwest.
R: Romanze, Ballade, Lied, religioeses Lied
M: 4/4
L: 1/8
K: G
d2 | d2B2G2A2 | G4
G3A | d2d2B2B2 | d4z4 | z2
d2d2c2 | B3BA2B2 | c4z2
A2 | A3Bc2e2 | d4z2
g2 | d3BB2d2 | d3cB2
d2 | B3AG2G2 | F2D2E2
D2 | G4G3A | B4A4 | G6

X:60
T: FrOT N/ OISE KREIS KRAIKT
N: E1000
O: Europa, Mitteleuropa, Deutschland
R: Staende -, Soldaten -, Kriegs - Lied
M: 3/4
L: 1/8
K: Bb
F | B2B2GF | B2A2
F2 | G2F2F2 | D2z2
B2 | A2G2A2 | B4
F2 | G2A2B2 | A4F2 |
G3FG2 | F2F2z2 | F2AAB2 | F4
FF | B2B2B2 | B2A2
F2 | G3FE2 | F2z2
FB | A2G2B2 | B2z2
B2 | c2A2B2 | c2
B2F2 | B3GF2 | G2F2
E2 | B3AG2 | c3BAG | F6 | F2z2
Bz | G2B2A2 | G2F2
FF | E4G2 | F2C2
A2 | B3zcB | A4z2

X:266
T: WIR WEFESTEN IRR SCHLAT AUES DET ICH MIS GENEKRUGEN
N: B0162
O: Europa, Mitteleuropa, Deutschland, Hessen, Oberhassel, Jandorfend
R: Staende -, Soldaten -, Kriegs - Lied
M: 4/4
L: 1/8
K: F
(3F2D2 | F3EG2G2 | F6
G2 | G3EF2G2 | A4z2
F2 | A2G2F2E2 | E3GB2B2 | A2GFG2G2 | F4z2
A2 | A2A2A2c2 | cAAGF2
C2 | FFA2B2B2 | B2B2A2F2 |
F2G2A2B2 | c4z2
AB | c3Ac2c2 | B4A2
B2 | c2c2B2c2 | A4z2
AB | c2e2e2d2 | c3AA2B2 | c2A2G2G2 | F3
```

The title names reflect the Germanic majority of songs (though they're probably nonsensical because of the nature of the training of this neural network which focused on song production rather than title production). 

Also keep in mind these songs are completely original, because of how recurrent neural networks learn, one character at a time. With more time and processing speed, perhaps, it would learn more and have a more in depth understanding of music. For now, I'm happy that the songs, at least don't sound grating and could pass for a tune.


Here are the AI-composed folk songs:
<iframe width="100%" height="166" scrolling="no" frameborder="no" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/252257626%3Fsecret_token%3Ds-cox1t&amp;color=ff5500&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false"></iframe>
<iframe width="100%" height="166" scrolling="no" frameborder="no" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/252257710%3Fsecret_token%3Ds-YxR2j&amp;color=ff5500&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false"></iframe>
<iframe width="100%" height="166" scrolling="no" frameborder="no" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/252256826%3Fsecret_token%3Ds-nkc0q&amp;color=ff5500&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false"></iframe>
<iframe width="100%" height="166" scrolling="no" frameborder="no" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/252256979%3Fsecret_token%3Ds-15Rqj&amp;color=ff5500&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false"></iframe>
<iframe width="100%" height="166" scrolling="no" frameborder="no" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/252257207%3Fsecret_token%3Ds-lNouq&amp;color=ff5500&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false"></iframe>
<iframe width="100%" height="166" scrolling="no" frameborder="no" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/252257451%3Fsecret_token%3Ds-AWQvU&amp;color=ff5500&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false"></iframe>
<iframe width="100%" height="166" scrolling="no" frameborder="no" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/252257543%3Fsecret_token%3Ds-cxFem&amp;color=ff5500&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false"></iframe>





