---
layout: post
title: "Constructing A Hidden Markov Model"
excerpt: "A Spell Checker Utilizing AI Techniques"
tags: [AI, markov, spell, checker, hidden, java]
comments: true
<!-- image:
  feature: queen2.jpg
  credit: AlmostQueen
  creditlink: http://www.almostqueen.com/photos/ -->
---

##

Markov models are wonderful things. They're probabilistic models used to model dynamic states. There are four commonly used Markov models, and the type of model depends on whether the system is autonomous or controlled, and whether or not the system state is fully or partially observable.

Markov chains are autonomous systems wherein the system state is fully observable. All Markov processes are dependent on two properties: conditional independence (the same as is used in Bayes theorem), and the Markov property. The Markov property is as follows: the probability distribution of the next state depends only on the current state and *not* on the sequence of events that preceded it.

A hidden Markov model is a Markov chain wherein the state is only *partiallY* observable. Observations into the system are insufficient to determine the state of the system. There are several algorithms involving HMMs, but today I'll be exploring the Viterbi. The Viterbi algorithm is used to find the most likely sequence of states in a HMM. Common uses of the Viterbi are speech recognition, spelling correction, and handwriting recognition.

Today, I'll be demonstrating a HMM that corrects for spelling errors.
