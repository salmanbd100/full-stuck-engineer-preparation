# Time & Space Complexity Analysis

## Pattern Overview

Understanding time and space complexity is fundamental to algorithm analysis and technical interviews. Every solution you propose must be analyzed for both time and space efficiency.

**Big O Notation** describes the upper bound of algorithm performance as input size grows.

## 🎯 When to Consider Complexity

- During coding interviews (always state complexity)
- Choosing between multiple solutions
- Optimizing existing code
- Estimating scalability
- Comparing algorithm efficiency

## 📊 Common Time Complexities

### O(1) - Constant Time
**Performance:** Same time regardless of input size

**Examples:**
- Array access by index
- Hash map get/set operations
- Math operations
- Variable assignment

```javascript
// O(1) - Constant time
function getFirstElement(arr) {
  return arr[0]; // Single operation, doesn't depend on array size
}

function hashLookup(map, key) {
  return map.get(key); // Hash map lookup is O(1) average case
}
```

```python
# O(1) - Constant time
def get_first_element(arr):
    return arr[0]  # Single array access

def hash_lookup(hash_map, key):
    return hash_map.get(key)  # Dictionary lookup
```

---

### O(log n) - Logarithmic Time
**Performance:** Doubles input, adds one operation

**Examples:**
- Binary search
- Balanced tree operations
- Finding in sorted array

```javascript
// O(log n) - Binary search
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (arr[mid] === target) return mid;
    if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }

  return -1;
}

// Time: O(log n) - halves search space each iteration
// Space: O(1) - only uses a few variables
```

```python
# O(log n) - Binary search
def binary_search(arr, target):
    left, right = 0, len(arr) - 1

    while left <= right:
        mid = (left + right) // 2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1

# Time: O(log n)
# Space: O(1)
```

**Why O(log n)?**
- Each iteration halves the search space
- n → n/2 → n/4 → n/8 → ... → 1
- Number of steps = log₂(n)

---

### O(n) - Linear Time
**Performance:** Proportional to input size

**Examples:**
- Single loop through array
- Linear search
- Finding min/max in unsorted array

```javascript
// O(n) - Linear time
function findMax(arr) {
  let max = arr[0];

  // Loop runs n times
  for (let i = 1; i < arr.length; i++) {
    if (arr[i] > max) {
      max = arr[i];
    }
  }

  return max;
}

// Time: O(n) - one pass through array
// Space: O(1) - only one variable
```

```python
# O(n) - Linear time
def find_max(arr):
    max_val = arr[0]

    for num in arr:
        if num > max_val:
            max_val = num

    return max_val

# Time: O(n)
# Space: O(1)
```

**Multiple Loops:**
```javascript
// Still O(n) - sequential loops
function processArray(arr) {
  // First loop: O(n)
  for (let i = 0; i < arr.length; i++) {
    console.log(arr[i]);
  }

  // Second loop: O(n)
  for (let i = 0; i < arr.length; i++) {
    console.log(arr[i] * 2);
  }

  // Total: O(n) + O(n) = O(2n) = O(n)
  // We drop constants in Big O
}
```

---

### O(n log n) - Linearithmic Time
**Performance:** Linear time with logarithmic factor

**Examples:**
- Efficient sorting (Merge Sort, Quick Sort, Heap Sort)
- Divide and conquer algorithms

```javascript
// O(n log n) - Merge Sort
function mergeSort(arr) {
  if (arr.length <= 1) return arr;

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));   // O(log n) divisions
  const right = mergeSort(arr.slice(mid));     // O(log n) divisions

  return merge(left, right);                   // O(n) merge
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;

  while (i < left.length && j < right.length) {
    if (left[i] < right[j]) {
      result.push(left[i++]);
    } else {
      result.push(right[j++]);
    }
  }

  return result.concat(left.slice(i)).concat(right.slice(j));
}

// Time: O(n log n) - log n recursive divisions, n work per level
// Space: O(n) - temporary arrays for merging
```

```python
# O(n log n) - Merge Sort
def merge_sort(arr):
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])

    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0

    while i < len(left) and j < len(right):
        if left[i] < right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    result.extend(left[i:])
    result.extend(right[j:])
    return result

# Time: O(n log n)
# Space: O(n)
```

**Why O(n log n)?**
- Divides array log n times (like binary search)
- Each level does O(n) work (merging)
- Total: O(n) × O(log n) = O(n log n)

---

### O(n²) - Quadratic Time
**Performance:** Nested operations over input

**Examples:**
- Nested loops
- Bubble sort, Selection sort
- Comparing all pairs

