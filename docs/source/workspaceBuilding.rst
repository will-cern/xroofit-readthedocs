Day 2: Workspace Building
*************************

Today you will learn how to build a workspace containing a model and a dataset. 

We start by creating a new ``RooWorkspace`` owned by an ``xRooNode`` smart pointer:

.. code-block:: python

  import ROOT
  import ROOT as XRF # or for ROOT's implementation of xRooFit, do: import ROOT.Experimental.XRooFit as XRF

  w = XRF.xRooNode("RooWorkspace","combined","my workspace")
  # an alternative to the above line involves creating the RooWorkspace directly and then wrapping it
  # in an xRooNode. The xRooNode will not own the workspace in this case, i.e. it wont be destroyed when
  # the node is destroyed. Example is:
  #   ws = ROOT.RooWorkspace("combined", "my workspace") # creating a RooWorkspace
  #   w = XRF.xRooNode(ws) # wrapping the workspace in the node


Anatomy of a model
==================

When we talk about a model for a dataset, we usually mean we are defining a likelihood for the dataset:

.. math::

  L(\underline{\underline{x}},\underline{a}|\underline{\theta}) = p_a(\underline{a}|\underline{\theta})\frac{\lambda(\underline{\theta})^{W}e^{-\lambda(\underline{\theta})}}{W!} \prod_{i=1}^{N} p_x(\underline{x}_i|\underline{\theta})^{w_i},

where :math:`\lambda(\underline{\theta})` is the total yield (predicted total number of events from the (extended) PDF), :math:`p_a(\underline{a})` is the constraint PDF for all the global observables, :math:`p_x(\underline{x}_i|\underline{\theta})` is the probability for the ith entry of the dataset, and :math:`w_i` is the weight of the ith entry. :math:`W` is the sum of the weights in the dataset (:math:`\sum_i w_i`).

.. _NLL Definition:
The negative log likelihood (NLL) is given by:

.. math::

  -\log L(\underline{\underline{x}},\underline{a}) = -\log p_a(\underline{a}) + \lambda - W\log(\lambda) + \log(W!) + \sum_{i=1}^{N} (-w_i\log(p_x(\underline{x}_i)))

where it is understood :math:`\lambda`, :math:`p_a(\underline{a})`, and :math:`p_x(\underline{x}_i)` are all implicitly dependent on the parameters :math:`\underline{\theta}`. The two terms :math:`\lambda - W\log(\lambda)` are collectively called the `extended term`, the :math:`-\log p_a(\underline{a})` is the `constraint term`, and the :math:`(-w_i\log(p_x(\underline{x}_i)))` is the ith `entry value`. The term :math:`\log(W!)` is known as the `dataset term` because it only depends on the dataset. It is often omitted from the NLL evaluation because it is independent of the parameters. The part of the NLL that is not the constraint term is known collectively as the `main term` (i.e. it is made up of the extended term and the sum of the entry values, and any dataset terms). 

The constraint term is usually (but not always) a product of individual PDFs for each of the global observables. Typically they are one of two types: Gaussian, or Poisson. The latter are almost exclusive used for constraints on MC-stat nuisance parameters (often known as "gamma" parameters). Normally-constrained nuisance parameters (known as "alpha" parameters) use a gaussian constraint with variance of 1 and global observable nominal value of 0. In HistFactory models, the luminosity parameter is the only parameter that receives a gaussian constraint with a variance different to 1 and nominal observed value different to 0. 

Top-level multi-channel PDF
---------------------------
Our attention now turns to the :math:`p_x(\underline{x}|\underline{\theta})` probability density. This PDF is conventionally split up into channels (sometimes called *regions*), meaning that there is a discrete observable (traditionally called ``channelCat``) in the dataset that determines which channel PDF should be used for the entry (i.e. which channel the entry *belongs* to). In RooFit, this channel-dependent PDF is represented by an instance of the ``RooSimultaneous`` PDF class. To create a new PDF of this type with xRooFit, just do:

.. code-block:: python

  w["pdfs"].Add("simPdf") # simPdf is the traditional name of a multi-channel PDF, but can be any name.

