lifelines
=======
 
[What is survival analysis and why should I learn it?](http://nbviewer.ipython.org/urls/raw.github.com/CamDavidsonPilon/lifelines/master/Tutorial%20and%20Examples.ipynb) Survival analysis was originally developed and applied heavily by the actuarial and medical community. Its purpose was to answer *why do events occur now versus later* under uncertainity (where *events* might refer to deaths, disease remission, etc.). This is great for researchers who are interested in measuring lifetimes: they can answer questions like *what factors might influence deaths?*

But outside of medicine and actuarial science, there are many other interesting and exicting applications of this lesser-known technique. SaaS providers are interested in measuring customer lifetimes; sociologists are interested in measure political parties lifetimes, or relationships, or marriages; Telcoms are interested in understand customer behaviours, etc. 

*lifelines* is a pure Python implementation of the best parts of survival analysis, with simple classes
and helper functions. 

####Dependencies:

The usual Python data stack: **numpy, pandas, matplotlib (optional)**


## (Quick) Intro to *lifelines* and survival analysis

**Work in progress (50%) **

If you are new to survival analysis, wondering why it is useful, or are interested in examples,
I recommend running the `Tutorial and Examples.ipynb` notebook in a IPython notebook session. Alternatively, you can view it online [`here`](http://nbviewer.ipython.org/urls/raw.github.com/CamDavidsonPilon/lifelines/master/Tutorial%20and%20Examples.ipynb).



## Documentation

**Work in progress (80%) **

I've added documentation to a notebook, `Documentation.ipynb`, that adds detail to 
the classes, methods and data types. You can use the IPython notebook to view it, or view it [`online`](http://nbviewer.ipython.org/urls/raw.github.com/CamDavidsonPilon/lifelines/master/Documentation.ipynb).



### Enough talk - just show me the examples!

These demonstrate some functionality. If you are unfamiliar with Survival Analysis, I would start [here](http://nbviewer.ipython.org/urls/raw.github.com/CamDavidsonPilon/lifelines/master/Tutorial%20and%20Examples.ipynb).

### Generating artificial datasets

    %pylab
    from lifelines.generate_datasets import *
    from lifelines.estimation import *

    n_ind = 4 # how many lifetimes do we observe
    n_dim = 5 # the number of covarites to generate. 
    t = np.linspace(0,40,400)

    hz, coefs, covart = generate_hazard_rates(n_ind, n_dim, t, model="aalen")
    # you're damn right these are dataframes

    hz.plot()

(this styling of Matplotlib is present in the `styles/` folder)
![Hazard Rates](http://i.imgur.com/O8Og76Ol.png)

    sv = construct_survival_curves(hz, t )
    sv.plot() #moar dataframes

![Survival Curves](http://i.imgur.com/jWu3CM9l.png)

    #using the hazard curves, we can sample from survival times.
    rv = generate_random_lifetimes(hz, t, 50 )
    print rv
    array([[ 9.4235589 ,  3.60902256,  3.0075188 ,  0.60150376],
           [ 1.00250627,  3.20802005,  0.70175439,  0.30075188],
           [ 5.71428571,  8.02005013,  5.41353383,  0.30075188],
           ...,
           [ 3.70927318,  4.41102757,  3.30827068,  0.30075188],
           [ 1.80451128,  1.5037594 ,  0.30075188,  0.40100251],
           [ 1.40350877,  1.5037594 ,  0.80200501,  0.10025063]])

    survival_times = rv[:,0][:,None]  

    #estimation is clean and built to resemble scikit learn's api.
    kmf = KaplanMeierFitter()
    kmf.fit(survival_times)
    kmf.survival_function_.plot()

![KaplanMeier estimate](http://i.imgur.com/aztRkvll.png)

    naf = NelsonAalenFitter()
    naf.fit(survival_times)
    naf.cumulative_hazard_.plot()

![NelsonAalen](http://i.imgur.com/xA9OBFNl.png)


### Censorship events and estimation
When there are right-censored events, the simplest case being there are still surviving individuals, we need to be more careful and factor these 
non-observed individuals in. The api for this is an obvious extension from above:


    t = np.linspace(0,40,1000)
    hz, coefs, covart = generate_hazard_rates(1, 2, t, model="aalen")

    #generate random lifetimes with uniform censoring. C is the boolean of censorship
    T, C = generate_random_lifetimes(hz, t, size=750, censor=True )

In the above line, `C` is a boolean array with `True` iff we observed the death event, otherwise, they individual was right-censored. `T` is the death event, or if censored, the most lifespan before censorship.  


    kmf = KaplanMeierFitter()
    kmf.fit(T,t,censorship=C) #add in the censorship here

    #plot it
    ax = kmf.survival_function_.plot()
    sv = construct_survival_curves(hz,t) 
    sv.plot(ax=ax) 

    ##what if we had ignored the censorship events?
    kmf.fit(T,t, column_name="KM-estimate without factoring censorship")
    kmf.survival_function_.plot(ax=ax)

    plt.show()

![SVest](http://i.imgur.com/jYm911Zl.png)


### Survival Regression

Currently implemented is Aalen's Additive model

    from lifelines.estimation import AalenAdditiveFitter

    #will fit the cumulative hazards
    aaf = AalenAdditiveFitter(fit_intercept=True)
    aaf.fit(T[None,:], X, censorship=C) #X is a dataframe of numpy array of covariatesg
    aaf.cumulative_hazards_.plot()

![AalenCumulative](http://i.imgur.com/1LupZvHl.png)

    #plot the kernel smoothed hazards
    aaf.smoothed_hazards(20).plot()

![AalenSmooth](http://i.imgur.com/ymVfsedl.png)


### Plotting and Styles

The styling present in the above graphs is from a custom `matplotlibrc` file, you can find it in the `styles/` directory. 

There is a plotting library in Lifelines, under `lifelines.plotting`. We can visualize the lifetimes of individuals (really only good for data checking for small samples).

    from lifelines.plotting import plot_lifetimes

    N = 20
    current = 10
    birthtimes = current*np.random.uniform(size=(N,))
    T, C= generate_random_lifetimes(hz, t, size=N, censor=current - birthtimes )
    plot_lifetimes(T, censorship=C, birthtimes=birthtimes)

![lifetimes](http://i.imgur.com/JDt3t3Xl.png)


## Moar examples?

There are some IPython notebook files in the repo, and you can view them online here (though they may 
contain syntax from older versions of *lifelines*.)

- [Divorce data](http://nbviewer.ipython.org/urls/raw.github.com/CamDavidsonPilon/lifelines/master/datasets/Divorces%2520Rates.ipynb)
- [Gehan's survival dataset](http://nbviewer.ipython.org/urls/raw.github.com/CamDavidsonPilon/lifelines/master/datasets/The%2520Gehan%2520Survival%2520Data.ipynb)




