---
title: "MLflow on Google Cloud Platform"
date: 2019-12-22T10:25:54-06:00
draft: false
---

With this post we will setup MLflow and experiment with the machine learning workflow. Specifically we will install MLflow on a compute engine in GCP.

#### Assumptions:
- Familiarity with Python, and scikit-learn
- Access to a linux system 
    - While the steps below don't necessarily require linux they were performed on linux so I can't speak to the ability to perform on Windows or Mac OS
- Google Cloud Platform familiarity and access to a GCP project

## Part 1: Install MLFlow

[MLflow](http://mlflow.org) is an "open source platform for the machine learning lifecycle". There is a great introduction post [Introducing MLflow: an Open Source Machine Learning Platform](https://databricks.com/blog/2018/06/05/introducing-mlflow-an-open-source-machine-learning-platform.html) which highlights the 4 main challenges that the mlflow platform intends to solve for the machine learning lifecycle:
1. There are a myriad of tools
2. It's hard to track experiments
3. It's hard to reproduce results
4. It's hard to deploy ML

In this post we won't specifically highlight the individual challenges, but simply get hands-on to install it and use for an example ml project.

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

Here is the result of executing the commands in Dec. 2019. The results will likely be different if you run the command and a newer version has been released with a different checksum result.

{{< img src="miniconda-checksum-results.png" >}}

and here is the matching value from the website:

{{< img src="miniconda-website-checksum.png" >}}

When you are comfortable that the download is valid you can start the install of miniconda:

```shell script
bash Miniconda3-latest-Linux-x86_64.sh
```
Along the way you can feel free to accept the default options.


##### (Optional) Note about conda init
The last option to initialize the system with `conda init` will modify your .bashrc scripts to include the miniconda directory in your path. Additionally it will have conda activated by default. I like this on a new server used only for this purpose. However, if performing these steps on my local/main workstation, I prefer to activate the conda shell manually with the command: `conda activate`. As such I would answer *no* to the previous `conda init` command. Additionally I would tell conda not to auto-activate with the following:

```shell script
conda config --set auto_activate_base false
```

#### Activate Conda

Assuming you had the installer execute the `conda init` command you can either open a new shell or just restart your shell:

```shell script
source .bashrc
```

#### Install git

If you don't already have git install you will need to perform the following:

```shell script
sudo apt-get update
sudo apt-get install git
```
#### Install mlflow

Ensure that you have your conda shell activated either by default or by manually typing `conda activate`. Your shell should generally have a "(base)" prefix like this:

{{< img src="shell_with_conda_activated.png" >}}

Then you are ready to actually install the mlflow package with extras:

```shell script
pip install mlflow[extras]
```
#### Add the Code

The following steps can be done manually as the amount of information is very minimal. However you can also get the code from my github repository: [https://github.com/j-buss/10-minute-tech-tutorials/tree/master/machine_learning](https://github.com/j-buss/10-minute-tech-tutorials/tree/master/machine_learning)

Let's add two directories: 

```shell script
mkdir mlflow-tutorial
mkdir mlflow-tutorial/sklearn-iris-classification
```
A brief explanation of each:

1. mlflow-tutorial: The folder for all of our content. We will run our `mlflow` commands from this folder so in addition to the folder we will explicitly add in the next step, the mlflow framework will add some folders and files as well
 2. mlflow-tutorial/sklearn-iris-classification: A directory for the three code files: conda.yaml, MLproject and train.py

##### mlflow-tutorial/sklearn-iris-classification/conda.yaml

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
##### mlflow-tutorial/sklearn-iris-classification/MLproject

The MLproject file is a mlflow configuration file that each project needs to identify the specific components of the model.

```yaml
name: sklearn_logistic_example

conda_env: conda.yaml

entry_points:
  main:
    command: "python train.py"
```
##### mlflow-tutorial/sklearn-iris-classification/train.py 

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


---

### 2. Run Iris Classification

With all of that prep work out of the we way we are now ready to run Iris Classification model through the mlflow framework.

#### Train the ML Model
At the command line within the mlflow-tutorial directory run the following command:

```shell script
mlflow run sklearn_iris_classification
```

This will ultimately train our logistic regression model. However, the first steps will build the conda environment by downloading the packages identified from within the conda.yaml file. This is the benefit of the conda framework as it creates an isolated environment for your code execution.

The output of the model run will be stored in a new directory called "mlruns". 

{{< img src="directory_structure_mlruns.png" >}}

Feel free to look around, but in just a minute we will use the mlflow UI to view the results in a more consumable fashion.

#### Open a Firewall port
Before we can use the mlflow UI we need a firewall port open on GCP. Within GCP navigate to
VPC Network->Firewall and create a firewall rule for our server. 

We will leverage the network tag "mlflow-ui" that we used when we created the server. Here are the selections for the firewall rule:

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
Specified protocols and ports|tcp:8080,8081

Click the "Create" button and we should be all set.

Image of completed firewall rule "mlflow-ui-allow":

{{< img src="firewall_rule_mlflow_ui_allow.png" >}}

#### View the mlflow UI results
With the firewall rule in place we are set to start up the mlflow UI.

Within our directory "mlflow-tutorial" start up the mlflow UI server by executing the following:

```shell script
mlflow ui -p 8080 -h 0.0.0.0
```

You should receive some output like the following:

{{< img src="start-mlflow-ui.png"  >}}

Open a browser and navigate to the external ip address of the server (with the port number ":8080" suffix). 

You will see one result for each of the training runs you have performed. In my case I ran trained the model two times.

{{< img src="mlflow-ui.pn">}}

There is much more you can explore in the UI by grouping experiments and comparing training results. However in the name of brevity I will skip it in this post and go to the model serving.

### 3. Serve the Model

Now that we have a trained model we can serve it with the mlflow framework. This will create a flask app and allow us to send a new value to the model and predict the output. We simply need to know where the file is that contains the model.

In our train.py code we had a line to save our model.

```python
mlflow.sklearn.log_model(logreg, "model")
```
Each time we train a model, or perform a "run", mlflow keeps a set of information about that specific "run", this includes the log files that we saw in the UI in addition to a trained model.

Clicking on one of the runs from the UI experiments page we can see some information about the run:

{{< img src="mlflow_ui_run_details.png" >}}

Specifically towards the bottom of the page we can expand the Artifacts section and see that a model file was kept for the individual run:

{{< img src="mlflow_ui_artifacts.png" >}}

#### Serve

Now having confirmed that we have a model file we can serve it up with a different port number so we could still use the mlflow ui if needed:

```shell script
# Replace <run-id> with specific run id
# mlflow models serve -m runs:/<run-id>/model -p 8081 -h 0.0.0.0
mlflow models serve -m runs:/bbb6f1f04fd14aec8a39dea4b18759cb/model -p 8081 -h 0.0.0.0
```
Now we need to get some data to test with from the scikit-learn iris dataset:

{{< img src="python_example_iris_values.png" >}}

Then from another shell on your local workstation you can actually send a payload to the server to obtain a prediction from our trained model:

```shell script
curl http://34.70.102.58:8081/invocations -H 'Content-Type: application/json; format=pandas-records' -d '[[5.1, 3.5, 1.4, 0.2]]'
```

We should get the prediction of "0" back which tells us the iris in question is a "setosa"

Let's try one more:
```shell script
curl http://34.70.102.58:8081/invocations -H 'Content-Type: application/json; format=pandas-records' -d '[[5.9, 3.0, 5.1, 1.8]]'
```

We should get the prediction of "2" which tells us the iris in question is a "virginica"



 
### Summary

So we have installed mlflow, trained a model and hosted it for making predictions. Make sure to turn off and/or delete any of the resources you are not going to keep using to ensure you are not hit with additional charges.

---

### Resources:
- All the code can be found in my [10-minute-tech-tutorials](https://github.com/j-buss/10-minute-tech-tutorials) 
- [mlflow Quickstart](https://www.mlflow.org/docs/latest/quickstart.html) 
- [mlflow Tutorial](https://www.mlflow.org/docs/latest/tutorials-and-examples/tutorial.html)

{{< code-clipboard >}}
