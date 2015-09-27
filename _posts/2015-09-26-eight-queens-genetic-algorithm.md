---
layout: post
title: "Baby's First Genetic Algorithm"
excerpt: A genetic algorithm approach and solution to the eight queens problem.
tags: 
  - eight_queens
  - genetic_algorithm
  - code_snippet
  - java
comments: true
image: 
  feature: "dna-genes.jpg"
  credit: PracticalEthics
  creditlink: "http://blog.practicalethics.ox.ac.uk/2015/04/is-sexual-offending-genetic/"
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

> Part Two of Four

If you're not familiar with the 8 Queens problem, please read my [previous post]({% post_url 2015-09-23-eight-queens-problem %}) or consult an online resource.[^1]

## Genetic Algorithms

Genetic algorithms are an artificial intelligence technique. Genetic algorithms are a type of search heuristic which inspired by biology. Genetic algorithms are a variant of the stochastic beam search, using sexual reproduction instead of asexual reproduction to populate the candidate solutions.

Genetic algorithms have four key properties:

* Selection
* Crossover
* Mutation
* Inheritance

Initially, the population pool is randomly generated.  In the eight queens problem, we can represent each position of an array as a column for a queen to be placed, and the value of each array as the row. Using this presentation, it's fairly trivial to generate an initial population group.

{% highlight java %}
private final int GRID_SIZE = 8;
private final int POPULATION_SIZE = 6;
private final Random rand = new Random();

private List<Integer[]> generatePopulation() {
    List<Integer[]> returnValue = new ArrayList<>();
    for (int i = 0; i < POPULATION_SIZE; i++) {
        Integer[] columnSet = new Integer[GRID_SIZE];
        for (int j = 0; j < GRID_SIZE; j++) {
            columnSet[j] = rand.nextInt(GRID_SIZE);
        }
        returnValue.add(columnSet);
    }
    return returnValue;
}
{% endhighlight %}

### Selection

Once the population pool has been randomly generated, parent individuals are assessed for their fitness. The best selection for the fitness function, in the case of the eight queens problem is based on the number of attacking queens. In the eight queens problem, the number of attacking queens is 28, but can be generalized for the n-queens problem as `(n-1) + (n-2) + ... + (n-(n-1)) + (n-n)`.

With that in mind, we can define a fit candidate as one that has no collisions, and scale its value to 1.0.

{% highlight java %}
double assessFitness(Integer[] candidate) {
    int collisions = 0;
    final int MAXIMUM_COLLISIONS = calculateMaxCollisions();
    for (int i = 0; i < GRID_SIZE - 1; i++) {
        for (int j = i + 1; j < GRID_SIZE; j++) {
            if ((candidate[i].equals(candidate[j])) || j - i == Math.abs(candidate[i] - candidate[j]))
                collisions++;
        }
    }
    return (MAXIMUM_COLLISIONS - collisions) / (double) MAXIMUM_COLLISIONS;
}
{% endhighlight %}

This value, while useful, is not sufficient for the selection of a candidate in a population. In order to select a candidate, overall fitness values are not sufficient. The candidate must be selected based on its *relative* fitness. We can do that by scaling the fitness values to the population.

{% highlight java %}
private Double[] probabilityOfMating(Double[] fitnessOfCandidates) {
    Double[] returnValue = new Double[POPULATION_SIZE];
    double sum = 0;
    for (Double d : fitnessOfCandidates) {
        sum += d;
    }

    for (int i = 0; i < fitnessOfCandidates.length; i++) {
        returnValue[i] = fitnessOfCandidates[i] / sum;
    }
    return returnValue;
}
{% endhighlight %}

Using the `probabilityOfMating()` function, we will receive a `Double[]` value of relative fitnesses. Once we have this value, we can utilize culling[^2] to select a parent based on their relative fitness.

{% highlight java %}
private final double CULLING_THRESHOLD = (1 / POPULATION_SIZE) * 0.85;
private Integer[] pickRandomParent(List<Integer[]> candidates, Double[] fitness, Integer[] invalidSet) {
    while (true) {
        for (int i = 0; i < POPULATION_SIZE; i++) {
            if (candidates.get(i) == invalidSet || fitness[i] < CULLING_THRESHOLD)
                continue;
            double val = rand.nextDouble();
            if (val <= fitness[i]) {
                return candidates.get(i);
            }
        }
    }
}
{% endhighlight %}

### Crossover and Mutation

Now that the parents have been selected, we can crossover the two parents. A random splice point is generated, and the child is made up of first parent up until the splice-point, where the second parent continues. At this point, random mutations are also introduced into the offspring.

{% highlight java %}
private final double MUTATION_RATE = 0.05; //1.0 = 100% mutation rate
private Integer[] crossover(Integer[] candidateOne, Integer[] candidateTwo) {
    int splitPoint = rand.nextInt(GRID_SIZE - 1) + 1; //Splits between 1 and (n - 1), where n = # of queens
    Integer[] returnValue = new Integer[GRID_SIZE];
    for (int i = 0; i < GRID_SIZE; i++) {
        if (rand.nextDouble() <= MUTATION_RATE)
            returnValue[i] = rand.nextInt(GRID_SIZE);
        else if (i < splitPoint)
            returnValue[i] = candidateOne[i];
        else
            returnValue[i] = candidateTwo[i];
    }
    return returnValue;
}
{% endhighlight %}

It's worth noting that mutation is occasionally implemented as an additional step after crossover.

### Inheritance

It's clear to see how the child has inherited portions of its parents. Often times, the population is quite diverse in early iterations of this process. This means that the crossover step makes very large steps in the state space early on, and smaller ones later in the process once the states are fairly similar.

Inheritance is a very important property of genetic algorithms, as it, through the crossover step, allows the algorithm to trend upwards.

For the full code sample used here, please visit [GitHub](https://github.com/hesham8/daily-challenge/tree/master/8Queens). 


If you found this interesting, check *[this](http://www.damninteresting.com/on-the-origin-of-circuits/)* out.

[^1]: <https://en.wikipedia.org/wiki/Eight_queens_puzzle>
[^2]: <https://en.wikipedia.org/wiki/Culling>
