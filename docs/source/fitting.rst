Day 3: NLL Construction and Minimization
========================================

This is otherwise known as the process of fitting a model to a dataset. The output of this process is a `Fit Result`.

>>> fitResult = w["pdfs/pdfName"].nll("datasetName").minimize()

Constructing the NLL Function
----------------------------
The NLL is the most commonly used :ref:`objective function <objective functions>`: for fitting. It is constructed using the ``nll`` method of a pdf node in the workspace:

>>> nll = w["pdfs/pdfName"].nll("datasetName",[list,of,nll,options])

The most important NLL options, which affect how the NLL is actually defined, are described in the section below. 

Whatever the NLL options are, the NLL function can always be factorized into two parts: the *main term* and the *constraint term*. The main term depends on the :ref:`regular observables <regular observables>`: and the constraint term depends on the :ref:`global observables <global observables>`:. The main term is the sum of the negative log likelihoods of each the entries in the dataset (accounting for any weights):

.. math::

  -\ln(L(\mu,\theta)) = -\ln(p(\underline{a}|\mu,\theta)) -\sum_{i=0}^{N}w_i\ln(p(\underline{x}_i|\mu,\theta))

where :math:`\underline{a}` are the global observables and :math:`\underline{x}_i` are the regular observables of the ith entry in the dataset, which has weight :math:`w_i`. :math:`p(\underline{a}|\mu,\theta)` is the product of just the constraint terms from the pdf, which feature just global observables. In the above expression, the first term is just called the constraint term and the second is called the main term. 

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
The ``minimize`` method accepts an optional fit configuration that contains hyperparameters that steer the minimization ...

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