Here we see that xRooFit defaults to assuming any new pdf you might want to create will be a multi-channel PDF.

Channels
---------
We now denote the the discrete channel observable as :math:`c`, and we write

.. math::

  p_x(\underline{x}|\underline{\theta}) = \frac{\lambda_c(\underline{\theta})}{\sum_j\lambda_j(\underline{\theta})}p_c(\underline{x}|\underline{\theta})

where :math:`p_c(\underline{x}|\underline{\theta})` is the channel's PDF, and :math:`\lambda_c(\underline{\theta})` is the yield (predicted number of events) for the channel :math:`c`. Note that :math:`p_c` will not be a PDF for the regular obserable :math:`c`, i.e. that regular observable is effectively ignored by the channel PDF.

To add a new channel to our multi-channel PDF, do e.g.:

.. code-block:: python

  w["pdfs/simPdf"].Add("SR").SetTitle("Signal Region") # adds the channel "SR" to the "simPdf" top-level pdf, and gives it a title

In xRooFit, a channel PDF is represented by a RooFit `RooProdPdf` which is a PDF class that can represent a product of PDFs. For technical reasons, this `RooProdPdf` will end up containing the constraint term PDFs (meaning if we evaluate the PDF it will include the constraint term contributions).

We now will see how a channel's PDF can be built out of samples ....

Samples
---------
Channels can be built out of samples. A sample is a sub-component of a channel, such that the PDF of the channel corresponds to the sum over the samples multiplied by some optional coefficients:

.. math::

  p_{c}(\underline{x}|\theta) = \frac{\sum_s c_{cs}(\theta)\phi_{cs}(\underline{x}|\theta)}{\int\sum_s c_{cs}\phi_{cs}(\underline{x}|\theta)dx}
  
where :math:`c_{cs}(\theta)` are known as the `coefficients` of the sample :math:`s` that appears in channel :math:`c` (technical points: the coefficients are "owned" by the channel rather than the sample). 

In RooFit the above PDF is represented by ``RooRealSumPdf`` (if the :math:`\phi_{cs}` are functions) or ``RooAddPdf`` (if the :math:`\phi_{cs}` are all PDFs, in which case the samples are known as 'components' and the coefficients will correspond to the yield for each component).

In the case where the :math:`\phi_{cs}` are functions, we can usually write this function as a product of factors that depend on the observables. The coefficients are also types of factor that do not depend on the observables.

To add a new sample to a channel, the channel must have a regular observable declared, with a binning. 

.. code-block:: python

  w["pdfs/simPdf/SR"].SetXaxis("obsName","obs title",nBins,low,high) # declare a regular observable for the channel
  w["pdfs/simPdf/SR/samples"].Add("bkg") # adds a "bkg" sample to the "SR" channel in the "simPdf" pdf

Alternatively, you can use a ROOT histogram to achieve the same results:

.. code-block:: python

  hBkg = ROOT.TH1D("bkg","Background;obs title",nBins,low,high)
  hBkg.GetXaxis().SetName("obsName") # note: ROOT's default x-axis name is 'xaxis' which will be used as the robs name otherwise
  w["pdfs/simPdf/SR/samples"].Add(hBkg)

Both of these approaches will create an initial `SimpleDensity` factor for the sample (see below). Subsequent factors can be included by multiplying the sample. 

.. note:: Samples created from histograms with bin errors:
   If you use a histogram to create a sample, any bin errors in the histogram will automatically trigger the creation or modification of a ShapeSys (a constrained shape factor, so next section) representing the statistical uncertainty. If you do not want this to happen, ensure your bins have 0 errors in them. You can also control which shapesys the bin errors contribute to by specifying the ``statPrefix`` option on the histogram, e.g. do ``hBkg.SetOption("statPrefix=stat_SR_bkg")``. The default statPrefix is effectively ``stat_<channelName>``. 

