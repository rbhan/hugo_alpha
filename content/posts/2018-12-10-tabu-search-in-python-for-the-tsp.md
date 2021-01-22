+++
title = "Tabu search in Python (for the TSP)"
slug = "tabu-search-in-python-for-the-tsp"
date = "2018-12-10"
categories = [ "archive", "notes" ]
tags = [ "0to30", "traveling salesman problem", "tabu search" ]
+++

[Travelling Santa](https://www.kaggle.com/c/traveling-santa-2018-prime-paths/overview) is the travelling salesman problem with a festive twist introduced by additonal prime city pathing requirement. This constraint becomes non-trivial to plug into a black box solver such as Concorde.

We attempted to get around this by optimizing the naive Concorde solution using tabu search with the prime penalty built into the fitness evaluation.

Surprisingly, there weren't many relevant Google hits for "tabu search in python" (and none contained immediately implementable code), so it further seemed like a worthwhile exercise to write our own tabu search.

The most useful results are [a Ruby implementation to solve the Berlin52 TSP](http://www.cleveralgorithms.com/nature-inspired/stochastic/tabu_search.html),  and [an example python algorithm](https://www.techconductor.com/algorithms/python/Search/Tabu_Search.php) which relies on building a list of nearest neighbour distances for each node. However, the latter isn't viable for our problem because we have to visit ~200k cities; our nearest neighbour dictionary would be too large to hold in memory. We needed a more general algorithm. The Ruby code wasn't a bad starting point, albeit Ruby is an appallingly unreadable language.

There are myriad formal graph notations for tabu search. More impressively, it's also apparently the topic of [at least one lecture](https://ocw.mit.edu/courses/sloan-school-of-management/15-053-optimization-methods-in-management-science-spring-2013/lecture-notes/MIT15_053S13_lec17.pdf) for Sloan management students. I'm no mathematician, but my eli5 understanding of tabu search is that it's an optimization method (*metaheuristic*) which tries to avoid getting stuck in local minima by 1) sometimes taking a step uphill instead of downhill, assuming we want to find a global minimum; and 2) adding solutions to a taboo (*tabu*) list of steps it's not allowed to visit for some short-term period, forcing exploration of new solutions even if they are worse. I don't know why it's spelled *tabu*.

Overview of the optimization routine:

  * Given an undirected graph, start with initial solution *T* -- this is an initial path through our cities
  * Establish some fitness function -- this is total distance of path *T*, calculated with the prime penalty

{{< gist rbhan 5529af9871ad6aec0ce6ece391c0efff >}}

  * Establish some termination criterion -- we set a maximum number of iterations
  * Find <i>x</i> non-tabu candidates in the neighbourhood of *T* -- not to be confused with geographical neighbourhoods of the cities, these are candidate paths in the solution space "neighbourhood" of the initial path, generated using a stochastic 2-opt swap which randomly crosses two edges and returns the resulting path

{{< gist rbhan f45ef997752a68e6c5ea9f8d8d247e53 >}}

  * Choose the candidate solution that most improves fitness -- whichever new path among the *x* candidates has the lowest distance, *even if it's higher than the previous best*

{{< gist rbhan a3308e23cdf339b12d4fa770eace91de >}}

  * Add this new candidate solution to the tabu list

And finally, the [holistic script](http://nbviewer.jupyter.org/github/rbhan/kaggle_tsp/blob/master/tabu_search_with_primes_jit.ipynb), although we will certainly be replacing the local search heuristic with higher order k-opt swaps, or better yet, the Lin-Kernighan heuristic (which Concorde uses). A key challenge for the latter is to implement it without pre-calculating the cost matrix.