[Home](../README.md) > [General Knowledge](README.md) > [Data Structures and Algorithms](Data%20Structures%20and%20Algorithms.md)
# Data Structures and Algorithms
| Data Structure        | Description | Read | Insert | Remove | Search | Special Cases | Memory Considerations |
|-----------------------|---------------------------|------|--------|--------|--------|---------------|-----------------------|
| **TArray** | Dynamic array with contiguous memory, supports fast index-based access and resizing. Best for sequential access but resizing can be costly. | O(1) | O(1) append, O(n) insert in middle | O(n) if shifting required | O(n) linear search, O(log n) if sorted | Resizing can be costly (O(n)) | Can allocate extra space to reduce reallocations. |
| **TMap** | Hash-based key-value dictionary with fast lookups but unordered iteration. Good for quick key-based retrieval. | O(1) avg, O(n) worst | O(1) avg | O(1) avg | O(1) avg, O(n) worst case (hash collision) | Hash collisions may degrade performance | Extra space for hash tables and pointers. |
| **TSet** | Unordered collection of unique elements using hashing for fast operations. Ensures no duplicates. | O(1) avg | O(1) avg | O(1) avg | O(1) avg | Hash collisions affect performance | Overhead due to hash table storage. |
| **TQueue** | FIFO queue, often used in multithreading. Lock-free in most cases, making it efficient for producer-consumer patterns. | O(1) | O(1) | O(1) | O(n) if searching | Designed for sequential access only | Minimal, only elements and head/tail pointers. |
| **TArrayView** | Lightweight reference to an existing TArray, allowing partial access. | O(1) | N/A | N/A | O(n) | Does not own memory | No extra allocation, just a reference. |
| **TDeque** | Double-ended queue allowing efficient insertions and removals at both ends. Better than TArray for such use cases. | O(1) | O(1) at front/back | O(1) front/back | O(n) | Indexing is O(n) due to segmented storage | Uses segmented blocks instead of continuous memory. |
| **TBitArray** | Compact storage for boolean values, using individual bits instead of full bytes. Efficient for flags. | O(1) | O(1) | O(1) | O(n) | Bitwise operations make it memory efficient | Lower memory usage than TArray<bool>. |
| **TCircularBuffer** | Fixed-size buffer that overwrites old data when full. Useful for real-time processing or logging. | O(1) | O(1) | O(1) | O(n) | Wraps around when full | Preallocated fixed size, no dynamic growth. |
| **TSparseArray** | Non-contiguous array with stable references, designed for efficient insertions and deletions without shifting elements. | O(1) | O(1) | O(1) (marking as free) | O(n) | Frees slots are reused, keeping references stable | Indirect indexing increases memory usage. |
| **TDynamicSparseArray** | Sparse array that dynamically grows while keeping efficient memory usage. Good for sparse but expandable storage. | O(1) | O(1) | O(1) | O(n) | Optimized for sparse data sets. Object Pooling. | Uses more memory than TSparseArray. |
| **TLinkedList** | Doubly linked list, allowing efficient insertions/removals without shifting elements. Ideal for frequent modifications in the middle. | O(n) | O(1) | O(1) | O(n) | No contiguous storage, better for frequent middle modifications | Higher memory due to extra pointers per node. |
| **TDoubleLinkedList** | More feature-rich linked list supporting bidirectional traversal. Similar to TLinkedList but better for iteration. | O(n) | O(1) | O(1) | O(n) | Slightly better iteration performance | Slightly more overhead than TLinkedList. |
| **TIndirectArray** | Owns the objects and auto-destroys them when removed or at the end of the scope. | O(1) | O(1) | O(n) | O(n) | Only allows pointer-based access | More memory overhead due to storing pointers. |


**O(1)**: Constant Time – Execution takes the same amount of time, regardless of the size of the input.

**O(log n)**: Logarithmic Time – Execution time grows very slowly as the input size increases.

**O(n)**: Linear Time – Execution time grows directly with the size of the input.

## Search Algorithms
### Binary Search
Best for **sorted arrays**, O(log n) time complexity, much faster than linear search for large datasets.
```cpp
int32 BinarySearch(const TArray<int32>& arr, int32 target) {
    int32 low = 0;
    int32 high = arr.Num() - 1;

    while (low <= high) {
        int32 mid = low + (high - low) / 2;

        if (arr[mid] == target) {
            return mid;
        }
        else if (arr[mid] < target) {
            low = mid + 1;
        }
        else {
            high = mid - 1;
        }
    }

    return -1;
}
```

