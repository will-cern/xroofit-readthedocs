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

You can draw the fit result (``fr.Draw()``) to see the post-fit parameter pulls. It is also very important to look at the correlation matrix of the fit (assuming the covariance matrix quality is good (code=3, see below). To visualize the correlation matrix you can do ``fr.Draw("CORR COLZ TEXT")``. To visualize just the parameters with the top-10 off-diagonal correlations you can do ``fr.Draw("CORR10 COLZ TEXT")``.

.. note:: The importance of checking the correlation matrix:
   The correlation matrix gives an indication of how skewed the likelihood minima is in the parameter space, in that large correlations are indicative of a potentially poorly-defined minima. A strong correlation (typically over 80%) can be evidence that your parameters are not sufficiently independent of one another and your model may then have *degeneracy* in that form of different points in the parameter space having the same (or very similar) model predictions (pdf values). If this is happening at or near where the true minima in the likelihood function lies, the minima itself would be poorly defined and the covariance matrix (and hence symmetric uncertainties) for the parameters cannot be trusted. 

The Fit Config
^^^^^^^^^^^^^^
The ``minimize`` method accepts an optional fit configuration that contains hyperparameters that steer the minimization. The nll function has its own copy of the fit config that is used if you do not pass your own fit config object to minimize. The settings in the fit config that is stored in the nll can be viewed by using ``nll.Print()``. The table below describes what each config option does and how to set it (by either passing an NLL option to the list in the :ref:`nll method <nll options>`, or change the setting after creating the nll object but before running a fit). In each case, the default value is given as the example setting:

.. list-table:: Fit Config settings
    :widths: 25 75
    :header-rows: 1

    * - Name
      - Details
    * - Tolerance
      - | **NLL Option**: ``XRF.xRooFit.Tolerance(0.01)``
        | **Setting after creation:** ``nll.fitConfig().MinimizerOptions().SetTolerance(0.01)``
        | **Description**: Controls when minimization stops. Tolerance equals 1000 times the maximum allowed value of the edm (estimated distance to minimum) of the fit before the fit is considered converged. E.g. the default value of 0.01 means that the edm must become less than 1e-5 for convergence. If this is not reached, the migrad status code will be 3. It is a "stopping condition" for the convergence, the smaller it is the closer to the true minimum your are likely to be.  Ideally leave it at the default. **It is not recommended to set this any higher than 10**, as problems with parameter uncertainties have been seen for fits with EDMs above 0.01 even though the covariance matrix was positive definite. 
    * - Strategy
      - | **NLL Option**: ``ROOT.RooFit.Strategy(-1)`` 
        | **Setting after creation:** ``nll.fitConfig().MinimizerOptions().SetStrategy(-1)``
        | **Description**: The starting minuit strategy. If set to -1 (the default), the starting strategy is the start of the StrategySequence setting (see below). The lower the strategy, the faster the minimization, but also the less accurate it will be, and it is possible that the fit may not accurately converge to the true minima (which manifests as poor quality covariance matrices or large post-hesse EDMs). In strategy 0 a fast but iterative approximation of the hessian matrix (which is required for each step of the minimization) is used. In strategy 2 the Hesse algorithm is used to compute the hessian matrix, using a forward finite-difference calculation. In strategy 3, the Hesse algorithm is used with central a finite-difference calculation, which is more accurate than the strategy 2 calculation but 4 times more expensive to compute. It has been found to be necessary in some analyses with high statistics. Strategy 1 is not advised as a starting strategy.
    * - StrategySequence
      - | **NLL Option**: ``XRF.xRooFit.StrategySequence("0s01s12s2s3m")``
        | **Setting after creation:** ``nll.fitConfigOptions().SetValue("StrategySequence","0s01s12s2s3m")``
        | **Description**: Determines the order of retries automatically performed if a fit fails. A number indicates a strategy setting, `s` indicates a rescan, and `m` indicates a switch to minuit1 (which will soon be deprecated). For example, a strategy sequence of "0s01s12s2m" means that if a strategy=0 fit fails it will try a rescan and then try the strategy=0 fit again, if that fails it will switch to strategy=1, and so on. 
    * - Hesse
      - | **NLL Option**: ``ROOT.RooFit.Hesse(True)``
        | **Setting after creation:** ``nll.fitConfig().SetParabErrors(True)``
        | **Description**: Controls if hesse should be run after the migrad minimization (if it wasn't already run with the necessary level of precision by the migrad minimization, which can sometimes happen and xRooFit will automatically determine this). If it is not run, the covariance matrix may not be accurate (quality != 3).
    * - HesseStrategy
      - | **NLL Option**: n/a
        | **Setting after creation:** ``nll.fitConfigOptions().SetValue("HesseStrategy",-1)``
        | **Description**: Controls which strategy is used first when hesse algorithm is run. If -1, will take first strategy in the HesseStrategySequence (see below)
    * - HesseStrategySequence
      - | **NLL Option**: n/a
        | **Setting after creation:** ``nll.fitConfigOptions().SetValue("HesseStrategySequence","23")``
        | **Description**: Similar to the StrategySequence setting, this controls the order of attempts made in the hesse algorithm, with an example of hesse failure being e.g. a non-positive definite covariance matrix (covQuality=1 in the case of hesse strategy 3 in the fit result). 

For example, to make the tolerance equal to 1 and the starting strategy equal to 1, you can do (assumes you have done e.g. ``import ROOT as XRF`` if using xRooFit compiled on top of ROOT):

>>> nll = w["pdfs/pdfName"].nll("datasetName",[XRF.xRooFit.Tolerance(1),ROOT.RooFit.Strategy(1)])

Or equivalently you can do:

>>> nll = w["pdfs/pdfName"].nll("datasetName")
>>> nll.fitConfig().MinimizerOptions().SetTolerance(1)
>>> nll.fitConfig().MinimizerOptions().SetStrategy(1)

A summary of the effects of Strategy and Tolerance are that higher strategies are generally slower but more robust, and lower tolerances are slower and/or more challenging to satisfy, but are more robust. So the tradeoff in setting these two hyperparameters is speed/convergence vs validity/success of the fit. A general "hyperstrategy" to follow might be to set the strategy as low as possible and increase the tolerance until your fits converge, then increase the strategy if the increased tolerance setting is causing problems such as the post-hesse EDM estimate being above tolerance. 

Status codes and covariance quality codes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
It is important to check the status codes and covariance quality codes of fits to confirm the fit is valid. A "valid" fit has a status code of 0 and a covariance quality of 3. Note that a valid fit might still have problems though, e.g. if there are large correlations between parameters. The codes can be checked with:

>>> fr.status()
>>> fr.covQual()

.. list-table:: Status codes
    :widths: 10 75
    :header-rows: 1

    * - Code
      - Description
    * - 0
      - The last algorithm to run in the fit ran successfully. Normally the last algorithm to run is the `Hesse` algorithm, which calculates the covariance matrix.
    * - 1
      - Covariance matrix forced positive-definite. This means that the place that the minimization converged does not appear to be a valid minimum; at a true minimum the covariance matrix (calculated from the Hessian) must be positive definite. It may be possible to overcome this error by increasing the strategy used by the Hesse algorithm from 2 to 3, although xRooFit by default will increase the strategy for you (look at the status code history to see if this happened). **If you see this status code, try increasing the Strategy**. 
    * - 2
      - Covariance matrix is invalid (usually this means it is not positive-definite). This status code occurs only with Hesse Strategy 3.
    * - 3
      - EDM above max threshold. EDM is estimated from the covariance matrix and is an estimate of how far from the true minimum might the fit be. The tolerance hyperparameter is what sets the threshold (see table above). **If you see this status code, try increasing the Tolerance** but be aware this can increase the uncertainties on quantities derived from fits such as likelihood ratio test statistics. 
    * - 4+
      - Some other error. The fit cannot be trusted. 


.. list-table:: Covariance quality codes
    :widths: 10 75
    :header-rows: 1

    * - Code
      - Description
    * - 0
      - Covariance matrix unavailable. This should only happen if there were no floating parameters for the fit. 
    * - 1
      - Approximation only. This code is returned by Hesse Strategy 3 if the covariance matrix is not positive-definite (the status will be 2).
    * - 2
      - Forced positive-definite. This code is returned by Hesse Strategy 2 (or lower) if the covariance matrix was not positive-definite (the status code will be 1). 
    * - 3
      - The covariance matrix is positive definite. Note that it is still possible that there are problems with the fit, particularly if the correlation matrix shows large correlations between variables. 

Goodness of fit
---------------
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

These are known as the symmetric or hessian uncertainties. They can be accessed for any parameter from a fit result as follows:

.. code-block:: python

  fr.floatParsFinal().find(parName).getVal() # the post-fit value
  fr.floatParsFinal().find(parName).getError() # the post-fit symmetric (hessian) uncertainty

Asymmetric uncertainties, :math:`\Delta_{\pm}\mu`, can be estimated using the *minos method*, which involves determining the values where the profile likelihood ratio curve for :math:`\mu` becomes equal to 1, which by definition occur at :math:`\mu = \hat{\mu}+\Delta_{\pm}\mu`. Given the additional computational requirements, you should select which parameters should have asymmetric uncertainties computed by flagging them with an attribute before you run the fit, then the asymmetric uncertainty can be accessed similarly to above:

.. code-block:: python

  nll.pars().find(parName).setAttribute("minos") # flag a specific parameter of the nll as requiring asymmetric uncertainties
  fr = nll.minimize()
  fr.floatParsFinal().find(parName).getErrorHi() # asymmetric up uncertainty
  fr.floatParsFinal().find(parName).getErrorLo() # asymmetric down uncertainty

.. _impact:
Impact and parameter correlations
-----------------------
The *impact* on some parameter, :math:`\mu`, due to another parameter :math:`\nu`, is defined as how much the best-fit value of :math:`\mu` changes by if :math:`\nu` is changed by its corresponding post-fit uncertainty and held constant. Specifically, impact is:

.. math::

  \Delta_{\nu\pm}\mu = \hat{\hat{\mu}}(\nu=\hat{\nu}+\Delta_{\pm}\nu) - \hat{\mu}

where :math:`\hat{\hat{\mu}}(\nu=\hat{\nu}\pm\Delta\nu)` signifies the conditional maximum likelihood estimator of :math:`\mu` for a fit with :math:`\nu` held constant at the given value. The (possibly-asymmetric) uncertainty on :math:`\nu` is given by :math:`\Delta_{\pm}\nu`. Impact can be calculated in xRooFit using the fit result object (note that these will trigger additional conditional fits):

.. code-block:: python
  
  fr.impact(muName,nuName,up=True) # computes delta_{nu+}mu impact on "muName" parameter due to the "nuName" parameter
  fr.impact(muName,nuName,up=True,prefit=True) # computes the 'prefit impact', meaning uncertainty on nu is the prefit uncertainty

Impact is very closely related to the correlation between two parameters, and in fact the *ranking plot* that is frequently produced in HEP analyses can be viewed as just a way of visualizing the row of the correlation matrix corresponding to the parameter of interest. In fact, the impact can be estimated from the covariance matrix as follows:

.. math::

  \Delta_{\nu\pm}\mu \approx \frac{\mathrm{cov}(\mu,\nu)}{\pm\Delta\nu} = \mathrm{corr}(\mu,\nu)(\pm\Delta\mu)

where the symmetric uncertainties from the covariance matrix diagonals are used. If the asymmetric uncertainties on :math:`\nu` have been calculated, the :math:`\pm\Delta\nu` can be replaced by :math:`\Delta_{\pm}\nu` in the formula above. We learn from the above expression that impact ranking is approximately the same thing as ranking the correlation coefficients. 

The approximated impact can be calculated in xRooFit with:

.. code-block:: python
  
  fr.impact(muName,nuName,up=True,approx=True) # computes approximated delta_{nu+}mu impact on "muName" parameter due to the "nuName" parameter
  fr.impact(muName,nuName,up=True,prefit=True,approx=True) # computes the approximated 'prefit impact', meaning uncertainty on nu is the prefit uncertainty

.. _breakdown:
Conditional Uncertainties and Uncertainty Breakdowns
----------------------------------------------
We saw above how to access the post-fit uncertainty for a parameter, but it is often desirable, particularly for parameters of interest, to know how much of that uncertainty was due to the presence of other uncertainties in the moodel. Impacts, as defined in the previous section, cannot be treated as uncertainty components and cannot be added in quadrature. What instead is required is known as the *conditional uncertainty*: the uncertainty on a parameter, :math:`\mu`, when another parameter, :math:`\nu`, is held constant at its post-fit (maximum likelihood estimator) value, :math:`\hat{\nu}`.

This can be approximated with the covariance matrix as follows:

.. math::

  \Delta\mu(\nu=\hat{\nu}) \approx \sqrt{\mathrm{cov(\mu,\mu)} - \frac{\mathrm{cov(\mu,\nu)^2}}{\mathrm{cov(\nu,\nu)}}}

This formula generalises to the case where we want to compute the conditional uncertainty on :math:`\mu`, conditioning on multiple other parameters. We split the list of parameters into those being conditioned on (class 2) and those not being conditioned on (class 1, this will include :math:`\mu`). Then covariance matrix is block-decomposed into matrices :math:`V_{11},V_{12},V_{21},V_{22}` according to the parameter grouping. Finally the *schur complement*, :math:`\bar{V_{22}}`, is computed according to the formula below, and the conditional uncertainty is extracted:

.. math::

  \bar{V_{22}} = V_{11} - V_{12} \cdot V_{22}^{-1} \cdot V_{21}\\
  \Delta\mu(\nu_i=\hat{\nu}_i) \approx \sqrt{\bar{V_{22}}(\mu,\mu)}
    
Conditional uncertainties can be calculated in xRooFit as follows:

.. code-block:: python

  cError = fr.conditionalError(muName,"list,of,nu",up=True,approx=True) # can use wildcard in list

The conditional uncertainty conditioned on a group of parameters can then be translated into an *uncertainty breakdown* (uncertainty component of parameter due to the group) by subtracting this conditional uncertainty from the total uncertainty in quadrature. For example, to obtain a systematic uncertainty component, one computes the conditional uncertainty conditioned on all the systematic uncertainty parameters, and subtracts this from the total uncertainty. In this particular case, the conditional uncertainty calculated *is* the statistical uncertainty (since statistical uncertainty is all the uncertainty that isn't systematic):

.. code-block:: python

  totError = fr.floatParsFinal().find(muName).getError()
  statError = fr.conditionalError(muName,"alpha_*,gamma_*",up=True,approx=True) # usual systematic parameters are prefixed by alpha_ and gamma_
  systError = ROOT.TMath.Sqrt(ROOT.TMath.Power(totErr,2) - ROOT.TMath.Power(statErr,2))

To breakdown the systematic uncertainty further, e.g. into mc-statistical (the `gamma` uncertainties) and model-sytematics (the `alpha` uncertainties) you can do:

.. code-block:: python

  totError = fr.floatParsFinal().find(muName).getError()
  statAndMCStatError = fr.conditionalError(muName,"alpha_*",up=True,approx=True) # condition just on model systematics
  modSystError = ROOT.TMath.Sqrt(ROOT.TMath.Power(totErr,2) - ROOT.TMath.Power(statAndMCStatError,2)) # model-systematics uncertainty component
  statError = fr.conditionalError(muName,"alpha_*,gamma_*",up=True,approx=True) # condition on all systematics to get stat error
  mcStatError = ROOT.TMath.Sqrt(ROOT.TMath.Power(statAndMCStatError,2) - ROOT.TMath.Power(statError,2)) # subtract stat error to get mc-stat uncertainty component

Finally, note that in the case where the group consists of a single parameter, when you calculate the uncertainty component due to this parameter by subtracting off its corresponding conditional uncertainty from the total uncertainty, you get precisely the (covariance-approximated) impact:

.. math::

  \sqrt{(\Delta\mu)^2 - (\Delta\mu(\nu=\hat{\nu}))^2} = \sqrt{\mathrm{cov}(\mu,\mu) - \left(\sqrt{\mathrm{cov(\mu,\mu)} - \frac{\mathrm{cov(\mu,\nu)^2}}{\mathrm{cov(\nu,\nu)}}}\right)^2} = \frac{\mathrm{cov}(\mu,\nu)}{\sqrt{\mathrm{cov}(\nu,\nu)}} = \mathrm{corr}(\mu,\nu)(\Delta\mu)

.. _breakdown2:
Shifted Global Observables (Shifted GO) method of uncertainty breakdowns
----------------------------------------------
A new (to 2025) technique for calculating systematic uncertainty breakdowns involves shifting global observables.

This can be approximated from the covariance matrix using the following method:

.. code-block:: python

   systError = ROOT.TMath.Sqrt(sum([pow(fr.impact(poi,np.GetName(),up=True,prefit=True,approx=True),2) for np in w.np().reduced("alpha_*","gamma_*")]))

i.e. you perform a quadrature sum of the covariance-approximated prefit impacts of the nuistance parameters

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



