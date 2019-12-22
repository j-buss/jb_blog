---
title: "Part 01"
date: 2019-12-22T10:25:54-06:00
draft: true
---

# TITLE

With this set of posts we will do a quick tutorial on using MLFlow to experiment with machine learning workflow.

#### Assumptions:
- Familiarity with Python, scikit-learn and machine-learning
- Access to a linux system

## Part 1: Install MLFlow

ipsum

### 1. Install MLFlow
Use the [MLFlow Tutorial](https://www.mlflow.org/docs/latest/tutorials-and-examples/tutorial.html)

- Install mlflow python library

 ```shell script
sudo pip3 install mlflow[extras]
```

- Install Conda

I tend to prefer the minimal type installations so I went for the [miniconda](https://docs.conda.io/en/latest/miniconda.html). Please check that you are using the latest version of the miniconda installer; however it appears these are permalinks so you should be good.

```shell script
# Download latest miniconda installer
curl -O -J https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
# Verify Checksums | compare with the value on the 
sha256sum Miniconda*
```

Here is the result of executing the command in Dec. 2019. The results will likely be different if you run a newer version.

{{< img src="miniconda-checksum-results.png" >}}

Here is the value from the website and we see that they match:

{{< img src="miniconda-website-checksum.png" >}}

Once we confirm that they match we are ready to install miniconda:

```shell script
bash Miniconda3-latest-Linux-x86_64.sh
```
Accept the default choices you are prompted with.

At the end you will be asked if you would like it to execute

```shell script
conda init
```

Feel free to select yes. This will modify your bash scripts to include the miniconda directory in your path.

I do however disagree with executing the `conda activate` by default every shell you open up. So I update the configuration with the following:

```shell script
conda config --set auto_activate_base false
```
- make a directory and create the following three files:

```shell script
mkdir mlflow-tutorial
cd mlflow-tutorial
```
The conda.yaml file will ensure conda loads the appropriate libraries and dependencies for our mlflow execution.
```yaml
name: sklearn-example
channels:
  - defaults
  - anaconda
dependencies:
  - python==3.6
  - scikit-learn=0.19.1
  - pip:
    - mlflow>=1.0
```

The MLproject file is a MLFlow configuration file that each project needs to identify the specific components of the model.

```yaml
name: sklearn_logistic_example

conda_env: conda.yaml

entry_points:
  main:
    command: "python train.py"
```

The train.py is the normal file we would think about to have our machine learning model.

```python
from __future__ import print_function

import numpy as np
from sklearn.linear_model import LogisticRegression

import mlflow
import mlflow.sklearn

if __name__ == "__main__":
    X = np.array([-2, -1, 0, 1, 2, 1]).reshape(-1, 1)
    y = np.array([0, 0, 1, 1, 1, 0])
    lr = LogisticRegression()
    lr.fit(X, y)
    score = lr.score(X, y)
    print("Score: %s" % score)
    mlflow.log_metric("score", score)
    mlflow.sklearn.log_model(lr, "model")
    print("Model saved in run %s" % mlflow.active_run().info.run_uuid)
```

```shell script
mlflow run sklearn_logistic_regression
```
- These files can all be found in the following the examples folder of the mlflow github [mlflow github](https://github.com/mlflow/mlflow/tree/master/examples/sklearn_logistic_regression)

### 2. Run MNIST Experiment

Now we are ready to run the 

```shell script
mlflow run mlflow-tutorial
```

### 3. Serve the Model


### Next Steps:


---

### Resources:
#### Code
- All the code can be found in my github: 
    - https://github.com/
- MLFlow [Quickstart](https://www.mlflow.org/docs/latest/quickstart.html) 
- MLFlow [Tutorial](https://www.mlflow.org/docs/latest/tutorials-and-examples/tutorial.html)

{{< code-clipboard >}}
