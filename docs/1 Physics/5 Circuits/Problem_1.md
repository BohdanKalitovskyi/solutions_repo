# Problem 1
# I. Problem Understanding and Modeling

## 1. Circuit Representation

**Definition 1.1 (Weighted Undirected Graph).**  
A circuit is modeled by a weighted undirected graph  
$$
G = (V, E, w),
$$  
where  
- $V = \{v_1, v_2, \dots, v_n\}$ is the set of **nodes** (junctions);  
- $E \subseteq \{\{v_i, v_j\} : v_i \neq v_j\}$ is the set of **edges** (resistors);  
- $w: E \to \mathbb{R}^+$ assigns each edge $\{v_i,v_j\}$ a resistance \(r_{ij} = w(\{v_i,v_j\})\).

**Adjacency Matrix $A$.**  
$$
A_{ij} =
\begin{cases}
r_{ij}, & \{v_i, v_j\} \in E,\\
0, & \text{otherwise},
\end{cases}
\quad
A \in \mathbb{R}^{n \times n},\ 
A_{ij}=A_{ji},\ 
A_{ii}=0.
$$

**Degree Matrix $D$ and Laplacian $L$.**  
Let
$$
D = \mathrm{diag}(d_1,\dots,d_n), 
\quad
d_i = \sum_{j=1}^n A_{ij}.
$$  
Then the **graph Laplacian** is  
$$
L = D - A.
$$  
Connectivity is equivalent to the algebraic connectivity satisfying  
$$
\lambda_2(L) > 0.
$$

**Incidence Matrix $B$.**  
For \(m=|E|\), define \(B\in\{-1,0,1\}^{n\times m}\) by arbitrarily orienting each edge \(e_k=\{v_i,v_j\}\):
$$
B_{ik} =
\begin{cases}
1, & \text{if } e_k \text{ is oriented } v_i\to v_j,\\
-1, & \text{if } e_k \text{ is oriented } v_j\to v_i,\\
0, & \text{otherwise}.
\end{cases}
$$

## 2. Input Specification

We adopt one of the following formats:

1. **Adjacency List.**  
   A function \(L: V \to 2^{V\times\mathbb{R}^+}\) where  
   $$
   L(v_i) = \{(v_j, r_{ij}) : \{v_i,v_j\} \in E\}.
   $$  
   _Example:_  
   $$
   L(v_1) = \{(v_2, R_{12}),\,(v_3, R_{13})\}.
   $$

2. **Adjacency Matrix.**  
   Directly input the symmetric matrix \(A\) as above:  
   $$
   A =
   \begin{pmatrix}
   0 & R_{12} & \cdots & R_{1n}\\
   R_{21} & 0 & \cdots & R_{2n}\\
   \vdots & \vdots & \ddots & \vdots\\
   R_{n1} & R_{n2} & \cdots & 0
   \end{pmatrix}.
   $$

3. **Edge List.**  
   A sequence of triples  
   $$
   E = \{(v_{u_k}, v_{v_k}, r_k)\}_{k=1}^m,
   $$  
   representing each resistor by its endpoints and resistance.

Each format ensures explicit storage of every resistor’s endpoints and resistance value.

## 3. Assumptions and Constraints

1. **Purely Resistive Network.**  
   All components obey Ohm’s law:  
   $$V = IR.$$

2. **Undirected Graph.**  
   Symmetry of resistance:  
   $$
   w(\{v_i,v_j\}) = w(\{v_j,v_i\}),\quad
   \forall\,\{v_i,v_j\}\in E.
   $$

3. **Connectivity Between Source and Sink.**  
   The graph is connected:  
   $$
   \forall\,u,v\in V,\;\exists\;\text{path }P_{u\to v}.
   $$  
   In particular, the designated source \(s\) and sink \(t\) must lie in the same connected component.

# II. Algorithm Design

## 1. Simplification Strategy

