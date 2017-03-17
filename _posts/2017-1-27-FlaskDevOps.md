---
layout: post
title: Flask DevOps
---

This post overviews `Flask` development operations for setting up a new project with an emphasis on the `Model-View-Controller` design pattern.

This guide makes the following assumptions:

* You are familiar with `Python`
* You have a high-level understanding of `SQL`
* You havea a high-level understanding of `REST` API development

This guide will be utilizing `PostgreSQL` to drive persistent storage on the backend.  I will be releasing and linking integration guides for other popular databases like `Couchbase`, `MongoDB`, etc. in the near future.  

## Get PyPI

This guide depends on you being able to easily download Python modules.  In order to do so, you should get `PyPI`.  Follow the basic guide [here](https://pip.pypa.io/en/stable/installing/).  

## Virtualenv - The Key to Python Projects

Before even touching `Flask`, let me introduce you to `virtualenv` (if you have not already seen this amazing tool). `Virtualenv` allows you to create an isolated environment to build and run a Python project in.  All dependencies for the project can be freshly declared and utilized, and the project can, therefore, be built and executed in a modular and isolated fashion.  In addition, if you download a preexisting Python project, you can create a virtual environment with `virtualenv` to install and store all dependencies for the project.  Think of it like your `node_modules` file if you come from `Node.js`, or your project gems if you come from `Ruby on Rails`.  To install, go [here](https://virtualenv.pypa.io/en/stable/installation/).  For dead-simple usage, go [here](https://virtualenv.pypa.io/en/stable/userguide/).

Once you have `virtualenv` setup, create an actual virtual environment with the following command:

{% highlight bash %}
virtualenv venv
{% endhighlight %}

In the above example, I chose to name the environment `venv`, but you can name it whatever you'd like.

To activate and enter the virtual environment, run the following:

{% highlight bash %}
source venv/bin/activate
{% endhighlight %}

The following command line prompt will indicate that you're in the virtual environment:

{% highlight bash %}
(venv) >
{% endhighlight %}

To deactivate the virtual environment, run the following:

{% highlight bash %}
deactivate
{% endhighlight %}

In order to maintain you dependencies, you should create a `requirements.txt` file, where your installed modules will go.  The typical workflow will be to install something via `pip`, and then run the following:

{% highlight bash %}
pip freeze > requirements.txt
{% endhighlight %}

The command `pip freeze` actually lists the dependencies encapsulated by the virtual environment.  The above command copies those into a text file that will allow one to run the following on downloading and using the Python project:

{% highlight bash %}
pip install -r requirements.txt
{% endhighlight %}

**NOTE:**  Be sure to `gitignore` your virtual environment.  

## Install Flask

We now have a solid setup to start installing our modules and setting up our scripts to initially run our app.  However, before we do anything, we must install `Flask`.  Simple run the following:

{% highlight bash %}
pip install flask
# To add this requirement to our dependency file
pip freeze > requirements.txt
{% endhighlight %}

## File Organization

A `Flask` app has some utility scripts at the top-level, and has a modular organization when defining any sort of functionality.  Dividing up a `Flask` app into modules allows one to separate resource / logic concerns.  

The utility scripts at the top level include the following:

{% highlight bash %}
config.py # describes different environments that app runs in
manage.py # holds functionality for migrating your database (changing its schema)
run.py    # runs the app on a port
{% endhighlight %}

The entire functional backend of a `Flask` app is housed in a parent module called `app`.  You can create this by creating a directory `app` and populating it with an `__init__.py` file.  Then, inside that `app` directory, you can create modules that describe the resources of your app.  These modules should be as de-coupled and reusable as possible.  For example, let's say I need a bunch of user authentication logic described by a couple of endpoints and helper functions.  These might be useful in another `Flask` app and can be comfortably separated from other functionality.  As a result, I would make a module called `accounts` inside my app directory.  Each module (including `app`) should also have a `templates` directory if you plan on adding any `HTML` views to your app.   

In general, your filesystem should resemble something like this:

{% highlight bash %}
.
├── README.md
├── app
│   ├── __init__.py
│   ├── accounts
│   │   └── __init__.py
│   └── templates
├── config.py
├── manage.py
├── requirements.txt
└── run.py

{% endhighlight %}

## Database Setup

In this example, as stated, `PostgreSQL` (or `Postgres`) will be the database leveraged.  `Postgres` can be installed a multitude of ways, but if you're on `OSX` I recommend utilizing the [`Postgres App`](https://postgresapp.com/).  

Once you have `Postgres` setup and have your `$PATH` configured accordingly, run the following:

{% highlight bash %}
# Enter postgres command line interface
$ psql
# Create your database
CREATE DATABASE my_app_db;
# Quit out
\q
{% endhighlight bash %}

The above creates the actual database that will be used for this application.  

Rather than writing raw-SQL for this application, I have chosen to utilize [`SQLAlchemy`](http://flask-sqlalchemy.pocoo.org/2.1/) (specifically, `Flask-SQLAlchemy`) as a database `Object-Relational-Model` (`ORM`, for short).  In addition, for the purposes of serialization (turning these database entities into organized [`JSONs`](http://www.json.org/) that we can send over the wire) and deserialization (turning a `JSON` into a entity once again), I have chosen to use [`Marshmallow`](https://marshmallow-sqlalchemy.readthedocs.io/en/latest/) (specifically, `marshmallow-SQLAlchemy`).  

Several modules are needed to completely integrate `Postgres` into a `Flask` app, but several of these modules are co-dependent on one another.  If we run the following, all modules needed for this tutorial relating to `Postgres` will be added:

{% highlight bash %}
pip install flask-migrate marshmallow-sqlalchemy psycopg2
pip freeze > requirements.txt
{% endhighlight %}

Once you have the above installed, let's write the migration script, `manage.py` that will be used to capture changes you make over time to the schemas of your various models.  

This script will not work out of the gate and refers to components we have not yet defined in our app, but I will describe these below:

{% highlight python %}
import os
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand
from app import app, db

migrate = Migrate(app, db)
manager = Manager(app)

manager.add_command("db", MigrateCommand)

if __name__ == "__main__":
  manager.run()
{% endhighlight %}

In the above, `app` refers to the module we created above in the **File Organization** section.  `db` refers to our reference to the database connection that we have yet to define in the `app` module.  

This script can be used in the following way to migrate your database, on changing your models:

{% highlight bash %}
# Initialize migrations
python manage.py db init
# Create a migration
python manage.py db migrate
# Apply it to the DB
python manage.py db upgrade
{% endhighlight %}

## Configuration Setup

Now that we have setup our database and have handled our `manage.py` script, we can create our `config.py` script, which involves the database and various other configuration information specific to `Flask`.  This file will be used in our initialization of the `Flask` app in the `app` module in the near future.  

An example of a `config.py` file looks like this:

{% highlight python %}
import os
basedir = os.path.abspath(os.path.dirname(__file__))

# Different environments for the app to run in

class Config(object):
  DEBUG = False
  CSRF_ENABLED = True
  CSRF_SESSION_KEY = "secret"
  SECRET_KEY = "not_this"
  SQLALCHEMY_DATABASE_URI = os.environ['DATABASE_URL']

class ProductionConfig(Config):
  DEBUG = False

class StagingConfig(Config):
  DEVELOPMENT = True
  DEBUG = True

class DevelopmentConfig(Config):
  DEVELOPMENT = True
  DEBUG = True

class TestingConfig(Config):
  TESTING = True
{% endhighlight %}

The above defines several classes used to instantiate configuration objects in the creation of a `Flask` app.  Let's go through some of the variables:

* `DEBUG` indicates whether or not debug stack traces will be logged by the server.
* `CSRF_ENABLED`, `CSRF_SESSION_KEY`, and `SECRET_KEY` all relate to `Cross-Site-Request-Forgery`, which you can read more about [here](https://goo.gl/qkGU9).  
* `SQLALCHEMY_DATABASE_URI` refers to the database URL (a server running your database).  In the above example, I refer to an environment variable `'DATABASE_URL'`.  I will be discussing environment variables in the next section, so stay tuned.

## Environment Variables

Environment variables allow one to specify credentials like a sensitive database URL, API keys, secret keys, etc.  These variables can be manually `export-ed` in the shell that you are running your server in, but that is a clunky approach.  The tool [`autoenv`](https://github.com/kennethreitz/autoenv) solves this problem.  

`autoenv` allows for environment variable loading on `cd`-ing into the base directory of the project. Follow the following command line arguments to install `autoenv`:

{% highlight bash %}
# Install the package from pip
pip install autoenv
# Override cd by adding this to your .?rc file (? = bash, zsh, fish, etc), I'll use
echo "source `which activate`" >> ~/.?rc
# Reload your shell
source ~/.?rc
# Make a .env file to hold variables
touch .env
{% endhighlight %}

As mentioned in the above code, your `.env` file will be where you hold variables, and will look something like this:

{% highlight bash %}
# Set the environment type of the app (see config.py)
export APP_SETTINGS=config.DevelopmentConfig
# Set the DB url to a local database for development
export DATABASE_URL=postgresql://localhost/my_app_db
{% endhighlight %}

As you can see above in the example, I reference a specific configuration class (`DevelopmentConfig`), meaning I plan on working in my development environment.  I also have my database URL.  

**NOTE:**  Be sure to `gitignore` your `.env` file.  

## Flask App Setup


Up until now, we haven't been able to run our server.  We must write 3 more files in order to do so.  

We have never setup our `Flask` app with it configurations anywhere, so let's do that.  We will be doing so in `./app/__init__.py`.  The file should look like this:

{% highlight python %}
# Gevent needed for sockets
from gevent import monkey
monkey.patch_all()

# Imports
import os
from flask import Flask, render_template
from flask_sqlalchemy import SQLAlchemy
from flask_socketio import SocketIO

# Configure app
socketio = SocketIO()
app = Flask(__name__)
app.config.from_object(os.environ["APP_SETTINGS"])
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = True

# DB
db = SQLAlchemy(app)

# Import + Register Blueprints
# WORKFLOW:
# from app.blue import blue as blue_print
# app.register_blueprint(blue_print)

# Initialize app w/SocketIO
socketio.init_app(app)

# HTTP error handling
@app.errorhandler(404)
def not_found(error):
  return render_template("404.html"), 404
{% endhighlight %}  

Let's unpack this file piece by piece.  The top initializes `Gevent`, [a coroutine-based Python networking library](http://www.gevent.org/) for `Socket.IO` (a very useful socket library useful in adding real-time sockets to your app).  The next section imports several libraries, initializes our `Flask` app, set our app up with the configurations from the appropriate `config.py` class, creates the `db` connection pool, bootstraps `Socket.IO` to our `Flask` app, and sets the `404` error page that the app should present on not finding a resource.  **NOTE**: the comments regarding registering a "blueprint" will be where we register our sub-modules with our main, parent `Flask` app.  

Since we have involved `Socket.IO` and `Gevent`, you should run the following:

{% highlight bash %}
pip install flask-socketio gevent
pip freeze > requirements.txt
{% endhighlight %}

Now that our actual `Flask` app instance exists, we should create our `404.html` page.  Place it in `./app/templates/404.html`:

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
  </head>
  <body>
    <h1>404: Resource not found</h1>
  </body>
</html>
{% endhighlight %}

Finally, in order to actually start our server, we need to create a top-level `run.py` script.  This script can be placed at the root of our project:

{% highlight python %}
from app import app, socketio

if __name__ == "__main__":
  print "Flask app running at http://0.0.0.0:5000"
  socketio.run(app, host="0.0.0.0", port=5000)

{% endhighlight %}


Now, at the root of your application, you can run:

{% highlight bash %}
python run.py
{% endhighlight %}

Your server is now running!

**NOTE:** If you get issues regarding `APP_SETTINGS` or `DATABASE_URL`, you should ensure your `.env` is setup properly, and you should `cd` out of and back into your project root.

## That's it, for now...

This marks the end of project configuration for a well-constructed `Flask` app following `MVC`.  However, for additional development-related advice regarding project setup, keep reading.  

# Accounts Blueprint

Now we will be diving into writing `Models` and `Controllers` for an `accounts` blueprint that will serve as reusable `users-sessions` module that can be added to any application desiring a sign-up system.  We make some short-cuts along the way in order to increase this guide's brevity, while still providing meaningful sample code and explanations for the different components of the system.  

We must create our module within `app`, such that it contains the following structure:

{% highlight bash %}
.
├── __init__.py
├── controllers
│   ├── __init__.py
│   ├── sessions_controller.py
│   └── users_controller.py
└── models
    ├── __init__.py
    ├── session.py
    └── user.py
{% endhighlight %}

Let's start with `./app/accounts/__init__.py`.  This file contains a couple of lines of information specifying the `Flask Blueprint` information of this module, as well import `controllers`:

{% highlight python %}
from flask import Blueprint

# Define a Blueprint for this module (mchat)
accounts = Blueprint('accounts', __name__, url_prefix='/accounts')

# Import all controllers
from controllers.users_controller import *
from controllers.sessions_controller import *
{% endhighlight %}

In addition, register your new blueprint in `./app/__init__.py` by changing the lines:

{% highlight python %}
# Import + Register Blueprints
# WORKFLOW:
# from app.blue import blue as blue_print
# app.register_blueprint(blue_print)
{% endhighlight %}

to:

{% highlight python %}
# Import + Register Blueprints
from app.accounts import accounts as accounts
app.register_blueprint(accounts)
{% endhighlight %}

## The Models

The models created will correspond, field-by-field, to the database tables that will be setup as a result of you running the series of migration commands listed above in the **Database Setup** section.

### Base Model and Imports

The following must be run before we dive into coding up our `Base` model (the abstract model parent class that all models will extend from):

{% highlight bash %}
pip install werkzeug
pip freeze > requirements.txt
{% endhighlight %}

This installs a module that is required to hash user-passwords ([you never want to store passwords in plain-text](https://arstechnica.com/security/2016/09/plaintext-passwords-and-wealth-of-other-data-for-6-6-million-people-go-public/)).  We should, technically, [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) the passwords too, but given that this is just an example, I'll leave that detail to your implementation.  

In `./models/__init__.py`, (accessible to the entire `models` module) we should write the following:

{% highlight python %}
from app import db # Grab the db from the top-level app
from marshmallow_sqlalchemy import ModelSchema # Needed for serialization in each model
from werkzeug import check_password_hash, generate_password_hash # Hashing
import hashlib # For session_token generation (session-based auth. flow)

class Base(db.Model):
  """Base PostgreSQL model"""
  __abstract__ = True
  id         = db.Column(db.Integer, primary_key =True)
  created_at = db.Column(db.DateTime, default    =db.func.current_timestamp())
  updated_at = db.Column(db.DateTime, default    =db.func.current_timestamp())

{% endhighlight %}

The above imports modules and objects necessary for use in your models, as well as defines a `Base` model class that every model should extend to inherit the book-keeping and necessary fields, `id`, `created_at`, and `updated_at`.  

### User Model

The `User` model in `./models/user.py` will consist of the following:

{% highlight python %}
from . import *

class User(Base):
  __tablename__ = 'users'

  email           = db.Column(db.String(128), nullable =False, unique =True)
  fname           = db.Column(db.String(128), nullable =False)
  lname           = db.Column(db.String(128), nullable =False)
  password_digest = db.Column(db.String(192), nullable =False)

  def __init__(self, ** kwargs):
    self.email           = kwargs.get('email', None)
    self.fname           = kwargs.get('fname', None)
    self.lname           = kwargs.get('lname', None)
    self.password_digest = generate_password_hash(kwargs.get('password'), None)

  def __repr__(self):
    return str(self.__dict__)


class UserSchema(ModelSchema):
  class Meta:
    model = User

{% endhighlight %}

The above is pretty self-explanatory.  It outlines several `db` fields, declares a constructor, defines a default string-based representation of the `User` model, and declares a `UserSchema` class that will be used to serialize and deserialize the `User` model to / from `JSON`.  

### Session Model

The `Session` model in `./models/session.py` will consist of the following:

{% highlight python %}
from . import *

class Session(Base):
  __tablename__ = 'sessions'

  user_id       = db.Column(db.Integer, db.ForeignKey('users.id'), unique =True, index =True)
  session_token = db.Column(db.String(40))
  update_token  = db.Column(db.String(40))
  expires_at    = db.Column(db.DateTime)

  def __init__(self, ** kwargs):
    user = kwargs.get('user', None)
    if user is None:
      raise Exception() # Shouldn't be the case

    self.user_id       = user.id
    self.session_token = self.urlsafe_base_64()
    self.update_token  = self.urlsafe_base_64()
    self.expires_at    = datetime.datetime.now() + datetime.timedelta(days=7)

  def __repr__(self):
    return str(self.__dict__)

  def urlsafe_base_64(self):
    return hashlib.sha1(os.urandom(64)).hexdigest()


class SessionSchema(ModelSchema):
  class Meta:
    model = Session

{% endhighlight %}

As we can see, the session model will belong to the user, and be part of a [token-based authentication flow](http://stackoverflow.com/questions/1592534/what-is-token-based-authentication).  Everything in the above code is self-explanatory, and serves as an example session implementation.  


### Exposing Models to the App

We must import our models into our app in order to have them be exposed to our `manage.py` script responsible for migrating our `db` to match our programmatic schema.  In `./controllers/__int__.py`, add the following (we'll throw in the imports while we're at it):

{% highlight python %}
from functools import wraps
from flask import request, jsonify, abort
import os

# Import module models
from app.accounts.models.user import *
from app.accounts.models.session import *

{% endhighlight %}


Now you can safely run the following in the root of your project to migrate your database:

{% highlight bash %}
# Initialize migrations
python manage.py db init
# Create a migration
python manage.py db migrate
# Apply it to the DB
python manage.py db upgrade
{% endhighlight %}

Now if you connect to your `Postgres` database, you should see two new tables, `users` and `sessions`!  The migration also creates an index on foreign-key `user_id` in `sessions`, for fast access of sessions by their owning user's `id`.  

That's it for models.  Stay tuned for the controller logic!  
