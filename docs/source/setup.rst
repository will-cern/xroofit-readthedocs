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

The main class of xRooFit is the `xRooNode <https://gitlab.cern.ch/will/xroofit#using-xroonode>`_. It is a smart interface class to objects in RooFit, in that it acts a bit like a smart pointer to a RooFit object and in this manner it provides you with a set of additional methods for the object it is "wrapping". 

xRooFit works in both C++ and python, however, it is much easier to use from python than from C++. The reason for this is because when you call a method of ``xRooNode`` in python, if it is not a method of the node but is actually a method of the *object* that the node contains then it will call that underlying method. So you can think of each ``xRooNode`` as having the methods defined for ``xRooNode`` (see `here <https://root.cern.ch/doc/master/classRooFit_1_1Detail_1_1XRooFit_1_1xRooNode.html>`_) *as well as* the methods of the RooFit object it is wrapping. In contrast, when working in C++ you have to use the ``get`` method of ``xRooNode`` with the specific class of the object in order to call the method. For example, to access the ``factory`` method of a workspace wrapped by an ``xRooNode`` in python you would do:

>>> w.factory( ... )

In C++ you would have to do:

.. code-block:: c++

   w.get<RooWorkspace>()->factory( ... )

In C++ you also have to be aware that some methods of ``xRooNode`` will return a pointer to an ``xRooNode`` whereas others will return an ``xRooNode``, and so you have to remember after each method if you should use ``.`` or ``->`` which isn't an issue in python where it is always ``.``. For these reasons, in this course we will stick with python in all the examples as the code is less verbose and more uniform in these cases.

