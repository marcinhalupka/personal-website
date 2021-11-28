---
title: "Model Deployment With Flask"
date: 2021-11-26T13:03:45+01:00
draft: false
---

In a typical analytical project we usually start with the problem definition which is followed by multiple steps like data collection, understanding, preparation, exploration and modeling.
If we're successful and create a useful model it's time to make it available for the end-users but as you can guess, sending them a piece of code or a Jupyter notebook is not an option.
Here comes a Flask to help us.

# Few words about the Flask

![Plot 1](/posts/images/flask-logo.png)

Flask is a web application framework written in Python. It's designed to make getting started quick and easy with the ability to scale up to complex applications.
It began a simple wrapper around *Werkzeug* and *Jinja* and has become one of the most popular Python web application frameworks.

Two scary names appeared here so let's explain what is hidden behind them:

* `Werkzeug` (German noun: tool) - a comprehensive web application library that provides tools necessary for communication between web server and web applications
* `Jinja2` - full-featured template engine for Python

Flask offers suggestions but doesn't enforce neither dependencies nor project layout - it's super flexible and there are many extensions provided by the community that make adding new functionality
easy.

It's often used to build small scale websites but we don't have to go that big - the variety of tools it provides can be used to build a REST API
with python in a really simple way - that is all we need!

## Installation

The recommendation is to use the virtual environment to manage the dependencies for your project.

The more Python projects you have, the more likely it is that you need to work with different versions of Python libraries or even Python itself. Newer versions of libraries for one project can break
compatibility in another project.

Virtual environments are independent groups of Python libraries, one for each project. Packages installed for one project will not affect other projects or the operating system's packages.

Python comes bundled with the `venv` module to create virtual environments.

**macOS/Linux**

```console
$ mkdir myproject
$ cd myproject
$ python3 -m venv venv
```

**Windows**

```console
> mkdir myproject
> cd myproject
> python3 -m venv venv
```
Before you work on your project, activate the coresponding environment:

**macOS/Linux**

```console
$ . venv/bin/activate
```

**Windows**

```console
> venv\Scripts\activate
```

Your shell prompt will change to show the name of the activated environment.

Now you're ready to install Flask, within the activated environment use the following command to install Flask:

```console
> pip install Flask
```

## A minimal application

A minimal Flask application looks something like this:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello, World!</p>"
```

So what did that code do?

1. First we imported the `Flask` class. An instance of this class will be our application.
2. Next we create an instance of this class. The first argument is the name of the application's module or package. `__name__` is a convenient shortcut for this that is appropriate for most cases.
This is needed so Flask knows where to look for resources.
3. We then use the `route()` decorator to tell Flask what URL should trigger our function.
4. The function returns the message we want to display in the user's browser. The default content type is HTML, so HTML in the string will be rendered by the browser.

Save it as `app.py` and run using command `flask run` command from your terminal. You should see a similar output (with few more extra lines):

```console
> flask run
 * Running on http://127.0.0.1:5000/
```

That means the app is running! When you head over to [http://127.0.0.1:5000/](http://127.0.0.1:5000/) you should see your hello world greeting.

![Plot 2](/posts/images/flask-hello-world.png)

# Training a simple model

Knowing the basics, it's time to prepare a basic model that we'll soon make accessible to the world.

We'll use the `Iris flower dataset` which contains three classes (three species of flowers) with 50 observations per class and build a classifier to tell us which class a new observation belongs to.

Each observation consists of four numbers:

* sepal length (cm)
* sepal width (cm)
* petal length (cm)
* petal width (cm)

The image bellow explains the meaning of these features:

![Plot 3](/posts/images/iris-description.png)

We can create a quick pairplot to have a look at data. Each graph on the 4x4 grid presents a combination of two features - on the diagonal it's a densitiy plot (as it's two times the same feature) and off the diagonal
it's a scatterplot. You can see that the grid is symmetric. Each species is represented with a different color.

![Plot 4](/posts/images/iris-pairplot.png)

As different species seem to be separable (setosa from other two ideally, versicolor from virginica - there's a small overlap on the marginal plots), we'll create a simple classifier and deploy it using Flask so
one with a yet unknown iris species can obtain a prediction.

Logistic regression from the `sklearn` library can be used for that purpose. As the features should be scaled before training, `Pipeline` object can be used as a wrapper on top of our sequentially applied transformation
and estimation. When all is done, our model can be serialized and dumped to a file (with a `pickle` library) so later it can be loaded from anywhere and used to make new predictions.

All of these steps translated to python code will look as follows:

```python
# Load libraries
import pickle
from sklearn import datasets
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

