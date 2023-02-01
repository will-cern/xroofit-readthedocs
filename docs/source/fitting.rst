Day 3: NLL Construction and Minimization
========================================

This is otherwise known as the process of fitting a model to a dataset. The output of this process is a `Fit Result`.

>>> fitResult = w["modelName"].nll("datasetName").minimize()

NLL Options
------------
These are passed to the ``nll`` method (alongside the dataset name) and determine specifically how the NLL objective function is constructed ...

Fit Config
------------
The ``minimize`` method accepts an optional fit configuration that contains hyperparameters that steer the minimization ...
