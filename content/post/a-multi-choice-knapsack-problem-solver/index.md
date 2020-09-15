---
title: A multi-choice knapsack problem solver
subtitle: Application to user scheduling over channels
date: 2020-09-13T15:20:37.176Z
draft: false
featured: true
authors:
  - Riade Benbaki
tags:
  - ""
categories:
  - Algorithmics
image:
  filename: featured
  focal_point: Smart
  preview_only: false
---
The Multi Choice Knapsack problem (referred to as MCKP) is a generalization of the ordinary knapsack problem, where the set of items in partitioned into different classes. The problem consists of finding exactly one item per class in order to maximize the total gain while respecting a total capacity constraint.

Multiple approaches are possible, both to the exact and relaxed problem. In the this article, we present a solver that solves instances of the MCKP, and apply it to a practical issue : User scheduling over communication channels. 

You can find the Github repo for the solver [here](https://github.com/riadeb/MultiChoiceKP)

{{% staticref "files/MCKP_for_user_scheduling.pdf" "newtab" %}}The complete description of the project can be found here{{% /staticref %}}


# Introduction

An antenna transmits data packets to smartphones (or users) through a
wireless medium, which is divided into a set of frequency channels.
Figure 1 is an example of simultaneous transmission towards three users
using three channels.

The higher the power dedicated to a user, the higher the data rate it
can experience. The exact dependence between power and data rate is
however user and channel specific. With the same transmit power, a user
close to the antenna will enjoy for example a higher data rate than a
user far away. A wireless packet scheduler is thus responsible to
allocated channels to users and to divide the total power budget of the
antenna among the available channels. The goal of this project is to
design optimal packet schedulers in this context.

# Problem formulation

We consider a system with a set $\mathcal{K}$ of $K$ users to be served
over a set $\mathcal{N}$ of $N$ channels. Every channel shall be used to
serve a single user and cannot be left unallocated for efficiency
reasons; a user can be served using several channels; however some users
may not be served. The scheduler chooses a transmit power
$p_{k,n}, k \in K, n \in N$ to serve user $k$ on channel $n$. If user
$k$ is not served on channel $n$, we have $p_{k,n} = 0$. When user $k$
is served over channel $n$ with power $p_{k,n}$, its data rate is
$r_{k,n} = u_{k,n}(p_{k,n})$, where the function $u_{k,n}$ is called the
rate utility function of user $k$ on channel $n$. This utility function
is assumed to be known by the scheduler. In practical systems, $u_{k,n}$
is a non-decreasing step function that takes a finite number of non-zero
values, say $M$ for all $k$ and $n$ and such that $u_{k,n}(0) = 0$ for
all $k$ and $n$. To fix notations, we thus define:
    $$\begin{array}{ll}
        u_{k,n}(p_{k,n}) =    & 0           & if & p_{k,n} < p_{k,1,n} \\\\
                        &  r_{k,1,n}       & if & p_{k,1,n} \leq p_{k,n} < p*{k,2,n} \\\\
                        & \dots & \dots \\\\
                        &  r_{k,M,n}        & if & p_{k,M,n} < p_{k,n}\
    \end{array}$$

The task of the scheduler is to allocate channels to users and power levels to  so as to maximise the total data rate
of the system under the constraint of a total power budget $p$
and the constraint of having exactly one user served per channel. For
simplicity, we assume that all coefficients $r_{k,m,n}, p_{k,m,n}$ and
$p$ are non-negative integers.\
To formulate the problem as an integer linear program (ILP), we aim to
maximise $\sum_{k,m,n}x_{k,m,n}r_{k,m,n}$ subject to

1. $\sum_{k,m,n}x_{k,m,n}p_{k,m,n} \leq p$
2. $\sum_{k,m}x_{k,m,n} = 1$
3. $x_{k,m,n} \in \mathbb{N}$

for all $0 \leq k  \leq K, 0 \leq m \leq M$ and $0 \leq n \leq N$ .\
\
This is a multiple-choice knapsack problem (MCKP) which is a
generalization of the ordinary knapsack problem, where the set of items
is partitioned into classes, corresponding to the channels. We need to select exactly one item of each class of
items\[@knapsack]. This ILP is known to be NP hard. By relaxing the
integrality constraint on $x_{k,m,n}$, we obtain a linear program (LP),
which provides an upper bound for our problem. If the solution to the LP
is integer, then we have a solution for the ILP. In the following
sections, we first make some preprocessing to reduce problem instance
size, then solve the LP and the IP.

## Solving the LP-relaxed problem

The LP-relaxed problem (removing the integrality constraint) can be solved with an off the shelf LP solver, or using a greedy approach.


We tested both approaches on the 2 largest files (test4.txt and test5.txt in testfiles folder) and compared their performance in term of CPU runtime (in ms).
          

| Testfile                               | test4.txt      |test5.txt    |
| -------------------------------------- | -------------- | ----------- |
| `Preprocessing`                        |168.98987497    |0.62586537   |
| `Greedy (on preprocessed instance) `   |2.08267759      |0.03906349   |
| `LPsolver (on preprocessed instance) ` |386.13193925    |5.14502407   |
| ` LPsolver (without preprocessing) `   |35256           |29           |

You can compare runtime of the different approaches on another file using the command :
``
javac Main -r {filepath}
``


## Algorithms for solving the ILP

As the relaxed solution cannot be implemented in practice (this would
require the possibility to schedule two users on the same channel), we
tackle in this section the ILP problem (when the power budget is an
integer). We are looking here for an
optimal solution by first relying on Dynamic Programming (DP).

## Dynamic Programming Solution

The first DP solution is based on a function $R(n,p)$ that returns the maximum rate achievable with channels from 1 to n, and with a power budget of p. This function verifies the following :\
$$R(n,p) = \max_{\substack{pair \in channel_n \ pair.p \leq p }} R(n-1,p-pair.p) + pair.r$$\


Time complexity : it takes at most $O(KM)$ to find the maximum of the
equation (1), so, regarding the two main loops of our algorithm, we have
a time complexity of $O(PNKM)$.

Space complexity : $O(P)$ as we use two arrays of length $P$.

## A second dynamic programming approach

An alternative DP approach is to consider the problem of finding the
minimal power budget necessary to achieve a data rate $r$. The equation of finding
minimal power allocations is :\
$$P(n,r) = \min_{\substack{pair \in channel_n \ pair.r \leq r }} P(n-1,r-pair.r) + pair.p$$

To use this approach, we need to know an upper bound of the maximum achievable rate, $U$. This upper-bound can be found by either taking the maximum rate in each channel, or by solving the relaxed LP-problem first.

Time complexity : O(NUKM)

 Space complexity : O(U)

## Branch And Bound Solution

Another classical way of solving IPs is Branch-and-Bound (BB).
The principle of BB is as follows. We construct a tree of sub-problems,
whose root corresponds to the initial problem. Each vertex v corresponds
to a sub-problem, which is generated from its parent in the tree by
adding an additional constraint. At node $v$ (a branch of the tree), the
relaxed sub-problem is solved in order to get an upper bound $\bar{z}_v$
of the optimal value for the sub-problem. This branch is not further
explored if (1) $\bar{z}_v$ is less than a current feasible solution we
have already or (2) $\bar{z}_v$ is associated to an integer solution, in
which case we can update the current feasible solution or (3) the
relaxed sub-problem is infeasible.

Each level of the tree represents a channel, and each node of each level
represents a feasible pair choice in that channel.So each path from the
source to a node in level k represents a possible configuration where
channels from 1 to k are assigned to a pair. At each node, we use the
relaxed problem to find an upper bound for the problem, using the greedy
algorithm. The greedy algorithm also gives us a feasible solution, so we
update the lower bound each time this feasible solution is better than
the one we already have. The worst case complexity is $O(KM*(KM)^N)$
because it's possible to go through all the nodes of the tree, but in
most cases, several branches will be eliminated, thus giving a lower
amortised complexity. The fact that we use a stack or a queue in the
algorithm changes the traversal order of the tree : DFS using a stack,
BFS using a queue. In this case, the later runtime results show that
using a DFS is faster.


## Results of IP algorithms

| Testfile          |1     |2    |3    |4       |5    |
| ----------------- | ---- | --- | --- | ------ | --- |
|`DP 1`             |1.493 |NA   |0.506|5722.799|9.421|
|`DP 2 `            |0.038 |NA   |0.042|4539.203|7.371|
|`BB (doing a DFS) `|0.004 |NA   |0.012|TLE     |0.078|
|`BB (doing a BFS) `|0.215 |NA   |0.271|TLE     |0.442|


