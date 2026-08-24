# Search Algorithms Reference Sheet

---

## Algorithm Summary

| Algorithm | Data Pre-requisite | Best Time | Average Time | Worst Time | Space Complexity | Primary Use Case |
| --- | --- | --- | --- | --- | --- | --- |
| **Linear Search** | None (Unsorted) | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | Small or unsorted collections |
| **Binary Search** | Sorted Array / Random Access | $\mathcal{O}(1)$ | $\mathcal{O}(\log n)$ | $\mathcal{O}(\log n)$ | $\mathcal{O}(1)$ | Large sorted static arrays |
| **Ternary Search** | Unimodal / Sorted Data | $\mathcal{O}(1)$ | $\mathcal{O}(\log n)$ | $\mathcal{O}(\log n)$ | $\mathcal{O}(1)$ | Finding local extrema / unimodal functions |
| **Exponential Search** | Sorted Unbounded Array | $\mathcal{O}(1)$ | $\mathcal{O}(\log i)$* | $\mathcal{O}(\log n)$ | $\mathcal{O}(1)$ | Infinite/unbounded streams or target near start |
| **Breadth-First Search (BFS)** | Graph / Tree Structure | $\mathcal{O}(1)$ | $\mathcal{O}(V + E)$ | $\mathcal{O}(V + E)$ | $\mathcal{O}(V)$ | Shortest path in unweighted graphs |
| **Depth-First Search (DFS)** | Graph / Tree Structure | $\mathcal{O}(1)$ | $\mathcal{O}(V + E)$ | $\mathcal{O}(V + E)$ | $\mathcal{O}(V)$ | Exhaustive search, topological sorting, cycle detection |

**Where $i$ is the index of the target element in the sequence.*

---

## Detailed Algorithm Specifications

### 1. Linear Search

#### Overview

Sequentially checks each element in the collection from start to finish until the target key is identified or the end of the collection is reached.

#### Properties

* **Data Requirement:** None (works on arrays, linked lists, and streams)
* **Access Requirement:** Sequential access
* **Optimality:** Optimal for completely unsorted, arbitrary data structures

#### Complexity Analysis

* **Best-Case Time:** $\mathcal{O}(1)$ (element found at the first position)
* **Average-Case Time:** $\mathcal{O}(n)$
* **Worst-Case Time:** $\mathcal{O}(n)$ (element absent or at the final position)
* **Auxiliary Space:** $\mathcal{O}(1)$

#### C++ Implementation

```cpp
#include <vector>

int linearSearch(const std::vector<int>& arr, int target) {
    for (int i = 0; i < static_cast<int>(arr.size()); ++i) {
        if (arr[i] == target) {
            return i;
        }
    }
    return -1;
}

```

---

### 2. Binary Search

#### Overview

A divide-and-conquer algorithm. Halves the search interval repeatedly by comparing the target to the middle element of a sorted sequence.

#### Properties

* **Data Requirement:** Sorted sequence
* **Access Requirement:** Random access ($\mathcal{O}(1)$ indexing support)
* **Variants:** Lower bound, upper bound, equal range

#### Complexity Analysis

* **Best-Case Time:** $\mathcal{O}(1)$ (target matches middle element on initial comparison)
* **Average-Case Time:** $\mathcal{O}(\log n)$
* **Worst-Case Time:** $\mathcal{O}(\log n)$
* **Auxiliary Space:** $\mathcal{O}(1)$ iterative ($\mathcal{O}(\log n)$ recursive due to call stack)

#### C++ Implementation

```cpp
#include <vector>

int binarySearch(const std::vector<int>& arr, int target) {
    int low = 0;
    int high = static_cast<int>(arr.size()) - 1;

    while (low <= high) {
        int mid = low + (high - low) / 2; // Prevents potential integer overflow
        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return -1;
}

```

---

### 3. Ternary Search

#### Overview

A divide-and-conquer technique that divides a sorted range into three equal sub-parts using two midpoints (`mid1` and `mid2`). Frequently used to find the maximum or minimum of a unimodal function.

#### Properties

* **Data Requirement:** Sorted sequence or unimodal function (strictly increasing then strictly decreasing, or vice versa)
* **Comparison Count:** Performs 2 comparisons per step, leading to more operations per level than binary search despite fewer recursive calls ($\log_3 n$)

#### Complexity Analysis

* **Best-Case Time:** $\mathcal{O}(1)$
* **Average-Case Time:** $\mathcal{O}(\log n)$
* **Worst-Case Time:** $\mathcal{O}(\log n)$
* **Auxiliary Space:** $\mathcal{O}(1)$

#### C++ Implementation

