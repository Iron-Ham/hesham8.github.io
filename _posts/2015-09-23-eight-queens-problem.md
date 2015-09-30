---
layout: post
title: "The Eight Queens Problem"
excerpt: "An introduction to the eight queens problem, and a recursive solution"
tags: [eight_queens, code_snippet]
comments: true
image:
  feature: queen2.jpg
  credit: AlmostQueen
  creditlink: http://www.almostqueen.com/photos/
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

> Part One of Four

This is the first part in a four part series on the eight queens problem. This first part will focus on introducing the Eight Queens problem, as well as recursively generating a set of all possible solutions.

## The 8 Queens Problem

The eight queens puzzle involves arranging eight queens on one `8x8` chessboard such that no queens are able to attack each other. This problem is one specific instance of the **n-queens** problem, where `n` queens occupy an `nxn` grid without attacking each other.

Computing each solution can be expensive for the 8-queens problem as there are roughly **4.5 billion** possible arrangements of eight queens on an 8x8 board, and only *92* solutions.

There are several techniques to do this. In the first three parts of this series, I will discuss a recursive solution, a genetic algorithm solution, and a simulated annealing solution. In the final part, I will compare the three approaches.

## The Recursive Solution

Solving this recursively is pretty straightforward once the logic is reasoned out. Let's represent our chessboard as an array. In order to do that, let's define the size of our grid (and the number of our queens) as a constant.

{% highlight java %}
private final int GRID_SIZE = 8;
{% endhighlight %}

Each value of the array can be treated as the measure the row, and each position in the array as a measure of its column. Let's examine the recursive logic to be implemented:

{% gist hesham8/68f0a31cd18c0546721d %}

Following this logic, this is actually a fairly straightforward solution to implement.

{% highlight java %}
//Does not include mirror image solutions. (i.e., 42061753 is not distinct from 35716024)
Integer[] placeQueens(int row, Integer[] columns, List<Integer[]> results) {
  Integer[] returnValue;
    if (row == GRID_SIZE) {
        return columns;
    } else {
        for (int col = 0; col < GRID_SIZE; col++) {
            if (checkValid(columns, row, col)) {
                columns[row] = col;
                returnValue = placeQueens(row + 1, columns, results);
                if (returnValue != null)
                    return returnValue;
            }
        }
    }
    return null;
}
private boolean checkValid(Integer[] columns, int row1, int column1) {
    for (int row2 = 0; row2 < row1; row2++) {
        int column2 = columns[row2];
        if (column1 == column2)
            return false;

        int columnDistance = Math.abs(column2 - column1);
        int rowDistance = row1 - row2;
        if (columnDistance == rowDistance)
            return false;
    }
    return true;
}
{% endhighlight %}

Currently, the `placeQueens()` method is written such that it can only written a single valid solution. Modifying it such that it returns *all* valid solutions is fairly trivial:

{% highlight java %}
//Does not include mirror image solutions. (i.e., 42061753 is not distinct from 35716024)
void placeQueens(int row, Integer[] columns, List<Integer[]> results, boolean withDuplicates) {
    if (row == GRID_SIZE) {
        results.add(columns.clone());
        if (withDuplicates) {
          Integer[] reverseSolution = new Integer[GRID_SIZE];
          for (int i = 0; i < GRID_SIZE; i++) {
              reverseSolution[i] = columns[columns.length - i - 1];
          }
          results.add(reverseSolution);
        }
    } else {
        for (int col = 0; col < GRID_SIZE; col++) {
            if (checkValid(columns, row, col)) {
                columns[row] = col;
                placeQueens(row + 1, columns, results);
            }
        }
    }
}
{% endhighlight %}

Note that in order to list all valid unique solutions, the mirror image of each solution generated this way must also be included.
