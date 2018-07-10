---
layout: post
title:  Herbie Regime Testing Update 1
date:   2018-07-09 16:30:00
description: First update to the Herbie regime testing&#58; better graphs.
---

One of the aspects lacking in the graphs of the [previous post](https://homes.cs.washington.edu/~dthien/blog/2018/HerbieRegimeTesting/) was more detailed information on how well Herbie does compared to the optimal regimes. After running some more regime tests on the [Compound Interest expression](http://herbie.uwplse.org/reports/1529397374:warfa:develop:b6189b1c10/numerics/25-CompoundInterest/graph.html), I now have some graphs that better illustrate this property.

![Herbie Regime % Improvement]({{ "/assets/img/posts/RegimeTestingUpdate1/RegimePercentImprovement1.png" | absolute_url }}){:width="700px"}

This graph plots each resampled point ($$i$$ vs $$n$$) in greyscale as a function of Herbie's percentage improvement defined by

$$ \texttt{herbie \% improvmement } = \frac{\texttt{baseline err } - \texttt{ herbie err}}{\texttt{span}} $$
where
$$ \texttt{span } = \texttt{baseline err } = \texttt{ oracle err} $$

The darker points are the those where the Herbie % improvement is low, and there is a minimum darkness to make all the points visible.

There are a few interesting things about this graph, most notably, the three triangular regions that Herbie does particularly badly. The areas very close to the branch condition also have a low Herbie % improvement, and there is an interesting banding effect near the right side of the graph. The three triangular regions are the first things to look at moving forward, as they constitute the largest source of error.

It should be noted that the graph shouldn't just be taken by itself, as there is currently no indication of how many bits of error this percent improvement makes up. The Compound Interest expressions has span of $$17.40$$ bits. Compare this to [this expression](http://herbie.uwplse.org/reports/1530112290:warfa:regime-testing:1beee99a87/physics/1-VandenBroeckandKellerEquation24/graph.html) which has a span of only $$0.12$$ bits and the following graph.

![Vanden Broeck and Keller Equation 24 % Improvement]({{ "/assets/img/posts/RegimeTestingUpdate1/RegimePercentImprovement2.png" | absolute_url }}){:width="700px"}

Despite the significant number of points where the oracle does better than Herbie on, there is little to be gained by further improving Herbie's regimes for this specific example.
