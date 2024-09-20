Day 2: Workspace Building
*************************

Today you will learn how to build a workspace containing a model and a dataset. 


Anatomy of a model
==================

.. note:: 
  In this section :math:`\theta` is used as the symbol to represent *all* types of parameter rather than just the nuisance parameters.



Channels
---------
Models are factorised into channels, which means that there is a discrete regular observable (traditionally called ``channelCat``) that indicates which channel each entry in the dataset belongs to. If we denote this observable as :math:`c` then the PDF (i.e. the model) is given by:

.. math::

  p(x|\theta) = \prod_g p_g(a_{g}|\theta)\prod_{i=0}^{N} p(x_{i}|\theta) = \prod_g p_g(a_{g}|\theta)\prod_c\prod_{j=0}^{N_{c}} p(x_{j}|\theta) = \prod_g p_g(a_{g}|\theta)\prod_i\left[\frac{\lambda_{c}(\theta)}{\sum_c \lambda_{c}(\theta)}p_{c}(x_{ic}|\theta)\right]



Samples
---------
Channels can be built out of samples. A sample is just a sub-component of a channel, such that the PDF of the channel corresponds to the sum over the samples:

.. math::

  p_{c}(x_{c}|\theta) = \frac{\sum_s c_{cs}f_s(x_{c}|\theta)}{\int\sum_s c_{cs}f_s(x_{c}|\theta)dx_c}
  
where :math:`c_{cs}` are known as the `coefficients` of the sample :math:`s` that appears in channel :math:`c` (technical points: the coefficients are "owned" by the channel rather than the sample). 

In RooFit the above PDF is represented by `RooRealSumPdf` (if the :math:`f_s` are functions) or `RooAddPdf` (if the :math:`f_s` are all PDFs, in which case the coefficients correspond to the yield for each sample).

Factors
--------
Samples can be built out of factors i.e. :math:`f_{s}(x_{c},\theta) = \prod_f f_{f}(x_{c}|\theta)` where :math:`f_{f}(x_{c}|\theta)` are the individual factors that are themselves functions of the channel observables :math:`x_{c}` and/or the model parameters :math:`\theta`.

The most fundamental type of factor we can imagine is literally just a fixed number. We will call this a `Const` factor. To go from this starting point to any other type of factor we conceptually need ways to make our factor either :math:`x_{c}`-dependent and/or :math:`p`-dependent.

We will call the discretely-:math:`x_{c}`-dependent (i.e. binned in the observable) version of a `Const` factor a `ConstHisto` factor. 

There are a multitude of ways we could make a factor :math:`\theta`-dependent. One strategy is to define a collection of "variations" for the factor (the variations can be arbitrary function but we can make them be a type of factor), locate them at points in a "variation space" with parameterized coordinates, and provide an interpolation/extrapolation rule to calculate the value of the factor at any point in the variation space. Very commonly the variation coordinates will explicitly be model parameters, and the points for which variations are defined will correspond to points where one of the coordinates equals either +1 or -1 and the remaining coordinates are 0. The +1 variation is called the `up` variation of that coordinate, and -1 variation is the `down` variation. Additionally the point where all the coordinates are 0 will be known as the "nominal" variation. We will call this type of factor a `Varied` factor. 

So far we have defined Const (:math:`x_{c}`- and :math:`\theta`-independent), ConstHisto (:math:`x_{c}`-dependent), and Varied (:math:`\theta`-dependent) factors. We will now define some special cases in terms of these generic factors:

   * `Histo` factor: a Varied factor where all the variations are `ConstHisto` factors.
   * `Shape` factor: a `Histo` factor with one explicit parameter variation-coordinate for each bin, with the nominal variation being 0 everywhere and each parameter +/-1 variation being 0 everywhere except for the single corresponding bin, which takes on value +/-1.
   * `Overall` factor: a Varied factor where all the variations are `Const` factors.
   * `Norm` factor: an `Overall` factor with exactly one parameter and the +/-1 variation is +/-1 (and nominal=0). This is equivalent to the factor being just the parameter explicitly. 

Let's re-iterate each of the factor types in turn....

Const Factors
^^^^^^^^^^^^^^
This is just a fixed number. These are represented in RooFit by ``RooConstVar`` and amount to just a scale factor.

ConstHisto Factors
^^^^^^^^^^^^^^
This is a discretely-:math:`x_{c}`-dependent (discretely = binned in the observable) unparameterized function, i.e. it's just a histogram over the observable. ConstHisto factors are often used as the variations of a Histo factor (which is how the Histo factor becomes implicitly dependent on the observable). In RooFit a ``ConstHisto`` factor is represented by the ``RooHistFunc`` class.

Varied Factors
^^^^^^^^^^^^^^
A Varied factor is a parameterized (i.e. -:math:`p`-dependent) function on one or more parameters. It should be thought of as being built up of a set of "variations" that correspond to particular points in the "variation space", specifically the points where one of the "variation coordinates" equals +1 or -1 and the remaining "variation coordinates" are all 0. The +1 point is the "up variation" corresponding to the given coordinate, and the -1 point is the "down variation" for that coordinate. There is also the "nominal variation"  corresponding to all coordinates equalling 0. The variations themselves could be functions of the parameters but are normally just ConstHisto factors (in which case the Varied factor is called a `Histo` factor). The variation coordinates are often explicitly model parameters, but could also be functions of the model parameters (i.e. the coordinates implicitly depend on the parameters). The Varied factor has an "interpolation code" that defines how exactly to interpolates/extrapolate from these defined points to any point in the variation space. The formulae for this interpolation are:....

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

   
