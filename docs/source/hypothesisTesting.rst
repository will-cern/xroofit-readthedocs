Day 4: Hypothesis Testing and Limit Setting
===========================================

Terminology and methodology
---------------------------

Before describing what limit setting is, I will introduce the concept of the `hypothesis space` (or `hypoSpace`). Every analysis should decide what its hypoSpace is, which means selecting which parameters will be used to define the hypotheses to be tested. You must then decide what values to set those hypoSpace parameters to, or otherwise assign the parameter to an axis of the hypoSpace (only one hypoSpace parameter can be assigned to any given axis of the hypoSpace). Some examples of hypoSpaces would be e.g. the Higgs Search used a hypoSpace of the higgs mass parameter and signal strength parameter, and assigned the former to the x-axis and latter to the y-axis. SUSY searches will often choose a hypoSpace with two mass parameters assigned to the x- and y- axes and a third hypoSpace parameter being signal strength set equal to 1. Note that it is normal that all the parameters of interest will be hypoSpace parameters, but they need not be selected as the axis parameters.

Limit setting is then the process of finding the `confidence intervals` for the chosen hypothesis space. These intervals correspond to contours of constant p-value in the hypoSpace (we haven't yet said how to calculate this p-value, but it has a value at any point in the hypoSpace). The value that defines the contour is the `critical value` of 1-CL where CL is the confidence level of the interval. For a 95% confidence interval, CL = 0.95 so that the critical value is 0.05. When one side of the confidence interval is going to be at the physical boundary of the parameter of interest then the other edge of the interval is called usually referred to as the `confidence limit`. 

So the task of computing confidence intervals/limits now is the process of calculating the `p-value` various points in the hypoSpace in order to find the contour of p-value = 0.05 (for 95% CL). Each point is called a `hypothesis point` (or hypoPoint), the coordinates of which defines the values of the hypoSpace parameters for the hypothesis test that is to be performed corresponding to that point. The p-value is the result of the hypothesis test. 

So each hypoPoint defines a set of values for the poi (and potentially other parameters), and we wish to calculate a p-value for each point. The p-value is extracted from the null- and alternative-hypothesis distributions of a `test statistic` (ts). Remember that a `test statistic` is just any function that converts a dataset into a single number. The null-hypothesis distribution of the ts is the distribution of the ts obtained for toys generated where the hypoSpace parameters are equal to the values of the hypoPoint. The alternative-hypothesis distribution is the ts distribution for toys generated where the hypoSpace parameters are equal to values that define some "alternative hypothesis". When setting limits, the "null hypothesis" is the signal+background hypothesis with signal contribution corresponding to the hypoPoint being tested, and the "alt hypothesis" is bacakground-only hypothesis. Once these two distributions are determined (by throwing the toys or by other methods) then the p-value for that point is given by one of the following:

   * null p-value (``pNull``): the fraction of null-hypothesis toys with ts greater than the target ts value. Use this value for calculating `CLs+b` limits.
   * alt p-value (``pAlt``): the fraction of alt-hypothesis toys with ts greater than the target ts-value.
   * cls p-value (``pCLs``): ratio ``pNull/pAlt``. Use this p-value when calculating `CLs` limits.

Note that the target ts value is dependent on what type of limit/interval is being found. If we are finding the observed limit, the target ts value is the ts value calculated with the observed dataset. If the we are finding the N-sigma limit the target ts value is whatever ts value has a pAlt equal to :math:`\Phi(N)` (so for the expected limit, the target ts value has ``pAlt = 0.5``). 

Note that by definition ``pNull`` and ``pAlt`` are between 0 and 1, but ``pCLs`` can be any positive number. Also note that we can assign an uncertainty to these p-values, e.g. when determining them with toys we can compute the uncertainty on the p-values in the same way we would calculate an uncertainty on an efficiency. This way it is clear that the more toys we generate, the more certain the p-value becomes, and the clearer defined the interval/limit edge will be.

So all that remains is to define what actual test statistic to use at each hypoPoint. We always use a variant of the `Profile Likelihood Ratio` test statistic. 

Finally it is worth noting that it is computationally expensive to calculate a p-value for a hypoPoint, therefore it is normal for a (sometimes quite sparse) grid of hypoPoints to be calculated and a choice made for how to interpolate the p-value between these points. A common choice is to perform linear interpolation of the log of the p-value. But you should be aware of what method you use.

Limit Setting Checklist
-----------------------
You should be able to answer the following questions:

  * What are your hypoSpace parameters, and what values are they set to (or otherwise assigned to x-axis and y-axis)?
  * What hypoPoints are you testing?
  * What p-value type are you using (pNull or pCLs)?
  * How are you interpolating the p-value across the hypoSpace (linear, or log-linear, or something else)?
  * What PLR test-statistic variant are you using (two-sided, one-sided-positive, one-sided-negative, one-sided-absolute, ...)?
  * Are you determining the ts distributions with toys or with asymptotic formulae?
  * What is the uncertainty on the p-value of each point? 
  * Did any of the fits (for toys or obs data) fail?


CLs limits with asymptotic formulae
-----------------------------------

95\% CLs limits on a hypoSpace defined with just the parameter of interest (assigned to x-axis) can be calculated with the asymptotic formulae with:

>>> w["modelName"].nll("datasetName").hypoSpace().limits()

which returns a dictionary of the observed and expected limits on the parameter of interest. The keys of the dictionary are "-2","-1","0","1","2" for the expected limits and "obs" for the observed limits. If no dataset is specified in the construction of the `nll` then the asimov expected dataset is used as the "observed" dataset and the "obs" limits are omitted (because the observed limits will be exactly the same as the expected limits). 

The values of the dictionary are pairs of numbers where the first number is the limit, and the second number is the uncertainty on that limit. 

Many fits are involved in the process of calculating the limits. If at any point a fit fails, the limit being calculated will be set to `NaN` and the next limit will be calculated. 

