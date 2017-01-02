---
author: nhuhu
comments: false
date: 2016-02-06 02:15:19+00:00
layout: post
link: https://artificialcognition.wordpress.com/2016/02/06/using-a-deep-belief-neural-network-to-diagnose-breast-cancer/
slug: using-a-deep-belief-neural-network-to-diagnose-breast-cancer
title: 'Project: Using a Deep-Belief Neural Network to Diagnose Breast Cancer'
wordpress_id: 361
categories:
- Experiments
---

I started learning more about deep-belief networks because of all the latest news surrounding this tool. Basically, they are Restricted Boltzmann Machines stacked together and have been shown to be extremely useful in classifying things.

I followed a [tutorial](http://www.pyimagesearch.com/2014/09/22/getting-started-deep-learning-python/) and had to learn a few libraries required for the DBN. And then, I decided to apply this to classify the [Wisconsin Breast Cancer (Diagnostic) Data Set](https://archive.ics.uci.edu/ml/datasets/Breast+Cancer+Wisconsin+(Diagnostic)) to see if it would work on a real life (and extremely important) problem.

One shortcut I used to scale the data was as follows:

{% highlight python %}
b = []
for d in dataset.data:
    for y in d:
        b.append(y)

# found using max point
scaled = dataset.data/ max(b)
{% endhighlight %}

Anyway, the results were promising with an accuracy of 94% overall in diagnosing breast cancer with a malignant diagnosis of 97%. This is very good because it means that it's more likely than not to diagnose a false positive than a false negative.

Here's the code:

{% highlight python %}
from sklearn.cross_validation import train_test_split
from sklearn.metrics import classification_report
from sklearn import datasets
from nolearn.dbn import DBN
import numpy as np
import cv2

dataset = datasets.load_breast_cancer()

b = []
for d in dataset.data:
    for y in d:
        b.append(y)

# found using max point
scaled = dataset.data/ max(b)

(trainX, testX, trainY, testY) = train_test_split(
    scaled, dataset.target, test_size = 0.33)

dbn = DBN(
	[trainX.shape[1], 500, 500, 2],
	learn_rates = 0.5,
	learn_rate_decays = 0.6,
	epochs = 10,
	verbose = 1)

dbn.fit(trainX, trainY)

preds = dbn.predict(testX)

print dataset.DESCR

print "0: " + str(dataset.target_names[0])
print "1: " + str(dataset.target_names[1])
print classification_report(testY, preds)
{% endhighlight %}

This produces:

```
[DBN] fitting X.shape=(381, 30)
[DBN] layers [30, 500, 500, 2]
[DBN] Fine-tune...
100%
Epoch 1:
  loss 0.667588567734
  err  0.41875
  (0:00:00)
100%
Epoch 2:
  loss 0.61961773634
  err  0.353125
  (0:00:00)
100%
Epoch 3:
  loss 0.544265073538
  err  0.215625
  (0:00:00)
100%
Epoch 4:
  loss 0.410316121578
  err  0.16875
  (0:00:00)
100%
Epoch 5:
  loss 0.314362972975
  err  0.10625
  (0:00:00)
100%
Epoch 6:
  loss 0.306012681127
  err  0.13125
  (0:00:00)
100%
Epoch 7:
  loss 0.242035767436
  err  0.11875
  (0:00:00)
100%
Epoch 8:
  loss 0.286982005835
  err  0.14375
  (0:00:00)
100%
Epoch 9:
  loss 0.22955622673
  err  0.121875
  (0:00:00)
100%
Epoch 10:
  loss 0.247347301245
  err  0.09375
  (0:00:00)
Breast Cancer Wisconsin (Diagnostic) Database

Notes
-----
Data Set Characteristics:
    :Number of Instances: 569

    :Number of Attributes: 30 numeric, predictive attributes and the class

    :Attribute Information:
        - radius (mean of distances from center to points on the perimeter)
        - texture (standard deviation of gray-scale values)
        - perimeter
        - area
        - smoothness (local variation in radius lengths)
        - compactness (perimeter^2 / area - 1.0)
        - concavity (severity of concave portions of the contour)
        - concave points (number of concave portions of the contour)
        - symmetry
        - fractal dimension ("coastline approximation" - 1)

        The mean, standard error, and "worst" or largest (mean of the three
        largest values) of these features were computed for each image,
        resulting in 30 features.  For instance, field 3 is Mean Radius, field
        13 is Radius SE, field 23 is Worst Radius.

        - class:
                - WDBC-Malignant
                - WDBC-Benign

    :Summary Statistics:

    ===================================== ======= ========
                                           Min     Max
    ===================================== ======= ========
    radius (mean):                         6.981   28.11
    texture (mean):                        9.71    39.28
    perimeter (mean):                      43.79   188.5
    area (mean):                           143.5   2501.0
    smoothness (mean):                     0.053   0.163
    compactness (mean):                    0.019   0.345
    concavity (mean):                      0.0     0.427
    concave points (mean):                 0.0     0.201
    symmetry (mean):                       0.106   0.304
    fractal dimension (mean):              0.05    0.097
    radius (standard error):               0.112   2.873
    texture (standard error):              0.36    4.885
    perimeter (standard error):            0.757   21.98
    area (standard error):                 6.802   542.2
    smoothness (standard error):           0.002   0.031
    compactness (standard error):          0.002   0.135
    concavity (standard error):            0.0     0.396
    concave points (standard error):       0.0     0.053
    symmetry (standard error):             0.008   0.079
    fractal dimension (standard error):    0.001   0.03
    radius (worst):                        7.93    36.04
    texture (worst):                       12.02   49.54
    perimeter (worst):                     50.41   251.2
    area (worst):                          185.2   4254.0
    smoothness (worst):                    0.071   0.223
    compactness (worst):                   0.027   1.058
    concavity (worst):                     0.0     1.252
    concave points (worst):                0.0     0.291
    symmetry (worst):                      0.156   0.664
    fractal dimension (worst):             0.055   0.208
    ===================================== ======= ========

    :Missing Attribute Values: None

    :Class Distribution: 212 - Malignant, 357 - Benign

    :Creator:  Dr. William H. Wolberg, W. Nick Street, Olvi L. Mangasarian

    :Donor: Nick Street

    :Date: November, 1995

This is a copy of UCI ML Breast Cancer Wisconsin (Diagnostic) datasets.
https://goo.gl/U2Uwz2

Features are computed from a digitized image of a fine needle
aspirate (FNA) of a breast mass.  They describe
characteristics of the cell nuclei present in the image.
A few of the images can be found at
http://www.cs.wisc.edu/~street/images/

Separating plane described above was obtained using
Multisurface Method-Tree (MSM-T) [K. P. Bennett, "Decision Tree
Construction Via Linear Programming." Proceedings of the 4th
Midwest Artificial Intelligence and Cognitive Science Society,
pp. 97-101, 1992], a classification method which uses linear
programming to construct a decision tree.  Relevant features
were selected using an exhaustive search in the space of 1-4
features and 1-3 separating planes.

The actual linear program used to obtain the separating plane
in the 3-dimensional space is that described in:
[K. P. Bennett and O. L. Mangasarian: "Robust Linear
Programming Discrimination of Two Linearly Inseparable Sets",
Optimization Methods and Software 1, 1992, 23-34].

This database is also available through the UW CS ftp server:

ftp ftp.cs.wisc.edu
cd math-prog/cpo-dataset/machine-learn/WDBC/

References
----------
   - W.N. Street, W.H. Wolberg and O.L. Mangasarian. Nuclear feature extraction
     for breast tumor diagnosis. IS&T/SPIE 1993 International Symposium on
     Electronic Imaging: Science and Technology, volume 1905, pages 861-870,
     San Jose, CA, 1993.
   - O.L. Mangasarian, W.N. Street and W.H. Wolberg. Breast cancer diagnosis and
     prognosis via linear programming. Operations Research, 43(4), pages 570-577,
     July-August 1995.
   - W.H. Wolberg, W.N. Street, and O.L. Mangasarian. Machine learning techniques
     to diagnose breast cancer from fine-needle aspirates. Cancer Letters 77 (1994)
     163-171.

0: malignant
1: benign
             precision    recall  f1-score   support

          0       0.97      0.85      0.90        66
          1       0.92      0.98      0.95       122

avg / total       0.94      0.94      0.94       188
```

