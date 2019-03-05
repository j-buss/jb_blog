---
layout: post
title:  "Introduction"
date:   2019-03-01 18:49:37 -0600
permalink: /:categories/:title.html
categories: text-gen-lstm
---
Let’s make a robo-Nietzsche...you know some electronic embodiment of Friedrich Nietzsche...a an electronic doppleganger of sorts...

<p float="left">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Nietzsche187a.jpg/256px-Nietzsche187a.jpg" style="width: 48%; margin-right: 1%; margin-bottom: 0.5em;" title="Friedrich Hartmann [Public domain], via Wikimedia Commons">
<img src="{{ site.baseurl }}/assets/images/robot.png" style="width: 48%; margin-right: 1%; margin-bottom: 0.5em;" title="Bilboq [Public domain], via Wikimedia Commons">
</p>

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

<img src="{{site.baseurl}}/assets/images/build_train.jpg" alt="build and train" width="400"/>

However I wanted something more like the this:

<img src="{{site.baseurl}}/assets/images/build_train_serve_interact.jpg" alt="build , train, serve and interact" width="400"/>

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*This would allow me to not only **make** a working model, but to **use** it as well*

So with this set of posts we will explore the build and implementation of a machine learning model. 
With the intent of using and learning about many tools along the way: Google Colaboratory, Google Cloud Platform, Kubernetes, Cloud Functions, Stackdriver, Computer Engine, Cloud ML, etc. 

So if you are ready...let's dive right in...

### Start the Build

As with the other chapters of the book I wrote all the code by hand to help with the acquisition. 
My requirements for this code was:

1. Use a “widely” regarded development framework
2. Save code in an online repository
3. Leverage a “cloud” development environment

Using Google’s [Colaboratory](https://colab.research.google.com/) satisfies all of these requirements. 
It is completely free, it leverages [Jupyter](https://jupyter.org/) notebooks and stores the code in your Google Drive or 
can easily be saved in GitHub.

When you open up the Colab environment the default screen contains a “Getting Started” section that will walk 
you through many introductory steps to perform. 

<img src="{{site.baseurl}}/assets/images/welcome_colaboratory.png" alt="Welcome to Google Colaboratory" width="800"/>

However for many individuals that may have used Jupyter notebooks in the past this will be very familiar.

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

<img src="{{site.baseurl}}/assets/images/dlwp_notebook_01.png" alt="Connect Google Colaboratory to Github" width="800"/>

Save your notebook to Github:

<img src="{{site.baseurl}}/assets/images/text_gen_lstm_save_to_github.png" alt="Connect Google Colaboratory to Github" width="800"/>

If this is your first time saving from Colab to Github you may need to authorize your

<img src="{{site.baseurl}}/assets/images/authorize_colaboratory.png" alt="Connect Google Colaboratory to Github" width="800"/>

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

<img src="{{site.baseurl}}/assets/images/text_gen_lstm_colab_to_github_warning.png" alt="Connect Google Colaboratory to Github" width="800"/>

Click the “Run Anyway” option

<img src="{{site.baseurl}}/assets/images/text_gen_lstm_reset_all_runtimes.png" alt="Connect Google Colaboratory to Github" width="800"/>

Click “Yes”


Within the notebook you can select each cell and use the key-stroke “Shift+Enter” to execute each cell.

<img src="{{site.baseurl}}/assets/images/dlwp_notebook_01.png" alt="Connect Google Colaboratory to Github" width="800"/>

Execute each of the cells until:

<img src="{{site.baseurl}}/assets/images/notebook_core_logic.png" alt="Connect Google Colaboratory to Github" width="800"/>

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
