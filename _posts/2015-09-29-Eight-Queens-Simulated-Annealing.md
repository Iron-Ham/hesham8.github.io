---
layout: post
title: "Cooling Molten Problem States"
excerpt: "A simulated annealing approach to solving Eight Queens"
tags: [eight_queens, simulated_annealing, code_snippet, java]
comments: true
image:
  feature: metallurgy.jpg
  credit: Institute Of Metallurgical and Foundry Engineering
  creditlink: http://metont.uni-miskolc.hu/en/?page_id=2890
published: true
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>Overview</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

> Part Three of Four

If you're not familiar with the 8 Queens problem, please read my [previous post]({% post_url 2015-09-23-eight-queens-problem %}). Make sure to also check out the [genetic algorithm approach]({% post_url 2015-09-26-eight-queens-genetic-algorithm %}).

## Hill Climbing Search

{% gist hesham8/99f107dfdbe7e7a7bcfa %}
Above is the pseudo-code for a standard hill-climbing search. It's a simple loop that moves, well, up-hill. Hill climbing doesn't look beyond the current state's neighbors, and so it can get stuck in a local maxima, ridges, and plateaus. Hill climbing is a greedy technique because it doesn't look ahead.

At some point, in many problems, the algorithm reaches a point where it can no longer make forward progress. Hill climbing gets stuck an astonishing *86%* of the time, but it's easy to see why. With slight modifications, hill climbing can be a successful algorithm *96%* of the time. Should we allow sideways moves (up to a finite cap, `m`), then we are much more likely to solve the problem. However, we still get stuck 4% of the time.

There are several ways of making this algorithm *complete*. Random-restart hill climbing is such a method. The idea behind random-restart is to continually restart the hill-climb if it gets stuck. Surely, this is a complete algorithm but could run for an effectively infinite amount of time.

Enter simulated annealing.

### Simulated annealing

A hill climbing algorithm that doesn't make downhill moves towards states with lower values cannot be complete; those algorithms tend to get stuck at local maxima. A purely random walk through the state-space is complete, but extremely inefficient. The combination of these two techniques is *simulated annealing*. Let's take a look at the pseudo code for simulated annealing.

{% gist hesham8/b1b079c53f4f12f40455 %}

If we compare the inner-most loop of hill-climbing to simulated annealing, we will see that they are quite similar. Instead of picking the **best** move, we pick a **random move**.

{% highlight java %}
private int[] generateNeighborState(int[] currentState) {
    int i = rand.nextInt(GRID_SIZE);
    int j = rand.nextInt(GRID_SIZE);
    currentState[i] = j;
    return currentState;
}
{% endhighlight %}

If the move improves the situation, the move is accepted. Otherwise, the move is only accepted with some probability `P`.

{% highlight java %}
int[] simulatedAnnealing() {
     int[] currentState = generateInitialState();
     int iter = 0;
     double temp = INITIAL_TEMP;
     while (MAX_ITERATIONS == -1 || iter < MAX_ITERATIONS) {
         temp = scheduleAnnealing(temp);
         double currentStateFitness = calculateStateFitness(currentState);
         if (currentStateFitness == 1.0) return currentState;
         int[] nextState = generateNeighborState(currentState);
         double nextStateFitness = calculateStateFitness(nextState);
         double delta = nextStateFitness - currentStateFitness;
         if (delta > 0) {
             currentState = nextState;
         } else if (rand.nextDouble() <= probabilityOfAcceptance(delta, temp)) {
             currentState = nextState;
         }

         iter++;
     }
     return currentState;
 }

 private double probabilityOfAcceptance(double delta, double temperature) {
     return Math.exp(delta / temperature);
 }
{% endhighlight %}

Beyond that, the simulated annealing approach shares much of its code with the genetic algorithm approach. The `calculateSateCost()` function is identical to the genetic algorithm's `assessFitness()` function. It generates its initial state in the same way as well. The only non-fixed component of simulated annealing is the annealing schedule.

### The Annealing schedule

Deciding on an annealing schedule is as much an art as it is a science. There's been [some research](http://www.fys.ku.dk/~andresen/BAhome/ownpapers/permanents/annealSched.pdf) on annealing schedules and their relative performances. Some adhere to logarithmic schedules, others to geometric ones, some to linear functions, and others to more complex functions[^1].

[^1]: <http://www.researchgate.net/publication/238977971_Thermodynamic_Length_and_Dissipated_Availability>

Here's an example geometric annealing schedule:

{% highlight java %}
private double scheduleAnnealing(double currentTemp) {
    return currentTemp * 0.97;
}
{% endhighlight %}


## Next Time

For the fourth and final part of this series, I will perform a series of run-time analyses on the three methods. Bear in mind that the relative performances of the genetic and simulated annealing algorithms have much to do with their constant figures. As always, the code used here is [available on GitHub](https://github.com/hesham8/daily-challenge/blob/master/8Queens/SimulatedAnnealing.java).
