**C++ Reference**: [std::sort](https://en.cppreference.com/w/cpp/algorithm/sort) | [std::stable_sort](https://en.cppreference.com/w/cpp/algorithm/stable_sort)
# Sorting Algorithms Reference Sheet

---

## Algorithm Summary

| Algorithm | Best Time | Average Time | Worst Time | Space Complexity | Stable | In-Place |
| --- | --- | --- | --- | --- | --- | --- |
| **Insertion Sort** | $\mathcal{O}(n)$ | $\mathcal{O}(n^2)$ | $\mathcal{O}(n^2)$ | $\mathcal{O}(1)$ | Yes | Yes |
| **Merge Sort** | $\mathcal{O}(n \log n)$ | $\mathcal{O}(n \log n)$ | $\mathcal{O}(n \log n)$ | $\mathcal{O}(n)$ | Yes | No |
| **Quick Sort** | $\mathcal{O}(n \log n)$ | $\mathcal{O}(n \log n)$ | $\mathcal{O}(n^2)$ | $\mathcal{O}(\log n)$ | No | Yes |
| **Bucket Sort** | $\mathcal{O}(n + k)$ | $\mathcal{O}(n + k)$ | $\mathcal{O}(n^2)$ | $\mathcal{O}(n + k)$ | Yes* | No |

**Dependent on the internal sorting algorithm used for individual buckets.*

---

## Detailed Algorithm Specifications

### 1. Insertion Sort

#### Overview

Builds a sorted array incrementally. Iterates through the input elements, inserting each element into its proper location within the already sorted left partition.

#### Properties

* **Type:** Comparison-based
* **Stability:** Stable (does not alter relative order of equal keys)
* **In-Place:** Yes
* **Adaptive:** Yes (runs in $\mathcal{O}(n)$ time for nearly sorted arrays)

#### Complexity Analysis

* **Best-Case Time:** $\mathcal{O}(n)$ (array already sorted)
* **Average-Case Time:** $\mathcal{O}(n^2)$
* **Worst-Case Time:** $\mathcal{O}(n^2)$ (array sorted in reverse order)
* **Auxiliary Space:** $\mathcal{O}(1)$

#### C++ Implementation

```cpp
#include <vector>

void insertionSort(std::vector<int>& arr) {
    int n = arr.size();
    for (int i = 1; i < n; ++i) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            --j;
        }
        arr[j + 1] = key;
    }
}

```

---

### 2. Merge Sort

#### Overview

A divide-and-conquer algorithm. Recursively splits the input array into two halves until sub-arrays contain a single element, then merges the sorted halves back together.

#### Properties

* **Type:** Comparison-based, Divide & Conquer
* **Stability:** Stable
* **In-Place:** No
* **Adaptive:** No (performance remains constant regardless of initial ordering)

#### Complexity Analysis

* **Best-Case Time:** $\mathcal{O}(n \log n)$
* **Average-Case Time:** $\mathcal{O}(n \log n)$
* **Worst-Case Time:** $\mathcal{O}(n \log n)$
* **Auxiliary Space:** $\mathcal{O}(n)$ for buffer allocation during merge operations

#### C++ Implementation

```cpp
#include <vector>

void merge(std::vector<int>& arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;

    std::vector<int> L(n1), R(n2);
    for (int i = 0; i < n1; ++i) L[i] = arr[left + i];
    for (int j = 0; j < n2; ++j) R[j] = arr[mid + 1 + j];

    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k++] = L[i++];
        } else {
            arr[k++] = R[j++];
        }
    }

    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

void mergeSort(std::vector<int>& arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}

```

---

### 3. Quick Sort

#### Overview

A divide-and-conquer algorithm. Selects a pivot element and partitions the array such that all elements smaller than the pivot precede it, and all greater elements follow it. Recursively applies this process to the sub-arrays.

#### Properties

* **Type:** Comparison-based, Divide & Conquer
* **Stability:** Unstable
* **In-Place:** Yes
* **Cache Behavior:** High locality of reference, fast execution in practice

#### Complexity Analysis

* **Best-Case Time:** $\mathcal{O}(n \log n)$
* **Average-Case Time:** $\mathcal{O}(n \log n)$
* **Worst-Case Time:** $\mathcal{O}(n^2)$ (occurs with unbalanced partitions, e.g., already sorted array with end pivot)
* **Auxiliary Space:** $\mathcal{O}(\log n)$ stack space (worst-case $\mathcal{O}(n)$ stack depth)

#### C++ Implementation

```cpp
#include <vector>
#include <utility>

int partition(std::vector<int>& arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;

    for (int j = low; j < high; ++j) {
        if (arr[j] < pivot) {
            ++i;
            std::swap(arr[i], arr[j]);
        }
    }
    std::swap(arr[i + 1], arr[high]);
    return i + 1;
}

void quickSort(std::vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

```

---

### 4. Bucket Sort

#### Overview

A distribution-based algorithm. Divides elements across multiple bucket containers based on range. Each bucket is individually sorted using another sorting algorithm (or recursively) before concatenating all buckets sequentially.

#### Properties

* **Type:** Non-comparison / Distribution-based
* **Stability:** Stable (assuming stable sorting inside individual buckets)
* **In-Place:** No
* **Prerequisite:** Input elements must be uniformly distributed across a known interval (e.g., $[0, 1)$).

#### Complexity Analysis

* **Best-Case Time:** $\mathcal{O}(n + k)$ where $k$ is the number of buckets
* **Average-Case Time:** $\mathcal{O}(n + k)$
* **Worst-Case Time:** $\mathcal{O}(n^2)$ (occurs when all elements land in a single bucket)
* **Auxiliary Space:** $\mathcal{O}(n + k)$

#### C++ Implementation

```cpp
#include <vector>
#include <algorithm>

void bucketSort(std::vector<float>& arr) {
    int n = arr.size();
    if (n <= 0) return;

    std::vector<std::vector<float>> buckets(n);

    // Distribute elements into buckets
    for (int i = 0; i < n; ++i) {
        int bucketIndex = static_cast<int>(n * arr[i]);
        if (bucketIndex >= n) bucketIndex = n - 1;
        buckets[bucketIndex].push_back(arr[i]);
    }

    // Sort individual buckets
    for (int i = 0; i < n; ++i) {
        std::sort(buckets[i].begin(), buckets[i].end());
    }

    // Concatenate buckets into original array
    int index = 0;
    for (int i = 0; i < n; ++i) {
        for (float val : buckets[i]) {
            arr[index++] = val;
        }
    }
}

```

---

## Primary Trade-Off Matrix

* **Small datasets ($n \le 50$):** Insertion sort often outperforms $\mathcal{O}(n \log n)$ algorithms due to lower constant factor overhead and zero heap allocations.
* **Guaranteed performance & stability required:** Use Merge Sort when $\mathcal{O}(n \log n)$ worst-case execution time and order preservation are strict constraints.
* **Maximum in-memory speed:** Use Quick Sort for cache efficiency and standard runtime performance when stability is not required.
* **Uniformly distributed floating-point numbers:** Use Bucket Sort for linear time $\mathcal{O}(n)$ execution.