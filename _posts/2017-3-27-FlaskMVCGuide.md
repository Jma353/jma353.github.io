---
layout: post
title: Flask MVC Guide
---

This post outlines how to write an `Flask Blueprint` within your application, following the `Model-View-Controller` design pattern.  This guide relies on a project structure from the [`Flask DevOps Guide`](/FlaskDevOps/) on my website.

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