# Load digits dataset
iris = datasets.load_iris()

# Create feature matrix
X = iris.data

# Create target vector
y = iris.target

# Split the data into a training and test set
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=0)

# Combine all the steps into the pipeline
pipe = Pipeline([('scaler', StandardScaler()), ('lr', LogisticRegression())])

# Fit the model
pipe.fit(X_train, y_train)

# And save it to the local directory
pickle.dump(pipe, open('pipe_iris.pkl', 'wb'))
```

We can load it and check if everything works properly:

```python
loaded_pipe = pickle.load(open('pipe_iris.pkl', 'rb'))
print(loaded_pipe.score(X_test, y_test))
```

If yes, the following output should be printed:

```python
0.9736842105263158
```

Our model is ready to be deployed, time to move to creating a simple API with Flask.


# Building the template of the webpage

This article is not about HTML and CSS but not much is needed to create a basic, good looking interface so we'll do it.

Two templates are needed:

* `home.html` - here users enter the data
* `output.html` - and here they see a species prediction

Both files will be stored in the separate folder - in our home directory with an `app.py` file we'll create a
folder `templates` and put `home.html` and `output.html` there).

Code for the `home.html`:

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Iris species</title>
	<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bulma/0.9.0/css/bulma.min.css">
	<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
    <style>
        html{
            overflow: hidden;
        }

        body{
            position: absolute;
            width: 100%;
            height: 100%;
            margin: 0;
            padding: 0;
        }

        #login-form-container{
            position: absolute;
            width: 100%;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
        }
    </style>
</head>	
<body>
    <div id="login-form-container">
        <form action="classify" method="POST">
            <div class="card" style="width: 400px">
            <div class="card-content">
                <div class="media">
                <div class="is-size-4 has-text-centered">Iris Species Classification</div>
                </div>
                <div class="content">

                <div class="field">
                    <p class="control">
                    Sepal Length (cm): <input class="input" type="number" value='0.00' step='0.01' name="slen" id="slen">
                    </p>
                </div>

                <div class="field">
                    <p class="control">
                    Sepal Width (cm): <input class="input" type="number" value='0.00' step='0.01' name="swid" id="swid">
                    </p>
                </div>

                <div class="field">
                    <p class="control">
                    Petal Length (cm): <input class="input" type="number" value='0.00' step='0.01' name="plen" id="plen">
                    </p>
                </div>

                <div class="field">
                    <p class="control">
                    Petal Width (cm): <input class="input" type="number" value='0.00' step='0.01' name="pwid" id="pwid">
                    </p>
                </div>
                
                <div class="field">
                    <button class="button is-fullwidth is-rounded is-success">Submit</button>
                </div>
                </div>
            </div>
        </form>
    </div>
</body>
</html>
```

And the result:

![Plot 5](/posts/images/flask-interface.png)

To summarize - we created few `div` tags within a `form` with `action=classify` and `method=POST`. When user clicks the submit button,
the `POST` request will grab all the values filled by the user from the form and send them to the backend server.

