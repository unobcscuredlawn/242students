# Lab 7: Observing Graphs and Graph Algorithms

**CPTR 242 — Sequential and Parallel Data Structures and Algorithms**
Walla Walla University

---

## Overview

In this lab you will **run complete programs and observe their behavior**. There is no coding required. Your job is to read the output carefully, record what you see, and reason about what it means.

By the end you should be able to:

- Explain the difference between adjacency list and adjacency matrix representations and identify which is preferable given graph density
- Trace BFS and DFS on the same graph and describe how the choice of frontier data structure determines traversal order
- Follow Dijkstra's algorithm step by step and explain why it fails on negative-weight edges
- Follow Bellman-Ford step by step and explain how it detects negative-weight cycles
- Compare the time and space complexity of BFS, DFS, Dijkstra, Bellman-Ford, topological sort, Prim's MST, and Floyd-Warshall across graph types and sizes

---

## Background: Vertices, Edges, and the Cost of Representation

Every data structure in this course so far has imposed a strict shape on its data — arrays are sequential, trees have a parent–child hierarchy. A **graph** imposes no shape at all. It is simply a set of **vertices** (nodes) and **edges** (connections between them). Edges may be directed or undirected, weighted or unweighted, and the same pair of vertices may be connected by multiple edges or none at all.

That freedom creates an immediate question: **how do you store a graph in memory?**

Two standard answers exist:

- An **adjacency list** stores, for each vertex, the list of vertices it connects to. Space is O(V + E) — proportional to what actually exists.
- An **adjacency matrix** stores a V × V grid where entry [i][j] holds the edge weight (or 0/∞ if no edge). Space is always O(V²), regardless of how many edges exist.

For a **sparse** graph (E ≪ V²) the list wastes far less memory. For a **dense** graph (E ≈ V²) the matrix offers O(1) edge lookup with no pointer chasing.

The central question: **does the choice of representation change what an algorithm can do, and how much does it cost?**

---

## Setup

All programs are self-contained. Each generates its own graph and prints results directly to the terminal.

**Compiler command for C++ programs:**
```bash
g++ -O0 -std=c++17 -o program program.cpp && ./program
```

> `-O0` disables compiler optimizations so you observe true algorithm behavior. `-std=c++17` is required for structured bindings.

> Some programs will also be in Python.

I encourage you to use a Jupyter notebook for the Python.
You can use regular Python files with the interpreter, but you have to import the inital 8-puzzle base code in every Python search algorithm.

---

## Model 1: Adjacency List vs. Adjacency Matrix — Representation and Memory

Before running any algorithm you need to understand what a graph looks like in memory. This program builds the same undirected weighted graph using both representations, prints each structure, and reports the memory each one occupies at several graph sizes.

### Program 1 — `graph_representations.cpp`

```cpp

```

### Observation Table 1a — Small Graph Structure

Using the printed output for the 6-vertex graph, fill in the neighbor list for each vertex and the matrix entry at the specified positions.

| Vertex | Neighbors (v, weight) |
|---|---|
| 0 | (1,w=4) (2,w=2) |
| 1 | (0,w=4) (2,w=5) (3,w=10) |
| 2 | (0,w=2) (1,w=5) (4,w=3) |
| 3 | (1,w=10) (5,w=7) |
| 4 | (2,w=3) (5,w=1) |
| 5 | (3,w=7) (4,w=1) |

| Matrix position | Value | What it means |
|---|---|---|
| mat[0][1] | 4 | |
| mat[1][0] | 4 | |
| mat[0][3] | 0 | |
| mat[3][5] | 7 | |

### Observation Table 1b — Memory Usage

| V | E | List (bytes) | Matrix (bytes) | Ratio (matrix/list) |
|---|---|---|---|---|
| 10  | 12     | 432 | 400 | 0.9x |
| 10  | 45     | 960 | 400 | 0.4x |
| 100 | 150    | 4800 | 40000 | 8.3x |
| 100 | 4,950  | 81600 | 40000 | 0.5x |
| 500 | 600    | 21600 | 1000000 | 46.3x |
| 500 | 124,750 | 2008000 | 1000000 | 0.5x |

---

### Critical Thinking Questions — Model 1

**Q1.** At V = 100 with E = 150 (sparse), which representation is smaller? At V = 100 with E = 4590, which is smaller? Identify the crossover condition in terms of E and V.