```javascript
// O(n²) - Nested loops
function findDuplicates(arr) {
  const duplicates = [];

  // Outer loop: n iterations
  for (let i = 0; i < arr.length; i++) {
    // Inner loop: n iterations
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j] && !duplicates.includes(arr[i])) {
        duplicates.push(arr[i]);
      }
    }
  }

  return duplicates;
}

// Time: O(n²) - nested loops
// Space: O(n) - worst case all are duplicates
```

```python
# O(n²) - Bubble Sort
def bubble_sort(arr):
    n = len(arr)

    for i in range(n):
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]

    return arr

# Time: O(n²)
# Space: O(1) - in-place sorting
```

**Optimization:**
```javascript
// O(n) - Better approach using hash set
function findDuplicates(arr) {
  const seen = new Set();
  const duplicates = new Set();

  for (const num of arr) {
    if (seen.has(num)) {
      duplicates.add(num);
    } else {
      seen.add(num);
    }
  }

  return Array.from(duplicates);
}

// Time: O(n) - single pass
// Space: O(n) - using sets
```

---

### O(2ⁿ) - Exponential Time
**Performance:** Doubles with each input increase

**Examples:**
- Recursive Fibonacci (naive)
- Generating all subsets
- Brute force solutions

```javascript
// O(2ⁿ) - Naive Fibonacci
function fibonacci(n) {
  if (n <= 1) return n;

  // Each call makes 2 more calls
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Time: O(2ⁿ) - exponential branching
// Space: O(n) - recursion depth

// For n=5: Makes 15 function calls
// For n=10: Makes 177 function calls
// For n=20: Makes 21,891 function calls!
```

```python
# O(2ⁿ) - Naive Fibonacci
def fibonacci(n):
    if n <= 1:
        return n

    return fibonacci(n - 1) + fibonacci(n - 2)

# Time: O(2ⁿ)
# Space: O(n) - call stack depth
```

**Optimization with Memoization:**
```javascript
// O(n) - Optimized with memoization
function fibonacci(n, memo = {}) {
  if (n <= 1) return n;
  if (memo[n]) return memo[n];

  memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo);
  return memo[n];
}

// Time: O(n) - each value calculated once
// Space: O(n) - memo storage + recursion stack
```

---

### O(n!) - Factorial Time
**Performance:** Multiplies for each input

**Examples:**
- Generating all permutations
- Traveling salesman (brute force)
- Combinatorial problems

```javascript
// O(n!) - Generate all permutations
function permutations(arr) {
  if (arr.length <= 1) return [arr];

  const result = [];

  for (let i = 0; i < arr.length; i++) {
    const current = arr[i];
    const remaining = arr.slice(0, i).concat(arr.slice(i + 1));
    const remainingPerms = permutations(remaining);

    for (const perm of remainingPerms) {
      result.push([current, ...perm]);
    }
  }

  return result;
}

// Time: O(n!) - n choices, then n-1, then n-2, ...
// Space: O(n!) - storing all permutations

// For n=5: 120 permutations
// For n=10: 3,628,800 permutations!
```

```python
# O(n!) - Generate all permutations
def permutations(arr):
    if len(arr) <= 1:
        return [arr]

    result = []

    for i in range(len(arr)):
        current = arr[i]
        remaining = arr[:i] + arr[i+1:]

        for perm in permutations(remaining):
            result.append([current] + perm)

    return result

# Time: O(n!)
# Space: O(n!)
```

---

## 📦 Space Complexity Analysis

### O(1) - Constant Space
**Only uses fixed amount of extra space**

```javascript
// O(1) space
function sumArray(arr) {
  let sum = 0; // Only one variable

  for (const num of arr) {
    sum += num;
  }

  return sum;
}

// Time: O(n)
// Space: O(1) - only 'sum' variable
```

---

### O(n) - Linear Space
**Space grows with input size**

```javascript
// O(n) space - Creating new array
function doubleValues(arr) {
  const result = []; // New array of size n

  for (const num of arr) {
    result.push(num * 2);
  }

  return result;
}

// Time: O(n)
// Space: O(n) - result array
```

**Recursion Stack Space:**
```javascript
// O(n) space - Recursion depth
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

// Time: O(n)
// Space: O(n) - call stack stores n function calls
```

---

### O(log n) - Logarithmic Space
**Space for balanced tree or divide-and-conquer**