Factors
--------
As stated above, there are two types of factors: observable-dependent, and observable-independent. Conventionally, the observable-independent factors of a sample are made  the coefficients of the sample (:math:`c_{cs}(\theta)`), while the sample itself is just made from the observable-dependent factors (:math:`\phi_{cs} = \prod_k f^{(k)}_{cs}(\underline{x}|\theta)`).

Furthermore, a factor can be parameterized (:math:`\theta`-dependent) or unparameterized. 

Here are the basic factor types:

  * `Const` factor: An observable-independent pre-specified parameter or constant. RooFit class: ``RooConstVar``.
  * `Norm` factor: An observable-independent floatable parameter. RooFit class: ``RooRealVar``.
  * `Simple` factor: An observable-dependent parameter-independent function. Commonly represents a histogram of bin yields. RooFit class: ``RooHistFunc``.
  * `Density` factor: A special case of Simple factor where the bin value is equal to 1/binWidth. RooFit class: ``RooBinWidthFunction``.
  * `Shape` factor: A parameterized and observable-dependent factor where each bin in the observable is scaled by an individual norm factor. RooFit class: ``ParamHistFunc``

The above factor types can be created and included in a sample as follows:

.. code-block:: python

  w["pdfs/simPdf/SR/samples/bkg"].Multiply("myFactor","shape") # multiplying the sample by an observable-depdenent factor type
  w["pdfs/simPdf/SR/samples/bkg"].coefs().Multiply("mu_bkg","norm") # observable-independent factors conventionally go in as coefficients

If a factor with the same name already exists in the workspace, the factor type is ignored and the existing factor is used. This allows factors to be shared between samples both in the same channel and across channels. 

Parameterizing factors (Varied factors)
--------
Other than the trivial case where the factor is a parameter itself (i.e. norm factor), there are a multitude of ways we could make a factor :math:`\theta`-dependent. One strategy is to define a collection of "variations" for the factor (the variations are themselves types of factor), locate them at points in a "variation space" with parameterized coordinates, and provide interpolation+extrapolation rules to calculate the value of the factor at any point in the variation space. Very commonly the variation coordinates will explicitly be parameters, and the points for which variations are defined will correspond to points where one of the coordinates equals either +1 or -1 and the remaining coordinates are 0. The +1 variation is called the `up` variation of that coordinate, and -1 variation is the `down` variation. Additionally the point where all the coordinates are 0 will be known as the "nominal" variation. These `Varied` factors can be created in xRooFit by `varying` one of the parameter-independent factors above.

So the additional factor types are:

  * `Varied` factor: A parameterized factor with variations and an interpolation+extrapolation rule. RooFit class: ``PiecewiseInterpolation``.
     * `Overall` factor: A special case of Varied factor where the variations are const factors. RooFit class: ``RooStats::HistFactory::FlexibleInterpVar`` or ``PiecewiseInterpolation``.
     * `Histo` factor: A special case of Varied factor where the variations are simple factors. RooFit class: ``PiecewiseInterpolation``.
  * `Func` factor: a generic parametric function. RooFit class: ``RooFormulaVar``. 

To vary an existing factor, you can do:

.. code-block:: python

  # in this example the sample called `sampleName` is given a variation in its 2nd bin, corresponding to a shift
  # in the parameter called `npName` from value 0 to value 1.
  w["pdfs/simPdf/SR/samples/sampleName"].SetBinContent(2,val,"npName",1)

Alternatively, you can use a ROOT histogram as a variation (**important: you will want to ensure there are no bin errors on your variation histogram, otherwise xRooFit will try to create errors-on-errors which are not fully supported at this time**):

.. code-block:: python

  hVaryHist.SetName("npName=1") # must follow convention here of npName followed by a value, conventionally the nSigma of the variation
  w["pdfs/simPdf/SR/samples/sampleName"].Vary(hVaryHist)

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


