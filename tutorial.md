# Creating a new Flask app
I'm going to make a starter Flask app with user login based on a GCP Firebase backend complete with a NoSQL database, testing suite, blueprints, Google Cloud functions, version control, virtual environments, React and generation of static assets. I'll use this for all new projects going forward. In the future I'll add stripe payments, teams and other important admin functionality.

In this notes section I'll document everything I did to create this project step by step, so I can keep coming back to it and improving it. It'll act as a changelog but also a tutorial if I ever need to spin one up slightly differently.

## Tutorials I used for inspiration / unmerciful copypasting
- [The Flask Mega Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)
- [How to build a simple personal website with python, flask and netlify](https://medium.com/@francescaguiducci/how-to-build-a-simple-personal-website-with-python-flask-and-netlify-d800c97c283d)
- [The Ultimate Flask Front-End](https://realpython.com/the-ultimate-flask-front-end/)
- [Flask Tutorial](https://flask.palletsprojects.com/en/1.1.x/tutorial/)
- [Flask and Firebase and Pyrebase Oh My](https://blog.upperlinecode.com/flask-and-firebase-and-pyrebase-oh-my-f30548d68ea9)
- [Heating up with firebase tutorial on how to integrate firebase into your app](https://blog.devcolor.org/heating-up-with-firebase-tutorial-on-how-to-integrate-firebase-into-your-app-6ce97440175d)

## Step by step instructions for recreating
This is where I'll record as I go exactly what I did to create this project. It can serve as a kind of tutorial. I will assume at this point you have Python3 and VSCode installed on your computer and you're using windows, and are ssh'd into github.

### 1: Create folder set up environment
First open VSCode and get to the projects folder on your computer, then create a folder for your new project.

`cd Documents\Projects`

`mkdir FireFlask`

`cd FireFlask`

Let's create a virtual environment with this command.

`python -m venv venv`

Now you can activate it with this command.

`venv\Scripts\activate`

If you want to deactive it, just type this.

`deactivate`

Now you want to install Flask.

`pip install flask`

Now let's initialize git

`git init`

You also want to add a .gitignore file.

Make sure the encoding to UTF-8 or git can't read it.

Copy and paste a template across, like this one:

[Python.gitignore](https://github.com/github/gitignore/blob/master/Python.gitignore)

You want to save your commitments and a readme.

`echo "# FireFlask" > readme.md`

`pip freeze > requirements.txt`

Now do your first commit.

`git status`

`git add .`

`git commit -m "set up folder, requirements, gitignore, added readme"`

Next step is to connect to github. Log in and create a new repository.

Give it a unique name and brief description, and add the MIT license if making it public.

Now click 'Clone or download' and copy the SSH key.

`git remote add origin git@github.com:mjt145/FireFlask.git`

`git remote -v`

`git push --set-upstream origin master`

If git push not working, you need to pull the license changes first

`git pull origin master --allow-unrelated-histories`

This lets you ignore the problem with different histories.


### 2: Set up the initial Flask application
Now it's time to create our first working application.

`mkdir app`


Make a new file in the app folder called __init__.py
```
from flask import Flask

app = Flask(__name__)

from app import routes
```

Next we need to add the routes file which determines where traffic is sent
```
from app import app

@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
```

Finally to make the app work, you have to have a script at the top level that you'd run to start the application.

`from app import app`

Now you just need to tell windows what file to run for your flask app

`set FLASK_APP=fireflask.py`

You can run the app using this commmand

`flask run`

Then just press CTRL + c to exit

To make the environment variable be remembered across sessions, install python-dotenv

`pip install python-dotenv`

Remember to freeze your requirements after installing

`pip freeze > requirements.txt`

Then add a .flaskenv file to the top level

`FLASK_APP=fireflask.py`

Make sure you add .flaskenv to your gitignore

Add the changes to git and commit

From now on we're going to make changes in branches and then merge to master. Create a new branch and check it out.

`git checkout -b "templates"`

Then see all your branches here
`git branch`

Switch back to head like this
`git checkout master`

To push the branch to github now you run this
`git push --set-upstream origin templates`

If you need to add deletes to git then do this
`git add -u .`

The correct workflow to integrate it back again is this
`git checkout master`
`git pull origin master`
`git merge templates`
`git push origin master`

Then you can delete the branch like this
`git branch -d templates`
`git push origin --delete templates`

Ok now onto the templates changes. First change the routes.py file to return some HTML.
```
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    return '''
<html>
    <head>
        <title>Home Page - FireFlask</title>
    </head>
    <body>
        <h1>Hello, ''' + user['username'] + '''!</h1>
    </body>
</html>'''
```

If you run the flask server and go to http://127.0.0.1:5000/ you'll see it now says 'Hello Miguel'

Next step is to abstract away that HTML into a different file.
`mkdir app/templates`

In app/templates make a html file called index.html
```
<html>
    <head>
        <title>{{ title }} - FireFlask</title>
    </head>
    <body>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
```

Now go back to routes.py and change it to this
```
from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    return render_template('index.html', title='Home', user=user)
```

Next let's try out conditional statements. Change the template to this:
```
<html>
    <head>
        {% if title %}
        <title>{{ title }} - FireFlask</title>
        {% else %}
        <title>Welcome to FireFlask!</title>
        {% endif %}
    </head>
    <body>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
```

If we want to add loops, this is what we do to the routes.py to add a list of dictionarys we can loop through.
```
from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Michael'}
    posts = [
        {
            'author': {'username': 'John'},
            'body': 'Beautiful day in Portland!'
        },
        {
            'author': {'username': 'Susan'},
            'body': 'The Avengers movie was so cool!'
        }
    ]
    return render_template('index.html', title='Home', user=user, posts=posts)
```

Now in the template we need to change it to loop through the posts
```
<html>
    <head>
        {% if title %}
        <title>{{ title }} - FireFlask</title>
        {% else %}
        <title>Welcome to FireFlask</title>
        {% endif %}
    </head>
    <body>
        <h1>Hi, {{ user.username }}!</h1>
        {% for post in posts %}
        <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
        {% endfor %}
    </body>
</html>
```

The next thing we'll want to do is set up some template inheritance, so we can have a nav bar.

Create a new html file called base.html in templates
```
<html>
    <head>
      {% if title %}
      <title>{{ title }} - FireFlask/title>
      {% else %}
      <title>Welcome to FireFlask</title>
      {% endif %}
    </head>
    <body>
        <div>FireFlask: <a href="/index">Home</a></div>
        <hr>
        {% block content %}{% endblock %}
    </body>
</html>
```

The block content part allows you to pull in other html files. Now we can simplify index.html and update the nav bar across every file in future.

```
{% extends "base.html" %}

{% block content %}
    <h1>Hi, {{ user.username }}!</h1>
    {% for post in posts %}
    <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
    {% endfor %}
{% endblock %}
```

### 3: Set up the web forms
Next step is to accept input from users, for example to publish posts or sign up via web forms. For this we use WTForms.

First let's install WTForms and update requirements
`pip install flask-wtf`
`pip freeze > requirements.txt`

Now create a new branch in git as before
`git checkout -b "forms"`

Change the __init__.py file to this
```
from flask import Flask
from config import Config

app = Flask(__name__)
app.config.from_object(Config)

from app import routes
```

That will let you import a config file, which should look like this:
```
import os

class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
```

It should be in the top level directory.

Now create a file called forms.py in the app folder, with the following contents:
```
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired

class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Sign In')
```

The next step is to add the template for the login page, as login.html:
```
{% extends "base.html" %}

{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post" novalidate>
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
```

Now let's create a view function in the routes.py file

Add this to the top
`from app.forms import LoginForm`
and then this to the bottom:
```
@app.route('/login')
def login():
    form = LoginForm()
    return render_template('login.html', title='Sign In', form=form)
```

To make it easy to navigate to, include the link to the login page from the nav in the base.html template

```
<div>
    FireFlask:
    <a href="/index">Home</a>
    <a href="/login">Login</a>
</div>
```

Now we just need to add a way to handle the form data submitted. In the routes.py file we should change the login path to this:
```
from flask import render_template, flash, redirect

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        flash('Login requested for user {}, remember_me={}'.format(
            form.username.data, form.remember_me.data))
        return redirect('/index')
    return render_template('login.html', title='Sign In', form=form)
```

Note the import line should replace the one at the top, and then the rest of the code replaces just the login route (leave the other routes there)

Finally let's add the flash message to the base template so they show on any page.

```
<html>
    <head>
        {% if title %}
        <title>{{ title }} - FireFlask</title>
        {% else %}
        <title>FireFlask</title>
        {% endif %}
    </head>
    <body>
        <div>
            FireFlask:
            <a href="/index">Home</a>
            <a href="/login">Login</a>
        </div>
        <hr>
        {% with messages = get_flashed_messages() %}
        {% if messages %}
        <ul>
            {% for message in messages %}
            <li>{{ message }}</li>
            {% endfor %}
        </ul>
        {% endif %}
        {% endwith %}
        {% block content %}{% endblock %}
    </body>
</html>
```

We also need to pass the error messages back to the form as well, in the login.html templates so they're rendered when users enter something wrong.

```
{% extends "base.html" %}

{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post" novalidate>
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}<br>
            {% for error in form.username.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}<br>
            {% for error in form.password.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
```

Last thing to tidy up, is the links, because they shouldn't be hardcoded and instead should use url_for().

Change the two links in the base.html template:
```
<a href="{{ url_for('index') }}">Home</a>
<a href="{{ url_for('login') }}">Login</a>
```

Import url_for in the routes.py
`from flask import render_template, flash, redirect, url_for`

Change the redirect on form validate on submit
`return redirect(url_for('index'))`


Random note: if you set `FLASK_DEBUG=1` in the .flaskenv, you don't have to keep restarting the server to see changes.

The correct flow to integrate a branch and push changes to master is as follows:

Check you're on the right branch
`git branch`

Make sure you're up to date with your commits
`git status`
`git commit -m "added forms to the application"`

Push the branch changes upstream to github
`git push --set-upstream origin forms`

Now you can integrate the branch locally, first checkout master branch
`git checkout master`

Pull down from github to make sure you're up to date
`git pull origin master`

Now merge the forms branch into your local master
`git merge forms`

Check out the git log and see where you're at
`git log --oneline`

Check out the diff if needed by getting the hash from git log
`git diff 30519e4`
exit with `q`

Once you're happy you can push to master
`git push origin master`

Now delete the branch you don't need anymore, locally and remotely
`git branch -d forms`
`git push origin --delete forms`