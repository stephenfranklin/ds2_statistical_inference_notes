# Bayesian.Rmd
---
title       : Bayesian inference

## Bayesian analysis
- Bayesian statistics posits a *prior* on the parameter
  of interest
- All inferences are then performed on the distribution of 
  the parameter given the data, called the posterior
- In general,
  $$
  \mbox{Posterior} \propto \mbox{Likelihood} \times \mbox{Prior}
  $$
- $\propto$ means "proportional to"
- Therefore (as we saw in diagnostic testing) the likelihood is
  the factor by which our prior beliefs are updated to produce
  conclusions in the light of the data

---
## Prior specification
- The beta distribution is the default prior
  for parameters between $0$ and $1$.
- The beta density depends on two parameters $\alpha$ and $\beta$
$$
\frac{\Gamma(\alpha +  \beta)}{\Gamma(\alpha)\Gamma(\beta)}
 p ^ {\alpha - 1} (1 - p) ^ {\beta - 1} ~~~~\mbox{for} ~~ 0 \leq p \leq 1
$$
- $\Gamma$ is the Gamma function.
- $p$ is the random variable in the beta density.
- $\alpha$ and $\beta$ are parameters, 
    - not to be confused with the type 1 and type 2 error rates.
- The mean of the beta density is $\alpha / (\alpha + \beta)$
- The variance of the beta density is 
$$\frac{\alpha \beta}{(\alpha + \beta)^2 (\alpha + \beta + 1)}$$
- The uniform density is the special case where $\alpha = \beta = 1$

---

We can play with the beta density by using the R manipulate library. This code won't work as html, but here is the code.  Run it in Rstudio.

```r
## Exploring the beta density
library(manipulate)
pvals <- seq(0.01, 0.99, length = 1000)
manipulate(plot(pvals, dbeta(pvals, alpha, beta), type = "l", lwd = 3, frame = FALSE), 
    alpha = slider(0.01, 10, initial = 1, step = 0.5), beta = slider(0.01, 10, 
        initial = 1, step = 0.5))
```


---
## Posterior 
- Suppose that we chose values of $\alpha$ and $\beta$ so that
  the beta prior is indicative of our degree of belief regarding $p$
  in the absence of data
- Then using the rule that
  $$
  \mbox{Posterior} \propto \mbox{Likelihood} \times \mbox{Prior}
  $$
  and throwing out anything that doesn't depend on $p$, we have that
$$
\begin{align}
\mbox{Posterior} &\propto  p^x(1 - p)^{n-x} \times p^{\alpha -1} (1 - p)^{\beta - 1} \\
                 &  =      p^{x + \alpha - 1} (1 - p)^{n - x + \beta - 1}
\end{align}
$$
- This density is just another beta density with parameters
  $\tilde \alpha = x + \alpha$ and $\tilde \beta = n - x + \beta$
- Think of $\alpha$ as the a priori number of successes; $\beta$ as the a priori number of failures.


---
## Posterior mean

$$
\begin{align}
E[p ~|~ X] & =   \frac{\tilde \alpha}{\tilde \alpha + \tilde \beta}\\ \\
& =  \frac{x + \alpha}{x + \alpha + n - x + \beta}\\ \\
& =  \frac{x + \alpha}{n + \alpha + \beta} \\ \\
& =  \frac{x}{n} \times \frac{n}{n + \alpha + \beta} + \frac{\alpha}{\alpha + \beta} \times \frac{\alpha + \beta}{n + \alpha + \beta} \\ \\
& =  \mbox{MLE} \times \pi + \mbox{Prior Mean} \times (1 - \pi)
\end{align}
$$
- $0 < \pi < 1$
- The action of $\pi ~ 1$ is a large MLE, and small prior mean.
- The action of $\pi ~ 0$ is a small MLE, and large prior mean.

---
## Thoughts

- The posterior mean is a mixture of the MLE ($\hat p$) and the
  prior mean
- $\pi$ goes to $1$ as $n$ gets large; for large $n$ the data swamps the prior
- For small $n$, the prior mean dominates 
- Generalizes how science should ideally work; as data becomes
  increasingly available, prior beliefs should matter less and less
- With a prior that is degenerate (if we make the variance of the beta distribution equal zero) at a value, no amount of data
  can overcome the prior