**Series Reduction.**  
Identify a path $P = (v_0, v_1, \dots, v_k)$ such that each internal node $v_i$ has degree $2$, and edges $e_i = \{v_{i-1},v_i\}$ with resistances $r_i$. Replace $P$ by a single edge $\{v_0,v_k\}$ with equivalent resistance  
$$
R_{\mathrm{series}}
= \sum_{i=1}^k r_i.
$$

**Parallel Reduction.**  
Identify all edges $\{e_i\}_{i=1}^p$ connecting the same node pair $\{u,v\}$ with resistances $r_i$. Replace by a single edge $\{u,v\}$ with  
$$
\frac{1}{R_{\mathrm{parallel}}}
= \sum_{i=1}^p \frac{1}{r_i}
\quad\Longrightarrow\quad
R_{\mathrm{parallel}}
= \Bigl(\sum_{i=1}^p \tfrac{1}{r_i}\Bigr)^{-1}.
$$

## 2. Graph Traversal and Pattern Detection

### 2.1 Depth-First Search (DFS)  
Use DFS to explore and flag nodes of degree 2 for series-chain detection.  
```pseudo
procedure DFS(u):
  visited[u] ← true
  for each neighbor v of u do
    if not visited[v] then
      DFS(v)
    end if
  end for
end procedure
```

### 2.2 Breadth-First Search (BFS)  
Use BFS for level-order scans to locate chains iteratively.  
```pseudo
procedure BFS(s):
  queue ← [s]
  visited[s] ← true
  while queue not empty do
    u ← dequeue(queue)
    for each neighbor v of u do
      if not visited[v] then
        visited[v] ← true
        enqueue(queue, v)
      end if
    end for
  end while
end procedure
```

### 2.3 Parallel-Edge Detection  
For each node pair $(u,v)$, gather edge set  
$$
E_{uv} = \{e : e\text{ connects }u\text{ and }v\}.
$$  
If $|E_{uv}| > 1$, apply parallel reduction.

## 3. Termination Condition

Iterate series and parallel reductions until the graph reduces to exactly two nodes $\{s,t\}$ and a single edge between them. The weight of this final edge is the equivalent resistance:  
$$
G_{\mathrm{final}}:\ 
V = \{s,t\},\ 
E = \{\{s,t\}\},\ 
R_{\mathrm{eq}} = w(\{s,t\}).
$$

# III. Pseudocode Development

## 1. High-Level Pseudocode

```plaintext
main():
    inputData ← readInput()
    G ← buildGraph(inputData)
    simplifyGraph(G)
    R_eq ← computeEquivalentResistance(G)
    print("Equivalent Resistance:", R_eq)

procedure buildGraph(inputData):
    G ← new Graph()
    for each (u, v, r) in inputData.edgeList do
        if not G.containsNode(u) then G.addNode(u)
        if not G.containsNode(v) then G.addNode(v)
        G.addEdge(u, v, weight = r)
    return G

procedure simplifyGraph(G):
    repeat
        if seriesTriple ← findSeriesTriple(G) then
            reduceSeries(G, seriesTriple)
        else if parallelPair ← findParallelPair(G) then
            reduceParallel(G, parallelPair)
        else
            break
    until false

procedure findSeriesTriple(G):
    for each node v in G.nodes do
        if v ≠ s and v ≠ t and G.degree(v) == 2 then
            let [u, w] = G.neighbors(v)
            let r1 = G.edgeWeight(u, v)
            let r2 = G.edgeWeight(v, w)
            return (u, v, w, r1, r2)
    return null

procedure reduceSeries(G, (u, v, w, r1, r2)):
    R_series = r1 + r2
    G.removeEdge(u, v)
    G.removeEdge(v, w)
    G.removeNode(v)
    addOrCombineEdge(G, u, w, R_series)

procedure findParallelPair(G):
    for each unordered pair (u, v) in G.nodePairs do
        edges = G.edgesBetween(u, v)
        if edges.size > 1 then
            resistances = [G.edgeWeight(e) for e in edges]
            return (u, v, resistances)
    return null

procedure reduceParallel(G, (u, v, resistances)):
    R_parallel = (sum(1/r for r in resistances))⁻¹
    for each e in G.edgesBetween(u, v) do G.removeEdge(e)
    addOrCombineEdge(G, u, v, R_parallel)

procedure addOrCombineEdge(G, u, v, R_new):
    if G.hasEdge(u, v) then
        r_old = G.edgeWeight(u, v)
        R_combined = (1/r_old + 1/R_new)⁻¹
        G.updateEdgeWeight(u, v, R_combined)
    else
        G.addEdge(u, v, weight = R_new)

procedure computeEquivalentResistance(G):
    // After simplification, only nodes s and t remain connected by one edge
    return G.edgeWeight(s, t)
```

