---
layout: page 
title: Implementing a positive semidefinite completion algorithm
shorttitle: Pos-def completion algorithm
permalink: /posdefcomp/
group: "navigation"
order: 4
---

### 1. Introduction
Consider a sparse symmetric matrix $$A \in \mathbb{R}^{n \times n}$$. The sparsity pattern of A can be represented by an undirected graph G(V,E). The set V represents the vertices of the graph that correspond to the row/column index of A. E represents the set of edges connecting the vertices. E is defined as $$ E \subseteq \{\{i,j\}\mid i,j \in \{\{1,2,\dots,n\}\}$$. For matrix A that means that all entries $$A_{ij} =A_{ji} = 0 \text{ if } \{i,j\} \not \in E$$.

{% highlight ruby %}
def foo
  puts 'foo'
end
{% endhighlight %}


Now consider that all entries of A that are part of the sparsity pattern are fixed and all other entries can be freely chosen. In a positive semidefinite completion problem one tries to find values for the free entries that make the whole matrix A positive semidefinite.

This problem arises for example if one wants to decompose the constraints of a Semidefinite Program (SDPs) into multiple smaller SDPs. After assembling the solution matrix out of several smaller solutions, there remain several unchosen entries of the matrix that have to be filled to make the whole solution matrix positive semidefinite.

In the following a suitable algorithm is presented and implementation details are explained together with a simple example. The algorithm and notation is taken from (Vandenberghe - Chordal Graphs and Semidefinite Optimization)

### 2. Chordal Graph Background
Let's quickly define some necessary terms and properties of graphs that are required for the algorithm. Again assume the undirected graph G(V,E) with vertex set V and edge set E. Two vertices are *neighbors* if there exists an edge between them. A path inside the graph is a sequence of connected vertices $$(v,v_1,v_2,\dots,v_k)$$. A *cycle* is a closed path $$(v,v_1,v_2,\dots,v)$$. A *chord* in a path is an edge between non-consecutive vertices. A graph is *chordal* if all cycles of four or more have a chord. Chordal graphs have some beneficial properties that are used later.

 A graph is called *complete* if there exists an edge between every pair of its vertices. A subgraph $$G(\hat{V},\hat{E})$$ is constructed by considering only some of the original graph's vertices $$\hat{V} \subseteq V$$, and corresponding edges $$\hat{E} \subseteq E$$. Any subgraph that is complete is called a *clique*. A clique is maximal if its vertices don't form a proper subset of another clique.

An ordering of G assigns an order to each vertex in V. A so-called *perfect elimination ordering* is achieved if for each vertex all neighbors with a higher order induce a complete subgraph. For chordal graphs one can always find a perfect elimination ordering.

### 3. Positive Semidefinite Completion Algorithm
- The algorithm procedure
- Let E be the sparsity pattern of matrix A. Then A is only completable if all submatrices indexed by the clique vertces are positive semidefinite:
$$ A \in \Pi_E(S^n_+) \text{  iff  } A_{\beta\beta} ⪰0 $$
- why does this work

### 4. Implementation Details / Example


$$A = \begin{bmatrix} 10 & 0 & 0 & 0 & 0 & -5.5 & 0 & 0\\
                      0  & 6 & 0 & -3.5 & 0 & 0 & 0 & 1.5\\
                      0  & 0 & 6 & 0 & 0 & 1.5 & 0 & 0\\
                      0 & -3.5 & 0 & 11.5 &0 &2.0 & -2.5 & -2.5\\
                      0 & 0 & 0 & 0 &4 & -4.5 & 0 & 0 \\
                      -5.5 & 0 & 1.5 & 2.0 & -4.5 & 9.5 & -2.5 & -1.5\\
                      0 & 0 & 0 & -2.5 & 0 & -2.5 & 8 & 0 \\
                      0 & 1.5 & 0 & -2.5 & 0 & -1.5 & 0 & 7.5  \end{bmatrix} \in \mathbf{R}^{8\times8}$$

![Graph representation of sparsity pattern](/../assets/images/psd_exp_graph.png){:width="300px"}
![Clique tree](/../assets/images/psd_exp_tree.png){:width="300px"}

### 5. Further Reading

### 6. Code
A sample implementation, written in Julia, can be found on <a href="https://github.com/miga89/PSD_Completion" target="_blank">Github</a>.

