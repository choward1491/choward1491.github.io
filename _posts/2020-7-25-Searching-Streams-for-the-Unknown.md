---
layout: post
title: Searching Streams for the Unknown
subtitle: An algorithmic approach
date:   2020-07-25 01:52:00 +0000
author: Christian
categories: algorithms math randomization probability experimental theory
---

In the modern climate of big data, it should not surprise anyone that we can be handed a dataset that is much too large to fit onto our personal computer's hard drive, let alone having the dataset all load into RAM. Yet, these limitations do not stop us from trying to crunch bigger datasets and harvest even more data! 

![alt text](/assets/streaming/big_data.jpg "I love when you call me, Big Data")

With such a data abundance, how do we construct efficient algorithms to handle the limitations of our hardware? Depending on the sort of things you wish to compute with the data, the techniques one might use can be extensive. While one common technique is to use distributed parallel computing to tackle large problems with lots of data at scale. An orthogonal direction to this that we will focus on, though, is something referred to streaming algorithms. 

### Streaming Algorithm Basics
So one might ask, what is a streaming algorithm? To start, let us consider what a stream might be. A stream is a sequence of data that we start and then run through in the order of the sequence. In some contexts, one can only pass through a stream once while in other contexts one pass through the stream as many times as needed. A streaming algorithm generally corresponds to an algorithm that operates on a stream to compute or estimate some quantity. Commonly, streaming algorithms operate on the assumption that the stream has so much data within it that algorithms operating on the stream should use a logarithmic or constant amount of data relative to the stream size n. If we consider, as an example, a 1 Terabyte file of regression data we want to process to estimate some neural network parameters, it becomes clear having an algorithm that uses much less than 1 terabyte of space to run would be very useful to tackling such a problem. Thus, streaming algorithms can be extremely useful.

### Catching the most common fish in the stream

To solidify the idea of a streaming algorithm, we are going to dive into tackling the following problem:

> Suppose we are given a stream $S = (s_1, s_2, \cdots, s_n)$ of size $n$ where each variable $s_i$ corresponds to real number. We are then told that there exists a unique number that appears at least $X$ times in the stream. Describe an algorithm that can find this unique number and describe the number of passes this algorithm requires through the stream of data, given you are handed the value $X$.

### Naive Algorithm
Before we get to the more interesting algorithm, let us consider what might be viewed as the more naive algorithm to handle this problem. The main idea is going to be deterministically looping over the stream, where at iteration $i$ we set the $i^{th}$ value $s_i$ as a representative and then loop over the stream $S$ and count the number of times this number appears. If this number appears at least $X$ times, we know we have found the desired number and we can terminate. Otherwise, we increment $i$ and continue our search. The explicit pseudocode for this algorithm is defined below.

<div class="algorithm">
    <div class="algorithm_name" text="Naive Number Search"></div>
    <div class="algo_input">$(S, X)$</div>
    <div class="algo_output">$(r)$</div>
    <p>$c := 0$ <span class="my_comment">counter</span></p>
    <p>$r := 1$ <span class="my_comment">representative variable</span></p>
    <div class="for_loop"> $i$ from $1$ to $n$
        <ul>
            <li>Reset stream $S$</li>
            <li>Read in stream $S$ until we reach element $s_i$</li>
            <li>Set $r \leftarrow s_i$</li>
            <li>Reset stream $S$</li>
            <li>Set $c \leftarrow 0$ </li>
            <li>Read in stream $S$ and set $c \leftarrow c + 1$ for each value that matches $r$</li>
            <li>
                <div class="if_cond"> $c \geq X$
                    <ul>
                        <li>
                            <div class="return">$(r)$</div>
                        </li>
                    </ul>
                </div>
                <div class="endif"></div>
            </li>
        </ul>
    </div>
    <!--<div class="return">$(\text{NaN})$ <span class="my_comment">if no number exists, return not a number</span></div>-->
</div>

#### Some Analysis

It is pretty clear that this algorithm has us trying every type of number in the stream and seeing if its frequency is at least $X$, implying we will eventually find our desired number if it exists. Since for each value of $i$ we have to pass through the stream $2$ times, once to get the $i^{th}$ number as a representative and once to compute the frequency of that number, this implies in the worst case we will pass through the stream at most $2n$ times. Further, we can see that the only extra space we use other than the inputs is a few counters and a temporary number representing the representative. This implies our space complexity is at most $O(\log(n))$. So this algorithm is simple, correct, and has a small space footprint. Can we do any better?

