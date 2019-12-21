---
title: "Flask Part 1 - Simple App"
date: 2019-12-13T08:00:00-00:00
draft: false
---

# Deploying Flask Apps into Google Cloud Platform

There are many Flask tutorials online that describe the details of using the lightweight Python framework. With this series I do not intend to recreate the exhaustive tutorials that have been done. I simply wanted a quick 3-post series to get up and running with deploying Flask to
GCP App Engine and using data from Big Query.

Within this post, Part 1, we will create a Flask app locally and then push it to GCP App Engine.

Given that this post pertains to very simple code you can easily type it all. However, feel free to get the code from my github: [example_flask - Part 1](https://github.com/j-buss/example_flask/tree/master/Part_01).

#### Assumptions:
- Familiarity with Python
- Familiarity with Google Cloud Platform
- Access to a GCP Project with ability to deploy App Engine
- Linux working environment 
  - This tutorial leverages GCP Cloud Shell
  - Steps are not dependent on Linux, but haven't been tested in other environments

## Part 1: Simple App

Let's take a simple Flask app and deploy it to GCP. Essentially we will take the "5-line minimum" and add a few other items in order to deploy it to GCP App Engine.

### 1. Minimum app tested locally:
Install Flask:
```bash
pip install flask
```
Create the "5-line minimum" main_local.py file:  
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
export FLASK_APP=main_local.py
```
Now we are ready to run it:
```bash
flask run
 ```
If all went well we should get a output like the following:

{{< img src="simple_flask_run_locally.png" alt="Simple Flask running locally" >}}

### 2. Run the app in GCP:

With a working local copy from the last section we need to make a few changes so that it can run on GCP. 

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

#### Add Two More Files
We also need to make two more file for GCP App Engine to know how to load the application we just made: 
1. app.yaml
1. requirements.txt

##### 1. app.yaml

There are many options you can add into the app.yaml file. We will simply be telling GCP what language we are using and the WSGI tool.
We will be using the [gunicorn](https://gunicorn.org/) tool as our WSGI. Additional information about the app.yaml can be found at the following: 
 https://cloud.google.com/appengine/docs/standard/python3/config/appref

```python
runtime: python37
entrypoint: gunicorn -b :$PORT main:app
```

 
##### 2. requirements.txt

The requirements file will contain the Python libraries that need to be loaded to be used in our application. We are using only two:

```python
Flask==1.1.1
gunicorn
```

#### Deploy

To deploy the App Engine application we will use the GCP SDK from the command line. As such we need to have an active project.

```bash
gcloud config set project [PROJECT_NAME]
```

Now it is time to perform the deployment:

```bash
gcloud app deploy
```
You will receive a few prompts to finish the deployment to App Engine.

##### Select Region:

{{< img src="app_engine_select_region.png" alt="Select Region" >}}

##### Confirmation

{{< img src="confirm_app_engine_deploy.png" alt="Simple Flask running locally" >}} 

##### Status Message

{{< img src="app_engine_deploy_summary.png" alt="App Engine Deployment Summary Message" >}}

#### Test the App

Go Into the GCP Console and select the link to the default application:

{{< img src="app_engine_console.png" alt="App Engine Console" >}}

If all goes well you should see the following:

{{< img src="flask_app_running_app_engine.png" alt="Simple Flask App Running on GCP App Engine" >}}

### Next Steps:

Ok that was a pretty easy one. In the next post we will look using our flask app to pull data from BigQuery!

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
