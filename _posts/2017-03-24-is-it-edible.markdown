---
title: 'Project: Is it edible?'
---

Exams are over, so I've decided to post some of the projects I've made, but haven't posted on here.

This was one that I made at SB Hacks, my first hackathon! Great times, new friends, and so much cool stuff! It was definitely an experience I won't forget! 

Anyway, I was playing around with the [Clarifai](https://www.clarifai.com) API, which does image recognition, and I got the idea to combine this API with [Wordnet](https://wordnet.princeton.edu), to create a program that can tell if something is edible.

Basically, it takes in an image, uses Clarifai to generate a set of terms that it thinks best describes the image (Computer Vision is awesome!), then looks it up in Wordnet, finds out if these words are in the "Food" category, then returns the probability that it's edible.

An example:
Let's say I have this cake image in the same folder as the code, named "cake.jpg"
![here]({{ site.url }}/assets/cake.jpg)

Then, I can run the code as so: 
{% highlight python %}
python edible.py cake.jpg
{% endhighlight %}

It will give the output:
{% highlight python %}
['chocolate', 'cake', 'sweet']
I'm at least 99.382865% certain that's edible! :D Eat it!
{% endhighlight %}

The first line of output is a list of possible objects in the image, which it is surprisingly accurate about. The second line is the calculated probility that the object depicted in the image is edible, and it's right! Of course a cake is perfectly edible!

This was just a fun little experiment; nothing too difficult. So here's the code:

{% highlight python %}
from clarifai.rest import ClarifaiApp
from clarifai.rest import Image as ClImage
from nltk.corpus import wordnet as wn
import sys

filename = sys.argv[-1]
app = ClarifaiApp("oDbaG5DlF5P9jnG82suYYjD6_gL60wpapw0EQV5h", 
        "c65ZCtwa2IcaoI7OGR-9mjVrDoY8NLUnwszio8yF")

model = app.models.get("general-v1.3")

# prediction:
image = ClImage(file_obj=open(filename, 'rb'))
t = model.predict([image])


analysed = [[str(x["name"]), str(x["value"])] for x 
        in t["outputs"][0]["data"]["concepts"]]

guesses = [x[0] for x in analysed][:3]

print guesses

sub_defs = []
for g in guesses:
    sub_defs += wn.synsets(g)

hypernyms = []
for r in sub_defs:
    hypernyms += (r.hypernym_paths())[0]

if (wn.synset('food.n.01') in hypernyms) or (wn.synset('food.n.02') in hypernyms) or (wn.synset('food.n.03') in hypernyms):
    print "I'm at least " + str(float(analysed[2][1])*100) + "% certain that's edible! :D Eat it!"
else:
    print "I'm at least " + str(float(analysed[2][1])*100) + "% certain you shouldn't eat it! :("

{% endhighlight %}


