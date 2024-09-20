Day 1: Variables, Models, Datasets, Workspaces
===========

Today we introduce the terminology adopted throughout the rest of this course. We summarise of the basic building blocks of statistical analysis.

What do I mean by statistical analysis?
-----------------------------

For now I will work with the following description that will cover many of the common analyses performed in HEP

.. note:: Statistical Analysis:
    The process of making inferences about the values of parameters from a dataset of observables, using a parameterized probability model for the dataset. 

Therefore we need to understand the following terms: parameters, observables, models, datasets. 

Variables
---------
:ref:`Variables` are the fundamental entities from which :ref:`Models` and :ref:`Datasets` are built. A variable has a value that is not derived from any other variable, as opposed to a `function` which has its value derived from variables (or other functions). There are two types:

  * `continuous`: represented in RooFit by ``RooRealVar``, they can optionally have one or more named `ranges` associated to them, each range being defined by a lower and upper `bound`. 
  * `discrete` or `categorical`: represented in RooFit by ``RooCategory``, they have a finite set of possible values (states) defined for them.

Datasets
---------
:ref:`Datasets` consist of a set of values for a collection of variables. The variables that appear in a dataset are known as `observables`. You should think of a dataset as a table of values, where each observable is one column of the table, and the rows of the table are known as `entries` in the dataset. The entries of a dataset usually correspond to a single event, but an entry can also have a `weight` for when dealing with weighted data (the weights can also have an error, which can be used if calculating the `chi-squared` test statistic, which will be introduced below). Additionally, datasets can have a special list of observables called `global observables`, which can be thought of as like metadata for the dataset: their values are not specific to any entry, and are even defined if the dataset has no entries. The observables that are used as columns in the dataset are known as `regular observables`. Datasets are represented in RooFit by classes inheriting from ``RooAbsData``, of which ``RooDataSet`` is really the only one you need to know about. 

.. figure:: dataset_concept.png
    :width: 80%
    :align: "center"
    
    A visual representation of a dataset.

PDFs
----------
`PDFs` are functions that evaluate to the probability density (or sometimes probability mass if all the observables are categorical) of observing an entry of a given dataset. This will include the probability of observing the global observable values of the dataset. Any variable that the model depends which isn't an observable is known as a `parameter`. We will learn below that models usually follow a common generic structure in HEP. Models are represented in RooFit by classes inheriting from ``RooAbsPdf``.

Parameters are in one of two possible states: they are either `floating` or `constant`. Parameters that can be in either state are `floatable` parameters. Non-floatable parameters must be `constant` - these types of parameters are also called `prespecified`. Categorical parameters can be floatable. Continuous variables can be non-floatable if they are represented with a `RooConstVar` in RooFit. The constant parameters are also sometimes called the `consts` of the model, and the floating parameters are the `floats`.

Additionally, for statistical analysis purposes, one or more floatable parameters can be labelled `parameters of interest` (poi). The remaining floatable parameters are deemed the `nuisance parameters` (np).

Test Statistics
-------------
`Test Statistics` are functions that map a dataset onto a single value. They are usually constructed/defined using a model, thereby the parameters of the model are parameters of the test statistic.

Some, but not all, test statistics take the form of the summation of a quantity over the entries of the dataset and therefore the calculation can readily be parallelized across the entries. Such batch-computable test statistics are represented in RooFit by classes inheriting from `RooAbsTestStatistic`. Two such statistics are:

  * `Negative Log Likelihoood`: represented by  ``RooNLLVar`` in RooFit.
  * `chi-squared`: represented by ``RooXYChi2Var`` in RooFit.


.. _objective functions:
Objective functions
-------------
`Objective Functions` are :ref:`Test Statistics` that are desirable to minimize with respect to the parameters. The term `objective function` is used in machine learning for functions that are designed to be extremised.

Fit Results
------------
`Fit Results` represent the result of a :ref:`minimization <minimization>` of an `objective function` through varying its floating parameters. In RooFit fit results are represented by the ``RooFitResult`` class, and it holds the initial and final floating parameter values of the objective function, along with the constant parameter values, the minimized objective-function value with an estimate of the difference to the true minimum, and a status code to indicate whether the minimization was successful or not. Fit Results can also hold estimates floating parameter errors, along with status codes for the algorithms that estimate these errors.

