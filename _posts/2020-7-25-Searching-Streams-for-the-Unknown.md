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

### Basic Streaming Algorithm Concepts

So one might ask, what is a streaming algorithm? To start, let us consider what a stream might be. A stream could be viewed as a sequence of data that we start and then run through in the order of the sequence. In some contexts, one can only pass through a stream once while in other contexts one pass through the stream as many times as needed. A streaming algorithm generally corresponds to an algorithm that operates on a stream to compute or estimate some quantity. Often, streaming algorithms operate on the assumption that the stream has so much data within it that algorithms operating on the stream should use a logarithmic or constant amount of data relative to the stream size $n$. If we consider, as an example, a $1$ TB sized file of regression data that we want to process to estimate some neural network parameters, it becomes clear having an algorithm that uses much less than $1$ TB of space to run would be very useful to tackling such a problem. Thus, streaming algorithms can be extremely useful.

### Catching the most common number in the stream

To solidify the idea of a streaming algorithm, we are going to dive into tackling the following problem:

> Suppose we are given a stream $S = (s_1, s_2, \cdots, s_n)$ of size $n$ where each variable $s_i$ corresponds to real number. We are then told that there exists a unique number that appears at least $X$ times in the stream. Describe an algorithm that can find this unique number and describe the number of passes this algorithm requires through the stream of data, given you are handed the value $X$.

This problem came up recently in a discussion with a colleague and I felt it was interesting enough to try and find my own solution for. The rest of this post will go over some algorithms that solve this problem, the first being a pretty obvious deterministic algorithm and the second being a fun one making use of randomness.

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
                            <div class="return">$r$</div>
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

It is pretty clear that this algorithm has us trying every type of number in the stream and seeing if its frequency is at least $X$, implying we will eventually find our desired number since it exists. Since for each value of $i$ we have to pass through the stream $2$ times, once to get the $i^{th}$ number as a representative and once to compute the frequency of that number, this implies in the worst case we will loop over the stream at most $2n$ times. Further, we can see that the only extra space we use other than the inputs is a few counters and a temporary number representing the representative. This implies our space complexity is at most $O(\log(n))$ bits. So this algorithm is simple, correct, and has a small space footprint. 

One question is, how might this algorithm do on some "average" input? Suppose we relax the problem slightly and for some fixed value $X$, the algorithm tries to find a unique number that shows up _exactly_ $X$ times, with all other numbers showing up with a lower frequency. For some sequence $S$ like this, we can consider all random permutations of this sequence as inputs to this algorithm and ask, what is the average runtime of this algorithm given such a distribution for the inputs? Let us fix some arbitrary sequence $S$ such that exactly one number appears $X$ times and all other numbers appear less than this. Let us then define $I_{X}$ as the random variable representing the number of loop iterations our algorithm performs for a random permutation of $S$. It is clear that our algorithm completes in exactly $i$ iterations if the first $(i-1)$ numbers in the sequence are not our desired number but the $i^{th}$ number is. This probability can be expressed by

$$\text{Pr}\left\lbrace I_{X} = i\right\rbrace = \frac{X}{n-i+1} \prod_{j=0}^{i-2} \left(1 - \frac{X}{n - j}\right)$$

This probability can be upper bounded by using a well known inequality that states for any $x \in \mathbb{R}$ that $1 - x \leq \exp(-x)$. This gives us that

$$\text{Pr}\left\lbrace I_{X} = i\right\rbrace \leq \frac{X}{n-i+1} \exp\left(-X \sum_{j=0}^{i-2} \frac{1}{n-j}\right)$$

From here, we can find a lower bound for $\sum_{j=0}^{i-2} \frac{1}{n-j}$ by using the following sequence of operations

$$\begin{align}
\sum_{j=0}^{i-2} \frac{1}{n-j} &= \sum_{j=n-i+2}^{n} \frac{1}{j} \\
&\geq \int_{n-i+2}^{n+1} \frac{dx}{x} \tag{$1/x$ decreases}\\
&= \log\left(\frac{n+1}{n-i+2}\right)
\end{align}$$