> Your answer: An ajacency list is smaller is both cases. The ajacency list will be smaller if E is less than V^2 - V.

---

## Model 2: Problem Formulation

### Description
This exercise demonstrates how to formulate a search problem by defining its five key components. Understanding problem formulation is the foundation for applying any search algorithm. We'll use the 8-puzzle as our example domain.

### Key Concepts
- **State**: A configuration of the environment (e.g., positions of tiles in the 8-puzzle)
- **Actions**: Possible moves from a given state (e.g., slide tile up, down, left, right)
- **Transition Model**: The result of applying an action to a state
- **Goal Test**: A function that determines if a state is the goal
- **Path Cost**: The cost of reaching a state (e.g., number of moves)

### Task
Run the code below and observe how a problem is formally defined. Pay attention to:
- How states are represented (what data structure is used?)
- How the `get_actions()` method determines valid moves
- What information the `result()` method returns
- How the goal test works

```python

```

### Reflection Questions

**Q2:** In the 8-puzzle, we represent states as tuples of 9 numbers. What is one advantage of this representation compared to using a 2D array or other data structures? Consider computational efficiency, memory allocation, mutability and ease of use.

> Your answer: Tuples are better for mutibiliy, hashability, and can use sets to show what has been explored.

---

## Model 3: Breadth-First Search (BFS)

### Description
This exercise demonstrates uninformed breadth-first search, which explores all nodes at depth *d* before exploring nodes at depth *d+1*. BFS uses a FIFO (first-in-first-out) queue to manage the frontier.

### Key Concepts
- **Uninformed Search**: Strategies that have no problem-specific knowledge beyond the problem definition
- **Frontier**: The set of nodes that have been generated but not yet expanded
- **FIFO Queue**: Data structure where the first element added is the first one removed
- **Completeness**: A search algorithm is complete if it always finds a solution when one exists
- **Optimality**: An algorithm is optimal if it finds a solution with the lowest path cost

### Task
Run the code and observe the search process. Focus on:
- The order in which nodes are explored (printed as "Exploring:")
- How the frontier size changes over time
- How many nodes are expanded before finding the goal
- The path returned by BFS

```python

```

### Reflection Questions

**Q3:** BFS explores nodes level by level (all nodes at depth 1, then all at depth 2, etc.). How does this exploration strategy guarantee that BFS finds the optimal solution for problems where all actions have the same cost?

> Your answer: By searching by level, it can garuntee that the first goal has the smallest cost.

**Q4:** Observe the frontier size as the search progresses. Why does the frontier grow so rapidly in BFS, and what does this tell you about the space complexity of this algorithm? How would this affect solving larger problems?

> Your answer: the space complexity is O(b^d) where b is the branching factor and d is the depth of the shallowest solution. Larger problems would have issues with the memeory limit.

---

## Model 4: Depth-First Search (DFS)

### Description
This exercise demonstrates depth-first search, which always expands the deepest node in the frontier using a LIFO (last-in-first-out) stack. Comparing DFS to BFS reveals important tradeoffs between different uninformed strategies.

### Key Concepts
- **LIFO Stack**: Data structure where the last element added is the first one removed
- **Depth-First Exploration**: Exploring as deeply as possible before backtracking
- **Space Complexity Advantage**: DFS stores fewer nodes in memory than BFS
- **Non-Optimal**: DFS may find a solution but not the shortest one
- **Depth Limiting**: Preventing infinite loops by limiting maximum depth

### Task
Run the code and compare DFS to BFS from Exercise 2. Notice:
- How the exploration order differs from BFS
- The maximum frontier size compared to BFS
- Whether the solution found is optimal (shortest)
- How the depth limit prevents infinite exploration

```python

```

### Reflection Questions

**Q5:** Compare the maximum frontier size between DFS and BFS. Why does DFS use significantly less memory, and in what situations would this memory advantage be crucial for solving a problem?

> Your answer: It uses less memory because it only needs to remember the current path and a few unexplored alternatives at each level. This is useful when ram is limited and/or the search space is large.

**Q6:** DFS found a solution, but was it the optimal (shortest) solution? Explain why DFS is not guaranteed to find the optimal solution even when one exists, and describe a scenario where DFS might find a very poor solution.

