---
title: "Simple Flask Example"
date: 2019-12-13T08:00:00-00:00
draft: false
---

# Deploying Flask Apps into Google Cloud Platform

There are many Flask tutorials online that describe the details of using the lightweight Python framework. With this series I do not intend to recreate the exhaustive tutorials that have been done. I simply wanted a quick 3 post series to get up and running with deploying Flask to
GCP App Engine and using data from Big Query.

## Part 1: Creating Your First Flask App

Let's take a simple Flask app and deploy it to GCP. Essentially we will take the "5-line minimum" and add a few other items in order to deploy it to GCP App Engine.

### 1. Minimum app tested locally:
Install Flask:
```bash
pip install flask
```
Create the "5-line minimum" main.py file:  
```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"
```
This file is the only code needed. 
flask is a script...so we will execute it with a simple `flask run` command. 
However we need to set an environment variable first.
 ```bash
export FLASK_APP=main.py
```
Now we are ready to run it:
```bash
flask run
 ```
If all went well we should get a output like the following:

{{< img src="simple_flask_run_locally.png" alt="Simple Flask running locally" >}}

### 2. Run the app in GCP:

With a working local copy from the last section we need to make a few changes so that it can run on GCP. 

#### Rename app.py to main.py

GCP expects the flask app to be called main.py. Without a module named "main" it will error.

```bash
mv app.py main.py
```
 
#### Modify the code slightly

We need to add a main block which will be the entry point when the app is executed from GCP. Additionally, we are setting the port number to be used for the webserver to 8080 for testing.

```python {hl_lines=["7-8"]}
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"
if __name__ == '__main__':
    app.run(port=8080)
``` 

#### app.yaml

We also need to make one more file for GCP App Engine to know how to load the application we just made. The [app.yaml](https://cloud.google.com/appengine/docs/standard/python3/config/appref) file will contain the language and that we would like to use [gunicorn](https://gunicorn.org/) as our WSGI. 

```python
runtime: python37
entrypoint: gunicorn -b :$PORT main:app
```
#### Deploy

To deploy the App Engine application we will use the GCP SDK from the command line. As such we need to have an active project.

```bash
gcloud config set project [PROJECT_NAME]
```
enable cloud build api
```bash
gcloud app deploy
```
If it went well we should receive the following...
#### INSERT PICTURE AND RETURN MESSAGE
### Resources:
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
