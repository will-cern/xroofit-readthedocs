Day 4: Hypothesis Testing and Limit Setting
===========================================

Terminology and methodology
---------------------------

Hypothesis Spaces
^^^^^^^^^^^^^^^^^
Before describing what limit setting is, I will introduce the concept of the `hypothesis space` (or `hypoSpace`). 
Every analysis should decide what its hypoSpace is, which means selecting which parameters will be used to define 
the hypotheses to be tested. You must then decide what values to set those hypoSpace parameters to, or otherwise 
assign the parameter to an axis of the hypoSpace (only one hypoSpace parameter can be assigned to any given axis 
of the hypoSpace). Some examples of hypoSpaces would be e.g. the Higgs Search used a hypoSpace of the higgs mass 
parameter and signal strength parameter, and assigned the former to the x-axis and latter to the y-axis. SUSY searches 
will often choose a hypoSpace with two mass parameters assigned to the x- and y- axes and a third hypoSpace parameter 
being signal strength set equal to 1. Note that it is normal that all the parameters of interest will be hypoSpace parameters, 
but they need not be selected as the axis parameters.

Limit setting is then the process of finding the `confidence intervals` for the chosen hypothesis space. These intervals 
correspond to contours of constant p-value in the hypoSpace (we haven't yet said how to calculate this p-value, but it has 
a value at any point in the hypoSpace). The value that defines the contour is the `critical value` of 1-CL where CL is the 
confidence level of the interval. For a 95% confidence interval, CL = 0.95 so that the critical value is 0.05. When one side 
of the confidence interval is going to be at the physical boundary of the parameter of interest then the other edge of the 
interval is called usually referred to as the `confidence limit`. 

Hypothesis Points
^^^^^^^^^^^^^^^^^
So the task of computing confidence intervals/limits now is the process of calculating the `p-value` at various points in the 
hypoSpace in order to find the contour of p-value = 0.05 (for 95% CL). Each point is called a `hypothesis point` (or `hypoPoint`), 
the coordinates of which defines the values of the hypoSpace parameters for the hypothesis to be tested at that point. 
The p-value is the result of the hypothesis test. Note that it is usually required to say how p-values will be interpolated 
across the hypoSpace between hypoPoints. A common choice is to perform linear interpolation of the log of the p-value along 
axes corresponding to parameters of interest, and linear interpolation of the p-value along non-poi axes, however you may 
encounter different interpolations and it is important you know which you are doing. 

HypoPoint p-values
^^^^^^^^^^^^^^^^^^
In order to calculate the p-value of a hypoPoint, the null- and alternative- hypothesis distributions of a `test statistic` (ts) 
for that point are required. Remember that a `test statistic` is just any function that converts a dataset into a single number. 
The null-hypothesis distribution of the hypoPoint is the distribution of the ts obtained for toys generated where the hypoSpace parameters 
are equal to the values of the hypoPoint, i.e. it is the probability distribution :math:`p(ts|\mu=\mu_{\text{test})`. 
The alternative-hypothesis distribution is the ts distribution for toys generated where the hypoSpace parameters are equal to 
values that define some "alternative hypothesis", i.e. :math:`p(ts|\mu=\mu_{\text{alt}})`. Note that ts is calculated in exactly the same way 
for both the null and alternative hypothesis distributions of a given hypoPoint. 

When setting limits, the "null hypothesis" is the signal+background hypothesis with signal contribution corresponding to the 
hypoPoint being tested, and the "alt hypothesis" is the background-only hypothesis. Once these two distributions are determined 
(by throwing the toys or by other methods) then the p-value for that point is given by one of the following:

   * null p-value (``pNull``): the fraction of null-hypothesis toys with ts greater than the target ts value. Use this value for 
   calculating `CLs+b` limits.
   * alt p-value (``pAlt``): the fraction of alt-hypothesis toys with ts greater than the target ts-value.
   * cls p-value (``pCLs``): ratio ``pNull/pAlt``. Use this p-value when calculating `CLs` limits. Note that it is strictly not a p-value
   because it can take on values greater than 1; it is a ratio of probabilities, not a probability itself.

Note that the target ts value is dependent on what type of limit/interval is being found. If we are finding the observed limit, 
the target ts value is the ts value calculated with the observed dataset. If the we are finding the N-sigma limit the target 
ts value is whatever ts value has a ``pAlt`` equal to :math:`\Phi(N)` (so for the expected limit, the target ts value 
has ``pAlt = 0.5``). 

By definition ``pNull`` and ``pAlt`` are between 0 and 1, but ``pCLs`` can be any positive number. 
Also note that we can assign an uncertainty to these p-values, e.g. when determining them with toys we can compute the 
uncertainty on the p-values in the same way we would calculate an uncertainty on an efficiency. This way it is clear that the 
more toys we generate, the more certain the p-value becomes, and the clearer defined the interval/limit edge will be.