### A Randomized Algorithm
The previous algorithm is what we would consider deterministic, meaning that for the same input the algorithm will return the same answer in the same amount of runtime. In this case, we will consider a Las Vegas styled randomized algorithm, meaning for the same input the algorithm will return the same answer but its runtime can vary. This algorithm is going to be very similar to the previous one, but the twist leads to some interesting results.

At the start of this algorithm, on input of the stream $S$ and the value $X$, we will enter a loop that only ends if we find a representative $r$ from the stream that appears at least $X$ times in the stream. From there, we will pass through the stream once and uniformly at random choose a representative $r$ from the stream. From here, like the naive algorithm, we will reset the stream and loop through it and count how many times the representative $r$ appears. If the counter returns a value at least as large as $X$, then we know we are done. The key differences between this algorithm and the previous one is how we select the representative and how the loop terminates. In this case we could run forever while the naive algorithm has at most $n$ iterations of the main loop. The explicit algorithm details are listed below.

<div class="algorithm">
    <div class="algorithm_name" text="Randomized Number Search"></div>
    <div class="algo_input">$(S, X)$</div>
    <div class="algo_output">$(r)$</div>
    <p>$c := 0$ <span class="my_comment">counter</span></p>
    <p>$r := 1$ <span class="my_comment">representative variable</span></p>
    <div class="while_loop"> $c < X$
        <ul>
            <li>Reset stream $S$</li>
            <li>$r \leftarrow \text{UniformRandomSample}(S)$ <span class="my_comment">randomly choose representative</span></li>
            <li>Reset stream $S$</li>
            <li>Set $c \leftarrow 0$ </li>
            <li>Read in stream $S$ and set $c \leftarrow c + 1$ for each value that matches $r$</li>
        </ul>
    </div>
    <div class="return">$(r)$</div>
</div>

Now before going into the analysis of the above algorithm, how can we select a representative uniformly at random from the stream? Well one approach is to start reading from the stream and assign the $i^{th}$ number in the stream a random weight uniformly at random from the interval $[0,1]$, over time keeping which ever number ends up with the largest randomly assigned value. This works because the probability the $i^{th}$ number in the stream gets the maximum value is equal to $1/n$, where $n$ is again the size of the stream. The algorithm details for this random representative sampling is shown below. 

<div class="algorithm">
    <div class="algorithm_name" text="Uniform Random Sample"></div>
    <div class="algo_input">$(S)$</div>
    <div class="algo_output">$(r)$</div>
    <p>$r := 1$ <span class="my_comment">representative variable</span></p>
    <p>$u := 0$ <span class="my_comment">minimum weight</span></p>
    <div class="for_loop"> $s_i$ in $S = (s_1, s_2, \cdots, s_n)$
        <ul>
            <li>Randomly sample $w$ from interval $[0,1]$</li>
            <li>
                <div class="if_cond"> $w > u$
                    <ul>
                    	<li>$u \leftarrow w$</li>
                    	<li>$r \leftarrow s_i$</li>
                    </ul>
                </div>
                <div class="endif"></div>
            </li>
        </ul>
    </div>
    <div class="return">$(r)$ <span class="my_comment">return random representative</span></div>
</div>

Now let us consider the correctness for the randomized number search algorithm. This algorithm chooses a representative at random over and over until the representative corresponds to the number we are looking for, so the correctness of the algorithm is clear. Also, it is clear the space used is at most $O(\log(n))$ since this algorithm uses similar counter and number variables used in the naive algorithm. The tough question then for this algorithm is, what is the runtime? This is less trivial because it is possible the algorithm never ends up setting the representative to be the desired number, making the algorithm just run forever. For our purposes, we will model the runtime in terms of how many times we pass through the stream. On each iteration of the loop, just as in the naive algorithm, we will have $2$ passes through the stream. So to work out the runtime of the algorithm, we must answer how many iterations of the loop to expect. This is where the real fun begins!

#### Some Analysis

Recall that the assumption built into the problem is that there exists exactly one number that will appear in the stream at least $X$ times. This implies that when we randomly choose a representative, the probability we do not choose the desired number is at most $(1-X/n)$. Okay, so now what we would like to figure out is what the probability is of not choosing our desired number $k$ times in a row. Since each time we choose a random representative is independent of any other times, this implies that this probability is at most

$$ \text{Pr}\left\lbrace \right\rbrace \leq \left(1 - X/n\right)^k \leq \exp\left(-\frac{X k}{n}\right) $$