### Linear Search
Best for **unsorted arrays**, O(n) time complexity.
```cpp
int32 LinearSearch(const TArray<int32>& arr, int32 target) {
    for (int32 i = 0; i < arr.Num(); ++i) {
        if (arr[i] == target) {
            return i;
        }
    }
    return -1;
}
```

## Sort Algorithms

| **Algorithm**      | **Best Case** | **Average Case** | **Worst Case** | **Memory Complexity** | **When to Use**                                   | **When Not to Use**                       |
|-------------------|-------------|----------------|--------------|----------------------|--------------------------------------|--------------------------------|
| **Quick Sort**   | O(n log n)   | O(n log n)     | O(n²)        | O(log n)             | Large dataset sorting, cache efficiency, low memory usage | Worst-case performance-sensitive cases |
| **Merge Sort**   | O(n log n)   | O(n log n)     | O(n log n)   | O(n)                 | Stable sorting (preserves relative order of equal elements), linked lists, external storage sorting, predictable performance | Low memory availability |
| **Heap Sort**    | O(n log n)   | O(n log n)     | O(n log n)   | O(1)                 | Strict memory constraints, Priority-based sorting, Predictable performance, sorting while inserting | Cache efficiency matters |
| **Insertion Sort** | O(n)       | O(n²)          | O(n²)        | O(1)                 | Nearly sorted data, small datasets   | Large datasets, random order data |



### QuickSort
```cpp
void QuickSort(TArray<int32>& arr, int32 low, int32 high) {
    if (low < high) {
        int32 pivot = arr[high];
        int32 i = low - 1;
        for (int32 j = low; j < high; ++j) {
            if (arr[j] <= pivot) {
                i++;
                arr.Swap(i, j);
            }
        }
        arr.Swap(i + 1, high);
        int32 pi = i + 1;

        QuickSort(arr, low, pi - 1);
        QuickSort(arr, pi + 1, high);
    }
}
```

### MergeSort
```cpp
void Merge(TArray<int32>& arr, int32 left, int32 mid, int32 right) {
    int32 n1 = mid - left + 1;
    int32 n2 = right - mid;

    TArray<int32> leftArr, rightArr;
    for (int32 i = 0; i < n1; ++i) leftArr.Add(arr[left + i]);
    for (int32 i = 0; i < n2; ++i) rightArr.Add(arr[mid + 1 + i]);

    int32 i = 0, j = 0, k = left;
    while (i < n1 && j < n2) {
        if (leftArr[i] <= rightArr[j]) {
            arr[k] = leftArr[i];
            i++;
        } else {
            arr[k] = rightArr[j];
            j++;
        }
        k++;
    }

    while (i < n1) {
        arr[k] = leftArr[i];
        i++;
        k++;
    }

    while (j < n2) {
        arr[k] = rightArr[j];
        j++;
        k++;
    }
}

void MergeSort(TArray<int32>& arr, int32 left, int32 right) {
    if (left < right) {
        int32 mid = left + (right - left) / 2;
        MergeSort(arr, left, mid);
        MergeSort(arr, mid + 1, right);
        Merge(arr, left, mid, right);
    }
}
```

### HeapSort
```cpp
void Heapify(TArray<int32>& arr, int32 n, int32 i) {
    int32 largest = i;
    int32 left = 2 * i + 1;
    int32 right = 2 * i + 2;

    if (left < n && arr[left] > arr[largest]) {
        largest = left;
    }

    if (right < n && arr[right] > arr[largest]) {
        largest = right;
    }

    if (largest != i) {
        arr.Swap(i, largest);
        Heapify(arr, n, largest);
    }
}

void HeapSort(TArray<int32>& arr) {
    int32 n = arr.Num();
    
    for (int32 i = n / 2 - 1; i >= 0; i--) {
        Heapify(arr, n, i);
    }
    
    for (int32 i = n - 1; i >= 0; i--) {
        arr.Swap(0, i);
        Heapify(arr, i, 0);
    }
}
```

### InsertionSort
```cpp
void InsertionSort(TArray<int32>& arr) {
    int32 n = arr.Num();
    for (int32 i = 1; i < n; i++) {
        int32 key = arr[i];
        int32 j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 