---
date: 2013-12-11T00:00:00+00:00
title: "Simulated Maximum Likelihood with R"
tags: [R]
menu:
  main:
    parent: Blog
    identifier: /blog/smm_R
    weight: 10
---

<head>
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

</head>

<body>

<p>This document details section <em>12.4.5. Unobserved Heterogeneity 
Example</em> from Cameron and Trivedi's book - MICROECONOMETRICS: Methods and 
Applications. The original source code giving the results from table 12.2 are 
available from the authors&#39; site <a 
href="http://cameron.econ.ucdavis.edu/mmabook/mmaprograms.html">here</a> and 
written for Stata. This is an attempt to translate the code to R. I'd like to 
thank Reddit user <a 
href="http://www.reddit.com/user/anonemouse2010">anonemouse2010</a> for his 
advice which helped me write the function.</p>

<p>Consult the original source code if you want to read the authors&#39; comments. If you want the R source code without all the commentaries, grab it <a href='/assets/code/simulated_max_lik.R'>here</a>. This is not guaranteed to work, nor to be correct. It could set your pet on fire and/or eat your first born. Use at your own risk. I may, or may not, expand this example. Corrections, constructive criticism are welcome.</p>

<p>The model is \( y=\theta+u+\varepsilon \) where \( \theta \) is a scalar parameter equal to 1. \( u \) is extreme value type 1 (Gumbel distribution), \( \varepsilon \leadsto \mathbb{N}(0,1) \). For more details, consult the book.</p>

<h3>Import the data</h3>

<p>You can consult the original source code to see how the authors simulated the data. To get the same results, and verify that I didn&#39;t make mistakes I prefer importing their data directly from their website.</p>

<pre><code class="r">data &lt;- read.table(&quot;http://cameron.econ.ucdavis.edu/mmabook/mma12p2mslmsm.asc&quot;)
u &lt;- data[, 1]
e &lt;- data[, 2]
y &lt;- data[, 3]
numobs &lt;- length(u)
simreps &lt;- 10000
</code></pre>

<h3>Simulation</h3>

<p>In the code below, the following likelihood function:

$$\log{\hat{L}_N(\theta)} = \dfrac{1}{N} \sum_{i=1}^N\log{\big( \dfrac{1}{S}\sum_{s=1}^S \dfrac{1}{\sqrt{2\pi}} \exp \{ -(-y_i-\theta-u_i^s)^2/2 \}\big)}$$

which can be found on page 397 is programmed using the function <code>sapply</code>.</p>

<pre><code class="r">denssim &lt;- function(theta) {
    loglik &lt;- mean(sapply(y, function(y) log(mean((1/sqrt(2 * pi)) * exp(-(y - theta + log(-log(runif(simreps))))^2/2)))))
    return(-loglik)
}
</code></pre>

<p>This likelihood is then maximized:</p>

<pre><code class="r">system.time(res &lt;- optim(0.1, denssim, method = &quot;BFGS&quot;, control = list(maxit = simreps)))
</code></pre>

<pre><code>##    user  system elapsed 
##   21.98    0.08   22.09
</code></pre>

<p>Convergence is achieved pretty rapidly, to </p>

<pre><code>## [1] 1.101
</code></pre>

<p>which is close to the true value of the parameter 1 (which was used to generate the data). </p>

<p>Let&#39;s try again with another parameter value, for example \( \theta=2.5 \). We have to generate y again:</p>

<pre><code class="r">y2 &lt;- 2.5 + u + e
</code></pre>

<p>and slightly modify the likelihood:</p>

<pre><code class="r">denssim2 &lt;- function(theta) {
    loglik &lt;- mean(sapply(y2, function(y2) log(mean((1/sqrt(2 * pi)) * exp(-(y2 - 
        theta + log(-log(runif(simreps))))^2/2)))))
    return(-loglik)
}
</code></pre>

<p>which can then be maximized:</p>

<pre><code class="r">system.time(res2 &lt;- optim(0.1, denssim2, method = &quot;BFGS&quot;, control = list(maxit = simreps)))
</code></pre>

<pre><code>##    user  system elapsed 
##   12.56    0.00   12.57
</code></pre>

<p>The value that maximizes the likelihood is: </p>

<pre><code>## [1] 2.713
</code></pre>

<p>which is close to the true value of the parameter 2.5 (which was used to generate the data). </p>

</body>