Applying this lower bound and the $1 - x \leq \exp(-x)$ inequality again gives us that our probability is at most

$$\text{Pr}\left\lbrace I_{X} = i\right\rbrace \leq \frac{X}{n-i+1} \left(\frac{n-i+2}{n+1}\right)^X \leq \frac{X}{n-i+1} \exp\left(-X\frac{(i-1)}{n+1}\right)$$

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
    <div class="return">$r$</div>
</div>

Now before going into the analysis of the above algorithm, how can we select a representative uniformly at random from the stream? Well one approach is to start reading from the stream and assign the $i^{th}$ number in the stream a random weight uniformly at random from the interval $[0,1]$, over time keeping which ever number ends up with the largest randomly assigned value. This works because the probability the $i^{th}$ number in the stream gets the maximum value is equal to $1/n$, where $n$ is again the size of the stream. The algorithm details for this random representative sampling is shown below. 

<div class="algorithm">
    <div class="algorithm_name" text="Uniform Random Sample"></div>
    <div class="algo_input">$(S)$</div>
    <div class="algo_output">$(r)$</div>
    <p>$r := 1$ <span class="my_comment">representative variable</span></p>
    <p>$u := 0$ <span class="my_comment">minimum weight</span></p>
    <div class="foreach_loop"> $s_i$ in $S = (s_1, s_2, \cdots, s_n)$
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
    <div class="return">$r$ <span class="my_comment">return random representative</span></div>
</div>

Now let us consider the correctness for the randomized number search algorithm. This algorithm chooses a representative at random, over and over, until the representative corresponds to the number we are looking for, so the correctness of the algorithm is clear. Also, it is clear the space used is at most $O(\log(n))$ bits since this algorithm uses similar counter and number variables used in the naive algorithm. 

The tough question then for this algorithm is, what is the runtime? This is less trivial because it is possible the algorithm never sets the representative to be the desired number, making the algorithm run forever. For our purposes, we will model the runtime in terms of how many times we pass through the stream. On each iteration of the loop, just as in the naive algorithm, we will have $2$ passes through the stream. So to work out the runtime of the algorithm, we must answer how many iterations of the loop to expect. This is where the real fun begins!

#### Some Analysis

###### Expected Runtime

Recall that the assumption built into the problem is that there exists exactly one number that will appear in the stream at least $X$ times. This implies that when we randomly choose a representative, the probability we do choose the desired number is at least $X/n$. With this fact, we have the following result

<div class="theorem" >
    <div class="theorem_name" text="Average Runtime"></div>

    Suppose we are given an arbitrary input instance $(S,X)$ to the randomized number search algorithm, where $S$ is a size $n$ stream. The expected number of times we will loop over the stream $S$ is at most

    $$\left(\frac{2}{X}\right) n$$
</div>
<div class="proof">
	As noted before, the probability we randomly choose a representative that is the desired number is at least $X/n$. Suppose the exact probability of successfully choosing the number of the representative is $p$, then we can model the number of iterations $I$ of our main loop as a geometric random variable with probability of success equal to $p$, denoted $G(p)$. Since $I \sim G(p)$, the expected value of $I$ is equal to $1/p$ and can be upper bounded by

	$$\mathbb{E}\left(I\right) = \frac{1}{p} \leq \frac{n}{X}$$

	At each iteration, we loop over the stream $S$ exactly $2$ times, implying that the expected number of times we will loop over the stream's contents is at most

	$$\left(\frac{2}{X}\right) n$$
</div>

---

When we consider the above, it is clear that the average performance of this algorithm improves upon the naive algorithm when $X > 1$ when we measure their performance in terms of the number of times we loop over the stream $S$. This observation implies that for most instances, choosing the randomized algorithm should have, in expectation, a better average runtime than the naive algorithm. Of course, the expected runtime does not take into account any variation we might find in runtimes due to the randomness of the algorithm. To get a more clear picture of the runtime for this algorithm, we can instead try to use a high probability variant to the analysis.

###### High Probability Runtime