HypoPoint Test Statistics
^^^^^^^^^^^^^^^^^^^^^^^^^
So all that remains is to define what actual test statistic to use at each hypoPoint. We always use a variant of the 
`Profile Likelihood Ratio` test statistic. The (two-sided) profile likelihood ratio is given by:

.. math::

  t_\mu \equiv -2\log\left(\frac{L(\mu,\hat{\hat{\nu}},\theta)}{L(\hat{\mu},\hat{\nu},\theta)}\right)
  
(where :math:`\mu` are the poi, :math:`\nu` are the np, and :math:`\theta` are the pre-specified parameters). This test statistic requires 
a choice of :math:`\mu` and :math:`\theta` values, which are set equal to the hypoPoint coordinate values. 

The test statistic we usually use for upper limits is the *one-sided (capped-above) lower-bound profile likelihood ratio*, 
:math:`\tilde{q}_\mu`:

.. math::

  \tilde{q}_\mu \equiv \begin{cases}
    t_\mu \text{ if $\hat\mu < \mu$,} \\
    0 \text{ if $\hat\mu >= \mu$,} \\
    t_\mu-t_{\mu=\mu_L} \text{ if $\hat\mu < \mu_L$}. \\
    \end{cases}
    
where :math:`\mu_L` is the lower-bound of the physical range of the parameter of interest, which is normally equal to 0. The 
notation is such that q inidicates this likelihood ratio is one-sided (the second condition), and the ~ above the q indicates it is lower-bound (the third condition).
It will take up to two fits to evaluate this test statistic (the unconditional fit, the conditional fit at either the test-value of :math:`\mu` 
or at :math:`\mu=\mu_L`).

The test statistic conventionally used for discovery is the *one-sided (capped-below) profile likelihood ratio*:

.. math::

  q_0 \equiv \begin{cases}
    t_{\mu=0} \text{ if $\hat\mu > 0$,} \\
    0 \text{ if $\hat\mu <= 0$}.
    \end{cases}

However, in recent years a different test statistic variant has become popular for discovery - the *uncapped profile likelihood ratio*:

.. math::
  u_0 \equiv \begin{cases}
    t_{\mu=0} \text{ if $\hat\mu > 0$,} \\
    -t_{\mu=0} \text{ if $\hat\mu <= 0$}.
    \end{cases}


Asymptotic p-values
^^^^^^^^^^^^^^^^^^^
With just what is defined above one could calculate p-values for a hypoPoint by building up the test statistic distributions from toys. 
However, because each evaluation of the test statistic will involve one (the unconditional fit) or two (the conditional fit) fits, this can end up being a costly calculation to perform 
(especially for hypoPoints where the p-value turns out to be small, which will require many toys to determine accurately).

An approximation can be obtained using asymptotic formulae for test statistic distributions based on the Wald approximation.

Properties and Quantities of a HypoPoint
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the next section you will learn how to create a hypoSpace and run a scan in it, which will create hypoPoints in the space. 

Here is a table of the quantities that can be computed for a hypoPoint:



Limit Setting Checklist
-----------------------
You should be able to answer the following questions:

  * What are your hypoSpace parameters, and what values are they set to (or which are used as axis parameters)?
  * What hypoPoints are you testing?
  * What p-value type are you using (pNull or pCLs)?
  * How are you interpolating the p-value across the hypoSpace (linear, or log-linear, or something else)?
  * What PLR test-statistic variant are you using (two-sided, one-sided-capped-above, one-sided-capped-below, uncapped, one-sided-absolute, ...)?
  * Are you determining the ts distributions with toys or with asymptotic formulae?
  * What is the uncertainty on the p-value of each point? 
  * Did any of the fits (for toys, asimov, or obs data) fail?


xRooFit Demo: CLs limits with asymptotic formulae
-----------------------------------

Here is a complete and verbose example python script for computing a CLs limit on an existing workspace. It is intended to demonstrate how you can control many aspects of how the limit scan is performed.  Additional commentary on the code follows the script.

