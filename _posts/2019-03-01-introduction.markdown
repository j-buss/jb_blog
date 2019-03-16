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

## Introduction

This all started when I wanted to learn about deep learning. I was following along with the exercises in the book [Deep Learning with Python by François Chollet][DLwP_Book]. 

When I got to Chapter 8 - Generative deep learning François had a great example of creating text with a model after being trained on a corpus of text.
It was fascinating working through all the code. However I wanted a way to create something more than just reproducing code. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*I wanted to make an **application**...something I could interact with*

So that was how my journey began to build a text generating application using an LSTM model.

### What is an LSTM model?

A long short-term memory ([LSTM](https://en.wikipedia.org/wiki/Recurrent_neural_network)) model is a [recurrent neural network](https://en.wikipedia.org/wiki/Recurrent_neural_network) which incorporates a temporal aspect of the information that trains the model. Basically, past information will impact what the model is learning now. 

In this way we will essentially *teach* English to the analytical model. At least it will be the english as derived from the writings of [Friedrich Nietzsche](http://www.gutenberg.org/ebooks/author/779).

Basically an LSTM is a way of training a neural network to retain information...and forget it...as needed

>Humans don’t start their thinking from scratch every second. As you read this essay, you understand each word based on your understanding of previous words. You don’t throw everything away and start thinking from scratch again. Your thoughts have persistence.

>Recurrent neural networks address this issue. They are networks with loops in them, allowing information to persist.
If you would like a much deeper understanding of LSTM please the following article: [Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/) is a great description of LSTM networks in more depth. 

Ok...so now we have a very basic idea of what an LSTM is….

### What are we going to do?

Well let’s walk through the steps of building a generative model using LSTM, just like in the book; however

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Let's **actually** build a web app to generate text in real time*

We are going to use Python, Keras and Google Colaboratory to build the code. Furthermore we will leverage Google Cloud Platform, Kubernetes, Cloud Functions, Stackdriver, Compute Engine, and some other tools to build the application.

### Why?

Simple. To learn. What started as a experiment to learn how to build an analytical model expanded into a project involving all the steps to implement an application. 

My hope with this set of posts is that someone would find them useful to learn just as I did. 

### Who is the audience?

The ideal audience to this set of blogs would be someone who has had some experience with python, machine learning and Google Cloud Platform and likewise, wants to learn some steps along the way.

So if you are ready…let’s dive right in…

## Start the Build

Early in this project I wanted to use a development environment that satisfied the following three requirements:
1. Use a well-known development framework
2. Cloud only environment
3. Save the code in an online repository

Using Google’s [Colaboratory](https://colab.research.google.com/) satisfies all three of these requirements. 
It uses  
[Jupyter](https://jupyter.org/) notebooks, which are ubiquitess in the analytics community.
It is hosted on Google's infrastructure so it can be used from anywhere and it allows you to save your code in either Google Drive or Github. 

As the [Colab FAQ](https://research.google.com/colaboratory/faq.html) describes:
>Colaboratory is a research tool for machine learning education and research. It’s a Jupyter notebook environment that requires no setup to use.

### Colaboratory

When you open up the colab environment for the first time you will likely come to the "Welcome to Colaboratory" screen.

{% include image.html file="welcome_to_colaboratory.png" description="Welcome to Colaboratory! Notebook" %}

For these tutorials we will only describe the most basic of the functionality for colaboratory for code execution.
If you have not used Jupyter notebooks before feel free to walk through the introductory notebook as it will teach you many aspects of the environment.
There are two main sections to the Colab window.
1. Side Panel
2. Code Panel

The **Side Panel** contains a table of contents, example code snippets and a file explorer. While we will not have much use of these tools for the current tutorial there are some really powerful components to be found there.

{% include image.html file="welcome_to_colaboratory_side_panel.png" description="Google Colaboratory Side Panel" %}

The **Code Panel** is the area of the screen which we will spend most of our time. This area allows us to write and execute pieces of code.

{% include image.html file="welcome_to_colaboratory_code_panel.png" description="Google Colaboratory Code Panel" %}

Now that you have the most basic aspects of the Colaboratory...let's do something use and *load* an existing notebook.

### Open a Notebook from GitHub

Within Colab we are going to open a notebook that François Chollet made available in the Github repo accompanying the book: [deep-learning-with-python-notebooks](https://github.com/fchollet/deep-learning-with-python-notebooks).

Specifically we will be loading the notebook for section [8.1 Text generation with LSTM](https://github.com/fchollet/deep-learning-with-python-notebooks/blob/master/8.1-text-generation-with-lstm.ipynb)

{% include image.html file="dlwp_github_8_1_notebook.png" %}

Colab gives us a great way to load a notebook from github. Within colab select “File”:

{% include image.html file="colab_file.png" description="Notebook file" %}

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