Again recalling our desired number is in the stream at least $X$ times, this implies that at some iteration the representative chosen randomly is _not_ the desired number with probability at most $(1 - X/n)$. Now suppose our algorithm does $k$ iterations of the main loop. Let us call $E_{k}$ the event that none of these $k$ iterations find the desired number using the randomly sampled representative. Since each iteration is independent of all others, this implies that this probability is at most

$$ \begin{align}
\text{Pr}\left\lbrace E_k \right\rbrace &\leq \left(1 - X/n\right)^k 
\end{align}$$

Now the above probability corresponds to a _worst-case_ probability that our algorithm does not finish in $k$ iterations. This fact leads us to the following result

<div class="theorem" >
    <div class="theorem_name" text="High Probability Runtime"></div>

    Suppose we are given an arbitrary input instance $(S,X)$ to the randomized number search algorithm, where $S$ is a size $n$ stream. For any real number $1 > \delta > 0$, we have with probability at least $1 - \delta$ that our algorithm terminates after the number of times we loop over the stream $S$ is

    $$\left(\frac{2\log(1/\delta)}{X}\right) n$$
</div>
<div class="proof">
	So to prove this theorem, we note we are given a maximum allowable probability $\delta$ that our algorithm fails to finish within $k$ iterations. Let us recall from before that the probability our algorithm does not finish within $k$ iterations is at most $(1 - X/n)^k$ and further note the well known identity stating that $1 - x \leq \exp(-x)$ for all $x \in \mathbb{R}$. With these together, choosing $k$ to satisfy the inequality

	$$\exp\left(-\frac{Xk}{n}\right) \leq \delta$$

	ensures that $(1 - X/n)^k \leq \delta$ since $(1 - X/n)^k \leq \exp\left(-\frac{Xk}{n}\right)$. If we invert for $k$, we can find that $k$ must satisfy the inequality 

	$$ k \geq \left(\frac{\log(1/\delta)}{X}\right) n $$

	which implies we can just take the minimum value of this and know that with probability at least $1 - \delta$, our algorithm finishes within $k = \left(\frac{\log(1/\delta)}{X}\right) n$ iterations. Since each iteration loops over the stream exactly $2$ times, this implies the result.
</div>

---

Now as we decrease $\delta$ towards $0$, our confidence of at least $1 - \delta$ is brought closer and closer to a probability of $1$ that our bound on the runtime holds true. Intuitively, these smaller values for $\delta$ that correspond to higher confidence should force the upper bound on the number of times we loop over the stream grow because if I wanted to be more confident in an upper bound, I would make it larger to be more conservative. Thus, the analysis shows a $\log(1/\delta)$ penalty in the upper bound to ensure our confidence of at least $1 - \delta$. Pretty interesting. Another notable observation is that as $X$ grows, the number of iterations we will do quickly shrinks. This clearly makes sense since the larger $X$ is, the larger our chance is of randomly choosing our desired number as a representative.

Further, the analysis shows that if $\left(\log(1/\delta)/X\right) < 1$, then with probability at most $1 - \delta$ this algorithm has a runtime that is faster than the naive algorithm's worst-case runtime. This is an important observation because this implies there are potentially many instances where this algorithm can out perform the naive algorithm in a high probability sense. For example, if we choose $\delta = 10^{-10}$, then inputs where $X > 10 \log(10) > 23$ ensure with probability at least $1 - 10^{-10}$ that our runtime beats the worst-case runtime for the naive algorithm. Compared to the average runtime inequality that the runtime beats the naive algorithm when $X > 1$, it is certainly a bit more conservative but the guarantee is also much stronger since it looks at the tails of the distribution for the runtime.

### Experimental Comparisons

To ground the differences between both algorithm in reality, we will showcase some experimental results based on a C++ implementation. So to help us judge how useful this algorithm might be, let us see how the runtime is for different values of $X$ and $\delta$. In particular, recall that with each iteration of the algorithm we do $2$ passes of the stream. This implies with probability at least $1-\delta$ that the number of times we pass through the stream is


If we choose $X = 100$ and $\delta =$

<pre data-enlighter-language="cpp">
#include "iostream"
int main(int argc, char** argv){
	std::cout << "Hi there!" << std::endl;
	return 0;
}
</pre>
