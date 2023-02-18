Day 2: Workspace Building
*************************

Today you will learn how to build a workspace containing a model and a dataset. 


Anatomy of a model
==================

Channels
---------

Samples
---------

Factors
--------
Samples can be built out of factors i.e. :math:`f_{s}(x_{c},p) = \prod_f f_{f}(x_{c},p)` where :math:`f_{f}(x_{c},p)` are the individual factors that are themselves functions of the channel observables :math:`x_{c}` and the model parameters :math:`p`. 

The HistFactory model specification defines the following types of factors: Histo, Shape, Overall, and Norm. In fact, the latter three can all technically be viewed as "special cases" of the first, as we will see. For completeness I will extend these types of factors to include "Const" and "ConstHisto" factors. Let's discuss each of these factor types in turn....

Histo Factors
^^^^^^^^^^^^^^

A Histo factor is a discretely-:math:`x_{c}`-dependent function ('discretely' = its binned in the observable) and has bin contents parameterized on one or more parameters. It should be thought of as being built up of a set of "variations" that correspond to particular points in the "variation space", specifically the points where one of the "variation coordinates" equals +1 or -1 and the remaining "variation coordinates" are all 0. The +1 point is the "up variation" corresponding to the given coordinate, and the -1 point is the "down variation" for that coordinate. There is also the "nominal variation"  corresponding to all coordinates equalling 0. The variations themselves could be functions of the parameters but are normally just a function of the observable (i.e. they are just an unparameterized histogram). The variation coordinates are often explicitly model parameters, but could also be functions of the model parameters (i.e. the coordinates implicitly depend on the parameters). The Histo factor has an "interpolation code" that defines how exactly to interpolates/extrapolate from these defined points to any point in the variation space. The formulae for this interpolation are:....

.. math::

   test

In RooFit Histfactory models Histo factors are represented by the ``PiecewiseInterpolation`` class. The fixed histograms that are usually the variations of a Histo factor are represented by the ``RooHistFunc`` class, and we will call these `ConstHisto` factors.

Shape Factors
^^^^^^^^^^^^^^^^

A Shape factor is equivalent to a Histo factor where there is exactly one variation coordinate for every bin in the observable, the nominal histogram is 0 everywhere, and each +1 or -1 variation histogram with a single bin being set equal to +1 or -1 respectively. Effectively this means that content of each bin of a shape factor is precisely equal to the value of one (and only one) of the variation coordinates. And normally each variation coordinate is explicitly one of the model parameters. 

In RooFit Histfactory models Shape factors are represented by ``ParamHistFunc`` class.

Overall Factors
^^^^^^^^^^^^^^^^^^

An overall factor is 
