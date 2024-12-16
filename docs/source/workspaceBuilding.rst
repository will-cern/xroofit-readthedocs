Day 2: Workspace Building
*************************

Today you will learn how to build a workspace containing a model and a dataset. 


Anatomy of a model
==================

When we talk about a model for a dataset, we usually mean we are defining a likelihood for the dataset:

.. math::

  L(\underline{\underline{x}},\underline{a}|\underline{\theta}) = \frac{\lambda(\underline{\theta})^{N}e^{-\lambda(\underline{\theta})}}{N!} p(\underline{a}|\underline{\theta})\prod_{i=1}^{N} p(\underline{x}_i|\underline{\theta})^{w_i},

where :math:`\lambda(\underline{\theta})` is the total yield (predicted total number of events from the (extended) PDF), :math:`p_a(\underline{a})` is the constraint PDF for all the global observables, :math:`p_x(\underline{x}_i|\underline{\theta})` is the probability for the ith entry of the dataset, and :math:`w_i` is the weight of the ith entry. 

The negative log likelihood is given by:

.. math::

  -\log L(\underline{\underline{x}},\underline{a}) = \lambda - N\log(\lambda) + \log(N!) + (-\log p_a(\underline{a})) + \sum_{i=1}^{N} (-w_i\log(p_x(\underline{x}_i)))

where it is understood :math:`\lambda`, :math:`p_a(\underline{a})`, and :math:`p_x(\underline{x}_i)` are all implicitly dependent on the parameters :math:`\underline{\theta}`. The first two terms :math:`\lambda - N\log(\lambda)` are collectively called the `extended term`, the :math:`(-\log p_a(\underline{a}))` is the `constraint term`, and the :math:`(-w_i\log(p_x(\underline{x}_i)))` is the ith `entry term`.

The constraint term is usually (but not always) a product of individual PDFs for each of the global observables. Typically they are one of two types: Gaussian, or Poisson. The latter are almost exclusive used for constraints on MC-stat nuisance parameters (often known as "gamma" parameters). Normally-constrained nuisance parameters (known as "alpha" parameters) use a gaussian constraint with variance of 1 and global observable nominal value of 0. In HistFactory models, the luminosity parameter is the only parameter that receives a gaussian constraint with a variance different to 1 and nominal observed value different to 0. 

Our attention now turns to :math:`p_x(\underline{x}|\underline{\theta})`, which is conventionally split up into channels ...

Channels
---------
PDFs of models are usually factorised into channels, which means that there is a discrete regular observable (traditionally called ``channelCat``) that indicates which channel each entry in the dataset belongs to. If we denote this observable as :math:`c`, then we can write

.. math::

  p_x(\underline{x}|\underline{\theta}) = \frac{\lambda_c(\underline{\theta})}{\sum_j\lambda_j(\underline{\theta})}p_c(\underline{x}|\underline{\theta})

where :math:`p_c(\underline{x}|\underline{\theta})` is the channel's PDF, and :math:`\lambda_c(\underline{\theta})` is the yield (predicted number of events) for the channel :math:`c`. Note that :math:`p_c` will not be a PDF for the regular obserable :math:`c`, i.e. that regular observable is effectively ignored by the channel PDF.

PDFs which are factorized into channels are represented in RooFit by the class ``RooSimultaneous``.

We now will see how a channel's PDF can be built out of samples ....

Samples
---------
Channels can be built out of samples. A sample is a sub-component of a channel, such that the PDF of the channel corresponds to the sum over the samples multiplied by some optional coefficients:

.. math::

  p_{c}(\underline{x}|\theta) = \frac{\sum_s c_{cs}(\theta)f_{cs}(\underline{x}|\theta)}{\int\sum_s c_{cs}f_{cs}(\underline{x}|\theta)dx}
  
where :math:`c_{cs}(\theta)` are known as the `coefficients` of the sample :math:`s` that appears in channel :math:`c` (technical points: the coefficients are "owned" by the channel rather than the sample). 

In RooFit the above PDF is represented by `RooRealSumPdf` (if the :math:`f_s` are functions) or `RooAddPdf` (if the :math:`f_s` are all PDFs, in which case the samples are known as 'components' and the coefficients will correspond to the yield for each component).

In the case where the :math:`f_s` are functions are functions, we can usually write this function as a product of factors that depend on the observables. The coefficients are also types of factor that do not depend on the observables.

Factors
--------
As stated above, there are two types of factors: observable-dependent, and observable-independent. Conventionally, the observable-independent factors of a sample are made  the coefficients of the sample (:math:`c_{cs}(\theta)`), while the sample itself is just made from the observable-dependent factors (:math:`f_{cs}(\underline{x}|\theta)`).

Furthermore, a factor can be parameterized (:math:`\theta`-dependent) or unparameterized. Other than the trivial case where the factor is a parameter itself, there are a multitude of ways we could make a factor :math:`\theta`-dependent. One strategy is to define a collection of "variations" for the factor (the variations are themselves types of factor), locate them at points in a "variation space" with parameterized coordinates, and provide interpolation+extrapolation rules to calculate the value of the factor at any point in the variation space. Very commonly the variation coordinates will explicitly be parameters, and the points for which variations are defined will correspond to points where one of the coordinates equals either +1 or -1 and the remaining coordinates are 0. The +1 variation is called the `up` variation of that coordinate, and -1 variation is the `down` variation. Additionally the point where all the coordinates are 0 will be known as the "nominal" variation.

