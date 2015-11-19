---
layout: post
title: "Constructing A Hidden Markov Model"
excerpt: "A Spell Checker Utilizing AI Techniques"
tags: [AI, markov, spell, checker, hidden, java]
comments: true
---

Markov models are wonderful things. They're probabilistic models used to model dynamic states. There are four commonly used Markov models, and the type of model depends on whether the system is autonomous or controlled, and whether or not the system state is fully or partially observable.

Markov chains are autonomous systems wherein the system state is fully observable. All Markov processes are dependent on two properties: conditional independence (the same as is used in Bayes theorem), and the Markov property. The Markov property is as follows: the probability distribution of the next state depends only on the current state and *not* on the sequence of events that preceded it.

A hidden Markov model is a Markov chain wherein the state is only *partiallY* observable. Observations into the system are insufficient to determine the state of the system. There are several algorithms involving HMMs, but today I'll be exploring the Viterbi. The Viterbi algorithm is used to find the most likely sequence of states in a HMM. Common uses of the Viterbi are speech recognition, spelling correction, and handwriting recognition.

Today, I'll be demonstrating a HMM that corrects for spelling errors.

## Our Data Set

We need something relatively long. A document of sufficient length and lexical variety is the Unabomber's manifesto. I must state that this document *has not* been chosen for political reasons, but only for its size and lexical properties. For simplicity, all numbers and punctuation have been converted to white space, and all letters were converted to lower case. The remaining text is a sequence sequence of only lower-case characters and the space character, which has been represented by an underscore. Typos were then artifically added to the data -- with 90% probability, the correct letter is transcribed but with 10% probability a neighboring keyboard letter is struck. Spaces are always correctly transcribed.

Our data set looks like this:

    b b
    y y
    _ _
    t y
    h j
    e w
    i i
    r r
    _ _
    o o
    w q
    n n
    _ _
    a a
    c c
    c f
    o o
    u u
    n n
    t f
    _ _
    v v
    i i
    o o
    l l
    e d
    n n
    c c
    e w
    _ _


## The Viterbi Algorithm

The Viterbi algorithm requires three sets of probabilities:

1. The prior probability of any one event occurring.
2. The emission probability (the chance of event *o* occuring given state *s* -- P(o\|s)).
3. The transition probability -- the chance of moving from state *s* to state *t*.

Calculating the prior probability of this data set is trivial, and there are three easy, valid methods of doing so:

