---
title: "Plotly Introduction with Docker on GCP"
date: 2020-01-02T16:27:02-06:00
draft: false
---

One of the keys to data analysis is a flexible tool to explore data and/or share those results with others. Within a corporate setting this usually translates to leveraging Microsoft's [Power BI](https://powerbi.microsoft.com/en-us/) or [Tableau](https://www.tableau.com/). I have used both of these tools and enjoy them. However, for my independent analysis I was looking for a tool that was open source as well as fully programmable/integrated with python.

I stumbled across [Plotly Dash](https://plot.ly/dash/) and it satisfied both of these requirements. This post will be focused on some simple visualizations and specifically hosting them in Google Cloud Platform. While we could do this a few different ways I am choosing to create docker images and load in [App Engine Flex](https://cloud.google.com/appengine/docs/flexible/).

We will build up with 3 steps:

1. Simple Local
2. Stand-up a Compute Engine to build Docker
3. Build Docker Image and Deploy to GCP App Engine

#### Assumptions:
- Familiarity with Python
- Familiarity with Google Cloud Platform
- Access to a GCP Project with ability to deploy App Engine
- Linux working environment 
  - This tutorial leverages GCP Cloud Shell
  - Steps are not dependent on Linux, but haven't been tested in other environments
  
#### Disclaimer:
- The intent of this post is for learning to get the tool up and running and to share my experience. 
- If you follow along there will be a cost for the cloud resources. However if you build it and then delete the project the cost will be minimal (< $2 USD). 
- In a future post we may examine how we could optimize the cost of the solution.

# 1. Simple Local

This will be our first step and will serve as the basis for creating a plotly dash visualization. It is going to leverage the first app from Chapter 2 of the [Dash Tutorial](https://dash.plot.ly/getting-started) and serve it up locally. We will just be adding a few lines of code to change the solution a bit and have it be part of a flask app. 

The solution consists of just two files:
- main.py
- requirements.txt

## main.py

The changes to main are focused on loading the flask app library, starting the flask server and changing the default port to serve the content.

```python {linenos=inline,hl_lines=["1","6","10-14","38"]}
import flask
import dash
import dash_core_components as dcc
import dash_html_components as html

server = flask.Flask(__name__)

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(
        __name__, 
        server=server,
        external_stylesheets=external_stylesheets
)

app.layout = html.Div(children=[
    html.H1(children='Hello Dash'),

    html.Div(children='''
        Dash: A web application framework for Python.
    '''),

    dcc.Graph(
        id='example-graph',
        figure={
            'data': [
                {'x': [1, 2, 3], 'y': [4, 1, 2], 'type': 'bar', 'name': 'SF'},
                {'x': [1, 2, 3], 'y': [2, 4, 5], 'type': 'bar', 'name': u'Montréal'},
            ],
            'layout': {
                'title': 'Dash Data Visualization'
            }
        }
    )
])

if __name__ == '__main__':
    app.run_server(port=8080, debug=True)
```

## requirements.txt 

Our requirements.txt file is pretty straightforward as the Dash library tends to load all the associated libraries that are needed.

```text
dash==1.7.0
```

## Test It

When we have those files in our environment we can test to ensure the solution works:

```shell script
pip3 install -r requirements.txt
export FLASK_APP=main.y
flask run
```

With any luck we should see output with a URL:

{{< img src="flask_run_output.png" >}}

Clicking on the url we should see the following:

{{< img src="dash_simple_local_output.png" >}}

I know it's super simple, but always good to get the application to load properly. Now let's set our sights to moving our app to the cloud.

# 2. Create Compute Engine

Ok. There are many many ways that one could build our app. With this post we are going to create a docker image. This will give us extreme flexibility for hosting this app just about anywhere.

You could build your docker image just about anywhere. However I will stand up a compute engine in the cloud in order to build the Docker image. I agree this is overkill; however I was doing this at a time that I was remote and needed a way to build and push the image to GCP without relying on my phone as a hotspot. 

## Create Compute Engine
I am assuming you have some familiarity with setting up GCP components. The following are the notable selections I made in setting up the server in the cloud:

<style>
table, th, td {
      padding: 10px;
      border: 1px solid black; 
      border-collapse: collapse;
}
th{
    background-color: grey;
    color: white;
}
</style>

Field|Value
:-----:|:-----:
Name|docker-build-vm-001
Region|us-central1
Zone|us-central1-a
Machine Type|n1-standard-1
Access Scopes|Allow full access to all Cloud APIs

Once the server is up and running you can log onto it (via ssh) and start executing the following commands to install Git and Docker. 

### Install Git

To install git is quite straightforward on the compute engine:

```shell script
sudo apt-get update
sudo apt-get install git
```

Answer "Y" to the prompts of requiring additional space for the install.

### Install Docker
We will use the following steps as the basis of our work: [Install Docker on Debian](https://docs.docker.com/install/linux/docker-ce/debian/).

#### Install packages allowing apt over HTTPS:
```shell script
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
```
#### Add Docker's official GPG Key:

```shell script
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```

#### Verify the key

```shell script
sudo apt-key fingerprint 0EBFCD88
```

The output should resemble the following:

{{< img src="verify_docker_key_output.png" >}}

#### Set up Stable Repository

```shell script
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```
#### Update the apt package index

```shell script
sudo apt-get update
```

#### Install latest version of Docker Engine - Community and containerd 

```shell script
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
#### Run Hello-World to verify Install
```shell script
sudo docker run hello-world
```

If the install went successfully you will receive the following message:

{{< img src="docker_hello_world_message.png" >}}

# 3. Build Image and Deploy to GCP App Engine

Now that we have a compute engine we can log on and build out the Docker image. We will accomplish this in a few small steps.

## 1. Download the Github Repo

Download the code needed to follow along and navigate to the directory in question.

```shell script
git clone http://github.com/j-buss/10-minute-tech-tutorial/
cd 10-minute-tech-tutorial/python/plotly-intro-gcp-docker/02_Simple_Docker_Cloud
```

## 2. Build the image

The following commands will build a docker image. We will add a few environment variables to make typing of the commands a bit easier.

```shell script
# Set some environment variables to facilitate ease of deploy
PROJECT_ID=my-plotly-tutorial-001
APP_NAME=02-simple-docker-cloud
# Build the docker image
sudo docker build -t gcr.io/$PROJECT_ID/$APP_NAME:v1 .
```
When the image is built you can view the image 
```shell script
sudo docker images
```
{{< img src="list_docker_images.png">}}

### Push Image to Container Registry

Now that the image exists locally (relatively speaking since we are on a cloud compute engine) we can push it to a container registry. In this case we will use the GCP Container Registry.

```shell script
# Authorize and push to GCP Container Registry
sudo gcloud auth configure-docker
sudo docker push gcr.io/$PROJECT_ID/$APP_NAME:v1
```

If this is successful we should be able to see the same image in the Container Registry:

{{< img src="docker_image_in_container_registry.png" >}}

### Deploy Application in GCP App Engine

Now we have all the pieces in place to actually deploy the application to the cloud.
```shell script
# Deploy the application in GCP App Engine
sudo gcloud app deploy --image-url gcr.io/$PROJECT_ID/$APP_NAME:v1
```

### Test It

If the push to GCP App Engine is successful you can log on and test it. The link can be found on the GCP Console (#2 in the image). However make certain to change to the correct service (#1 in the image):

{{< img src="GCP_App_Engine_URL.png" >}}

The app should load the same as it did locally. With the exception being a different URL.

{{< img src="GCP_App_Engine_test_dashboard.png" >}}

## Delete the Project

After you have finished with following along make certain to delete your project so that you will not run up additional costs.

---

## Summary 

We have built a simple Plotly Dash App, bundled it into a Docker Image and hosted it on GCP. We are poised to take on much more complex work!

---

### Resources:
#### Code
# UPDATE WITH MASTER BRANCH LINK AFTER FINAL COMMIT
- All the code can be found in my github: [10-minute-tech-tutorials/python/plotly-intro](https://github.com/j-buss/10-minute-tech-tutorials/tree/dev-plotly-01/python/plotly-intro) 
#### Tutorials
- [Dash Tutorial](https://dash.plot.ly/) The official tutorial released by Plotly. It is a great introduction to the library.

{{< code-clipboard >}}
