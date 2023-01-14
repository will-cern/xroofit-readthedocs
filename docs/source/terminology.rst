Terminology
===========

This section introduces the terminology convention adopted throughout the rest of this course. It also serves as a summary of the basic building blocks of statistical analysis.

What is statistical analysis?
-----------------------------

It can be many things, but I will offer the following description that will cover many of the common analyses performed in HEP

.. attention:: What is statistical analysis?
    Statistical analysis is the process of make inferences about the values of parameters from a dataset of observables, using a parameterized probability model for the dataset. 

Therefore we need to understand the following terms: parameters, observables, models, datasets. 

Variables
---------
`Variables` are the fundamental types from which `models` and `datasets` are built. A variable has a value. There are two types:

  * `continuous`: represented in RooFit by ``RooRealVar``. They can optionally have one or more named `ranges` associated to them, each range being defined by a lower and upper `bound`. 
  * `discrete` or `categorical`: represented in RooFit by ``RooCategory``, they have a finite set of possible values (states) defined for them.
  
In a given statistical analysis all the variables are either an `observable` or a `parameter`. The observables are the variables that appear in a `dataset`, and the parameters are all the variables in `model` that aren't in the dataset. 

There are two types of observable: `regular` and `global`. The regular observables should be thought of as defining the columns of a dataset, and each `entry` in the dataset will have a value for each of the regular observables; the entries of a dataset usually correspond to a single event, but an entry can also have a `weight` for when dealing with weighted data. The global observables, on the other hand, are like metadata of the dataset: they are defined for the dataset as a whole and can be present even if there are no entries in the dataset. Datasets are represented in RooFit by classes inheriting from ``RooAbsData``, of which ``RooDataSet`` is really the only one you need to know about. 

As stated above, the variables of a model that are not observables (as defined by a given dataset) are refered to as `parameters`. A model is a parameterized function of a dataset that evaluates to the probability density (or sometimes probability mass) of observing the dataset, given the parameter values. We will learn below that models usually follow a common generic structure in HEP. Models are represented in RooFit by classes inheriting from ``RooAbsPdf``.

Parameters are in one of two possible states: they are either `floating` or `constant`. Only continuous parameters can be floating (we will learn why a floating categorial parameter would cause issues). The constant parameters are also sometimes called the `arguments` of the model, and the floating parameters are the `floats`. 
  
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