```cpp
#include <vector>

int ternarySearch(const std::vector<int>& arr, int target) {
    int low = 0;
    int high = static_cast<int>(arr.size()) - 1;

    while (low <= high) {
        int mid1 = low + (high - low) / 3;
        int mid2 = high - (high - low) / 3;

        if (arr[mid1] == target) return mid1;
        if (arr[mid2] == target) return mid2;

        if (target < arr[mid1]) {
            high = mid1 - 1;
        } else if (target > arr[mid2]) {
            low = mid1 + 1;
        } else {
            low = mid1 + 1;
            high = mid2 - 1;
        }
    }
    return -1;
}

```

---

### 4. Exponential Search

#### Overview

Finds a range where the target key resides by expanding the search step exponentially ($1, 2, 4, 8, \dots$), then executes a standard Binary Search within that bounded range. Ideal for arrays of unknown or infinite bounds.

#### Properties

* **Data Requirement:** Sorted sequence
* **Efficiency:** Time complexity scales with the position index of the target ($i$), rather than the overall array length ($n$)

#### Complexity Analysis

* **Best-Case Time:** $\mathcal{O}(1)$
* **Average-Case Time:** $\mathcal{O}(\log i)$ where $i$ is the target index
* **Worst-Case Time:** $\mathcal{O}(\log n)$
* **Auxiliary Space:** $\mathcal{O}(1)$

#### C++ Implementation

```cpp
#include <vector>
#include <algorithm>

int binarySearchRange(const std::vector<int>& arr, int target, int low, int high) {
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}

int exponentialSearch(const std::vector<int>& arr, int target) {
    int n = arr.size();
    if (n == 0) return -1;
    if (arr[0] == target) return 0;

    int i = 1;
    while (i < n && arr[i] <= target) {
        i *= 2;
    }

    return binarySearchRange(arr, target, i / 2, std::min(i, n - 1));
}

```

---

### 5. Breadth-First Search (BFS)

#### Overview

Graph traversal algorithm that explores nodes layer-by-layer radiating outward from a source node. Operates using a First-In-First-Out (FIFO) queue.

#### Properties

* **Graph Types:** Directed, Undirected, Weighted, Unweighted
* **Optimality:** Guarantees shortest path distance in **unweighted** graphs
* **Memory Constraints:** Holds entire current frontier in memory

#### Complexity Analysis

* **Time Complexity:** $\mathcal{O}(V + E)$ where $V$ is vertices and $E$ is edges
* **Auxiliary Space:** $\mathcal{O}(V)$ for queue and visited storage

#### C++ Implementation

```cpp
#include <vector>
#include <queue>

std::vector<int> bfs(int startNode, const std::vector<std::vector<int>>& adj, int numVertices) {
    std::vector<bool> visited(numVertices, false);
    std::queue<int> q;
    std::vector<int> traversalOrder;

    visited[startNode] = true;
    q.push(startNode);

    while (!q.empty()) {
        int curr = q.front();
        q.pop();
        traversalOrder.push_back(curr);

        for (int neighbor : adj[curr]) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                q.push(neighbor);
            }
        }
    }
    return traversalOrder;
}

```

---

### 6. Depth-First Search (DFS)

#### Overview

Graph traversal algorithm that explores as deep as possible along each branch before backtracking. Operates recursively or using a explicit Last-In-First-Out (LIFO) stack.

#### Properties

* **Graph Types:** Directed, Undirected, Trees
* **Applications:** Topological sorting, strongly connected components (Kosaraju/Tarjan), cycle detection, path validation

#### Complexity Analysis

* **Time Complexity:** $\mathcal{O}(V + E)$
* **Auxiliary Space:** $\mathcal{O}(V)$ for recursion call stack in the worst-case (linear graph depth)

#### C++ Implementation

```cpp
#include <vector>

void dfsHelper(int u, const std::vector<std::vector<int>>& adj, std::vector<bool>& visited, std::vector<int>& result) {
    visited[u] = true;
    result.push_back(u);

    for (int v : adj[u]) {
        if (!visited[v]) {
            dfsHelper(v, adj, visited, result);
        }
    }
}

std::vector<int> dfs(int startNode, const std::vector<std::vector<int>>& adj, int numVertices) {
    std::vector<bool> visited(numVertices, false);
    std::vector<int> traversalOrder;
    dfsHelper(startNode, adj, visited, traversalOrder);
    return traversalOrder;
}

```

---

## Primary Trade-Off Matrix

* **Unsorted array or static list:** Use Linear Search ($\mathcal{O}(n)$). Pre-sorting an array takes $\mathcal{O}(n \log n)$, which is not cost-effective for a single lookup query.
* **Sorted contiguous memory (Array/Vector):** Binary Search is the default choice ($\mathcal{O}(\log n)$ lookup time).
* **Unbounded/Streaming sorted data:** Use Exponential Search to bound the target in $\mathcal{O}(\log i)$ steps without knowing total length $n$.
* **Unweighted shortest path on graphs:** Use BFS; guarantees minimum hop count.
* **Structural graph analysis (Cycles, Topological Sort, Connected Components):** Use DFS due to natural integration with execution order and stack mechanics.