Day 1: Variables, Models, Datasets
===========

Today we introduce the terminology convention adopted throughout the rest of this course. We summarise of the basic building blocks of statistical analysis.

What is statistical analysis?
-----------------------------

It can be many things, but I will offer the following description that will cover many of the common analyses performed in HEP

.. attention:: What is statistical analysis?
    Statistical analysis is the process of make inferences about the values of parameters from a dataset of observables, using a parameterized probability model for the dataset. 

Therefore we need to understand the following terms: parameters, observables, models, datasets. 

Variables
---------
`Variables` are the fundamental entities from which `models` and `datasets` are built. A variable has a value that is not derived from any other variable, as opposed to a `function` which has its value derived from variables (or other functions). There are two types:

  * `continuous`: represented in RooFit by ``RooRealVar``, they can optionally have one or more named `ranges` associated to them, each range being defined by a lower and upper `bound`. 
  * `discrete` or `categorical`: represented in RooFit by ``RooCategory``, they have a finite set of possible values (states) defined for them.

Datasets
---------
`Datasets` consist of a set of values for a collection of variables. The variables that appear in a dataset are known as `observables`. You should think of a dataset as a table of values, where each observable is one column of the table, and the rows of the table are known as `entries` in the dataset. The entries of a dataset usually correspond to a single event, but an entry can also have a `weight` for when dealing with weighted data. Additionally, datasets can have a special list of observables called `global observables`, which can be thought of as like metadata for the dataset: their values are not specific to any entry, and are even defined if the dataset has no entries. The observables that are used as columns in the dataset are known as `regular observables`. Datasets are represented in RooFit by classes inheriting from ``RooAbsData``, of which ``RooDataSet`` is really the only one you need to know about. 

Models
----------
`Models` are functions that evaluate to the probability density (or sometimes probability mass) of observing an entry of a given dataset. This will include the probability of observing the global observable values of the dataset. Any variable that the model depends which isn't an observable is known as a `parameter`. We will learn below that models usually follow a common generic structure in HEP. Models are represented in RooFit by classes inheriting from ``RooAbsPdf``.

Parameters are in one of two possible states: they are either `floating` or `constant`. Only continuous parameters can be floating (we will learn why a floating categorial parameter would cause issues). The constant parameters are also sometimes called the `arguments` of the model, and the floating parameters are the `floats`. 

Test Statistics
-------------
`Test statistics` are functions that map a dataset onto a single value. In RooFit they are represented by classes inheriting from ``RooAbsTestStatistic``.

Fit Results
------------
`Fit Results` represent the result of a minimization of a `test statistic`. In RooFit fit results are represented by the ``RooFitResult`` class.

Workspaces
------------
A workspace is a collection of one or more models with one or more datasets. The observables of a workspace are all the observables of the datasets. The parameters of a workspace are all the other variables of the models in the workspace. In RooFit these are the class ``RooWorkspace``. These can also store fit results. 

Summary of types of variable
----------------------------
The table below summarises the different type of variable described above:

+-----------+----------+----------------------------------------------------------------------------------------------------+
|observable | regular  | Columns of a dataset, can have different value for each entry                                      |
|           |----------+----------------------------------------------------------------------------------------------------+
|           | global   | Metadata of a dataset, same value for every entry (can be defined even if no entries in the datset)|
+-----------+----------+----------------------------------------------------------------------------------------------------+
|parameter  | floating | Non-constant non-observables of a model (with a given dataset)                                     |
|           |----------+----------------------------------------------------------------------------------------------------+
|           | constant | Constant non-observables of a model (with a given dataset)                                         |
+-----------+----------+----------------------------------------------------------------------------------------------------+



In a given statistical analysis all the variables are either an `observable` or a `parameter`. The observables are the variables that appear in a `dataset`, and the parameters are all the variables in `model` that aren't in the dataset. 

There are two types of observable: `regular` and `global`. The regular observables should be thought of as defining the columns of a dataset, and each `entry` in the dataset will have a value for each of the regular observables; the entries of a dataset usually correspond to a single event, but an entry can also have a `weight` for when dealing with weighted data. The global observables, on the other hand, are like metadata of the dataset: they are defined for the dataset as a whole and can be present even if there are no entries in the dataset. Datasets are represented in RooFit by classes inheriting from ``RooAbsData``, of which ``RooDataSet`` is really the only one you need to know about. 

As stated above, the variables of a model that are not observables (as defined by a given dataset) are refered to as `parameters`.  


  


