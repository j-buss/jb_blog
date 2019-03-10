---
layout: post
title:  "Introduction"
date:   2019-03-01 18:49:37 -0600
permalink: /:categories/:title.html
categories: text-gen-lstm
---

What do you get when you combine a German philosopher and some AI..

## Robo-Nietzsche

<p float="left">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Nietzsche187a.jpg/256px-Nietzsche187a.jpg" style="width: 48%; margin-right: 1%; margin-bottom: 0.5em;" title="Friedrich Hartmann [Public domain], via Wikimedia Commons">
<img src="{{ site.baseurl }}/assets/images/robot.png" style="width: 48%; margin-right: 1%; margin-bottom: 0.5em;" title="Bilboq [Public domain], via Wikimedia Commons">
</p>

# Introduction

This all started when I wanted to learn about deep learning. I was following along with the exercises in the book [Deep Learning with Python by François Chollet][DLwP_Book]. 

When I got to Chapter 8 - Generative deep learning François had a great example of creating text with a model after being trained on a corpus of text.
It was fascinating working through all the code. However I wanted a way to create something more than just reproducing code. I wanted to make an **application**...something I could interact with.

So that was how my journey began to build a text generating application using an LSTM model.

## What is an LSTM model?

A long short-term memory ([LSTM](https://en.wikipedia.org/wiki/Recurrent_neural_network)) model is a [recurrent neural network](https://en.wikipedia.org/wiki/Recurrent_neural_network) which incorporates a temporal aspect of the information that trains the model. Basically, past information will impact what the model is learning now. 

In this way we will essentially *teach* English to the analytical model. At least it will be the english as derived from the writings of [Friedrich Nietzsche](http://www.gutenberg.org/ebooks/author/779).

The following article, [Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/) is a great description of LSTM networks in more depth. 

I think a key highlight from the article is that LSTMs are a special type of neural network which allow for information to be retained...and forgotten...as needed.

>Humans don’t start their thinking from scratch every second. As you read this essay, you understand each word based on your understanding of previous words. You don’t throw everything away and start thinking from scratch again. Your thoughts have persistence.

>Recurrent neural networks address this issue. They are networks with loops in them, allowing information to persist.

Ok...so now we have a very basic idea of what an LSTM is….


## What are we going to do?

Well let’s walk through the steps of building a generative model using LSTM, just like in the book; however

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Let's **actually** build a web app to generate text in real time*

We are going to use Python, Keras and Google Colaboratory to build the code. Furthermore we will leverage Google Cloud Platform, Kubernetes, Cloud Functions, Stackdriver, Computer Engine, and some other tools to build the application.

## Why?

Simple. To learn. What started as a experiment to learn how to build an analytical model expanded into a project involving all the steps to implement an application. 

My hope with this set of posts is that someone would find them useful to learn just as I did. 

## Who is the audience?

The ideal audience to this set of blogs would be someone who has had some experience with python, machine learning and Google Cloud Platform and likewise, wants to learn some steps along the way.

So if you are ready…let’s dive right in…

# Start the Build



#### Where to build?
I wanted an environment that worked from me. I wanted to satisfy these three requirements:

1. Use a “widely” regarded development framework
2. Leverage a “cloud” development environment
3. Save code in an online repository

Using Google’s [Colaboratory](https://colab.research.google.com/) satisfies all three of these requirements. 
It uses  
[Jupyter](https://jupyter.org/) notebooks, which are ubiquitess in the analytics community,
hosted on Google's infrastructure so it can be used from anywhere and it allows you to save your code in either Google Drive or Github. 


When you open up the Colab environment the default screen contains a “Getting Started” section that will walk 
you through many introductory steps to perform. 

<img src="{{site.baseurl}}/assets/images/welcome_colaboratory.png" alt="Welcome to Google Colaboratory" width="800"/>

For many individuals who have used Jupyter notebooks in the past this will be very familiar.

This this introduction we will just load one of the jupyter notebooks that Francois has made available in his accompanying Github account.

## Load a Notebook from GitHub

Within Colab if you are looking to leverage a jupyter notebook directly from Github you can enter a url for the notebook in question with a colab preference and it will lode directly.

For example - we want to load the following notebook in colab: 8.1-text-generation-with-lstm.ipynb 

We can simply go to the following url:  http://colab.research.google.com/github 
This opens a prompt box allowing you to paste the address of the specific notebook hosted on github
(In this picture I have simply pasted the link: 
https://github.com/fchollet/deep-learning-with-python-notebooks/blob/master/8.1-text-generation-with-lstm.ipynb )


<img src="{{site.baseurl}}/assets/images/dlwp_colab_to_github_01.png" alt="Connect Google Colaboratory to Github" width="800"/>

When you click off of the entry blank the colab interogates the inforamtion and loads the informaitotn pertaining to the repository:

<img src="{{site.baseurl}}/assets/images/dlwp_colab_to_github_02.png" alt="Connect Google Colaboratory to Github" width="800"/>

Simply click the “Open notebook in new tab” button to start editing the code:

<img src="{{site.baseurl}}/assets/images/dlwp_colab_to_github_03.png" alt="Connect Google Colaboratory to Github" width="800"/>

The notebook will open:

<img src="{{site.baseurl}}/assets/images/dlwp_notebook_01.png" alt="Deep Learning with Python Notebook" width="800"/>

Save your notebook to Github:

<img src="{{site.baseurl}}/assets/images/text_gen_lstm_save_to_github.png" alt="Save Google Colaboratory Notebook to Github" width="800"/>

If this is your first time saving from Colab to Github you may need to authorize your

<img src="{{site.baseurl}}/assets/images/authorize_colaboratory.png" alt="Authorize Colaboratory to Github" width="800"/>

{% highlight shell %} 
git clone https://github.com/fchollet/deep-learning-with-python-notebooks.git
cp deep-learning-with-python-notebooks/8.1-text-generation-with-lstm.ipynb text_gen_lstm/learn/
cd text_gen_lstm
git status
git add 8.1-text-generation-with-lstm.ipynb
git status
git commit -m "added jupyter notebook from fchollet/deep-learning-with-python-notebooks"
{% endhighlight %}

After the notebook has been copied to your github repository we can head back over to colaboratory and start up the notebook:

<img src="{{site.baseurl}}/assets/images/text_gen_lstm_colab_to_github_01.png" alt="Connect Google Colaboratory to Github" width="800"/>

Click on the name in the middle of the pop-up screen

<img src="{{site.baseurl}}/assets/images/text_gen_lstm_colab_to_github_warning.png" alt="Warning that notebook was not authored by Google" width="800"/>

Click the “Run Anyway” option

<img src="{{site.baseurl}}/assets/images/text_gen_lstm_reset_all_runtimes.png" alt="Reset all runtimes" width="800"/>

Click “Yes”


Within the notebook you can select each cell and use the key-stroke “Shift+Enter” to execute each cell.

<img src="{{site.baseurl}}/assets/images/dlwp_notebook_01.png" alt="Execution of a cell in Colaboratory notebook" width="800"/>

Execute each of the cells until:

<img src="{{site.baseurl}}/assets/images/notebook_core_logic.png" alt="Notebook core logic" width="800"/>

This is the core of the logic. While the code will work correctly it will run for quite some time. 
So before running change the epoch from range(1, 60) to range(1,2)

Then it will start running 2 epochs. 
This is enough to give us comfort that we have succeeded in setting 
up our repository and have executed the setup steps successfully.

Hopefully this has given you a little introduction of 

I have simply walked through a few of the basics of using the colaboratory / jupyter notebook.
However the following notebook has a significant amount of examples that can be executed to learn additional functionality.
[Colaboratory Github Demo](https://colab.research.google.com/github/googlecolab/colabtools/blob/master/notebooks/colab-github-demo.ipynb)


[DLwP_Book]: https://www.manning.com/books/deep-learning-with-python
[DLwP_github]: https://github.com/fchollet/deep-learning-with-python-notebooks
