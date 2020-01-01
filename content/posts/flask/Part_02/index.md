---
title: "Flask Part 2 - Reading BigQuery"
date: 2019-12-29T05:54:03-06:00
draft: true
---

This is the next step in my Flask series. The previous post culminated into deploying an extremely simple application on GCP App Engine: [Flask Part 1 - Simple App]({{< ref "/posts/flask/Part_01/index.md" >}}). In this post we will extend the simple Flask App and read some data from a public dataset in BigQuery.

We will break up the work into 3 steps:
1. Simple Query Job
2. Display with Template
3. Run Asynchronously

#### Assumptions:
- Familiarity with Python
- Familiarity with Google Cloud Platform
- Access to a GCP Project with ability to deploy App Engine
- Linux working environment 
  - This tutorial leverages GCP Cloud Shell
  - Steps are not dependent on Linux, but haven't been tested in other environments
- Also, I am assuming you have followed along with the previous post: [Flask Part 1 - Simple App]({{< ref "/posts/flask/Part_01/index.md" >}})

## 1. Simple Query Job

We will take our simple flask app from last time and add in the BigQuery components.

### BigQuery Library
In order to use BigQuery we will need to install the Python BigQuery library into our environment. We update the requirements.txt file with the following:

```python {hl_lines=["3"]}
Flask==1.1.1
gunicorn
google-cloud-bigquery==1.22.0
```

Additionally, if you plan to test the code locally you should load the library in your shell:

```shell script
pip install google-cloud-bigquery
```


### A Simple Query 

Now let's focus on a really simple query from a public BigQuery database. Let's use a public dataset from the Census Bureau's [American Community Survey (ACS)](https://www.census.gov/programs-surveys/acs). The ACS is a series of monthly samples that produces an annual estimate for the decennial census information.

Feel free to use the BigQuery UI to explore the dataset some more, but for this post I will just use a count of records:

{{< img src="bigquery_ui_simple_query_results.png" >}}

### Modify main.py

So with our simple query in hand we will need to make 3 additions to our code:

1. BigQuery Library and client [Lines 3-4]
    - Add in the BigQuery Python Library and create a BigQuery client to connect to BigQuery
2. [query_job](https://googleapis.dev/python/bigquery/latest/generated/google.cloud.bigquery.job.QueryJob.html) [Lines 10-18]
    - The query_job is the BigQuery construct that will execute an asynchronous query to BigQuery
3. Handle results [Lines 20-24]
    - The query_job will return a [RowIterator](https://googleapis.dev/python/bigquery/latest/generated/google.cloud.bigquery.table.RowIterator.html#google.cloud.bigquery.table.RowIterator)
    - Iterate through the return object to display results

```python {linenos=inline,hl_lines=["3-4","10-18","20-24"]}
import flask

from google.cloud import bigquery
bigquery_client = bigquery.Client()

app = flask.Flask(__name__)

@app.route("/")
def main():
    # Define query_job and query object
    query_job = bigquery_client.query(
        """
        SELECT 
          COUNT(*) as Rec_Count 
        FROM 
          `bigquery-public-data.census_utility.fips_codes_all`
        """
    )

    # Handle query_job result and return to flask to display
    res = query_job.result()
    for row in res:
        output = "Record Count: " + str(row[0])
    return output

if __name__ == "__main__":
    app.run(port=8080)
```

So with the changes to our requirements.txt and main.py we are ready to test the app locally:

```shell script
export FLASK_APP=main.py
flask run
```

If the test is successful you can deploy to GCP App Engine:

```shell script
#Substitute your PROJECT ID in the following
#gcloud config set project <PROJECT_ID>
gcloud config set project my-flask-tutorial-002
gcloud app deploy
```

Once it is deployed give it a test and ensure that we get the results we expect:

{{< img src="simple_count_app_engine_results.png" >}}

### 2. Display with Template

Let's make a minimal upgrade to our application. The only thing we are going to change is the usage of a [jinja template](https://jinja.palletsprojects.com/en/2.10.x/templates/) to clean up the display of data from BigQuery. This will allow us 2 benefits:

- Cleaner Code
- Display Flexibility

Let's handle the first item first. 

#### Cleaner Code

Here is an update to our main.py file with some changes on lines 20-21. 

```python {linenos=inline,hl_lines=["20-21"]}
import flask

from google.cloud import bigquery
bigquery_client = bigquery.Client()

app = flask.Flask(__name__)

@app.route("/")
def main():
    # Define query_job and query object
    query_job = bigquery_client.query(
        """
        SELECT 
          COUNT(*) as Rec_Count 
        FROM 
          `bigquery-public-data.census_utility.fips_codes_all`
        """
    )

    # Handle query_job result by rendering with a template
    return flask.render_template("query_result.html", results=query_job.result())

if __name__ == "__main__":
    app.run(port=8080)
```
We are simply using the [flask.render_template](https://flask.palletsprojects.com/en/1.1.x/api/#flask.render_template) method to "render a template from the template folder with the given context". We are telling it to use the file `templates/query_result.html` with the results `query_job.result()`.

#### Display Flexibility

So what is this file `templates/query_result.html` that I just casually mentioned, but does not yet exist? We will get to that, but it is is very important to understand what we are accomplishing before we get to the code. With the flask templating we are able to create files that can handle code, but will render into static files, in this specific case they will become html.

```html {linenos=inline,hl_lines=["9","13","11"]}
<!DOCTYPE html>
<table>
    <thead>
        <tr>
            <th>Record Count</th>
        </tr>
    </thead>
    <tbody>
    {% for row in results %}
    <tr>
        <td>{{ row[0] }}</td>
    </tr>
    {% endfor %}
    </tbody>
</table>
```

With lines 9 and 13 we define the start and end of a for loop respectively. This means that the `results` coming into the template will be iterated in this loop. 

Then in line 11 we are taking the first element of the row and displaying it within the an html cell `<td>...</td>` within a row `<tr>...</tr>`

#### Test It

OK. The changes to main.py and the new templates/query_result.html file are the only changes to make. Feel free to give it a test locally and then in GCP App Engine. Your output should look like the following:

{{< img src="flask_template_version_output.png" >}}

Note the slight difference in the output as we placed the results into an html table in our template file.


### 3. Run Asynchronously
We have the template working, but we are going to add two features to make it more robust:

- Asynchronous Call
- Flask Redirect

#### Asynchronous Call

We have glossed over some details of our call to `bigquery_client.query` 

[bigquery_client.get_job](https://googleapis.dev/python/bigquery/latest/generated/google.cloud.bigquery.client.Client.html#google.cloud.bigquery.client.Client.get_job)

As soon as we made the call to `bigquery_client.query` it submits a BigQuery job. We can see this in the shell if we execute the following:

```shell script
#List all BigQuery Jobs
#bq ls -j <PROJECT_ID>
bq ls -j my-flask-tutorial-002
```



#### Flask Redirect

[flask.url_for](https://flask.palletsprojects.com/en/1.1.x/api/#flask.url_for)

[flask.redirect](https://flask.palletsprojects.com/en/1.1.x/api/#flask.redirect)

[flask.request.args](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Request.args)

[flask.request.args.get](https://werkzeug.palletsprojects.com/en/0.16.x/datastructures/#werkzeug.datastructures.MultiDict.get)




The output simply shows our makeshift html page with the BigQuery Job ID filled in:

{{< img src="webpage_from_submit_query.png" >}}




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
