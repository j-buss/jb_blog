---
title: "Flask Part 2 - Reading BigQuery"
date: 2019-12-21T05:54:03-06:00
draft: true
---

# Deploying Flask Apps into Google Cloud Platform

This is a continuation from the previous post [Flask Part 1 - Simple App]({{< ref "/posts/flask/Part_01/index.md" >}}). In this post we will extend the simple Flask App and read some data from a public dataset in BigQuery.

#### Assumptions:
- Familiarity with Python
- Familiarity with Google Cloud Platform
- Access to a GCP Project with ability to deploy App Engine
- Linux working environment 
  - This tutorial leverages GCP Cloud Shell
  - Steps are not dependent on Linux, but haven't been tested in other environments
- Also, I am assuming you have followed along with the previous post: [Flask Part 1 - Simple App]({{< ref "/posts/flask/Part_01/index.md" >}})

## Part 2: Read BigQuery

In order to read data from BigQuery we need to:
1. Load BigQuery Libraries
2. Edit App to __Read__ BigQuery
3. Edit App to __Display__ Data

Just like last post we will take this in steps. We will test it locally and then in GCP.

#### 1. Load BigQuery Libraries

In order to use BigQuery we will need to install the Python BigQuery library into our environment:

```shell script
pip install google-cloud-bigquery
```

Well...that was easy. Unfortunately the next two steps are much more involved.

### 2. Edit App to Read BigQuery

Now let's focus on a relatively simply query from a public BigQuery database. 

Update the requirements.txt file

```shell script
Flask==1.1.1
google-cloud-bigquery==1.22.0
```
### 3. Test it on GCP



---

### Resources:
#### Code
- All the code can be found in my github: 
    - https://github.com/j-buss/example_flask
#### Tutorials
- [Flask Mega Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)
 : The Flask Tutorial Gold Standard. In a clear and consistent format it walks you through all the basics and well into the intermediate range.
 Not only that, but the tutorial is a delight to work through. 
 The author Miguel Grinberg writes in a way that it is a pleasure to follow along. 
 His love of Flask and Python are evident.
- [Your First Flask App](https://hackersandslackers.com/your-first-flask-application/) is a series of posts which describe many facets of Flask.
The series includes blueprints, Jinja and SQLAlchemy. It is not intended to be a step-by-step tutorial, but instead describes the concepts in a very 
conversational way.
- [Deploying a Python Flask Web Application to App Engine Flexible](https://codelabs.developers.google.com/codelabs/cloud-vision-app-engine/index.html?index=..%2F..index#0)
is a Google Cloud Codelabs walk-through describing each of the steps to deploy a vision app from Google's Github repository: 
[python-docs-sample](https://github.com/GoogleCloudPlatform/python-docs-samples) 

#### Podcast:
- [Building Flask-based Web Apps](https://talkpython.fm/episodes/show/48/building-flask-based-web-apps) is a podcast interview on [Talk Python to Me]()

{{< code-clipboard >}}
