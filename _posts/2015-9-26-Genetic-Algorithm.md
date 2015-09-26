---
layout: post
title: Baby's First Genetic Algorithm
subtitle: Solving the 8 Queens Problem
---

Genetic algorithms have their roots in biological evolution and reproduction. Genetic algorithms are a variant of stochastic beam search, with the key difference being that the parent states reproduce sexually rather than asexually.

The genetic algorithm begins by generating a random set of states. Each state is assessed for fitness. Once the states have been assessed for fitness, the probability of reproducing is calculated for each of these states. Two individual states are selected from the population state pool and are instructed to "mate". The mating, or crossover, process involves selecting a random position in the state-string and splicing the two parent states together at that position. At this time, mutations may also be introduced into the child state. The child states eventually replace the parent states, and this process continues until a state that is fit enough has been found.

Let's take a look at some of the components of such an algorithm.

In this example, I'll be using a genetic algorithm to solve the 8-Queens problem. Let's begin by taking a look at the main function, and then we'll delve into the private utility functions.

```java
private final int GRID_SIZE = 8; //8 Queens but could be an N-queens prob. Fitness function will auto-magically adjust
private final int POPULATION_SIZE = 6;
private final double MUTATION_RATE = 0.05; //1.0 = 100% mutation rate
private final double CULLING_THRESHOLD = (1 / POPULATION_SIZE) * 0.85;
private final int MAX_ITERATIONS = -1; //-1 == INFINITE
private final Random rand = new Random();

Integer[] geneticAlgorithmSolution() {
    List<Integer[]> population = generatePopulation();
    Double[] fitness = new Double[population.size()];
    int currentIteration = 0;
    for (int i = 0; i < POPULATION_SIZE; i++) {
        fitness[i] = assessFitness(population.get(i));
    }
    while (!populationContainsSolution(population, fitness) && ((MAX_ITERATIONS == -1) || currentIteration < MAX_ITERATIONS)) {
        Double[] matingProbability = probabilityOfMating(fitness);
        List<Integer[]> newPopulation = new ArrayList<>();
        while (newPopulation.size() < POPULATION_SIZE) {
            Integer[] parent1 = pickRandomParent(population, matingProbability, null);
            Integer[] parent2 = pickRandomParent(population, matingProbability, parent1);
            Integer[] candidate = crossover(parent1, parent2);
            newPopulation.add(candidate);
        }
        population = newPopulation;
        for (int i = 0; i < POPULATION_SIZE; i++) {
            fitness[i] = assessFitness(population.get(i));
        }
        currentIteration++;
    }
    Integer[] candidate = population.get(0);
    for (int i = 0; i < fitness.length; i++) {
        if (fitness[i] == 1.0) {
            return population.get(i);
        }
        if (assessFitness(candidate) < fitness[i])
            candidate = population.get(i);
    }
    return candidate;
}
```

The first thing many will notice is the large list of final constant values at the top of the file. Tweaking genetic functions is an art as much as a science, and without a meta-heuristic optimization of these numbers, it can be guess-and-check-work. The function is currently set up for 8 queens, but can easily be modified to more or less queens by modifying the `GRID_SIZE` variable.

This particular genetic algorithm includes population culling. Culling is the art of removing un-fit candidates, and the `CULLING_THRESHOLD` determines the cutoff. In this case, candidates that perform less than the base-line `1/POPULATION_SIZE` are removed.

Let's take a look at some of the more interesting functions:

```java
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
```

The crossover function is the large appeal to this technique. The crossover operation allows large blocks of letters that have evolved independently to perform useful functions. And yet, it can be proven mathematically that crossover provides no advantage if the initial positions of the genetic code are random. I'll leave that issue up to the mathematicians.

Intuitively, however, it makes sense why this would work. Independent blocks that have a partial match to a solution can be very useful in trending towards a final solution.


```java
/*
 Time Complexity: O(n^2). Where n = # of queens.
 Space complexity: O(1).
*/
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
```

For an N-Queens problem, the fitness function is relatively straightforward. There is a fixed number of possible collisions, MAX_COLLISIONS which can be calculated as `(n-1) + (n-2) + (n-3) + ... + 1 + 0`. Subtracting the number of collisions in the current configuration can provide a relative fitness. Dividing it by the maximum number of collisions will scale that number between 0.0 (worst fitness) and 1.0 (solution).

This number will have to be scaled once more to determine the probability of the members of the population to mate.

```java
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
```

As can be seen, genetic algorithm solutions are relatively straightforward to implement.

For the complete code used in this post, please visit [my GitHub page](https://github.com/hesham8/daily-challenge/tree/master/8Queens).