---
## Example

- Suppose that in a random sample of an at-risk population
$13$ of $20$ subjects had hypertension. Estimate the prevalence
of hypertension in this population.
- $x = 13$ and $n=20$
- Consider a uniform prior, $\alpha = \beta = 1$
- The posterior is proportional to (see formula above)
$$
p^{x + \alpha - 1} (1 - p)^{n - x + \beta - 1} = p^x (1 - p)^{n-x}
$$
That is, for the uniform prior, the posterior is the likelihood
- Consider the instance where $\alpha = \beta = 2$ (recall this prior
is humped around the point $.5$) the posterior is
$$
p^{x + \alpha - 1} (1 - p)^{n - x + \beta - 1} = p^{x + 1} (1 - p)^{n-x + 1}
$$
- The "Jeffrey's prior" which has some theoretical benefits
  puts $\alpha = \beta = .5$

---

Here is another manipulate() example:

```r
pvals <- seq(0.01, 0.99, length = 1000)
x <- 13
n <- 20
myPlot <- function(alpha, beta) {
    plot(0:1, 0:1, type = "n", xlab = "p", ylab = "", frame = FALSE)
    lines(pvals, dbeta(pvals, alpha, beta)/max(dbeta(pvals, alpha, beta)), lwd = 3, 
        col = "darkred")
    lines(pvals, dbinom(x, n, pvals)/dbinom(x, n, x/n), lwd = 3, col = "darkblue")
    lines(pvals, dbeta(pvals, alpha + x, beta + (n - x))/max(dbeta(pvals, alpha + 
        x, beta + (n - x))), lwd = 3, col = "darkgreen")
    title("red=prior,green=posterior,blue=likelihood")
}
manipulate(myPlot(alpha, beta), alpha = slider(0.01, 100, initial = 1, step = 0.5), 
    beta = slider(0.01, 100, initial = 1, step = 0.5))
```

The idea that is shown here is that the posterior value is going to be somewhere between the prior value and the likelihood from the new data. The Bayesian provides a parameter (I think it's the $\alpha$) which indicates the position of the prior; and a parameter ($\beta$ I think) which indicates how certain we are of the prior, or how strong the prior is. Thus the prior tugs against how likely the new data are and the posterior ends up somewhere in between. For example, a posterior will be closer to a very strong prior.

---
## Credible intervals
- A Bayesian credible interval is the  Bayesian analog of a confidence
  interval
- A $95\%$ credible interval, $[a, b]$ would satisfy
  $$
  P(p \in [a, b] ~|~ x) = .95
  $$
- The best credible intervals chop off the posterior with a horizontal
  line in the same way we did for likelihoods 
- These are called highest posterior density (HPD) intervals

---
## Getting HPD intervals for this example
- Install the \texttt{binom} package, then the command

```r
library(binom)
binom.bayes(13, 20, type = "highest")
```

```
##   method  x  n shape1 shape2   mean  lower  upper  sig
## 1  bayes 13 20   13.5    7.5 0.6429 0.4423 0.8361 0.05
```

gives the HPD interval. 
- The default credible level is $95\%$ and
the default prior is the Jeffrey's prior.

---
Manipulate() example: (Again, code presented here, but it won't function outside of RStudio.)

```r
pvals <- seq(0.01, 0.99, length = 1000)
x <- 13
n <- 20
myPlot2 <- function(alpha, beta, cl) {
    plot(pvals, dbeta(pvals, alpha + x, beta + (n - x)), type = "l", lwd = 3, 
        xlab = "p", ylab = "", frame = FALSE)
    out <- binom.bayes(x, n, type = "highest", prior.shape1 = alpha, prior.shape2 = beta, 
        conf.level = cl)
    p1 <- out$lower
    p2 <- out$upper
    lines(c(p1, p1, p2, p2), c(0, dbeta(c(p1, p2), alpha + x, beta + (n - x)), 
        0), type = "l", lwd = 3, col = "darkred")
}
manipulate(myPlot2(alpha, beta, cl), alpha = slider(0.01, 10, initial = 1, step = 0.5), 
    beta = slider(0.01, 10, initial = 1, step = 0.5), cl = slider(0.01, 0.99, 
        initial = 0.95, step = 0.01))
```

