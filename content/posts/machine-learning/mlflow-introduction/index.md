---
title: "MLflow on Google Cloud Platform"
date: 2019-12-22T10:25:54-06:00
draft: false
---

With this post we will setup MLflow and experiment with the machine learning workflow. Specifically we will install MLflow on a compute engine on GCP.

#### Assumptions:
- Familiarity with Python, and scikit-learn
- Access to a linux system 
    - While the steps below don't necessarily require linux they were performed on linus so I can't speak to the ability to perform on Windows or Mac OS
- Google Cloud Platform familiarity and access to a GCP project

## Part 1: Install MLFlow

[MLflow](http://mlflow/) is an "open source platform for the machine learning lifecycle". There is a great introduction post [Introducing MLflow: an Open Source Machine Learning Platform](https://databricks.com/blog/2018/06/05/introducing-mlflow-an-open-source-machine-learning-platform.html) which highlights the 4 main challenges that the mlflow platform intends to solve with regard to ML Lifecycles:
1. There are a myriad of tools
2. It's hard to track experiments
3. It's hard to reproduce results
4. It's hard to deploy ML

In this post we won't specifically highlight the individual challenges, but simply install it and use a few example ml projects for learning.

### 1. Install MLFlow on a Compute Engine

#### Create Compute Engine
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
Name|mlflow-test-vm-01
Region|us-central1
Zone|us-central1-a
Machine Type|n1-standard-1
Network tags|mlflow-ui

Once the server is up and running you can log onto it (via ssh) and start executing the following commands. We are using the steps outlined in [MLFlow Tutorial](https://www.mlflow.org/docs/latest/tutorials-and-examples/tutorial.html) as the basis of our work, but with the specifics needed for a new cloud instance.

#### Install Conda

I tend to prefer a minimal installation so I went for the [miniconda](https://docs.conda.io/en/latest/miniconda.html). In the following command I am using the 64 bit installer for Python 3.

```shell script
# Download latest miniconda installer
curl -O -J https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
# Verify Checksums | compare with the value on the 
sha256sum Miniconda*
```

Here is the result of executing the command in Dec. 2019. The results will likely be different if you run the command and a newer version has been released with a different checksum result.

{{< img src="miniconda-checksum-results.png" >}}

Here is the value from the website and we see that they match:

{{< img src="miniconda-website-checksum.png" >}}

When you are comfortable that the download is valid you can start the install of miniconda:

```shell script
bash Miniconda3-latest-Linux-x86_64.sh
```
Along the way you can feel free to accept the defaults.

The last option to initialize the system with `conda init` will modify your .bashrc scripts to include the miniconda directory in your path. On a new server that is not your primary workstation I would answer yes.

However, as a point of personal preference if I am running these commands on my local/main workstation I tend to prefer to activate the conda shell manually so that I am clear when conda has been activated with the following command: `conda activate`. As such I would answer *no* to the previous `conda init` command. Additionally I would tell conda not to auto-activate with the following:

```shell script
conda config --set auto_activate_base false
```

#### Activate Conda

Assuming you had the installer execute the conda init command you can restart your .bashrc

```shell script
source .bashrc
```
#### Add the Code

The following steps can be done manually as the amount of information is very minimal. However you can also get the code from my github repository: [https://github.com/j-buss/10-minute-tech-tutorials/tree/master/machine_learning](https://github.com/j-buss/10-minute-tech-tutorials/tree/master/machine_learning)

Let's add two directories: 

```shell script
mkdir mlflow-tutorial
mkdir mlflow-tutorial/sklearn-iris-classification
```
A brief explanation of each:

1. mlflow-tutorial: The folder for all of our content. We will run our `mlflow` commands from this folder so in addition to the folder we will explicitly add in the next step, the fmlflow framework will add some folders and files as well
 2. mlflow-tutorial/sklearn-iris-classification: A directory for the three code files: conda.yaml, MLproject and train.py

##### conda.yaml

The conda.yaml file will ensure conda loads the appropriate libraries and dependencies for our mlflow execution.
```yaml
name: sklearn-example
channels:
  - defaults
  - anaconda
dependencies:
  - python==3.6
  - scikit-learn=0.19.1
  - pip=19.3.1
  - pip:
    - mlflow>=1.0
```
##### MLproject

The MLproject file is a MLFlow configuration file that each project needs to identify the specific components of the model.

```yaml
name: sklearn_logistic_example

conda_env: conda.yaml

entry_points:
  main:
    command: "python train.py"
```
##### train.py 

The train.py is the "normal" machine learning program file we would consider to be the basis of the ml model.

```python
from sklearn import datasets
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
import mlflow
import mlflow.sklearn

if __name__ == "__main__":
    iris = datasets.load_iris()
    X = iris.data
    Y = iris.target
    X_train, X_test, y_train, y_test = train_test_split(X, Y, test_size=0.33, random_state=44)
    logreg = LogisticRegression()
    logreg.fit(X_train, y_train)
    y_pred = logreg.predict(X_test)
    score = accuracy_score(y_test, y_pred)
    print("Score: %s" % score)
    mlflow.log_metric("score", score)
    mlflow.sklearn.log_model(logreg, "model")
    print("Model saved in run %s" % mlflow.active_run().info.run_uuid)
```
When you are done you should have a directory structure looking like the following:

{{< img src="mlflow_directory_structure.png" >}}

#### Install git

If you don't already have git install you will need to perform the following:

```shell script
sudo apt-get update
sudo apt-get install git
```
#### Install mlflow

Ensure that you have your conda shell activated either by default or by manually typing `conda activate`. Your shell should have a "(base)" prefix like this:

{{< img src="conda_shell_activated.png" >}}"

Then you are ready to actually install the mlflow package with extras:

```shell script
pip install mlflow[extras]
```

### 2. Run Iris Classification

Ok. Now we are ready to run Iris Classification model through the mlflow framework.

#### Train the ML Model
At the command line within the mlflow-tutorial directory run the following command

```shell script
mlflow run sklearn_logistic_regression
```

This will run the logistic regression. However, as part of the first step it will likely download the packages required from within the conda.yaml file. This is the benefit of the conda framework as it creates an isolated environment for your code execution.

The output of the model run will be stored in a new directory called "mlruns". While you could look at the output manually mlflow has a UI allowing us to review the results. But we need one additional step to open the firewall port before we can examine the results.

#### Open a Firewall port
Within the web interface for GCP go into the 
VPC Network->Firewall and create a firewall rule for our server. We will leverage the network tag "mlflow-ui" that we used when we created the server. Here are the selections for the firewall rule:

Field|Value
:-----:|:-----:
Name|mlflow-ui-allow
Network|default
Priority|1000
Direction of traffic|Ingress
Action on match|Allow
Targets|Specified target tags
Target tags|mlflow-ui
Source filter|IP ranges
Source IP ranges|0.0.0.0/0
Specified protocols and ports|tcp:8080

Click the "Create" button and we should be all set.

#### View the mlflow UI results
Within our directory "mlflow-tutorial" start up the mlflow UI server by executing the following:

```shell script
mlflow ui -h 0.0.0.0 -p 8080
```

You should receive some output like the following:

{{< img src="start-mlflow-ui.png"  >}}

Then in a browser you can navigate to the external ip address of the server with the addition of the port number on the end (in our case 8080) to see the results of our trained model. You will see one result for each of the training runs as I ran it 2x.

{{< img src="mlflow-ui.pn">}}

### 3. Serve the Model

We already saved the model

INSERT PICTURE OF Model Pkl from UI

#### Serve

```shell script
mlflow models serve -m runs:/6381286aefcb48c6b2db16297503c58c/model -p 8080 -h 0.0.0.0
```

### Next Steps:


---

### Resources:
#### Code
- All the code can be found in my github: 
    - https://github.com/
- MLFlow [Quickstart](https://www.mlflow.org/docs/latest/quickstart.html) 
- MLFlow [Tutorial](https://www.mlflow.org/docs/latest/tutorials-and-examples/tutorial.html)

{{< code-clipboard >}}
