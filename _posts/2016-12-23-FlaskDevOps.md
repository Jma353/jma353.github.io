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
