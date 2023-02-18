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

The HistFactory model specification defines the following types of factors: Histo, Shape, Overall, and Norm. For completeness I will also introduce the terminology of "Const" and "ConstHisto" factors. I will explain below how, in fact, Shape, Overall, and Norm factors can technically be viewed as "special cases" of the Histo factor. Let's discuss each of these factor types in turn....

Const Factors
^^^^^^^^^^^^^^
This is just a fixed number. These are represented in RooFit by ``RooConstVar`` and amount to just a scale factor.

ConstHisto Factors
^^^^^^^^^^^^^^
This is a discretely-:math:`x_{c}`-dependent (discretely = binned in the observable) unparameterized function, i.e. it's just a histogram over the observable. ConstHisto factors are often used as the variations of a Histo factor (which is how the Histo factor becomes implicitly dependent on the observable). In RooFit a ``ConstHisto`` factor is represented by the ``RooHistFunc`` class.

Histo Factors
^^^^^^^^^^^^^^
A Histo factor is an implicitly-:math:`x_{c}`-dependent function parameterized on one or more parameters. It should be thought of as being built up of a set of "variations" that correspond to particular points in the "variation space", specifically the points where one of the "variation coordinates" equals +1 or -1 and the remaining "variation coordinates" are all 0. The +1 point is the "up variation" corresponding to the given coordinate, and the -1 point is the "down variation" for that coordinate. There is also the "nominal variation"  corresponding to all coordinates equalling 0. The variations themselves could be functions of the parameters but are normally just ConstHisto factors. The variation coordinates are often explicitly model parameters, but could also be functions of the model parameters (i.e. the coordinates implicitly depend on the parameters). The Histo factor has an "interpolation code" that defines how exactly to interpolates/extrapolate from these defined points to any point in the variation space. The formulae for this interpolation are:....

.. math::

   TBD

In RooFit Histfactory models Histo factors are represented by the ``PiecewiseInterpolation`` class. 

Shape Factors
^^^^^^^^^^^^^^^^
A Shape factor is equivalent to a Histo factor where there is exactly one variation coordinate for every bin in the observable, the nominal variation being a ConstHisto that 0 everywhere, and each +1 or -1 variation is a ConstHisto with a single bin being set equal to +1 or -1 respectively. Effectively this means that content of each bin of a shape factor is precisely equal to the value of one (and only one) of the variation coordinates. And normally each variation coordinate is explicitly one of the model parameters. 

In RooFit Histfactory models Shape factors are represented by ``ParamHistFunc`` class.

Overall Factors
^^^^^^^^^^^^^^^^^^
An overall factor is equivalent to a Histo Factor where all bins in the observable are the same value, i.e. this type of factor is independent of the :math:`x_{c}` observables. Additionally overall factors are restricted to their variations being unparameterized i.e. the variations must be just Const factors. In RooFit Histfactory models Shape factors are represented by ``RooStats::HistFactory::FlexibleInterpVar`` class.

Norm Factors
^^^^^^^^^^^^^^^^^^^
Really these are just the floating version of a Const factor, i.e. where a ``RooRealVar`` is used instead of a ``RooConstVar``. We can make a Histo factor equivalent to this by creating a Histo factor with just one parameter and the nominal variation being a ConstHisto with 0 everywhere, +/-1 variation being ConstHisto with +/-1 everywhere, respectively.

   
