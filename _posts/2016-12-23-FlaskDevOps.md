---
layout: post
title: Flask DevOps
---

This post overviews `Flask` development operations for setting up a new project with an emphasis on the `Model-View-Controller` design pattern.

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

## Database Setup

In this example, as stated, `PostgreSQL` will be the database leveraged.  `PostgreSQL` can be installed a multitude of ways, but if you're on `OSX` I recommend utilizing the [`Postgres App`](https://postgresapp.com/).  

Once you have `PostgreSQL` setup and have your `$PATH` configured accordingly, run the following:

{% highlight bash %}
# Enter postgres command line interface
$ psql
# Create your database
CREATE DATABASE my_apps_db;
# Quit out
\q
{% endhighlight bash %}

## TODO: More to come 