> Your answer: DFS might find a solution on a deep branch even though a shorter solution might exist elsewhere. DFS might find a poor solution in the 8 puzzle.

**Q7:** We implemented a depth limit to prevent DFS from exploring infinitely deep paths. What problems could arise without this limit, and how does this relate to the concept of completeness in search algorithms?

> Your answer: Some branches might be infinite and prevent the algorithm from making progress. an algortim is complete when it is garunteed to find a solution, but an algorithm without a depth limit can prevent it from finding a solution.

---

## Model 5: Uniform-Cost Search (UCS) also known as Dijkstra's Algorithm

### Description
This exercise demonstrates uniform-cost search, which expands nodes in order of their path cost. UCS uses a priority queue and is optimal for problems with varying action costs.

### Key Concepts
- **Priority Queue**: Data structure where elements are removed in order of priority (lowest cost first)
- **Path Cost**: The total cost of reaching a node from the initial state
- **Uniform-Cost Property**: Always expanding the lowest-cost node guarantees optimality
- **Action Cost Variation**: Different actions may have different costs
- **Completeness with Optimality**: UCS is both complete and optimal

### Task
Run the code and observe how UCS differs from BFS. Pay attention to:
- How nodes are ordered by path cost rather than depth
- The role of the priority queue in selecting which node to expand
- How UCS handles varying action costs
- Why the solution is guaranteed to be optimal

```python

```

### Reflection Questions

**Q8:** How does UCS decide which node to expand next, and why does this strategy guarantee finding the optimal solution? Compare this to how BFS makes its expansion decisions.

> Your answer: at every step, UCS expands the node with the lowest total cost from initial state. it finds the optimal solution because all other paths cost more. BFS finds the lowest depth and UCS finds the lowest cost.

**Q9:** Observe the path found by UCS through the grid. Does it go directly toward the goal, or does it take a longer route? Explain why UCS chose this path in terms of path cost versus path length.

> Your answer: UCS takes a longer route in order to save on cost even if it moves more

**Q10:** In what types of problems would UCS perform identically to BFS? When would UCS clearly outperform BFS in terms of solution quality? Give specific examples of problem domains.

> Your answer: they would behave the same if the cost is the same. UCS would out perform when actions have different costs.

---

## Model 6: Algorithm Comparison — BFS, DFS, Dijkstra, Bellman-Ford, Topological Sort, Prim's MST, Floyd-Warshall

This model benchmarks all seven graph algorithms discussed in the course across three graph types — sparse, medium, and dense — and reports wall-clock time, memory overhead, and the correctness conditions for each. No new algorithm is traced step-by-step here; the goal is to synthesize everything into a single comparative picture.

### Program 6 — `algorithm_comparison.cpp`

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <climits>
#include <chrono>
#include <iomanip>
#include <numeric>
#include <random>
#include <algorithm>
using namespace std;
using Clock = chrono::high_resolution_clock;

const int INF = INT_MAX / 2;
using WGraph = vector<vector<pair<int,int>>>;  // adj list (neighbor, weight)
using EdgeList = vector<tuple<int,int,int>>;   // (u, v, w)

// -------- BFS --------
void bfs(const vector<vector<int>>& adj, int src) {
    int V = adj.size();
    vector<bool> vis(V, false);
    queue<int> q;
    vis[src] = true; q.push(src);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u]) if (!vis[v]) { vis[v]=true; q.push(v); }
    }
}

// -------- DFS (iterative) --------
void dfs(const vector<vector<int>>& adj, int src) {
    int V = adj.size();
    vector<bool> vis(V, false);
    stack<int> s;
    s.push(src);
    while (!s.empty()) {
        int u = s.top(); s.pop();
        if (vis[u]) continue;
        vis[u] = true;
        for (int v : adj[u]) if (!vis[v]) s.push(v);
    }
}

// -------- Dijkstra --------
void dijkstra(const WGraph& adj, int src) {
    int V = adj.size();
    vector<int> dist(V, INF);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    dist[src] = 0; pq.push({0, src});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;
        for (auto [v, w] : adj[u])
            if (dist[u]+w < dist[v]) { dist[v]=dist[u]+w; pq.push({dist[v],v}); }
    }
}