Workspaces
------------
A workspace is a collection of one or more models with one or more datasets. The observables of a workspace are all the observables of the datasets. The parameters of a workspace are all the other variables of the models in the workspace. In RooFit these are the class ``RooWorkspace``. These can also store fit results and any other type of ROOT object.

.. _regular observables:
.. _global observables:
Summary of types of variable
----------------------------
The table below summarises the different types of variables that were introduced above:

.. list-table:: Types of variable
    :widths: 25 10 65
    :header-rows: 1

    * - Type
      - xRooNode method
      - Description
    * - Observable
      - obs()
      - Variable that features in a dataset. Includes regular and global observables.
    * - - Regular observable
      - robs()
      - Observable that is a column of a dataset, and can have a different value for each entry.
    * - - Global observable
      - globs()
      - Metadata of a dataset, same value for every entry 
(can be defined even if no entries in the datset).
    * - Parameter
      - pars()
      - Not an observable. Includes prespecified and nuisance parameters, and parameters of interest.
    * - - Prespecified parameter
      - pp()
      - Non-floatable parameter, i.e. cannot be varied during a fit, nor assigned an uncertainty.
    * - - Parameter of interest
      - poi()
      - A floatable parameter that has been marked as "of interest".
    * - - Nuisance parameter
      - np()
      - A floatable parameter that is not a parameter of interest.
    * - - Floating parameter
      - floats()
      - A parameter that is currently marked as floating (will be subset of poi and np).
    * - - Constant parameter
      - consts()
      - A parameter that is currently marked as constant (all pp + any const poi or np). 

Exercises
----------------------------
 
Working with workspaces
^^^^^^^^^^^^^^^^^^^^^^^
Here are some ways to load a workspace into an `xRooNode`:
 
>>> w = XRF.xRooNode("filename.root")
>>> f = ROOT.TFile("filename.root"); ws = f.Get("wsname"); w = XRF.xRooNode(ws) # assumes wsname is name of workspace in file

Changing parameters from floating to constant and vice versa
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Once you have an `xRooNode` that wraps a workspace you can use methods of the node to access the different variables (see methods in table above). These each return a node that wraps a `RooArgList`. See https://root.cern.ch/doc/master/classRooArgList.html for documentation of that class. The individual elements of the list can be accessed by name or index via xRooNode e.g.: `w.obs()["obsName"]` which will return an xRooNode that wraps e.g. a `RooRealVar` with the name `obsName`. You can also produce another `xRooNode` that is a subset of the list using its `reduced <https://root.cern.ch/doc/master/classROOT_1_1Experimental_1_1XRooFit_1_1xRooNode.html#ac11e410ea9561991b44c2598be8c2659>`_ method, passing a comma separated list with or without wildcards. 

Any variable in RooFit can have boolean attributes on it, essentially a flag on the variable. All the constant parameters have the "Constant" attribute (note: it is case-sensitive). All the parameters of interest have the "poi" attribute. Attributes on individual variables can be set and retrieved with the `setAttribute <https://root.cern.ch/doc/master/classRooAbsArg.html#ac77328af4e29b2642c248a03f03deb73>`_ and `getAttribute <https://root.cern.ch/doc/master/classRooAbsArg.html#aa0e2616c8c43065117031c6797ac19d4>`_ methods of `RooAbsArg` (base class of almost everything in RooFit, including variables). `RooArgList` also has the method `setAttribAll` that can be used to set the same attribute on all the variables in the list.

Here are a few examples:

.. code-block:: python

    w.pars().reduced("alpha_*").setAttribAll("Constant")  # mark all parameters beginning with "alpha_" as constant
    w.pars()["myPar"].setAttribute("poi")                 # mark the parameter called 'myPar' as a parameter of interest
    w.poi()[0].setAttribute("Constant")                   # mark the first parameter of interest as constant
    w.pars()["myPar"].setAttribute("poi",False)           # demote it back to being a nuisance parameter
    w.floats().Print()                                    # list the currently floating parameters

As an exercise, see if you can list the parameters of your workspace, and play with which ones are constant and which ones are floating. 