Now the above probability corresponds to a worst case probability that our algorithm does not finish in $k$ iterations. Suppose we want to choose $k$ such that the probability we do not finish in $k$ iterations is at most $\delta$. This implies we want to choose $k$ such that the following inequality holds

$$ \exp\left(-\frac{X k}{n}\right) \leq \delta $$

which can be satisfied by choosing $k$ equal the below expression:

$$ k = \left(\frac{\log(1/\delta)}{X}\right) n $$

Now notice that this choice of $k$ grows as we decrease $\delta$, which makes sense because smaller choices of $\delta$ give us stronger confidence that our algorithm will terminate in $k$ iterations. We can also see that the more times our desired number appears, which is reflected by the number $X$, the smaller we should expect our runtime to be. This clearly makes sense since the larger $X$ is, the larger our chance is of randomly choosing our desired number as a representative. 

So to help us judge how useful this algorithm might be, let us see how the runtime is for different values of $X$ and $\delta$. In particular, recall that with each iteration of the algorithm we do $2$ passes of the stream. This implies with probability at least $1-\delta$ that the number of times we pass through the stream is


If we choose $X = 100$ and $\delta =$

<pre data-enlighter-language="cpp">
#include "iostream"
int main(int argc, char** argv){
	std::cout << "Hi there!" << std::endl;
	return 0;
}
</pre>

<div class="theorem" >
    <div class="theorem_name" text="My Theorem"></div>

    Suppose we have some sequence $S = (s_1, s_2, \cdots, s_n)$ and we would like to find the average value $\mu_S$ of this sequence. This can be done using the expression

    $$\mu_S = \frac{1}{n} \sum_{i=1}^n s_i$$

    Further, we can see that this is clearly awesome.
</div>

<div class="lemma" >
    <div class="lemma_name" text="My Super Cool JL Lemma"></div>

    <p>
    Hi there person sfdosdi weoi so fso ose go aoirg oa oigoa ofg woe iog ao oag oa rogo ao ao o goe rgia oo rg o aoweig oi ao o rigowe jfo oag oa gorgj a oa gjow go oag oaiw egijefo iwoej goaeigj aoij goij gaio gaoeijf aoeiga riguhairuhgairugha oghao g oir ga.
    </p>

    <p>
    oiwegj owe gowej foiej owiej gowej gowei gjoweig jwoeijf ewofjewg owegjweog wegjew gowgjiew goi weogjweog oiwgj ewiog owejg weiog jwegj eowg jweogjewjogj weogjweg jw gowg weogjowegj weig jw gaog jwoig woejga iegja oweg jaog ajgaowiegjawo igj ogj ao.
    </p>
</div>

<div class="proof">
    Suppose that I have some interesting thing to say here. By assumption, it is interesting and correct. We are done.
</div>

<div class="defn" >
    <div class="defn_name" text="Vector Space"></div>

    <p>
    Hi there person sfdosdi weoi so fso ose go aoirg oa oigoa ofg woe iog ao oag oa rogo ao ao o goe rgia oo rg o aoweig oi ao o rigowe jfo oag oa gorgj a oa gjow go oag oaiw egijefo iwoej goaeigj aoij goij gaio gaoeijf aoeiga riguhairuhgairugha oghao g oir ga.
    </p>

    <p>
    oiwegj owe gowej foiej owiej gowej gowei gjoweig jwoeijf ewofjewg owegjweog wegjew gowgjiew goi weogjweog oiwgj ewiog owejg weiog jwegj eowg jweogjewjogj weogjweg jw gowg weogjowegj weig jw gaog jwoig woejga iegja oweg jaog ajgaowiegjawo igj ogj ao.
    </p>
</div>

<div class="algorithm">
    <div class="algorithm_name" text="Newton's Method"></div>
    <div class="for_loop"> $i$ from $1$ to $n$
        <ul>
            <li>Compute Jacobian of $f(x)$</li>
            <li>Compute Newton step</li>
            <li>$x \leftarrow x + \Delta x$</li>
            <li>
                <div class="if_cond"> $|x| < \epsilon$
                    <ul>
                        <li>
                            <div class="return">$x$</div>
                        </li>
                    </ul>
                </div>
                <div class="else_cond">
                    <ul><li>
                        <div class="continue"></div>
                    </li></ul>
                </div>
                <div class="endif"></div>
            </li>
        </ul>
    </div>
</div>