// -------- Bellman-Ford --------
void bellmanFord(int V, const EdgeList& edges, int src) {
    vector<int> dist(V, INF); dist[src] = 0;
    for (int i = 0; i < V-1; i++)
        for (auto [u, v, w] : edges)
            if (dist[u] < INF && dist[u]+w < dist[v]) dist[v] = dist[u]+w;
}

// -------- Topological sort (Kahn's algorithm) --------
void topoSort(const vector<vector<int>>& adj) {
    int V = adj.size();
    vector<int> indeg(V, 0);
    for (int u = 0; u < V; u++) for (int v : adj[u]) indeg[v]++;
    queue<int> q;
    for (int u = 0; u < V; u++) if (indeg[u]==0) q.push(u);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u]) if (--indeg[v]==0) q.push(v);
    }
}

// -------- Prim's MST --------
void prim(const WGraph& adj, int src) {
    int V = adj.size();
    vector<int> key(V, INF);
    vector<bool> inMST(V, false);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    key[src] = 0; pq.push({0, src});
    while (!pq.empty()) {
        auto [k, u] = pq.top(); pq.pop();
        if (inMST[u]) continue;
        inMST[u] = true;
        for (auto [v, w] : adj[u])
            if (!inMST[v] && w < key[v]) { key[v]=w; pq.push({key[v],v}); }
    }
}

// -------- Floyd-Warshall --------
void floydWarshall(vector<vector<int>> dist) {
    int V = dist.size();
    for (int k = 0; k < V; k++)
        for (int i = 0; i < V; i++)
            for (int j = 0; j < V; j++)
                if (dist[i][k] < INF && dist[k][j] < INF)
                    dist[i][j] = min(dist[i][j], dist[i][k]+dist[k][j]);
}

// -------- Graph generators --------
struct TestGraph {
    int V, E;
    vector<vector<int>>    unweighted;
    WGraph                 weighted;
    EdgeList               edgeList;
    vector<vector<int>>    matrix;
};

TestGraph makeGraph(int V, int E, mt19937& rng) {
    TestGraph g;
    g.V = V; g.E = E;
    g.unweighted.assign(V, {});
    g.weighted.assign(V, {});
    g.matrix.assign(V, vector<int>(V, INF));
    for (int i = 0; i < V; i++) g.matrix[i][i] = 0;

    // Guarantee connectivity via a spanning chain
    for (int i = 0; i < V-1; i++) {
        int w = 1 + rng() % 20;
        g.unweighted[i].push_back(i+1);
        g.weighted[i].push_back({i+1, w});
        g.edgeList.push_back({i, i+1, w});
        g.matrix[i][i+1] = w;
    }
    // Fill remaining edges randomly
    uniform_int_distribution<int> vdist(0, V-1);
    int added = V-1;
    while (added < E) {
        int u = vdist(rng), v = vdist(rng);
        if (u == v) continue;
        int w = 1 + rng() % 20;
        g.unweighted[u].push_back(v);
        g.weighted[u].push_back({v, w});
        g.edgeList.push_back({u, v, w});
        if (g.matrix[u][v] == INF) g.matrix[u][v] = w;
        added++;
    }
    return g;
}

double bench(const string& algo, TestGraph& g) {
    auto t0 = Clock::now();
    if      (algo == "bfs")    bfs(g.unweighted, 0);
    else if (algo == "dfs")    dfs(g.unweighted, 0);
    else if (algo == "dijkstra") dijkstra(g.weighted, 0);
    else if (algo == "bellman")  bellmanFord(g.V, g.edgeList, 0);
    else if (algo == "topo")   topoSort(g.unweighted);
    else if (algo == "prim")   prim(g.weighted, 0);
    else if (algo == "floyd")  floydWarshall(g.matrix);
    return chrono::duration<double, milli>(Clock::now() - t0).count();
}

