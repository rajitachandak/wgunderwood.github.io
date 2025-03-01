---
layout: post
title:  "Local Polynomial Regression 2: Bandwidth Selection"
date:   2022-03-29
---

This post, the second in a series on local polynomial regression,
investigates bandwidth selection procedures for the
Nadaraya--Watson estimator introduced previously.

In [part one](/2021/09/05/local-polynomial-regression-1.html)
we defined the Nadaraya--Watson estimator
with kernel $K$ and bandwidth $h$
and showed how it can be used to estimate a regression function.
Throughout this post we will focus for simplicity on the
popular Epanechnikov kernel $K(x) = \frac{3}{4}(1-x^2)$
and consider methods for selecting the bandwidth $h$.

{% include mathjax.html %}

<div style="display:none">
  $\newcommand \E {\mathbb{E}}$
  $\newcommand \P {\mathbb{E}}$
  $\newcommand \R {\mathbb{R}}$
  $\newcommand \Var {\mathrm{Var}}$
  $\newcommand \Cov {\mathrm{Cov}}$
  $\newcommand{\diff}[1]{\,\mathrm{d}#1}$
  $\DeclareMathOperator{\MSE}{MSE}$
  $\DeclareMathOperator{\IMSE}{IMSE}$
  $\DeclareMathOperator{\LOOCV}{LOO-CV}$
</div>

## Bandwidth selection in theory

A key concept in understanding bandwidth selection is the *bias-variance tradeoff*.
Intuitively, the bias of an estimator is the error it makes on average,
while the variance is a measure of how unpredictable the estimator is.

Crucially, a more "complex" estimator has less bias but more variance,
while increasing the number of data points tends to reduce the variance
and does not affect the bias.
As such, we can control the bias-variance tradeoff by increasing the complexity
of the estimator as more data becomes available.

Let $\widehat \mu(x)$ be an estimator of the regression function
$\mu(x) = \E[y_i \mid x_i = x]$.
Then the bias and variance of $\widehat \mu(x)$ are

$$
B(x) = \E\big[ \widehat \mu(x) \big] - \mu(x), \qquad
V(x) = \E\Big[ \big(\widehat \mu(x) - \E\big[ \widehat \mu(x) \big] \big)^2 \Big]
$$

respectively.
Crucially, the bias and variance together determine the
pointwise mean squared error (MSE) of the estimator defined as

$$
\MSE(x)
= \E\Big[ \big(\widehat \mu(x) - \mu(x) \big)^2 \Big]
= B(x)^2 + V(x).
$$

This property is known as the *bias-variance decomposition*.
Since we want an estimator which performs well over all points $x$,
it is common to define the following integrated versions of the bias and variance:

$$
B^2 = \int_\R B(x)^2 \diff{x}, \qquad
V = \int_\R V(x) \diff{x}.
$$

We can then aim to minimize the integrated MSE given by
$\IMSE = B^2 + V$.


### Theory of the Nadaraya--Watson estimator

Recall from
[part one](/2021/09/05/local-polynomial-regression-1.html)
that the Nadaraya--Watson estimator is defined as

$$
\widehat \mu(x) =
\frac{\sum_{i=1}^n y_i K\left(\frac{x_i-x}{h}\right)}
{\sum_{i=1}^n K\left(\frac{x_i-x}{h}\right)}.
$$

#### Bias

Assume that the data $(x_i, y_i)$ are independent
and identically distributed.
Let $x_i$ have density function $f(x)$ and write
$\sigma_K^2 = \int_\R x^2 K(x) \diff{x}$.
Then we can calculate the approximate bias
using Taylor's theorem as

$$
\begin{align*}
\E\big[\widehat \mu(x)\big] - \mu(x)
&\approx
\frac{\E\left[y_i \frac{1}{h} K\left(\frac{x_i-x}{h}\right)\right]}
{\E\left[\frac{1}{h} K\left(\frac{x_i-x}{h}\right)\right]}
- \mu(x) \\
&\approx
\frac{\mu(x)f(x) + h^2 \sigma_K^2
\big(\mu'(x)f'(x) + \mu''(x)f(x)/2 + \mu(x)f''(x)/2\big)}
{f(x) + h^2 \sigma_K^2 f''(x)/2}
- \mu(x) \\
&\approx
h^2 \sigma_K^2
\left( \frac{\mu'(x)f'(x)}{f(x)} + \frac{\mu''(x)}{2} \right).
\end{align*}
$$

#### Variance

Similarly if we write
$\sigma_\varepsilon^2 = \Var[y_i \mid x_i = x]$
(this does not depend on $x$
so we say the model is homoscedastic)
and $R_K = \int_\R K(x)^2 \diff{x}$
then the variance can be approximated using
$\Var[X/Y] \approx
(\Var[X]\E[Y]^2 + \Var[Y]\E[X]^2 - 2\Cov[X,Y]\E[X]\E[Y])/\E[Y]^4$.

$$
\begin{align*}
\Var\left[\frac{1}{n} \sum_{i=1}^n
y_i \frac{1}{h} K\left(\frac{x_i-x}{h}\right)\right]
&\approx
\frac{1}{nh} f(x) R_K \big( \mu(x)^2 + \sigma_\varepsilon^2 \big), \\
\Var\left[\frac{1}{n} \sum_{i=1}^n
\frac{1}{h} K\left(\frac{x_i-x}{h}\right)\right]
&\approx
\frac{1}{nh} f(x) R_K, \\
\Cov\left[\frac{1}{n} \sum_{i=1}^n
y_i \frac{1}{h} K\left(\frac{x_i-x}{h}\right),
\frac{1}{n} \sum_{i=1}^n
\frac{1}{h} K\left(\frac{x_i-x}{h}\right) \right]
&\approx
\frac{1}{nh} \mu(x) f(x) R_K
\end{align*}
$$

so


$$
\begin{align*}
\Var\big[\widehat \mu(x)\big]
&\approx
\frac{1}{nh} \frac{R_K \sigma_\varepsilon^2}{f(x)}.
\end{align*}
$$

#### MSE

Therefore the MSE and IMSE of the Nadaraya--Watson
are approximately

$$
\begin{align*}
\MSE(x)
&\approx
h^4 \sigma_K^4
\left( \frac{\mu'(x)f'(x)}{f(x)} + \frac{\mu''(x)}{2} \right)^2
+ \frac{1}{nh} \frac{R_K \sigma_\varepsilon^2}{f(x)}, \\
\IMSE
&\approx
h^4 \sigma_K^4
\int_\R
\left( \frac{\mu'(x)f'(x)}{f(x)} + \frac{\mu''(x)}{2} \right)^2
\diff{x}
+ \frac{1}{nh}
\int_\R
\frac{R_K \sigma_\varepsilon^2}{f(x)}
\diff{x}.
\end{align*}
$$

In principle it is now possible to select the bandwidth
by minimizing the IMSE over $h$.
However the IMSE depends on
$\mu(x)$, $f(x)$ and $\sigma_\varepsilon^2$,
each of which is itself unknown.
Nonetheless, balancing the bias and variance terms
shows that the optimal bandwidth must be on the order of
$h \asymp n^{-1/5}$.







## Bandwidth selection in practice

Since minimizing the IMSE is not feasible,
we investigate some alternative methods for selecting
the bandwidth.

### Minimizing the empirical integrated mean squared error

A natural first attempt at practical bandwidth selection
is to minimize some estimate of the IMSE.
We could replace the true IMSE

$$
\IMSE(h)
= \int_\R
\E\Big[ \big(\widehat \mu(x) - \mu(x) \big)^2 \Big]
\diff{x}
$$

with a sample version
known as the empirical IMSE,

$$
\widehat\IMSE(h)
= \frac{1}{n} \sum_{i=1}^n
\big(y_i - \widehat \mu(x_i) \big)^2,
$$

and minimize this instead.
However this does not work, as evidenced in Figure 1,
since minimizing the empirical IMSE
will always choose a bandwidth extremely close to zero!

<figure style="display: block; margin-left: auto; margin-right: auto;">
<img style="width: 500px; margin-left: auto; margin-right: auto;"
src="/assets/graphics/posts/images_local-polynomial-regression/min_mse_bandwidths.png">
<figcaption>
  Fig. 1: Empirical IMSE decreases to zero as the bandwidth decreases to zero.
</figcaption>
</figure>

To see why this is,
note that whenever $h$ is less than
the smallest gap between any two points $x_i$,
then the kernel centered at $x_i$ cannot
"see" any other points, so

$$
\widehat \mu(x_i) =
\frac{y_i K\left(\frac{x_i-x_i}{h}\right)}
{K\left(\frac{x_i-x_i}{h}\right)}
= y_i
$$

and thus $\widehat\IMSE(h) = 0$.
This phenomenon is illustrated in Figure 2,
which shows how a small bandwidth causes severe overfitting.


<figure style="display: block; margin-left: auto; margin-right: auto;">
<img style="width: 500px; margin-left: auto; margin-right: auto;"
src="/assets/graphics/posts/images_local-polynomial-regression/min_mse_data.png">
<figcaption>
  Fig. 2: With a small enough bandwidth, the data is interpolated.
</figcaption>
</figure>





### Leave-one-out cross-validation

A popular method for avoiding this phenomenon of overfitting is
leave-one-out cross-validation (LOO-CV).
The idea is to remove a point $(x_i,y_i)$ from the data set
and fit the estimator on the remaining data.
This estimator, denoted $\widehat \mu_{-i}(x)$, is then evaluated
at the data point which was initially left out and its squared error is recorded.
The leave-one-out cross-validation error is the average of these errors
over removing each data point in turn.
Formally,

$$
\LOOCV(h) = \frac{1}{n} \sum_{i=1}^n
\big( y_i - \widehat \mu_{-i}(x_i) \big)^2.
$$

The advantage of minimizing LOO-CV is that the model is always trained
and evaluated on different samples,
so is unable to "memorize" the data set.
This improves its generalization ability
and avoids selecting bandwidths which are too small.
Figure 3 shows how LOO-CV is minimized at a particular bandwidth value,
and Figure 4 demonstrates this to be a reasonable choice.

<figure style="display: block; margin-left: auto; margin-right: auto;">
<img style="width: 500px; margin-left: auto; margin-right: auto;"
src="/assets/graphics/posts/images_local-polynomial-regression/min_loo_cv_bandwidths.png">
<figcaption>
  Fig. 3: LOO-CV achieves a global minimum.
</figcaption>
</figure>

<figure style="display: block; margin-left: auto; margin-right: auto;">
<img style="width: 500px; margin-left: auto; margin-right: auto;"
src="/assets/graphics/posts/images_local-polynomial-regression/min_loo_cv_data.png">
<figcaption>
  Fig. 4: LOO-CV is able to select a sensible bandwidth.
</figcaption>
</figure>

However LOO-CV requires fitting $n$ different models
for each candidate bandwidth $h$.
This can make it expensive to compute in practice,
and its batch analogue, called $k$-fold cross-validation,
is often preferred.
Another option is generalized cross-validation,
which approximates the LOO-CV error and is even faster to compute.








## Concluding remarks

In this post we saw how a bandwidth can be selected using leave-one-out cross-validation.
However it is worth pointing out that sometimes a good bandwidth does not even exist,
such as when the regression function $\mu(x)$ is much "smoother" at some points than at others,
as seen in Figure 5.
Since the approximate bias term we derived depends on the curvature $\mu^{\prime\prime}(x)$,
a larger bandwidth is preferred for smoother parts of the regression function.

<figure style="display: block; margin-left: auto; margin-right: auto;">
<img style="width: 500px; margin-left: auto; margin-right: auto;"
src="/assets/graphics/posts/images_local-polynomial-regression/topologist_sine_curve.png">
<figcaption>
  Fig. 5: Some regression functions have no good global bandwidth.
</figcaption>
</figure>



## Next time

In the next post we will return to the problem of boundary bias
and show how local polynomial regression
(in particular local linear regression)
can help to alleviate it.


## References

* The University of Oxford's course in
Applied and Computational Statistics,
taught by
[Geoff Nicholls](http://www.stats.ox.ac.uk/%7Enicholls/) in 2018

* [An Introduction to Statistical Learning](https://trevorhastie.github.io/ISLR/)
by Gareth James, Daniela Witten, Trevor Hastie and Robert Tibshirani, 2013
