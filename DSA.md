# Constraint Trick in DSA (Complete Guide)

##  What is Constraint Trick?

The **constraint trick** means choosing the correct algorithm based on the input size and limits given in the problem.

👉 Instead of guessing the approach, constraints guide you to the optimal solution.

---

## ⏱️ Time Complexity vs Input Size

| Input Size (n) | Allowed Complexity | Techniques                         |
| -------------- | ------------------ | ---------------------------------- |
| n ≤ 10         | O(n!)              | Permutations, Backtracking         |
| n ≤ 20         | O(2ⁿ)              | Bitmasking, Subsets                |
| n ≤ 10³        | O(n³)              | Triple loops, Floyd-Warshall       |
| n ≤ 10⁵        | O(n log n)         | Sorting, Binary Search             |
| n ≤ 10⁶        | O(n)               | Linear Scan, Prefix Sum            |
| n ≥ 10⁷        | O(n) (tight)       | Highly optimized code              |
| n ≥ 10⁹        | O(1) / O(log n)    | Math, Formula, Fast Exponentiation |

---

## 🧠 Technique Selection by Constraints

### 🔹 1. Very Small Input (n ≤ 20)

* Recursion / Backtracking
* Bitmasking
* Generate all subsets

---

### 🔹 2. Small-Medium Input (n ≤ 1000)

* Nested loops
* Dynamic Programming (O(n²) / O(n³))
* Graph algorithms

---

### 🔹 3. Medium-Large Input (n ≤ 10⁵)

* Hashing (unordered_map / set)
* Sliding Window
* Two Pointers
* Sorting + Binary Search
* Greedy

---

### 🔹 4. Very Large Input (n ≥ 10⁶)

* Single pass solutions
* Prefix Sum
* Difference Array
* Fast I/O

---

### 🔹 5. Huge Input (n ≥ 10⁹)

* Mathematical formulas
* Logarithmic solutions
* Avoid loops

---

## 🔥 Common Patterns Based on Constraints

| Clue in Problem       | Use This Approach            |
| --------------------- | ---------------------------- |
| "All subsets"         | Bitmasking                   |
| "n ≤ 20"              | Backtracking                 |
| "n ≤ 10⁵"             | O(n log n)                   |
| "Multiple queries"    | Prefix Sum / Segment Tree    |
| "Range queries"       | Fenwick Tree / Segment Tree  |
| "Values small (≤100)" | Frequency Array              |
| "Sorted input"        | Binary Search / Two Pointers |
| "Continuous subarray" | Sliding Window               |

---

## ⚡ Optimization Mapping

| Brute Force   | Optimized Approach              |
| ------------- | ------------------------------- |
| O(n²)         | O(n log n) (sorting)            |
| O(n²)         | O(n) (hashing)                  |
| O(2ⁿ)         | O(n) / O(n log n) (DP / Greedy) |
| Repeated calc | Prefix Sum                      |

---

## 🧩 Example

### Problem: Find pair with sum = K

#### Case 1: n ≤ 100

```cpp
// O(n^2)
for(int i = 0; i < n; i++)
  for(int j = i+1; j < n; j++)
    if(arr[i] + arr[j] == k)
      return true;
```

#### Case 2: n ≤ 10^5

```cpp
// O(n)
unordered_set<int> s;
for(int x : arr) {
  if(s.count(k - x)) return true;
  s.insert(x);
}
```

---

## 🎯 Key Insight

👉 Constraints are hidden hints to the solution.

* Large n → avoid nested loops
* Small n → brute force allowed
* Huge n → think math

---

## ⚠️ Common Mistakes

* Ignoring constraints
* Using O(n²) when n = 10⁵
* Not estimating time complexity
* Over-optimizing small inputs

---

## 🧠 Golden Rule

> Always check constraints before coding.

---

## 🏁 Summary

* Constraints decide algorithm
* Match input size → complexity
* Eliminate slow approaches early
* Practice pattern recognition

---

## 💡 Pro Tip

👉 In interviews, always say:
"I’ll first analyze constraints to decide the optimal approach."