---

# VI. Analysis of Algorithm Efficiency

## 1. Time and Space Complexity

Let $n = |V|$ be the number of nodes and $m = |E|$ the number of edges in the circuit graph.

1. **Series Reduction Search**  
   - Each call to `findSeriesTriple` scans all nodes and inspects degrees and neighbors in $O(n + m)$.  
   - In the worst case, up to $n-2$ series reductions occur (removing one internal node each time).  
   - Total cost for series detection and reduction:  
     $$
     \sum_{k=1}^{n-2} O(n_k + m_k)
     \;\approx\; O\bigl((n + m)\,n\bigr)
     = O(n^2 + nm).
     $$

2. **Parallel Reduction Search**  
   - Each call to `findParallelPair` may examine each edge or each unordered node‐pair.  
     - Edge‐based approach: grouping edges by endpoint pair can be done in $O(m\log m)$ (via sorting or hashing).  
   - In the worst case, up to $m-1$ parallel reductions occur.  
   - Total cost for parallel detection and reduction:  
     $$
     \sum_{k=1}^{m-1} O(m_k \log m_k)
     \;\approx\; O(m^2 \log m).
     $$

3. **Combined Simplification Loop**  
   - In each iteration, we attempt a series or parallel reduction.  
   - Worst‐case iterations $\le n + m$.  
   - Overall worst‐case time complexity:  
     $$
     O\bigl(n^2 + nm + m^2 \log m\bigr).
     $$

4. **Space Complexity**  
   - Storing the graph with adjacency lists:  
     $$
     O(n + m).
     $$  
   - Auxiliary data (visited flags, degree arrays, hash maps):  
     $$
     O(n + m).
     $$  
   - Total space:  
     $$
     O(n + m).
     $$

5. **Sparse vs. Dense Graphs**  
   - **Sparse** ($m = O(n)$):  
     $$ 
     \text{Time} = O(n^2 + n^2 \log n) = O(n^2 \log n). 
     $$  
   - **Dense** ($m = O(n^2)$):  
     $$
     \text{Time} = O(n^2 + n^3 + n^4 \log n) = O(n^4 \log n).
     $$  
   - In practice, most circuits are sparse, so performance is closer to $O(n^2 \log n)$.

## 2. Bottlenecks Identification

1. **Repeated Full-Graph Scans**  
   - Both series and parallel detection scan the entire graph each iteration.
   - Reducing this by maintaining dynamic candidate lists (e.g., queue of degree-2 nodes or hash buckets of parallel candidates) can lower per-iteration cost to near-constant time.

2. **Edge‐Grouping for Parallel Reduction**  
   - Sorting or hashing edges by endpoint pair each time is expensive.
   - A multi-map or adjacency‐list extension tracking parallel edges as they form would avoid rebuilding groups.

3. **Graph Mutation Overhead**  
   - Removing and adding nodes/edges in dynamic data structures can incur pointer or array‐shifting costs.
   - Using specialized union‐find or link‐cut trees may accelerate series/parallel merges.

4. **Worst‐Case Nested Reductions**  
   - Deeply nested series-parallel combinations may trigger many small reductions.
   - Caching subgraph reduction results (memoization) could prevent redundant work on isomorphic substructures.

---
