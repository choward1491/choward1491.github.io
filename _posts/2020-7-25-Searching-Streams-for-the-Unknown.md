---
layout: post
title: Searching Streams for the Unknown
mathjax: true
date:   2020-07-25 01:52:00 +0000
categories: algorithms math randomization probability experimental theory
---

## An algorithmic approach

In the modern climate of big data, it should not surprise anyone that we can be handed a dataset that is much too large to fit onto our personal computer's hard drive, let alone having the dataset all load into RAM. Yet, these limitations do not stop us from trying to crunch bigger datasets and harvest even more data! 
Generated using imgflip.comWith such a data abundance, how do we construct efficient algorithms to handle the limitations of our hardware? Depending on the sort of things you wish to compute with the data, the techniques one might use can be extensive. While one common technique is to use distributed parallel computing to tackle large problems with lots of data at scale. An orthogonal direction to this that we will focus on, though, is something referred to streaming algorithms. 

### Streaming Algorithm Basics
So one might ask, what is a streaming algorithm? To start, let us consider what a stream might be. A stream is a sequence of data that we start and then run through in the order of the sequence. In some contexts, one can only pass through a stream once while in other contexts one pass through the stream as many times as needed. A streaming algorithm generally corresponds to an algorithm that operates on a stream to compute or estimate some quantity. Commonly, streaming algorithms operate on the assumption that the stream has so much data within it that algorithms operating on the stream should use a logarithmic or constant amount of data relative to the stream size n. If we consider, as an example, a 1 Terabyte file of regression data we want to process to estimate some neural network parameters, it becomes clear having an algorithm that uses much less than 1 terabyte of space to run would be very useful to tackling such a problem. Thus, streaming algorithms can be extremely useful.


### Catching the most common fish in the stream

To solidify the idea of a streaming algorithm, we are going to dive into tackling the following problem:

Suppose we are given a stream $S = (x_1, x_2, \cdots, x_n)$ of size $n$ where each variable $x_i$ corresponds to an integer. We are then told that there exists a unique number that is the most common among all numbers that appear in the stream and appears at least $X$ times.

Describe an algorithm that can find this unique number and describe the number of passes this algorithm requires through the stream of data, given you are handed the value $X$.

### Naive Algorithm
Before we get to the more interesting algorithm, let us consider what might be viewed as the more naive algorithm to handle this problem. At the start of the algorithm, on input of the stream $S$ and the value $X$, we first initialize some counter $i$ to $i = 1$. From there, for any value of $i$, we first read the stream $S$ until we hit the ith number in the stream and set this value as our representative $r$. We then reset the stream and, starting from the beginning, count how many times we see our representative $r$ in the stream. If we reach the end of the stream and we have seen $r$ at least $X$ times, then we know our representative $r$ corresponds to the unique number we are looking for and so we return that and we are done. If we have instead seen $r$ less than $X$ times, then we know we have not found the desired number yet, and so we set $i \leftarrow i + 1$ and restart finding a new representative.

It is pretty clear that this algorithm has us trying every type of number in the stream and seeing if its frequency is at least $X$, implying we will eventually find our desired number. Since for each value of $i$ we have to pass through the stream $2$ times, once to get the $i^{th}$ number and once to compute the frequency of that number, this implies in the worst case we will pass through the stream at most $2n$ times. Seems like a simple enough algorithm! So can we do any better?

### A Randomized Algorithm
The previous algorithm is what we would consider deterministic, meaning that for the same input the algorithm will return the same answer in the same amount of runtime. In this case, we will consider a Las Vegas styled randomized algorithm, meaning for the same input the algorithm will return the same answer but its runtime can vary. This algorithm is going to be very similar to the previous one, but the twist leads to some interesting results.

At the start of this algorithm, on input of the stream $S$ and the value $X$, we will enter a loop that only ends if we find a representative $r$ from the stream that appears at least $X$ times in the stream. From there, we will pass through the stream once and uniformly at random choose a representative $r$ from the stream. Note that this random selection of the representative is really the defining difference between this algorithm and the previous one.

Now to achieve selecting a representative at random by streaming, we start reading from the stream and assign the ith number a random value uniformly at random from the interval $[0,1]$, over time keeping which ever number ends up with the largest randomly assigned value. This works because the probability the $i^{th}$ number in the stream gets the maximum value is equal to $1/n$, where $n$ is again the size of the stream. From here we then reset the stream and, starting from the beginning, count how many times we see our representative $r$ in the stream. If we reach the end of the stream and we have seen $r$ at least $X$ times, then we know our representative $r$ corresponds to the unique number we are looking for and so we return that and we are done. If we have instead seen $r$ less than $X$ times, then we know we have not found the desired number yet and we go back to the start of the loop so we can choose a random representative and repeat the process.

This algorithm effectively chooses a representative at random over and over until the representative corresponds to the number we are looking for, so the correctness of the algorithm is clear. However, what is the runtime of this algorithm? Well each iteration of the loop will, just as the deterministic algorithm, require $2$ passes through the stream. The real question is, how many iterations of the loop should we expect before we find our desired number? This is where the real fun begins.

Recall that the assumption built into the problem is that there exists exactly one number that will appear in the stream at least $X$ times. This implies that when we randomly choose a representative, the probability we do not choose the desired number is at most $(1-X/n)$. Okay, so now what we would like to figure out is what the probability is of not choosing our desired number $k$ times in a row. Since each time we choose a random representative is independent of any other times, this implies that this probability is at most

$$\text{Pr}\left\lbrace \right\rbrace \leq \left(1 - X/n\right)^k \leq \exp\left(-\frac{X k}{n}\right)$$

Now the above probability corresponds to a worst case probability that our algorithm does not finish in $k$ iterations. Suppose we want to choose $k$ such that the probability we do not finish in $k$ iterations is at most $\delta$. This implies we want to choose $k$ such that the following inequality holds

$$ \exp\left(-\frac{X k}{n}\right) \leq \delta $$

which can be satisfied by choosing $k$ equal the below expression:

$$ k = \left(\frac{\log(1/\delta)}{X}\right) n $$


Now notice that this choice of $k$ grows as we decrease $\delta$, which makes sense because smaller choices of $\delta$ give us stronger confidence that our algorithm will terminate in $k$ iterations. We can also see that the more times our desired number appears, which is reflected by the number $X$, the smaller we should expect our runtime to be. This clearly makes sense since the larger $X$ is, the larger our chance is of randomly choosing our desired number as a representative. 

So to help us judge how useful this algorithm might be, let us see how the runtime is for different values of $X$ and $\delta$. In particular, recall that with each iteration of the algorithm we do $2$ passes of the stream. This implies with probability at least $1-\delta$ that the number of times we pass through the stream is


If we choose $X = 100$ and $\delta =$