Here is a list of types of factors:

  * `Const` factor: An observable-independent pre-specified parameter or constant. RooFit class: ``RooConstVar``.
  * `Norm` factor: An observable-independent floatable parameter. RooFit class: ``RooRealVar``.
  * `Simple` factor: An observable-dependent parameter-independent function. Commonly represents a histogram of bin yields. RooFit class: ``RooHistFunc``.
  * `Density` factor: A special case of Simple factor where the bin value is equal to 1/binWidth. RooFit class: ``RooBinWidthFunction``.
  * `Varied` factor: A parameterized factor with variations and an interpolation+extrapolation rule RooFit class: ``PiecewiseInterpolation``.
    * `Overall` factor: A special case of Varied factor where the variations are const factors. RooFit class: ``RooStats::HistFactory::FlexibleInterpVar`` or ``PiecewiseInterpolation``.
    * `Histo` factor: A special case of Varied factor where the variations are simple factors. RooFit class: ``PiecewiseInterpolation``.
  * `Shape` factor: A parameterized and observable-dependent factor where each bin in the observable is scaled by an individual norm factor. RooFit class: ``ParamHistFunc``

When any of the parameters of a parameter-dependent factor also have a constraint term, the phrase `factor` can be replaced by `sys`, e.g. a `ShapeFactor` becomes a `ShapeSys`.

Interpolation and Extrapolation Rules of Varied Factors
^^^^^^^^^^^^^^
Varied factors have an "interpolation code" that determines its interpolation and extrapolation rule/scheme for a given parameter. Normally all the parameters in a varied factor will have the same interpolation code.

The equation for a varied factor with nominal variation :math:`f_0(x)` and up/down variations of :math:`f_{i+}(x)`/:math:`f_{i-}(x)` for parameter :math:`\theta_i` with interpolation code :math:`c_i` is:

.. math::

  f(x|\underline{\theta}) = f_0(x) + \sum_i I_{c_i}(\theta_i;f_{i-}(x), f_{0}(x), f_{i+}(x))

for additive interpolation codes and

.. math::

  f(x|\underline{\theta}) = f_0(x)\prod_i I_{c_i}(\theta_i;\frac{f_{i-}(x)}{f_{0}(x)}, 1, \frac{f_{i+}(x)}{f_{0}(x)})

for multiplicative interpolation codes, where the code types and interpolation functions are defined in the following table:




A Varied factor is a parameterized (i.e. -:math:`\theta`-dependent) function on one or more parameters. It should be thought of as being built up of a set of "variations" that correspond to particular points in the "variation space", specifically the points where one of the "variation coordinates" equals +1 or -1 and the remaining "variation coordinates" are all 0. The +1 point is the "up variation" corresponding to the given coordinate, and the -1 point is the "down variation" for that coordinate. There is also the "nominal variation"  corresponding to all coordinates equalling 0. The variations themselves could be functions of the parameters but are normally just ConstHisto factors (in which case the Varied factor is called a `Histo` factor). The variation coordinates are often explicitly model parameters, but could also be functions of the model parameters (i.e. the coordinates implicitly depend on the parameters). The Varied factor has an "interpolation code" that defines how exactly to interpolates/extrapolate from these defined points to any point in the variation space. The formulae for this interpolation are:....

.. math::

   TBD

In RooFit Histfactory models Histo factors are represented by the ``PiecewiseInterpolation`` class. 

Shape Factors
^^^^^^^^^^^^^^^^
A Shape factor is equivalent to a Histo factor where there is exactly one variation coordinate for every bin in the observable, the nominal variation being a Simple factor that is 1 everywhere, and each +1 or -1 variation is a Simple factor with a single bin being set equal to +1 or -1 respectively. Effectively this means that content of each bin of a shape factor is precisely equal to the value of one (and only one) of the variation coordinates. And normally each variation coordinate is explicitly one of the model parameters. 

In RooFit Histfactory models Shape factors are represented by ``ParamHistFunc`` class.

Overall Factors
^^^^^^^^^^^^^^^^^^
An overall factor is equivalent to a Histo Factor where all bins in the observable are the same value, i.e. this type of factor is independent of the :math:`x_{c}` observables. Additionally overall factors are restricted to their variations being unparameterized i.e. the variations must be just Const factors. In RooFit Histfactory models Shape factors are represented by ``RooStats::HistFactory::FlexibleInterpVar`` class.

Norm Factors
^^^^^^^^^^^^^^^^^^^
Really these are just the floating version of a Const factor, i.e. where a ``RooRealVar`` is used instead of a ``RooConstVar``. We can make a Histo factor equivalent to this by creating a Histo factor with just one parameter and the nominal variation being a ConstHisto with 0 everywhere, +/-1 variation being ConstHisto with +/-1 everywhere, respectively.

   