int main() {
    mt19937 rng(42);

    // (V, E) configs: sparse, medium, dense
    vector<tuple<int,int,string>> configs = {
        {500,  600,   "sparse (E≈V)"},
        {500,  5000,  "medium (E≈10V)"},
        {500,  50000, "dense  (E≈100V)"}
    };

    vector<string> algos = {
        "bfs","dfs","dijkstra","bellman","topo","prim","floyd"
    };
    vector<string> labels = {
        "BFS","DFS","Dijkstra","Bellman-Ford","Topo Sort","Prim MST","Floyd-Warshall"
    };

    for (auto& [V, E, desc] : configs) {
        TestGraph g = makeGraph(V, E, rng);
        cout << "=== " << desc << " (V=" << V << ", E=" << E << ") ===\n\n";
        cout << setw(18) << "Algorithm"
             << setw(14) << "Time (ms)"
             << "\n" << string(32, '-') << "\n";
        for (int i = 0; i < (int)algos.size(); i++) {
            // Floyd-Warshall only on small dense to keep runtime sane
            if (algos[i] == "floyd" && E > 10000) {
                cout << setw(18) << labels[i]
                     << setw(14) << "(skipped — O(V^3))" << "\n";
                continue;
            }
            double ms = bench(algos[i], g);
            cout << setw(18) << labels[i]
                 << setw(14) << fixed << setprecision(3) << ms << "\n";
        }
        cout << "\n";
    }

    // Complexity reference table
    cout << "=== Complexity Reference ===\n\n";
    cout << setw(18) << "Algorithm"
         << setw(18) << "Time"
         << setw(18) << "Space (extra)"
         << setw(16) << "Graph type"
         << setw(20) << "Handles neg. edges?"
         << "\n" << string(90, '-') << "\n";

    vector<array<string,5>> info = {
        {"BFS",           "O(V+E)",       "O(V)",      "unweighted",   "N/A"},
        {"DFS",           "O(V+E)",       "O(V)",      "any",          "N/A"},
        {"Dijkstra",      "O((V+E)logV)", "O(V)",      "non-neg wts",  "No"},
        {"Bellman-Ford",  "O(VE)",        "O(V)",      "any weights",  "Yes"},
        {"Topo Sort",     "O(V+E)",       "O(V)",      "DAG only",     "N/A"},
        {"Prim MST",      "O((V+E)logV)", "O(V)",      "undirected",   "No"},
        {"Floyd-Warshall","O(V^3)",       "O(V^2)",    "any weights",  "Yes"},
    };

    for (auto& row : info)
        cout << setw(18) << row[0] << setw(18) << row[1]
             << setw(18) << row[2] << setw(16) << row[3]
             << setw(20) << row[4] << "\n";
}
```

### Observation Table 6a — Benchmark Times (ms)

**Sparse (E ≈ V)**

| Algorithm | Time (ms) |
|---|---|
| BFS | |
| DFS | |
| Dijkstra | |
| Bellman-Ford | |
| Topo Sort | |
| Prim MST | |
| Floyd-Warshall | |

**Medium (E ≈ 10V)**

| Algorithm | Time (ms) |
|---|---|
| BFS | |
| DFS | |
| Dijkstra | |
| Bellman-Ford | |
| Topo Sort | |
| Prim MST | |
| Floyd-Warshall | |

**Dense (E ≈ 100V)**

| Algorithm | Time (ms) |
|---|---|
| BFS | |
| DFS | |
| Dijkstra | |
| Bellman-Ford | |
| Topo Sort | |
| Prim MST | |
| Floyd-Warshall | |

### Observation Table 6b — Complexity Reference

Fill in the table from the printed output.

| Algorithm | Time complexity | Extra space | Graph type required | Handles negative edges? |
|---|---|---|---|---|
| BFS | | | | |
| DFS | | | | |
| Dijkstra | | | | |
| Bellman-Ford | | | | |
| Topo Sort | | | | |
| Prim MST | | | | |
| Floyd-Warshall | | | | |

---

### Critical Thinking Questions — Model 6

**Q11.** From Table 6a, how does Bellman-Ford's time compare to Dijkstra's across the three densities? At what density does Bellman-Ford become significantly slower? Use the complexity formulas O(VE) vs O((V+E) log V) to explain what you observe.

> Your answer:

**Q12.** You are designing a navigation system (like Google Maps) for a city with 100,000 intersections and 300,000 road segments, all with positive travel times. You need to answer shortest-path queries from any source to any destination in under 50 ms. Which algorithm from Table 6b would you choose and why? Which would you rule out immediately?

> Your answer: hint: not all algorithms are shortest path algorithms

---


*Next lab: algorithm design — where the question shifts from "which algorithm" to "how do you prove one is correct and derive its complexity from scratch."*