Code for the `output.html`:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Iris Classification Result</title>
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bulma/0.9.0/css/bulma.min.css">
        <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
        <style>
            html{
                overflow: hidden;
            }

            body{
                position: absolute;
                width: 100%;
                height: 100%;
                margin: 0;
                padding: 0;
            }

            #login-form-container{
                position: absolute;
                width: 100%;
                height: 100%;
                display: flex;
                align-items: center;
                justify-content: center;
            }
        </style>
    </head>	
    <body>
        <div id="login-form-container">
            <div class="card" style="width: 400px">
                <div class="card-content">
                    <div class="media">
                        <div class="is-size-4 has-text-centered">Predicted class: {{ prediction }}</div>
                    </div>
                    <form action="home">
                        <div class="field">
                            <button class="button is-fullwidth is-rounded is-success">Retry</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </body>
</html>
```

And the result:

![Plot 6](/posts/images/flask-output.png)

In the code above, `{{ prediction }}` specifies the input argument that should be provided by the Flask server.
In Flask, such arguments can be passed along with the respective HTML file, to get rendered there.

# Connecting the webpage to our model

A minimal application presented earlier is a great starting point that needs some modifications.

To begin, let's start building our script by importing necessary libraries. We need flask and libraries used by the model we built earlier.

```python
from flask import Flask, request, render_template
import numpy as np
import pickle
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
```

Then we need to initialize the server and route it to the default URL path. The `template_folder` argument is used to point flask to the directory
where we store all the templates.

```python
app = Flask(__name__, template_folder='templates')

@app.route("/home")
def home():
    return render_template('home.html')
```

The default route is set by specifying the path in `@app.route()` decorator function. Whenever the `/home` route is called, the function gets executed.
Flask's `render_template()` function was used to set the default page to `home.html`.


```python
# Route 'classify' accepts POST request
@app.route('/classify', methods=['POST'])
def classify_type():
    try:
        sepal_len = request.form['slen'] # Get parameters for sepal length
        sepal_wid = request.form['swid'] # Get parameters for sepal width
        petal_len = request.form['plen'] # Get parameters for petal length
        petal_wid = request.form['pwid'] # Get parameters for petal width
        
        # Load the model's pipeline
        pipe = pickle.load(open(r'models/pipe_iris.pkl', 'rb'))
        
        # Reshape the input data to a required format
        user_input = np.array([sepal_len, sepal_wid, petal_len, petal_wid]).reshape(1, -1)
        
        # Get the output from the classification model, it predicts class label
        predicted_class_number = pipe.predict(user_input)[0]
        
        # Get the species name from class label
        predicted_species = ['Setosa', 'Versicolor', 'Virginica'][predicted_class_number]
        
        # Render the output in new HTML page
        return render_template('output.html', prediction=predicted_species)
    except:
        return 'Error'
```

In a similar way we created a separate route for our model: `/classify` with `POST` method as a default.

In this function we retrieve the data from action in `home.html` through `POST` request. We obtain the data from each of the input fields
in the form using its `name` attribute (`requests.form[name]`), load previously created pipeline, reshape the model to the expected format
and predict the class. As prediction is a class label (`[0, 1, 2]`) we map it to the corresponding species name (`['Setosa', 'Versicolor', 'Virginica']`).
The name is then passed as an argument to be rendered with an `output.html` file.

# Deployed model in action

Let's grab an example from the Iris dataset and see our model in action!

|sepal length (cm)|sepal width (cm)|petal length (cm)|petal width (cm)|Species|
|-----------------|----------------|-----------------|----------------|-------|
|5.1|3.5|1.4|0.2|Setosa|

Input:

![Plot 6](/posts/images/flask-input.png)


Output:

![Plot 6](/posts/images/flask-prediction.png)


# Conclusion

We went through a simple case study presenting how to train a simple model, connect it with a web application, create a basic frontend and
deploy everything locally with Flask.

That was just an introduction but if deploying machine learning model for you sounded like a complex
and heavy task, now you have an idea of what it is, how it works and can use our exercise as a starting point for your projects. 

