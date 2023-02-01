Day 4: Hypothesis Testing and Limit Setting

95\% CLs limits on the parameter of interest can be calculated with the asymptotic formulae with:

>>> w["modelName"].nll("datasetName").hypoSpace().limits()

which returns a dictionary of the observed and expected limits on the parameter of interest. The keys of the dictionary are "-2","-1","0","1","2" for the expected limits and "obs" for the observed limits. If no dataset is specified in the construction of the `nll` then the asimov expected dataset is used as the "observed" dataset and the "obs" limits are omitted (because the observed limits will be exactly the same as the expected limits). 

The values of the dictionary are pairs of numbers where the first number is the limit, and the second number is the uncertainty on that limit. 

Many fits are involved in the process of calculating the limits. If at any point a fit fails, the limit being calculated will be set to `NaN` and the next limit will be calculated. 

