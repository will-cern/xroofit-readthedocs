Day 0: Software Introduction and Setup
=====

This course uses the RooFit statistical analysis toolkit that is part of the `ROOT <https://root.cern/>`_ data analysis framework. RooFit is a collection of classes that can be used for building statistical models and datasets, and fitting the former to the latter. We will also make use of the `xRooFit <https://gitlab.cern.ch/will/xroofit>`_ library, which is a high-level API for working with RooFit.

The most helpful way to understand the interplay of xRooFit with RooFit is to think of the relationship between Keras and Tensorflow in machine learning. While it is certainly possible to build+train ML models directly with tensorflow objects, it can often be easier or more efficient to work with the tensorflow object through the keras API. Similarly, while RooFit can certainly be used to directly build+fit statistical models, doing this through the xRooFit interface can be more efficient. And certainly the terminology used in this course is intended to be consistent with what is used in the xRooFit API. 

.. _installation:

ROOT
------------

How to install ROOT:

.. code-block:: console

   (.venv) $ ...

xRooFit
----------------

xRooFit installation on top of ROOT ....