* The raw chance of selecting any one letter (1/26).
* The frequency that letters appear in the data set.
* The frequency that any given letter appears in words in the [English language](https://en.wikipedia.org/wiki/Letter_frequency#Relative_frequencies_of_letters_in_the_English_language).

The transition probability can be calculated by counting the movements between states, and similarly, the emissions can be calculated by counting the number of times a state and an output appear together. These counts, however, must be normalized. I'm using the Laplace normalization to normalize the data. The Laplace normalization is a simple formula: `(1 + count_Occurance) / (count_States + possible_States)`.

The primary calculation in the Viterbi algorithm is as follows:

    where s = 0
    k(s) = P_prior(state) * P_emission(observation|state)

    where s = 1..n
    k(s) = P(oldState) * P_trans(oldState -> newState) * P_emission(observation|newState)

These state path values are then maximized for each state *s*, and then the maximum of all possible states is selected as the output of the HMM.

### Calculating Our Probabilities

In the following code sample, I'll be assuming that the prior probability will be calculated using method two.

{% highlight java %}
private void trainModel(String filePath) {
    Scanner fileReader = null;

    char previousState = '_';

    try {
        fileReader = new Scanner(new File(filePath));

        while (fileReader.hasNext()) {
            String currentLine = fileReader.nextLine().trim();
            String[] lineComponents = currentLine.split("\\s+");

            char currentState = lineComponents[0].charAt(0);
            char output = lineComponents[1].charAt(0);

            // Increment state occurrence counter for calculating prior probabilities
            // Increment emissions counter
            // Increment transitions counter
            if (output != '_' && previousState != '_') {
                outputCount[getCharValue(currentState)] += 1;
                emissionsCount[getCharValue(currentState)][getCharValue(output)] += 1;
                transitionsCount[getCharValue(previousState)][getCharValue(currentState)] += 1;
            }

            previousState = currentState;
        }

    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } finally {
        if (fileReader != null) {
            fileReader.close();
        }
        calculatePriorProbability();
        calculateEmissions();
        calculateTransitions();
    }
}

private void calculatePriorProbability() {
    int sum = sumOfArray(outputCount);
    for (int i = 0; i < priorProbability.length; i++) {
        priorProbability[i] = (double) outputCount[i] / sum;
    }
}

private void calculateEmissions() {
    for (int i = 0; i < emissions.length; i++) {
        int sum = sumOfArray(emissionsCount[i]); // Get the count of state s

        for (int j = 0; j < emissions[i].length; j++) {
            emissions[i][j] = ((double) emissionsCount[i][j] + 1) / (sum + numberOfStates); // Smooth probability
        }
    }
}

private void calculateTransitions() {
    for (int i = 0; i < transitions.length; i++) {
        int sum = sumOfArray(transitionsCount[i]); // Get the count of state s

        for (int j = 0; j < transitions[i].length; j++) {
            transitions[i][j] = ((double) transitionsCount[i][j] + 1) / (sum + numberOfStates); // Smooth probability
        }
    }
}
{% endhighlight %}


### Implementing the Viterbi

{% highlight java %}
private void testHMM(String filePath) {
    PrintWriter pw = null;
    Scanner fileReader = null;

    try {
        pw = new PrintWriter(new File("HMM_OUTPUT.TXT"));
        fileReader = new Scanner(new File(filePath));

        char previousState = '_';
        double[] prevProb = new double[numberOfStates]; // Keeps track of previous state probability
        while (fileReader.hasNext()) {
            String currentLine = fileReader.nextLine().trim();
            String[] lineComponents = currentLine.split("\\s+");

            char output = lineComponents[1].charAt(0);

            // End of word
            if (output == '_') {
                previousState = '_';
                pw.println(output);
                continue;
            }

            // Start of word
            if (previousState == '_') {
                // Start = P_start(state) * P_obs(observation)
                for (int i = 0; i < prevProb.length; i++) {
                    prevProb[i] = Math.log(priorProbability[i]) + Math.log(emissions[i][getCharValue(output)]);
                }
            } else {
                /* These two nested for loops could probably be reduced into one loop */

                // S(i) = P(oldState) * P_trans(old -> new) * P(observation|new)
                double[][] tmp = new double[numberOfStates][numberOfStates];
                for (int i = 0; i < numberOfStates; i++) {
                    for (int j = 0; j < numberOfStates; j++) {
                        tmp[i][j] = prevProb[i] + Math.log(transitions[i][j]) + Math.log(emissions[j][getCharValue(output)]);
                    }
                }

                // Take most likely path of each state
                for (int i = 0; i < numberOfStates; i++) {
                    int max = 0;
                    for (int j = 1; j < numberOfStates; j++) {
                        if (tmp[max][i] < tmp[j][i]) {
                            max = j;
                            prevProb[i] = tmp[max][i];
                        }
                    }
                }
            }

            // Take most likely state path
            int max = 0;
            for (int i = 1; i < prevProb.length; i++) {
                if (prevProb[i] > prevProb[max]) {
                    max = i;
                }
            }

            pw.println(getCharFromInt(max));
            previousState = output;
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } finally {
        if (pw != null) {
            pw.close();
        }
        if (fileReader != null) {
            fileReader.close();
        }
    }
}
{% endhighlight %}

## Conclusion

The implementation provided in the samples reduces the spelling errors of a given text sample from ~20% to about ~10%. I haven't tested variations and optimizations that can be made to the probabilistic model, but if y'all find anything interesting while toying around with it please send me an email!

The full code for the HMM is available on my [GitHub](https://github.com/hesham8/daily-challenge/tree/master/HiddenMarkovModel) page.
