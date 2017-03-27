---
layout: post
title: Machine Learning - Getting the Hang of It
date: 2017-03-27
share: y
disqus: y
---

[coursera_ml]:https://www.coursera.org/learn/machine-learning/home/welcome
[john_wittenauer_nnet]:http://www.johnwittenauer.net/machine-learning-exercises-in-python-part-5/
[terminator]:https://en.wikipedia.org/wiki/Terminator_(franchise)
[space_apps]:https://2016.spaceappschallenge.org/locations/cluj-napoca-romania
[sphero_bb8]:http://www.sphero.com/starwars/bb8
[diddyborg]:https://www.piborg.org/diddyborg
[tensorflow]:https://www.tensorflow.org/
[andrew_ng]:http://www.andrewng.org/
[gist_url]:https://gist.github.com/andr0idsensei/92dd7e54a029242690555e84dca93efd

  Unless you've been living under a rock and not following the mainstream tech blogs and news in the last couple of years, you're no stranger to the slew of Artificial Intelligence
is eating the world articles that include scenarios from the most optimistic to [Terminator][terminator] like apocalyptic. To be honest, I've thought about writing an opinion piece about the sujbect right here on my blog, but since everybody is doing that and some are doing a much better job at it than I could, I decided to drop it. 
Instead, I am going to write about how I got started with Machine Learning together with some resources I used.

<br/><br/>
  For me, I think the fascination with machines that move on their own started when I was a kid got a wind-up robot toy as a gift (at the time there were no fancy toys such
 as [Sphero's BB8][sphero_bb8]). Of course I played with it until it broke, then I cryed and wanted a new one. I guess that my fascination with gadgets and robots still lived on and then when I grew up I became a software developer. With the recent surge of AI related buzz, the flame rekindled and I got myself in the flow reading a lot of the above mentioned articles. However just reading about it doesn't actually make you an expert on the subject and the mainstream tech outlets are only generating buzz, they don't offer technical details on how things really work, but at least they got me wanting to dig deeper. I even bought myself a [Diddyborg][diddyborg] rover so that I could play around with the electronics and software, but the initial kick into gears came last April with the first edition of [NASA Space Apps Challenge][space_apps] that was held in Romania, here in Cluj.

<br/><br/>
  Together with some friends we decided to participate and we picked one of the hardest challenges that was posted - Asteroid Hunting using Machine Learning. Of course we bit off more
than we could chew and we didn't manage to demo anything but just presented a concept in the end. I've been reading about [Tensorflow][tensorflow], looked at some tutorials and I thought:
how hard can it be to hack something into place and make a demo. Well...I guess that showed me :). This bump didn't discourage me however, and around October I had bit more than usual free time so I decided to actually learn how Machine Learning works so I enlisted in [Coursera's Machine Learning Course][coursera_ml] held by [Andrew Ng][andrew_ng], one of the currently World renowned Artificial Intelligence experts.

<br/><br/>
  I've enlisted and completed only 2 MOOCs so far and from my academic experience, Andrew Ng's Machnine Learning course was by far the best learning experience I had. His style is very
reassuring and accessible even to people without an academic and/or math background. There is very little math involved and Andrew Ng explains the Machine Learning concepts in a way that is easy to follow and understand. So, if you want a solid Machine Learning base, I highly recommend this course.

<br/><br/>
  Since Machine learning is a subset of AI and Deep Learning is a subset of Machine Learning, now I want to dig deeper and gain more knowledge especially about Deep Learning techniques,
but before I conclude this article I would like to post a Github Gist with a vectorized Python implementation of the neural network exercise from Coursera's ML course. The course code was required to be written in Octave or Matlab and since most Machine Learning libraries are written in Python and Python is a popular language in the Machine Learning
and Data Science communities, I wanted to try implementing the neural network from the course in Python. I thought this would be a good opportunity to better understand how the Backpropagation alogrithm works and after a quick search I've found out that other people implemented the course exercises in Python so I chose to have a look at [John Wittenauer's implementation][john_wittenauer_nnet], which actually is pretty comprehensive. I've used his code with some small tweaks and cosmetisations to implement the neural network and after I ran it, I thought it would be a good idea to try and vectorize the implementation. Vectorization is used throughout the course as Professor Ng recommends to use it whenever possible since the Linear Algebra libraries have optimizations which you can take advantage of to make your code a lot faster. He was right about that; the for loop implementation that I first used from John's article is a lot slower than the vectorized one. When I ran it, the for loop version took about 436 seconds to train the neural network with 5000 examples while the vectorized one took about 17 seconds with the same amount of example data. The code is only Python with raw numpy, no fancy ML libraries used and here is the [Gist][gist_url] I created on GitHub.

<br/><br/>
  I hope I didn't bore you with my stories and that the code I shared will prove to be useful for you. When I'll have some more writing time I'll come back with some more resources for the
the ones of you who want to study Machine Learning and are novices like me.