.. list-table:: Interpolation Schemes
    :widths: 25 10 55 10
    :header-rows: 1

    * - Code
      - Name
      - Definition
      - Notes

    * - 0
      - Additive Piecewise Linear 
      - :math:`I_0(\theta;x_{-},x_0,x_{+}) = \begin{cases}\theta(x_{+} - x_0) & \text{if} \theta>=0 \\ \theta(x_0 - x_{-}) & \text{otherwise}\end{cases}`
      - **Not recommended** except if using a symmetric variation, because of discontinuities in derivatives.

    * - 1             
      - Multiplicative Piecewise Exponential 
      - :math:`I_1(\theta;x_{-},x_0,x_{+}) = \begin{cases}(x_{+}/x_0)^{\theta} & \text{if} \theta>=0 \\ (x_{-}/x_0)^{-\theta} & \text{otherwise}\end{cases}`
      - **Not recommended.**

    * - 4
      - Additive Poly Interp. + Linear Extrap
      - :math:`I_4(\theta;x_{-},x_0,x_{+}) = \begin{cases}I_0(\theta;x_{-},x_0,x_{+}) & \text{if} |\theta|>=1 \\ \sum_{i=1}^6 a_i\theta^i & \text{otherwise}\end{cases}`
      - :math:`a_i` such that matching 0th,1st,2nd derivatives at :math:`|\theta|=1` boundaries. **Recommended for histo factors**.

    * - 5
      - Multiplicative Poly Interp. + Exponential Extrap.
      - :math:`I_5(\theta;x_{-},x_0,x_{+}) = \begin{cases}I_1(\theta;x_{-},x_0,x_{+}) & \text{if} |\theta|>=1 \\ 1 +\sum_{i=1}^6 a_i\theta^i & \text{otherwise}\end{cases}`
      - :math:`a_i` such that matching 0th,1st,2nd derivatives at :math:`|\theta|=1` boundaries. Recommended for normalization factors. In FlexibleInterpVar this is interpCode=4. **Recommended for overall factors**.

    * - 6
      - Multiplicative Poly Interp. + Linear Extrap.
      - :math:`I_6(\theta;x_{-},x_0,x_{+}) = 1+I_4(\theta;x_{-},x_0,x_{+})`. 
      - Recommended for normalization factors that must not have roots (i.e. be equal to 0) outside of :math:`|\theta|<1`.

To check or change an interpolation scheme of a `Histo` factor you can do:

.. code-block:: python

  w["pdfs/simPdf/SR/samples/sampleName/sampleName"].printAllInterpCodes() # list interpCode of the DensityHisto factor
  w["pdfs/simPdf/SR/samples/sampleName/sampleName"].setAllInterpCodes(code) # change the interp code of the DensityHisto factor

Factors vs Sys
--------------
When any of the parameters of a parameter-dependent factor also has a constraint term, the phrase `factor` can be replaced by `sys`, e.g. a `ShapeFactor` becomes a `ShapeSys`.

To promote a factor to a sys we would just add a constraint to the parameter(s) of the factor. E.g.

.. code-block:: python

  w["pdfs/simPdf"].pars()["npName"].Constrain("gaussian(0,1)") # create a gaussian constraint on the `npName` parameter of the model 
  w["pdfs/simPdf"].pars()["npName"].Constrain("normal") # alias to above

.. note:: Log-normal constraints:
   Sometimes you will encounter the phrase *log-normal* constraint. This is equivalent to replacing the parameter in the model with the exponentiated version of that parameter, and then applying a normal gaussian constraint to the parameter. The replacement in the model can be achieved with ``w["pdfs/simPdf"].pars()["npName"].Replace("expr::expo_npName('exp(npName)',npName)")``


Handling MC Stat Errors: A special ShapeSys for MC Stat errrors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The standard way to incorporate MC stat uncertainties on the sample nominal histograms is to have a dedicated ShapeSys for the uncertainty, where the parameters are constrained by Poissons (they are hence known as :math:`\gamma` mc stat parameters). This ShapeSys is nominally shared between all the samples in the channel that are participating in the calculation of the total MC stat uncertainty. However, I would advise that any sample that gets its own Norm factor should also have its own separate ShapeSys (or none at all), rather than share the ShapeSys. This is because the affect of the normalization factor isn't accounted for in the calculation of the Poisson constraint term unless the sample has its own stat ShapeSys with its own :math:`\gamma` parameters.

