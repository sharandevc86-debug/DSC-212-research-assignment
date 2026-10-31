# Modularity on the Karate Club Graph

**Course:** DSC212, Graph Theory Module (research assignment)<br>
**Author:** *Sharandev B*<br>
**ID:** *IMS25189*<br>
**Notebook:**[`DSC212_Zachary's_Karate_Club_Assignment.ipynb`](./DSC212_Zachary's_Karate_Club_Assignment.ipynb)

---

## 1. Goals

The goal of this assignment was to implement **recursive spectral modularity partitioning** from scratch (NumPy and NetworkX only) and to test it on Zachary's Karate Club network, the 34-member club that split into two after a dispute between its instructor (Mr. Hi, node 1) and its president (John A., node 34). Concretely, I set out to:

1. Build the modularity matrix of the graph,

$$
B_{ij} = A_{ij} - \frac{k_i k_j}{2m},
$$

   and use the leading eigenvector of $B$ to split the network into two communities.

2. Repeat the split recursively on each community until no split improves modularity.
3. Draw the graph after every split with a fixed layout and one colour per community.
4. Compute degree, betweenness, closeness and clustering for every node after each split and plot how they evolve.
5. Discuss which nodes stay central and how community structure shapes these metrics.

---

## 2. The problem I ran into

I first followed the handout exactly. For a community $C$ it defines the restricted modularity matrix as the plain block of $B$:

$$
B^{(C)}_{ij} = B_{ij}, \qquad i,j \in C,
$$

and says to stop when the leading eigenvalue $\lambda_1^{(C)} \le 0$.

The result looked reasonable at first, but the output had **five communities, four proper ones plus a singleton, node 12**. Node 12 has degree 1 and its only neighbour is node 1, so it should not form a community of its own. Looking closer:

- At the second split the plain block placed node 12 in the group $\{5,6,7,11,12,17\}$, away from node 1. Inside that group it has no ties at all.
- The plain block still gave $\lambda_1 \approx 2.15 > 0$, so the stopping rule never fired.
- At the fourth split node 12 was cut off alone. Modularity rose slightly (0.4081 to 0.4094), but only by detaching a degree-1 node from its only neighbour.

In my view the handout's simplified matrix is too diluted to support its own stopping rule, which is why I did more analysis on modularity itself.

### What I found

The all-ones vector is an eigenvector of the **whole-graph** matrix $B$ with eigenvalue 0 (Proposition 2 in the handout), because every row of $B$ sums to zero:

$$
B\mathbf{1} = A\mathbf{1} - \frac{\mathbf{k}\,(\mathbf{k}^\top \mathbf{1})}{2m} = \mathbf{k} - \mathbf{k} = \mathbf{0}.
$$

This is a property of the full matrix, not of an arbitrary subgroup. For the plain block $B^{(C)}$ the row sums are generally non-zero, so $\mathbf{1}$ need not be an eigenvector and $\lambda_1^{(C)} \le 0$ can fail to detect an indivisible group.

Following Newman (2006), I switched to the **generalised modularity matrix** for a subgroup:

$$
B^{(C)}_{ij} = B_{ij} - \delta_{ij} \sum_{l \in C} B_{il}, \qquad i,j \in C.
$$

This subtracts each row sum on the diagonal, which has three consequences:

1. $B^{(C)}\mathbf{1} = \mathbf{0}$ holds for every subgroup.
2. $\tfrac{1}{4m}\, s^\top B^{(C)} s$ equals the true change in modularity $\Delta Q$ when $C$ is split by the vector $s \in \{-1,+1\}^{|C|}$.
3. $0$ is always an eigenvalue, so $\lambda_1^{(C)} \ge 0$. When $\lambda_1^{(C)} = 0$, the leading eigenvector is $\mathbf{1}$, all its entries have the same sign, and the group is indivisible.

One further detail: $\lambda_1^{(C)} > 0$ only guarantees that *some real-valued* vector improves $Q$, not that rounding it to $\pm 1$ does. The 9-node group $\{2,3,4,8,13,14,18,20,22\}$ has $\lambda_1 = 0.060 > 0$, but its sign split has $\Delta Q = -0.016$. I therefore added a guard that rejects any split with $\Delta Q \le 0$.

---

## 3. Results

| Method | Communities | Final $Q$ |
|---|---|---|
| Handout's plain block | 5 (including singleton $\{12\}$) | 0.4094 |
| Newman's correction (used for the final results) | 4 | 0.3934 |

The plain-block version reaches a slightly higher $Q$, but only because it isolates a degree-1 node. I chose the corrected version because its stopping rule is valid and the communities are interpretable.

Final communities (node IDs 1 to 34):

| Community | Nodes |
|---|---|
| 1 | 1, 5, 6, 7, 11, 12, 17 |
| 2 | 2, 3, 4, 8, 13, 14, 18, 20, 22 |
| 3 | 9, 10, 15, 16, 19, 21, 23, 27, 30, 31, 33, 34 |
| 4 | 24, 25, 26, 28, 29, 32 |

- The first split ($Q = 0.372$) matches the real split of the club for 33 of 34 members. Only node 9 differs: he joined Mr. Hi's club, but three of his five ties go to John A.'s side.
- Nodes 1 and 34 stay the most central nodes in degree, betweenness and closeness across all splits. Node 3 loses most of its betweenness once the cross-community ties are cut.
- The full discussion is in Section 7 of the notebook.

**Note on the metrics.** Node metrics are computed inside each community subgraph, with iteration 0 being the full graph. Absolute values are therefore not comparable across iterations because they are normalised by community size. I compare ranks instead.

---

## 4. Repository contents

```
.
├── DSC212_Zachary's_Karate_Club_Assignment.ipynb   # full analysis: code, outputs, plots, discussion
├── LICENSE/                            # MIT LICENSE
└── README.md
```

## 5. How to run

```bash
pip install numpy networkx matplotlib pandas jupyter
jupyter notebook DSC212_Zachary's_Karate_Club_Assignment.ipynb
```

Run the notebook top to bottom ("Restart & Run All"). It needs no manual edits. The graph layout uses a fixed random seed, so the figures look the same on every run.

---

## 6. AI usage statement

I used an AI assistant (Claude, by Anthropic) as a tool for parts of this project where my knowledge was limited:

- **Python libraries.** I am not yet fluent in NumPy, NetworkX, pandas and matplotlib, so I used the assistant to learn which functions to call and to help write and debug code for the eigen-decomposition, the recursion, the plots and the tables.
- **LaTeX.** I used it to typeset the mathematics in this README and in the notebook's Markdown cells.

The assistant also drafted wording for parts of the notebook discussion.When I decided to follow the handout's method, I noticed the node 12 problem in my own results, and I wrote up my findings on Newman's correction myself. The assistant helped check that this reasoning was correct and pointed out where my explanation was imprecise, for example that zero row sums hold for the whole-graph matrix and not for an arbitrary subgroup. I am responsible for everything submitted here.

---

## 7. References

1. M. E. J. Newman (2006). Modularity and community structure in networks. *Proceedings of the National Academy of Sciences*, 103(23), 8577-8582.
2. W. W. Zachary (1977). An information flow model for conflict and fission in small groups. *Journal of Anthropological Research*, 33(4), 452-473.
3. DSC212 course handout, *Modularity on the Karate Club Graph*, Graph Theory Module.
