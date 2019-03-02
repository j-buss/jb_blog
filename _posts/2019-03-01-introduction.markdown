---
layout: post
title:  "Introduction"
date:   2019-03-01 18:49:37 -0600
permalink: /:categories/:title.html
categories: text-gen-lstm
---
Let’s make a robo-Nietzsche...you know some electronic embodiment of Friedrich Nietzsche...a an electronic doppleganger of sorts...

<a title="Friedrich Hartmann
 [Public domain], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Nietzsche187a.jpg"><img width="175" alt="Nietzsche187a" src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Nietzsche187a.jpg/256px-Nietzsche187a.jpg"></a>

<img src="{{ site.baseurl }}/images/robot.png" height="100" width="100">

![robot]({{ site.baseurl }}/images/robot.png)

The following set of blogs document the information for exploring the generation of text from an LSTM.
The impetus for this experiment was the book [Deep Learning with Python by François Chollet][DLwP_Book]. 

### Why?

Well I  wanted to learn how to use tensorflow and I wanted a more “complete” description than I could easily find online. 
So I picked up the book Deep Learning with Python by François Chollet. 
The book provides a wonderful explanation of Deep Learning using Keras. 
I followed along and performed all the steps from the book. 

It was going well but there was a question that was simmering below the surface that finally came to a head by the time I got to chapter 8  (out of 9....I am not the quickest learner) 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*How can I **actually** build something with one of these models?*

The book does a great job of teaching you how to build a model.
However I wanted to have something that used the model...an **application** if you will. 
Even if it was a silly application.

Previously, when I was studying machine learning there were many resources that walk you through the model:

However I wanted something more like the this:


#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*This would allow me to not only **make** a working model, but to **use** it as well*

So with this set of posts we will explore the build and implementation of a machine learning model. 
With the intent of using and learning about many tools along the way: Google Colaboratory, Google Cloud Platform, Kubernetes, Cloud Functions, Stackdriver, Computer Engine, Cloud ML, etc. 

So if you are ready...let's dive right in...

### Start the Build



{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}


[DLwP_Book]: https://www.manning.com/books/deep-learning-with-python
[DLwP_github]: https://github.com/fchollet/deep-learning-with-python-notebooks
[robot_img]: ({{site.url}}/images/robot.png)