.. code-block:: python

  import ROOT
  XRF = ROOT # or for ROOT's builtin xRooFit: XRF = ROOT.Experimental.XRooFit

  fileName  = "path/to/workspace.root"           # path to the workspace
  pdfName   = "simPdf"                           # name of the top-level pdf in the workspace
  channels  = "*"                                # comma-separated list of channels to include (n.b. you should not include VRs)
  dsName    = "obsData"                          # name of the observed dataset, use "" to use an asimov dataset for the obsData
  poiName   = ""                                 # name of the parameter of interest - leave blank to auto-infer if possible
  asimovVal = 0                                  # POI-value to assume for asimov dataset (if dsName="")
  scanMin   = 0                                  # lower boundary poi value for limit scan (can be more restricted than fitting range)
  scanMax   = 10                                 # upper boundary poi value for limit scan (can be more restricted than fitting range)
  scanN     = 0                                  # number of points to scan, leave as 0 for an auto-scan
  scanType  = "cls visualize"                    # leave out the 'visualize' if you don't want to see progress during scan
  constPars = ""                                 # comma-separated list of nuisance parameters to hold const, e.g. do "*" for a stat-only limit
  tsType    = XRF.xRooFit.TestStatistic.qmutilde # choices: tmu, qmu, qmutilde, q0, u0
  nSigmas   = [0,1,2,-1,-2,float('nan')]         # list of nSigmas to compute limits at ... "NaN" is used by xRooFit to indicate you want obs limit 
  outFile   = ""                                 # specify a path to save the post-scan workspace (with result) to

  w = XRF.xRooNode(fileName)
  if poiName == "": poiName = w.poi()[0].GetName() # requires POI to have been pre-specified in the workspace
  if constPars!= "": w.pars().reduced(constPars).setAttribAll("Constant") # mark required parameters constant
  w.pars()[poiName].setVal(asimovVal) # set to asimov value before building NLL, so that asimov dataset corresponding to this hypo is used if dsName=""
  hs = w[pdfName].reduced(channels).nll(dsName).hypoSpace(poiName,tsType) # creates a hypoSpace using the given pdf and dataset for the NLL, and poi = given parameter
  
  hs.scan(scanType,scanN,scanMin,scanMax,nSigmas)
  limits = hs.limits() # extracts the limits from the scan by interpolation, returns as a dict

  # show results ...
  print(limits)
  hasNaN = False
  for nSigma,lim in dict(limits).items(): # example of how to get result out of limits map
      if ROOT.TMath.IsNaN(lim.value()): hasNaN = True # use lim.error() to access the 'uncertainty' on the limit
  if hasNaN:
      # failed to find one of the limits, so print the hypoSpace for information about points that were scanned and their FitResult statuscodes
      hs.Print()

  # save the result to the workspace if requested, and then save the workspace
  if outFile != "":
      w.Add( hs.result() )
      w.SaveAs(outFile)
      w.Browse() # can inspect the workspace ... the hypoSpace will appear under the "scans" folder of workspace

A minimal version of running a limit would be:

.. code-block:: python

  import ROOT
  XRF = ROOT # or for ROOT's builtin xRooFit: XRF = ROOT.Experimental.XRooFit
  w = XRF.xRooNode("path/to/workspace.root")
  print( w.nll("datasetName").hypoSpace().limits() )

This assumes that the POI has already been declared in the workspace, there is only one top-level pdf in the workspace, and that the fitting range of the POI is appropriate to also be used as the scan range. 

The ``limits()`` method returns a dictionary of limits (each with a ``value()`` and ``error()``), with the keys of the dictionary being "-2","-1","0","1","2" for the expected limits and "obs" for the observed limits. If no dataset is specified in the construction of the `nll` then the asimov expected dataset is used as the "observed" dataset.

The values of the dictionary are pairs of numbers where the first number is the limit, and the second number is the uncertainty on that limit. 

Why does my CLs limit scan fail?
-----------------------------------
Many fits are involved in the process of calculating the limits. If at any point a fit fails, the limit being calculated will be set to `NaN` and the next limit will be calculated. 

You should print the hypoSpace or explore it in the browser, as demonstrated in the script above, in order to work out which hypothesis tests (hypoPoints) returned non-zero status codes. 

A common issue is that the range specified for the scan is too large, and the so the hypoPoints get created that are too discrepant with the dataset and the fit struggles to correctly evaluate the covariance matrix at the minima (the covariance matrix must be positive definite, but status code = 1 indicates that the matrix was forced positive definite, which means you are not at a valid minima). 

If you specified a sensible scan range, you should next try to identify if there is a particular (nuisance) parameter that is causing your fits to fail. You can use the demo code above to select groups of parameters to hold constant during the fit. Remember that ``w.pars().Print()`` will list all the parameters and ``w.floats().Print()`` will list all the currently-floating parameters.


xRooFit Demo: Computing Discovery Significance
----------------------------------------------
You can compute discovery significances using the example program above, where you scan just a single point, the hypoPoint corresponding to the background-only hypothesis. Instead of obtaining a limit though, you want to extract the null-hypothesis p-value for the point you scan. Namely, make the following changes:

.. code-block:: python

  scanMin = 0 # we want to test just the mu=0 hypothesis
  scanMax = 0 # so set min and max both to 0
  scanN = 1
  scanType = "pnull"

And instead of calling the ``limits`` method, extract the null pvalues as follows:

.. code-block:: python

  print("Observed p0:",hs[0].pNull_asymp()) # return type of pNull_asymp() has a .value() and .error() method
  print("Expected p0:",hs[0].pNull_asymp(0)) # significance under the mu=1 hypothesis
  print("Expected +1 sigma:",hs[0].pNull_asymp(1))
  print("Expected -1 sigma:",hs[0].pNull_asymp(-1))

Null p-values can be converted to significances using the standard gaussian quantile (aka normile) function. 
