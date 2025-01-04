Day 3: NLL Construction and Minimization
========================================

This is otherwise known as the process of fitting a model to a dataset. The output of this process is a `Fit Result`.

>>> fitResult = w["pdfs/pdfName"].nll("datasetName").minimize()

Constructing the NLL Function
----------------------------
The NLL is the most commonly used :ref:`objective function <objective functions>` for fitting. It is constructed using the ``nll`` method of a pdf node in the workspace:

>>> nll = w["pdfs/pdfName"].nll("datasetName",[list,of,nll,options])

The list of NLL options is optional. The most important NLL options, which affect how the NLL is actually defined, are described in the :ref:`section below <objective functions>`. 

Whatever the NLL options are, the NLL function can always be factorized into two parts: the *main term* and the *constraint term*. The main term depends on the :ref:`regular observables <regular observables>` and the constraint term depends on the :ref:`global observables <global observables>`. See :ref:`NLL Definition <NLL Definition>` for more info.

.. _nll options:
NLL Options
^^^^^^^^^^^
These are passed to the ``nll`` method (alongside the dataset name) and determine specifically how the NLL objective function is constructed ...


.. _minimization:
Running a minimization
----------------------

To minimize the NLL function, just call its ``minimize`` method, which will return a ``FitResult`` object:

>>> fr = nll.minimize()

You can draw the fit result (``fr.Draw()``) to see the post-fit parameter pulls. 

The Fit Config
^^^^^^^^^^^^^^
The ``minimize`` method accepts an optional fit configuration that contains hyperparameters that steer the minimization. The nll function has its own copy of the fit config that is used if you do not pass your own fit config object to minimize. The settings in the fit config that is stored in the nll can be viewed by using ``nll.Print()``. The table below describes what each config option does and how to set it (the first column is what to pass in the NLL option list to the :ref:`nll method <nll options>`, the second column is how to change the setting after creating the nll object). In each case, the default value is given as the example setting:

.. list-table:: Fit Config settings
    :widths: 25 10 65
    :header-rows: 1

    * - NLL Option
      - Setting after nll creation
      - Description
    * - ``XRF.xRooFit.Tolerance(0.01)``
      - ``nll.fitConfig().MinimizerOptions().SetTolerance(0.01)``
      - 1000 times the maximum allowed value of the edm (estimated distance to minimum) of the fit before the fit is considered converged. E.g. the default value of 0.01 means that the edm must become less than 1e-5 for convergence. If this is not reached, the migrad status code will be 3.
    * - ``ROOT.RooFit.Strategy(-1)``
      - ``nll.fitConfig().MinimizerOptions().SetStrategy(-1)``
      - The starting minuit strategy. If set to -1 (the default), the starting strategy is the start of the StrategySequence setting (see below). 
    * - ``XRF.xRooFit.StrategySequence("0s01s12s2s3m")``
      - ``nll.fitConfigOptions().SetValue("StrategySequence","0s01s12s2s3m")``
      - Determines the order of retries automatically performed if a fit fails. A number indicates a strategy setting, `s` indicates a rescan, and `m` indicates a switch to minuit1 (which will soon be deprecated). For example, a strategy sequence of "0s01s12s2m" means that if a strategy=0 fit fails it will try a rescan and then try the strategy=0 fit again, if that fails it will switch to strategy=1, and so on. 
    * - ``ROOT.RooFit.Hesse(True)``
      - ``nll.fitConfig().SetParabErrors(True)``
      - Controls if hesse should be run after the migrad minimization (if it wasn't already run with the necessary level of precision by the migrad minimization, which can sometimes happen and xRooFit will automatically determine this). If it is not run, the covariance matrix may not be accurate (quality != 3).
    * - n/a
      - ``nll.fitConfigOptions().SetValue("HesseStrategy",-1)``
      - Controls which strategy is used first when hesse algorithm is run. If -1, will take first strategy in the HesseStrategySequence (see below)
    * - n/a
      - ``nll.fitConfigOptions().SetValue("HesseStrategySequence","23")``
      - Similar to the StrategySequence setting, this controls the order of attempts made in the hesse algorithm, with an example of hesse failure being e.g. a non-positive definite covariance matrix (covQuality=1 in the case of hesse strategy 3 in the fit result). 

For example, to make the tolerance equal to 1 and the starting strategy equal to 1, you can do (assumes you have done e.g. ``import ROOT as XRF`` if using xRooFit compiled on top of ROOT):

>>> nll = w["pdfs/pdfName"].nll("datasetName",[XRF.xRooFit.Tolerance(1),ROOT.RooFit.Strategy(1)])

Or equivalently you can do:

>>> nll = w["pdfs/pdfName"].nll("datasetName")
>>> nll.fitConfig().MinimizerOptions().SetTolerance(1)
>>> nll.fitConfig().MinimizerOptions().SetStrategy(1)

Goodness of fit
^^^^^^^^^^^^^^^
xRooFit uses the ``saturated model`` to compute a goodness of fit (g.o.f) p-value for any state of the NLL function. First the NLL function is evaluated, then the NLL is effectively re-evaluated for a hypothetical scenario where the pdf is able to describe the data perfectly. For binned data, this scenario corresponds to the case where the prediction of the model in each bin was exactly equal to the dataset yield in that bin. For unbinned data, this scenario corresponds to the model where :math:`p(\underline{x}_i)=\frac{w_i}{\sum w_i}`. The difference between the two NLL values, multiplied by two, is called the ``saturated model likelihood ratio`` test statistic. It is then assumed that this test statistic is :math:`\chi^2` distributed with an appropriate choice of the number of degrees of freedom, which allows us to compute a p-value for the test statistic value. 

If the above calculation is performed with just the main term of the NLL, the number of degrees of freedom is equal to the number entries in the dataset (for binned data, this is the same as the number of bins in the model) minus the number of unconstrained parameters in the main term (i.e. parameters that do not appear in the constraint term). All of this information is accessed in xRooFit as follows:

.. code-block:: python

  nll.mainTerm().getVal() # the current value of the main term of the NLL
  nll.saturatedMainTerm() # the value of the mainTerm in the hypothetical scenario of a perfect model
  nll.mainTermNdof() # the number of degrees of freedom (nBins - nUnconstrained in the case of a binned model)
  nll.mainTermPgof() # = ROOT.TMath.Prob( 2*(nll.mainTerm().getVal() - nll.saturatedMainTerm()), nll.mainTermNdof() )

It is also possible to do the above calculation with the constraint term included; the constraint term can also have a hypothetical scenario where all its predictions exactly equal the global observable vaues. In this case the number of degrees of freedom is the number of entries in the dataset plus the number of global observables minus the number of floating parameters in the whole pdf. However, due to the way nominal global observable values are chosen for observed dataset (e.g. all normal-constraints corresponding to global observables use 0 for the global observable value in the observed dataset), such a g.o.f. p-value is biased towards larger values for the observed datasets. For a toy dataset, however, the p-value should be valid. Below are the methods for this version of the g.o.f calculation:

.. code-block:: python

  nll.getVal() # the current value of the NLL
  nll.saturatedVal() # the value of the NLL in the hypothetical
  nll.ndof() # the number of degrees of freedom (nBins + nGlobs - nFloats in a binned model)
  nll.pgof() # = ROOT.TMath.Prob( 2*(nll.getVal() - nll.saturatedVal()), nll.ndof() )

Parameter uncertainties
-----------------------
Post-fit parameter uncertainties are nominally estimated from the diagonal entries of the covariance matrix, i.e:

.. math::

  \Delta\mu = \sqrt{\mathrm{cov(\mu,\mu)}}

Asymmetric uncertainties, :math:`\Delta_{\pm}\mu`, can be estimated using the *minos method*, which involves determining the values where the profile likelihood ratio curve for :math:`\mu` becomes equal to 1, which by definition occur at :math:`\mu = \hat{\mu}+\Delta_{\pm}\mu`. 

.. _impact:
Impact and parameter correlations
-----------------------
The *impact* on some parameter, :math:`\mu`, due to another parameter :math:`\nu`, is defined as how much the best-fit value of :math:`\mu` changes by if :math:`\nu` is changed by its corresponding post-fit uncertainty and held constant. Specifically, impact is:

.. math::

  \Delta_{\nu\pm}\mu = \hat{\hat{\mu}}(\nu=\hat{\nu}+\Delta_{\pm}\nu) - \hat{\mu}

where :math:`\hat{\hat{\mu}}(\nu=\hat{\nu}\pm\Delta\nu)` signifies the conditional maximum likelihood estimator of :math:`\mu` for a fit with :math:`\nu` held constant at the given value. The (possibly-asymmetric) uncertainty on :math:`\nu` is given by :math:`\Delta_{\pm}\nu`.

Impact is very closely related to the correlation between two parameters, and in fact the *ranking plot* that is frequently produced in HEP analyses can be viewed as just a way of visualizing the row of the correlation matrix corresponding to the parameter of interest. In fact, the impact can be estimated from the covariance matrix as follows:

.. math::

  \Delta_{\nu\pm}\mu \approx \frac{\mathrm{cov}(\mu,\nu)}{\pm\Delta\nu} = \mathrm{corr}(\mu,\nu)(\pm\Delta\mu)

where the symmetric uncertainties from the covariance matrix diagonals are used. If the asymmetric uncertainties on :math:`\nu` have been calculated, the :math:`\pm\Delta\nu` can be replaced by :math:`\Delta_{\pm}\nu` in the formula above. We see from the above expression that impact ranking is approximately the same thing as ranking the correlation coefficients. 

.. _profilelikelihood:
Profiled Likelihood Scans
----------------------
To draw the profiled likelihood ratio for a given parameter, you can do:

.. code-block:: python

  hs = nll.hypoSpace("parName")
  hs.scan("plr",nPoints,minVal,maxVal)
  hs.Draw()

You will learn more about ``hypoSpace`` on the next day, but this object will allow you to access the conditional fits that are run in order to evaluate the profile likelihood ratio at each point in the scan. Alternatively, to do the conditional fits manually and make the plot by hand, you could e.g. do:

.. code-block:: python

  fr = nll.minimize()
  g = ROOT.TGraph()
  v = minVal
  while v < maxVal:
    cfr = fr.cfit(f"parName={v}") # should ideally check status codes etc of cfr
    g.AddPoint( v, 2*(cfr.minNll() - fr.minNll() ) ) # computes the 2*PLR value
    v += (maxVal-minVal)/(nPoints-1)
  g.Draw("ALP")



