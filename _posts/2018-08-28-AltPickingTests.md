---
layout: post
title:  Herbie Alt Picking Tests
date:   2018-08-28 12:30:00
description: Testing Herbie's Alt Picking Capabilities
---

In the [last post](https://homes.cs.washington.edu/~dthien/blog/2018/AltStats/), we gathered some useful statistics on how Herbie uses the alt table. This post will build on the previous one, and will use terminology defined there. This time, we want to look more specifically at the alt picking function.

## Current Picking Function

The picking function Herbie currently uses is to pick the fresh alt with the lowest overall error. Ties are broken by picking the alt with the lowest cost. (Cost is an internal metric associated with each operation meant to roughly correspond with that operation's performance cost. It is likely that Herbie doesn't do a very good job at estimating this right now.) Intuitively, we can think of this picking function as not wasting our time on alts that have some fundamental flaw in them preventing them from improving accuracy, while spending the most time on alts that have already had some improvement done to them. However, it's not obvious that this is always the best strategy.

## Measuring the Picking Function

Now that we have a good idea of the problem domain, we have to come up with some way to measure how good our picking function is doing. We already have an easy way of comparing different picking functions to see which one is better: just run Herbie once for each picking function to see which gives you the best result. However, it should be noted that this doesn't give us how good we absolutely are; that is how much room for improvement we have from a perfect or oracular picking function. There also exists the possibility in this method with over-fitting our picking function to the current model of how Herbie works. Changes later on in some other part of Herbie (like the previously measured regimes) might cause a different picking function to be better, so care needs to be taken with this method to say "this picking function is the best one tested on this version of Herbie" rather than "this is the best picking function we tested."

Instead of just testing a bunch of picking functions to see which is the best, we would like to get more precise metrics on how good a picking function is. To do this, we need to develop some kind of oracular picking function that will represent the best possible case. The best oracle would be the one that runs the remaining iterations of Herbie on all the alts in the alt table and then tells us which one we should have picked at each step based on Herbie's output. However, this method is very costly because it involves a full or near-full run of Herbie on each alt in the alt table, which is a little resource intensive for how useful we believe this information will be. It might be nice in the future to get more comprehensive data by running this oracle, but for now, we will restrict ourselves to a slightly less powerful version. 

We will instead use an oracle that will do one further iteration of improvement on that alt, and then calculate the overall error of the program using the regimes oracle we developed in the previous posts. This oracle is oracular in the sense that: given an input Herbie improvement algorithm and a way of measuring error (our regimes oracle), with one iteration of improvement, what is the best alt to have picked. Once again, we are relying on a fixed Herbie improvement specification, so care should be taken to not say that a given picking function is always the best because changes to Herbie could change the optimal picking function. As a reminder, the regimes oracle calculates the error of the best alt at every point and then combines them for the error score. This oracle is a good balance of a "true oracle" and run time practicality, but one can also imagine the same oracle being applied to any number of improvement iterations.

Another oracle we might consider is an oracle that runs an improvement cycle on every alt in the alt table, and then combines the resulting alt tables into that iteration's final alt table. For example, our alt table might have alts $$a$$, $$b$$, and $$c$$ which, after an improvement iteration, produce alt tables $$A$$, $$B$$, and $$C$$. This oracle's output alt table would be $$A \cup B \cup C$$. This oracle would tell us the effectiveness of a picking function in general; that is if we should even pick a singular alt to improve, or if we get large benefits from trying to improve every alt.

## Data

So far, the oracle has been run on the hamming and mathematics suites have been tested, the results of which are listed in the tables below. The absolute error score difference is the difference between the error score of Herbie and that of the oracle. The herbie% improvement is the improvement by herbie over the improvement by the oracle. If $$H$$ and $$O$$ are lists holding the absolute error scores for Herbie and the oracle's iterations respectively, then

$$
\texttt{herbie\%}(n) = \frac{H[n-1] - H[n]}{H[n-1] - O[n]}
$$.

This will tell us how close herbie came to the best improvement. Closer to 1 is better. For data collection purposes, divide by 0 has been treated as 1 (comes up when there are no bits of improvement left).

#### Hamming Absolute Error Score Difference

| | Iter 1 | Iter 2 | Iter 3 |
|-|--------|--------|--------|
| Mean | 0.0072 | 0.015 | 0.052 |
| Std Dev | 0.022 | 0.037 | 0.17 |
| Max | 0.098 | 0.15 | 0.81 |

#### Hamming Herbie%

| | Iter 1 | Iter 2 | Iter 3 |
|-|--------|--------|--------|
| Mean | 0.84 | 0.71 | 0.70 |
| Std Dev | 0.33 | 0.44 | 0.46 |
| Max | 1 | 1 | 1 |

#### Mathematics Absolute Error Score Difference

| | Iter 1 | Iter 2 | Iter 3 |
|-|--------|--------|--------|
| Mean | 0.76 | 0.79 | 0.88 |
| Std Dev | 3.91 | 3.70 | 3.85 |
| Max | 22.15 | 19.91 | 19.91 |

#### Mathematics Herbie%

| | Iter 1 | Iter 2 | Iter 3 |
|-|--------|--------|--------|
| Mean | 0.65 | 0.73 | 0.71 |
| Std Dev | 0.43 | 0.42 | 0.44 |
| Max | 1 | 1 | 1 |

We can see looking at the data that Herbie does a pretty good job on the hamming suite of benchmarks with a maximum absolute error score difference of only $$0.81$$. However, the same can't be said about the mathematics suite. There the maximum absolute error score difference is at a whopping $$22.15$$; Herbie's alt picking function does a really bad job. Looking through the data files, the benchmark corresponding with this number is the [2-ancestry mixing positive discriminant benchmark](http://herbie.uwplse.org/reports/1534939666:warfa:develop:4e924be312/mathematics/30-2ancestrymixingpositivediscriminant/graph.html). We can see that Herbie isn't able to improve the error in this case at all, so here we have a concrete example where a better picking function would lead to huge benefits.

Overall, it looks like Herbie does a pretty good job picking the right alt, even on the mathematics suite, which Herbie hasn't had nearly as much scrutiny under. The mean error score difference is below $$1$$ bit of error, and the Herbie% improvement numbers are very close to those of the hamming suite of benchmarks, particularly in the later iterations. But it does seem like there are a few cases where having a better picking function would make a big difference.

## Future Work

Now that we have a good way of benchmarking how good a picking function is, both relative to other ones and absolutely, we can start trying out different picking functions until we land on one that can solve the current problems we run into. There are also a few other benchmark suites which haven't been run on yet, and it would be useful to get more examples of where our current picking function fails.

Aside from testing the picking function, it may also be useful to look at the pruning step of Herbie to see if we are accidentally getting rid of valuable alts which may be required for further improvement.

## Files
Data was collected through Herbie and extracted to the following files:
* [Hamming data]({{ "/assets/files/posts/AltPickingTests/hamming.log" | absolute_url }})
* [Mathematics data]({{ "/assets/files/posts/AltPickingTests/mathematics.log" | absolute_url }})
