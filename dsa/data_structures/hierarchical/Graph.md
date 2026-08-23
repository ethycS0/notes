# Graph

## Overview
A Graph $G = (V, E)$ is a non-linear data structure composed of a set of **Vertices (Nodes, $V$)** and **Edges (Connections, $E$)**.

## Graph Classifications & Types
- **Directed vs Undirected**: Directed graphs have one-way edges ($u \to v$); undirected graphs have two-way edges ($u \leftrightarrow v$).
- **Weighted vs Unweighted**: Edges carry values (costs, distances) or represent simple connections.
- **Cyclic vs Acyclic**: Cyclic graphs contain paths starting and ending at the same node; Acyclic graphs have no cycles (e.g., **DAG** - Directed Acyclic Graph).
- **Dense vs Sparse**: Dense graphs have $E \approx V^2$; sparse graphs have $E \ll V^2$.

## Graph Representations
| Representation | Space Complexity | Edge Lookup $(u, v)$ | Iterate Neighbors of $u$ | Ideal For |
| :--- | :--- | :--- | :--- | :--- |
| **Adjacency Matrix** | $O(V^2)$ | $O(1)$ | $O(V)$ | Dense Graphs |
| **Adjacency List** | $O(V + E)$ | $O(\text{Degree}(u))$ | $O(\text{Degree}(u))$ | Sparse Graphs |

## Core Algorithms & Complexities
| Algorithm | Purpose | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- |
| **BFS** | Shortest path in unweighted graph | $O(V + E)$ | $O(V)$ |
| **DFS** | Connectivity, topological sort, cycle detection | $O(V + E)$ | $O(V)$ |
| **Dijkstra** | Shortest path with non-negative edge weights | $O((V + E) \log V)$ | $O(V)$ |
| **Bellman-Ford** | Shortest path with negative weights | $O(V \cdot E)$ | $O(V)$ |
| **Floyd-Warshall**| All-pairs shortest path | $O(V^3)$ | $O(V^2)$ |

## C++ Representation & Usage

Graphs in C++ are constructed using standard containers (`std::vector`, `std::unordered_map`).

### Adjacency List & BFS Code Usage
```cpp
#include <iostream>
#include <vector>
#include <queue>

// Graph represented as Adjacency List
class Graph {
    int V;
    std::vector<std::vector<int>> adj;

public:
    Graph(int v) : V(v), adj(v) {}

    void addEdge(int u, int v, bool directed = false) {
        adj[u].push_back(v);
        if (!directed) {
            adj[v].push_back(u);
        }
    }

    void bfs(int start) {
        std::vector<bool> visited(V, false);
        std::queue<int> q;

        visited[start] = true;
        q.push(start);

        while (!q.empty()) {
            int node = q.front();
            q.pop();
            std::cout << node << " ";

            for (int neighbor : adj[node]) {
                if (!visited[neighbor]) {
                    visited[neighbor] = true;
                    q.push(neighbor);
                }
            }
        }
    }
};

int main() {
    Graph g(4);
    g.addEdge(0, 1);
    g.addEdge(0, 2);
    g.addEdge(1, 2);
    g.addEdge(2, 3);

    std::cout << "BFS starting from node 0:\n";
    g.bfs(0); // Output: 0 1 2 3
}
```