The complete model likelihood
---------
Combining the factors, samples, and channels together into a single likelihood gives:

.. math::

  L(\underline{\underline{x}},\underline{a}|\underline{\theta}) = p_a(\underline{a}|\underline{\theta})\frac{\lambda(\underline{\theta})^{W}e^{-\lambda(\underline{\theta})}}{W!} \prod_{i=1}^{N} \left(\frac{\lambda_{c_i}(\underline{\theta})}{\sum_j\lambda_j(\underline{\theta})}\frac{\sum_s c_{c_is}(\theta)\prod_k f^{(k)}_{c_is}(\underline{x}_i|\theta)}{\int\sum_s c_{c_is}(\theta)\prod_k f^{(k)}_{c_is}(\underline{x}|\theta)dx}\right)^{w_i}

where the product over :math:`k` is for the observable-dependent factors in the sample in the channel, and the :math:`c_{c_is}` coefficient is the product of the observable-independent factors in the sample in the channel. Conventionally the yield of the channel, :math:`\lambda_{c_i}`, is the same as the normalization term for the channel, :math:`\int\sum_s c_{c_is}\prod_k f^{(k)}_{c_is}(\underline{x}|\theta)dx`, and hence the likelihood can also be written as:

.. math::

  L(\underline{\underline{x}},\underline{a}|\underline{\theta}) = p_a(\underline{a}|\underline{\theta})\frac{\lambda(\underline{\theta})^{W}e^{-\lambda(\underline{\theta})}}{W!} \prod_{i=1}^{N} \left(\frac{\sum_s c_{c_is}(\theta)\prod_k f^{(k)}_{c_is}(\underline{x}_i|\theta)}{\lambda(\underline{\theta})}\right)^{w_i}.

Binned datasets
^^^^^^^^^^^^^^^
For the special case where the dataset is a `binned dataset`, the :math:`x_i` each correspond to a different bin center, and the :math:`w_i` are the observed yields in that bin. The prediction in the ith bin of the model (which is in channel c) is given by :math:`\lambda_{i} = \Delta_{i}\sum_s c_{c_is}(\theta)\prod_k f^{(k)}_{c_is}(\underline{x}_i|\theta)` where :math:`\Delta_{i}` is the width of the bin and  :math:`\underline{x}_i` are the coordinates of the bin center. Hence the likelihood can be written as:

.. math::

  L(\underline{\underline{x}},\underline{a}|\underline{\theta}) = p_a(\underline{a}|\underline{\theta})\frac{\lambda(\underline{\theta})^{W}e^{-\lambda(\underline{\theta})}}{W!} \prod_{i=1}^{N} \left(\frac{\lambda_{i}}{\lambda\Delta_{i}}\right)^{w_i}

After some manipulation this can be shown to be equal to:

.. math::

  L(\underline{\underline{x}},\underline{a}|\underline{\theta}) = p_a(\underline{a}|\underline{\theta})\frac{1}{W!}\prod_{i=1}^{N} \frac{e^{-\lambda_i}\lambda_i^{w_i}}{w_i!}\frac{w_i!}{\Delta_i^{w_i}}

The second part of product, :math:`\frac{w_i!}{\Delta_i^{w_i}}` is independent of the parameters and hence can be included in the `dataset term` of the NLL, and the first part of the product then amounts to just a product of Poissons. Hence people will often claim their model is a product of Poissons, with a constraint PDF for nuisance parameters constrained by auxilliary measurements (aka global observables).


Creating datasets in the workspace
==================
If you have a histogram representing the observed yield in a particular channel of the model you are working on, you can add that histogram's content to a dataset in the workspace by first ensuring the histogram's name matches the desired dataset name, and then adding the histogram to the ``datasets()`` of the channel:


.. code-block:: python

  hData.SetName("obsData")
  w["pdfs/simPdf/channelName"].datasets().Add(hData)

Alternatively, you can specify data content bin-by-bin as follows:

.. code-block:: python

  w["pdfs/simPdf/channelName"].SetBinData(binNumber, value, dsName) # dsName defaults to "obsData" if unspecified