```javascript
// O(log n) space - Binary search (recursive)
function binarySearchRecursive(arr, target, left = 0, right = arr.length - 1) {
  if (left > right) return -1;

  const mid = Math.floor((left + right) / 2);

  if (arr[mid] === target) return mid;
  if (arr[mid] < target) {
    return binarySearchRecursive(arr, target, mid + 1, right);
  } else {
    return binarySearchRecursive(arr, target, left, mid - 1);
  }
}

// Time: O(log n)
// Space: O(log n) - recursion depth is log n
```

---

## 🎯 Analyzing Complexity: Step-by-Step

### Step 1: Identify Operations
Count how many times key operations execute relative to input size.

```javascript
function example(arr) {
  let sum = 0;                    // O(1) - single operation

  for (let i = 0; i < arr.length; i++) {   // O(n) - loops n times
    sum += arr[i];                // O(1) per iteration
  }

  return sum;                     // O(1) - single operation
}

// Total Time: O(1) + O(n) × O(1) + O(1) = O(n)
// Space: O(1) - only 'sum' variable
```

### Step 2: Count Nested Loops
Multiply complexities of nested operations.

```javascript
function example(arr) {
  for (let i = 0; i < arr.length; i++) {        // O(n)
    for (let j = 0; j < arr.length; j++) {      // O(n)
      console.log(arr[i], arr[j]);              // O(1)
    }
  }
}

// Time: O(n) × O(n) × O(1) = O(n²)
```

### Step 3: Sequential Operations
Add complexities of sequential operations.

```javascript
function example(arr) {
  // First loop
  for (let i = 0; i < arr.length; i++) {     // O(n)
    console.log(arr[i]);
  }

  // Second loop
  for (let i = 0; i < arr.length; i++) {     // O(n)
    console.log(arr[i] * 2);
  }

  // Nested loops
  for (let i = 0; i < arr.length; i++) {     // O(n)
    for (let j = 0; j < arr.length; j++) {   // O(n)
      console.log(arr[i] + arr[j]);
    }
  }
}

// Time: O(n) + O(n) + O(n²) = O(n²)
// Drop lower-order terms, keep dominant term
```

### Step 4: Drop Constants and Lower Terms
Big O cares about growth rate, not exact count.

```javascript
// O(3n² + 5n + 10) → O(n²)
// Drop constants: 3, 5, 10
// Drop lower terms: n compared to n²
// Keep dominant term: n²
```

---

## 📊 Complexity Comparison

### Growth Rates (n = 1000)

| Complexity | Operations | Example |
|------------|-----------|---------|
| O(1) | 1 | Hash lookup |
| O(log n) | ~10 | Binary search |
| O(n) | 1,000 | Linear search |
| O(n log n) | ~10,000 | Merge sort |
| O(n²) | 1,000,000 | Bubble sort |
| O(2ⁿ) | 10³⁰⁰+ | Naive Fibonacci |
| O(n!) | Astronomical | All permutations |

**For n = 1,000,000:**
- O(log n): ~20 operations ✅ Very fast
- O(n): 1,000,000 operations ✅ Fast
- O(n log n): ~20,000,000 operations ✅ Acceptable
- O(n²): 1,000,000,000,000 operations ❌ Too slow

---

## 🔍 Common Interview Scenarios

### Scenario 1: Two Sum Problem

**Brute Force - O(n²):**
```javascript
function twoSum(nums, target) {
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) {
        return [i, j];
      }
    }
  }
  return [];
}

// Time: O(n²) - nested loops
// Space: O(1) - no extra space
```

**Optimized - O(n):**
```javascript
function twoSum(nums, target) {
  const map = new Map();

  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];

    if (map.has(complement)) {
      return [map.get(complement), i];
    }

    map.set(nums[i], i);
  }

  return [];
}

// Time: O(n) - single pass
// Space: O(n) - hash map storage
// Trade-off: Use more space to get better time
```

---

### Scenario 2: Finding Duplicates

**Brute Force - O(n²):**
```javascript
function containsDuplicate(nums) {
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] === nums[j]) return true;
    }
  }
  return false;
}

// Time: O(n²)
// Space: O(1)
```

**Sort First - O(n log n):**
```javascript
function containsDuplicate(nums) {
  nums.sort((a, b) => a - b);

  for (let i = 1; i < nums.length; i++) {
    if (nums[i] === nums[i - 1]) return true;
  }

  return false;
}

// Time: O(n log n) - sorting dominates
// Space: O(1) or O(n) depending on sort implementation
```

**Hash Set - O(n):**
```javascript
function containsDuplicate(nums) {
  const seen = new Set();

  for (const num of nums) {
    if (seen.has(num)) return true;
    seen.add(num);
  }

  return false;
}

// Time: O(n) - single pass
// Space: O(n) - set storage
```

---

## 💡 Complexity Optimization Strategies

### 1. Use Hash Maps/Sets
**Trade space for time**
- O(n²) → O(n) for lookup-heavy operations
- Example: Two Sum, finding duplicates

### 2. Sort First
**Preprocessing for efficiency**
- Enables binary search: O(n) → O(log n)
- Enables two pointers: O(n²) → O(n)
- Cost: O(n log n) sort upfront

### 3. Two Pointers
**Reduce nested loops**
- O(n²) → O(n) for array problems
- Example: Pair sum in sorted array, container with most water

### 4. Sliding Window
**Avoid recalculating**
- O(n × k) → O(n) for subarray problems
- Example: Max sum subarray of size k

### 5. Divide and Conquer
**Break problem into smaller parts**
- Often achieves O(n log n)
- Example: Merge sort, quick sort

### 6. Dynamic Programming
**Avoid redundant calculations**
- O(2ⁿ) → O(n²) or O(n) with memoization
- Example: Fibonacci, climbing stairs

---

## 🎤 Interview Communication

### Always State Complexity

**Bad:**
```
"This solution works fine."
```

**Good:**
```
"This solution runs in O(n) time and uses O(n) space because we iterate
through the array once and store up to n elements in the hash map."
```

### Explain Trade-offs

**Example:**
```
"I have two approaches:

Approach 1: Sort then search
- Time: O(n log n) for sort + O(log n) for search = O(n log n)
- Space: O(1) if we can modify the input

Approach 2: Hash set
- Time: O(n) for single pass
- Space: O(n) for set storage

For large datasets where time is critical, I'd use approach 2. If memory
is constrained, approach 1 is better."
```

### Discuss Optimization

**Example:**
```
"My initial solution is O(n²) with nested loops. I can optimize this to
O(n) by using a hash map to store seen values, allowing O(1) lookup
instead of O(n) inner loop. The trade-off is using O(n) extra space for
the map, which is acceptable for most use cases."
```

---

## 📚 Practice Problems

### Easy
1. **Two Sum** - Optimize from O(n²) to O(n)
2. **Valid Anagram** - Compare different approaches
3. **Contains Duplicate** - Hash set vs sorting

### Medium
4. **3Sum** - Reduce from O(n³) to O(n²)
5. **Container With Most Water** - Two pointers O(n)
6. **Longest Substring Without Repeating** - Sliding window O(n)

### Hard
7. **Median of Two Sorted Arrays** - Achieve O(log(m+n))
8. **Trapping Rain Water** - O(n) with O(1) space
9. **Longest Increasing Subsequence** - O(n log n) vs O(n²)

---

## ✅ Complexity Cheat Sheet

### Time Complexity Order (Best to Worst)
```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)
```

### Common Data Structure Operations

| Operation | Array | Hash Map | Binary Search Tree | Heap |
|-----------|-------|----------|-------------------|------|
| Access | O(1) | - | O(log n) | - |
| Search | O(n) | O(1) avg | O(log n) avg | O(n) |
| Insert | O(n) | O(1) avg | O(log n) avg | O(log n) |
| Delete | O(n) | O(1) avg | O(log n) avg | O(log n) |

### Common Algorithm Complexities

| Algorithm | Time (Avg) | Time (Worst) | Space |
|-----------|------------|--------------|-------|
| Bubble Sort | O(n²) | O(n²) | O(1) |
| Merge Sort | O(n log n) | O(n log n) | O(n) |
| Quick Sort | O(n log n) | O(n²) | O(log n) |
| Binary Search | O(log n) | O(log n) | O(1) |
| DFS/BFS | O(V+E) | O(V+E) | O(V) |

---

## 🎯 Key Takeaways

1. **Always analyze both time AND space complexity**
2. **State complexity in interviews before coding**
3. **Explain trade-offs between different approaches**
4. **Optimize from brute force → better complexity**
5. **Common optimization: Trade space for time (hash maps)**
6. **Drop constants and lower-order terms in Big O**
7. **Focus on worst-case unless specified otherwise**

---

## 🔗 Related Patterns

- [Prefix Sum](./02-prefix-sum.md) - O(n) preprocessing for O(1) queries
- [Two Pointers](./03-two-pointers.md) - O(n) instead of O(n²)
- [Sliding Window](./04-sliding-window.md) - O(n) for subarray problems
- [Modified Binary Search](./10-modified-binary-search.md) - O(log n) search

---

**Remember:** In interviews, explaining complexity is as important as writing correct code! 🎯
