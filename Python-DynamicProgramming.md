# DSA Interview Notes — Python
> Complete reference for Arrays & Strings, Linked Lists, Stacks & Queues, Hashing & Heaps

---

# Table of Contents

1. [Arrays & Strings](#1-arrays--strings)
   - [Array Insert / Delete](#11-array-insert--delete)
   - [Array Reversal](#12-array-reversal)
   - [2D Arrays](#13-2d-arrays-matrices)
   - [Pattern Matching](#14-pattern-matching)
   - [Array Rotation](#15-array-rotation)
   - [Palindrome Check](#16-palindrome-check)
2. [Linked Lists](#2-linked-lists)
   - [Singly Linked List](#21-singly-linked-list)
   - [Doubly Linked List](#22-doubly-linked-list)
   - [Cycle Detection](#23-cycle-detection)
   - [List Reversal](#24-list-reversal)
   - [Fast / Slow Pointers](#25-fast--slow-pointers)
   - [Merge Lists](#26-merge-lists)
3. [Stacks & Queues](#3-stacks--queues)
   - [Stack Implementation](#31-stack-implementation)
   - [Queue Implementation](#32-queue-implementation)
   - [Parentheses Matching](#33-parentheses-matching)
   - [Expression Evaluation](#34-expression-evaluation)
   - [Next Greater Element](#35-next-greater-element)
   - [Circular Queue](#36-circular-queue)
4. [Hashing & Heaps](#4-hashing--heaps)
   - [HashMap / HashSet](#41-hashmap--hashset)
   - [Hash-Based Problems](#42-coding-hash-based-problems)
   - [Min Heap / Max Heap](#43-min-heap--max-heap)
   - [Priority Queue](#44-priority-queue)
   - [Heapify Operations](#45-heapify-operations)
   - [Heap Sort](#46-heap-sort)

---

# 1. Arrays & Strings

---

## 1.1 Array Insert / Delete

### Key Concepts
- Python lists are **dynamic arrays** (amortized O(1) append, O(n) insert/delete at arbitrary index)
- Underlying array doubles in size when capacity exceeded → **amortized O(1)** for append
- Insert/delete at index i → shifts all elements after i → **O(n)**

### Interview Details
- Always clarify: insert at index, at end, in sorted order?
- Deletion by **value** vs deletion by **index** — different operations
- Watch for **index out of bounds** edge cases
- In-place vs returning new array

### Time/Space Complexity

| Operation | Time | Space |
|---|---|---|
| Insert at end | O(1) amortized | O(1) |
| Insert at index i | O(n) | O(1) |
| Delete by index | O(n) | O(1) |
| Delete by value | O(n) | O(1) |

```python
# ---- INSERT ----
arr = [1, 2, 4, 5]

arr.append(6)           # Insert at end → [1, 2, 4, 5, 6]
arr.insert(2, 3)        # Insert at index 2 → [1, 2, 3, 4, 5, 6]

# Manual insert without built-ins (common in interviews)
def insert_at(arr, index, val):
    arr.append(None)                    # Extend length
    for i in range(len(arr)-1, index, -1):
        arr[i] = arr[i-1]              # Shift right
    arr[index] = val
    return arr

# ---- DELETE ----
arr = [1, 2, 3, 4, 5]

arr.pop()               # Remove last → O(1)
arr.pop(2)              # Remove at index 2 → O(n)
arr.remove(3)           # Remove first occurrence of value 3 → O(n)

# Manual delete without built-ins
def delete_at(arr, index):
    for i in range(index, len(arr)-1):
        arr[i] = arr[i+1]              # Shift left
    arr.pop()                           # Remove last (now duplicate)
    return arr

# ---- Edge cases to mention ----
arr = []
# arr.pop()     → IndexError
# arr.remove(x) → ValueError if x not in arr
```

---

## 1.2 Array Reversal

### Key Concepts
- **Two-pointer technique** — swap left and right, move inward
- Python has `arr[::-1]` (returns new) and `arr.reverse()` (in-place)
- Interviews often ask for **in-place** reversal with O(1) extra space

### Interview Details
- Clarify: in-place vs new array
- Reversing a **subarray** (between indices l and r) — very common variant
- Reversing **words in a sentence** = split → reverse list → join
- Rotation problems often reduce to **three reversals**

```python
# ---- Basic reversal ----
arr = [1, 2, 3, 4, 5]

# Pythonic (new array)
rev = arr[::-1]               # [5, 4, 3, 2, 1]

# Built-in in-place
arr.reverse()                  # Modifies arr directly

# Two-pointer (interview preferred)
def reverse_array(arr):
    left, right = 0, len(arr) - 1
    while left < right:
        arr[left], arr[right] = arr[right], arr[left]
        left += 1
        right -= 1
    return arr

# ---- Reverse a subarray [l, r] ----
def reverse_subarray(arr, l, r):
    while l < r:
        arr[l], arr[r] = arr[r], arr[l]
        l += 1
        r -= 1
    return arr

arr = [1, 2, 3, 4, 5]
reverse_subarray(arr, 1, 3)   # [1, 4, 3, 2, 5]

# ---- Reverse words in a sentence ----
s = "hello world foo"
result = " ".join(s.split()[::-1])   # "foo world hello"

# ---- Recursive reversal ----
def reverse_recursive(arr, l=0, r=None):
    if r is None:
        r = len(arr) - 1
    if l >= r:
        return arr
    arr[l], arr[r] = arr[r], arr[l]
    return reverse_recursive(arr, l+1, r-1)
```

---

## 1.3 2D Arrays (Matrices)

### Key Concepts
- Represented as **list of lists** in Python
- Row-major order: `matrix[row][col]`
- Common operations: transpose, rotate 90°, spiral traversal, search
- **In-place rotation** = transpose + reverse rows (or reverse cols + transpose)

### Interview Details
- Always confirm: is it square (n×n) or rectangular (m×n)?
- Off-by-one errors at boundaries — very common bug
- Layer-by-layer thinking for rotation
- BFS/DFS on 2D grids → treat as graph with 4 or 8 neighbors

```python
# ---- Create & access ----
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
rows, cols = len(matrix), len(matrix[0])
print(matrix[1][2])   # 6  → row 1, col 2

# ---- Transpose (swap rows and cols) ----
def transpose(matrix):
    n = len(matrix)
    for i in range(n):
        for j in range(i+1, n):             # Only upper triangle
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
    return matrix

# Pythonic transpose
transposed = [list(row) for row in zip(*matrix)]

# ---- Rotate 90° clockwise (in-place) ----
# Step 1: Transpose
# Step 2: Reverse each row
def rotate_90_clockwise(matrix):
    n = len(matrix)
    # Transpose
    for i in range(n):
        for j in range(i+1, n):
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
    # Reverse each row
    for row in matrix:
        row.reverse()
    return matrix

# Rotate 90° counter-clockwise = reverse each row, then transpose

# ---- Spiral order traversal ----
def spiral_order(matrix):
    result = []
    top, bottom = 0, len(matrix) - 1
    left, right = 0, len(matrix[0]) - 1

    while top <= bottom and left <= right:
        for col in range(left, right + 1):       # Left → Right
            result.append(matrix[top][col])
        top += 1
        for row in range(top, bottom + 1):       # Top → Bottom
            result.append(matrix[row][right])
        right -= 1
        if top <= bottom:
            for col in range(right, left - 1, -1):   # Right → Left
                result.append(matrix[bottom][col])
            bottom -= 1
        if left <= right:
            for row in range(bottom, top - 1, -1):   # Bottom → Top
                result.append(matrix[row][left])
            left += 1
    return result

# ---- Search in row-sorted, col-sorted matrix ----
# Start at top-right corner: O(m+n)
def search_matrix(matrix, target):
    row, col = 0, len(matrix[0]) - 1
    while row < len(matrix) and col >= 0:
        if matrix[row][col] == target:
            return (row, col)
        elif matrix[row][col] > target:
            col -= 1
        else:
            row += 1
    return (-1, -1)

# ---- 4-directional neighbors (BFS/DFS on grid) ----
def get_neighbors(r, c, rows, cols):
    directions = [(0,1),(0,-1),(1,0),(-1,0)]
    return [(r+dr, c+dc) for dr,dc in directions
            if 0 <= r+dr < rows and 0 <= c+dc < cols]
```

---

## 1.4 Pattern Matching

### Key Concepts
- **Naive approach**: O(n×m) — slide pattern over text, compare char by char
- **KMP (Knuth-Morris-Pratt)**: O(n+m) — uses failure function (prefix table)
- **Rabin-Karp**: O(n+m) average — rolling hash
- Python's `in` operator and `str.find()` use optimized C — mention but code naive/KMP in interviews

### Interview Details
- KMP is the gold standard for interviews asking O(n+m)
- Know how to build the **LPS (Longest Proper Prefix which is also Suffix)** array
- Rabin-Karp useful when asked about hashing approach
- Wildcard / regex matching → DP approach

```python
# ---- Naive O(n*m) ----
def naive_search(text, pattern):
    n, m = len(text), len(pattern)
    matches = []
    for i in range(n - m + 1):
        if text[i:i+m] == pattern:
            matches.append(i)
    return matches

# ---- KMP O(n+m) ----
def build_lps(pattern):
    """
    LPS[i] = length of longest proper prefix of pattern[0..i]
             that is also a suffix
    """
    m = len(pattern)
    lps = [0] * m
    length = 0    # length of previous longest prefix-suffix
    i = 1
    while i < m:
        if pattern[i] == pattern[length]:
            length += 1
            lps[i] = length
            i += 1
        else:
            if length != 0:
                length = lps[length - 1]   # Fallback — key insight
            else:
                lps[i] = 0
                i += 1
    return lps

def kmp_search(text, pattern):
    n, m = len(text), len(pattern)
    lps = build_lps(pattern)
    matches = []
    i = j = 0                # i → text, j → pattern
    while i < n:
        if text[i] == pattern[j]:
            i += 1
            j += 1
        if j == m:
            matches.append(i - j)
            j = lps[j - 1]              # Continue with fallback
        elif i < n and text[i] != pattern[j]:
            if j != 0:
                j = lps[j - 1]
            else:
                i += 1
    return matches

# Example
text = "aabaacaadaabaaba"
pattern = "aaba"
print(kmp_search(text, pattern))   # [0, 9, 12]

# ---- Rabin-Karp (rolling hash) ----
def rabin_karp(text, pattern, base=256, mod=10**9+7):
    n, m = len(text), len(pattern)
    if m > n:
        return []
    h_pat = h_txt = 0
    high = pow(base, m-1, mod)        # base^(m-1) for rolling
    matches = []

    for i in range(m):                # Initial hash
        h_pat = (base * h_pat + ord(pattern[i])) % mod
        h_txt = (base * h_txt + ord(text[i])) % mod

    for i in range(n - m + 1):
        if h_pat == h_txt:
            if text[i:i+m] == pattern:   # Verify (handle collision)
                matches.append(i)
        if i < n - m:
            h_txt = (base*(h_txt - ord(text[i])*high) + ord(text[i+m])) % mod

    return matches

# ---- Wildcard matching with DP ----
# '?' matches single char, '*' matches any sequence
def wildcard_match(s, p):
    m, n = len(s), len(p)
    dp = [[False]*(n+1) for _ in range(m+1)]
    dp[0][0] = True
    for j in range(1, n+1):
        if p[j-1] == '*':
            dp[0][j] = dp[0][j-1]
    for i in range(1, m+1):
        for j in range(1, n+1):
            if p[j-1] == '*':
                dp[i][j] = dp[i-1][j] or dp[i][j-1]
            elif p[j-1] == '?' or p[j-1] == s[i-1]:
                dp[i][j] = dp[i-1][j-1]
    return dp[m][n]
```

---

## 1.5 Array Rotation

### Key Concepts
- **Left rotation by k**: first k elements move to end
- **Right rotation by k**: last k elements move to front
- Most efficient: **Three-reversal algorithm** — O(n) time, O(1) space
- Alternative: Extra array copy — O(n) time, O(n) space

### Interview Details
- Normalize k: `k = k % n` (rotating by n = no change)
- Clarify direction: left or right?
- Three-reversal is the expected optimal answer
- Rotation is related to **circular array** problems

```python
# ---- Three-Reversal Algorithm ----
# Right rotate by k:
#   1. Reverse entire array
#   2. Reverse first k elements
#   3. Reverse remaining n-k elements

def reverse(arr, l, r):
    while l < r:
        arr[l], arr[r] = arr[r], arr[l]
        l += 1; r -= 1

def rotate_right(arr, k):
    n = len(arr)
    k = k % n                  # Handle k >= n
    reverse(arr, 0, n-1)       # Reverse all
    reverse(arr, 0, k-1)       # Reverse first k
    reverse(arr, k, n-1)       # Reverse rest
    return arr

def rotate_left(arr, k):
    n = len(arr)
    k = k % n
    reverse(arr, 0, k-1)       # Reverse first k
    reverse(arr, k, n-1)       # Reverse rest
    reverse(arr, 0, n-1)       # Reverse all
    return arr

arr = [1, 2, 3, 4, 5, 6, 7]
print(rotate_right(arr[:], 3))   # [5, 6, 7, 1, 2, 3, 4]
print(rotate_left(arr[:], 3))    # [4, 5, 6, 7, 1, 2, 3]

# ---- Slice approach (O(n) space) ----
def rotate_right_slice(arr, k):
    n = len(arr)
    k = k % n
    return arr[-k:] + arr[:-k]

def rotate_left_slice(arr, k):
    n = len(arr)
    k = k % n
    return arr[k:] + arr[:k]

# ---- Check if one array is rotation of another ----
def is_rotation(a, b):
    if len(a) != len(b):
        return False
    doubled = a + a
    return is_subarray(doubled, b)

def is_subarray(text, pattern):
    n, m = len(text), len(pattern)
    for i in range(n - m + 1):
        if text[i:i+m] == pattern:
            return True
    return False

# ---- Find minimum in rotated sorted array ----
# Binary search approach: O(log n)
def find_min_rotated(arr):
    lo, hi = 0, len(arr) - 1
    while lo < hi:
        mid = (lo + hi) // 2
        if arr[mid] > arr[hi]:
            lo = mid + 1          # Min is in right half
        else:
            hi = mid              # Min is in left half (including mid)
    return arr[lo]
```

---

## 1.6 Palindrome Check

### Key Concepts
- **String**: equal to its reverse; use two pointers from ends inward
- **Number**: convert to string OR reverse digits mathematically
- Ignore non-alphanumeric + case-insensitive for "valid palindrome" variants
- **Longest palindromic substring** → expand-around-center: O(n²) / Manacher's: O(n)

### Interview Details
- Two-pointer is O(n) time O(1) space — always preferred over slicing
- Watch for: empty string (palindrome), single char (palindrome), even vs odd length
- Manacher's algorithm rarely asked to code, but good to mention it exists
- **Palindrome partitioning** and **count palindromic substrings** use DP

```python
# ---- Basic string palindrome ----
def is_palindrome(s):
    left, right = 0, len(s) - 1
    while left < right:
        if s[left] != s[right]:
            return False
        left += 1
        right -= 1
    return True

# Pythonic (O(n) space — new string created)
def is_palindrome_pythonic(s):
    return s == s[::-1]

# ---- Valid palindrome (alphanumeric only, case-insensitive) ----
# LeetCode #125 style
def is_valid_palindrome(s):
    left, right = 0, len(s) - 1
    while left < right:
        while left < right and not s[left].isalnum():
            left += 1
        while left < right and not s[right].isalnum():
            right -= 1
        if s[left].lower() != s[right].lower():
            return False
        left += 1
        right -= 1
    return True

print(is_valid_palindrome("A man, a plan, a canal: Panama"))  # True

# ---- Number palindrome (no string conversion) ----
def is_palindrome_number(x):
    if x < 0 or (x % 10 == 0 and x != 0):
        return False
    reversed_half = 0
    while x > reversed_half:
        reversed_half = reversed_half * 10 + x % 10
        x //= 10
    return x == reversed_half or x == reversed_half // 10

# ---- Longest Palindromic Substring (expand around center) ----
def longest_palindrome(s):
    def expand(l, r):
        while l >= 0 and r < len(s) and s[l] == s[r]:
            l -= 1
            r += 1
        return s[l+1:r]          # Slice after loop exits

    result = ""
    for i in range(len(s)):
        odd  = expand(i, i)       # Odd length center
        even = expand(i, i+1)     # Even length center
        if len(odd)  > len(result): result = odd
        if len(even) > len(result): result = even
    return result

print(longest_palindrome("babad"))   # "bab" or "aba"
print(longest_palindrome("cbbd"))    # "bb"

# ---- Count palindromic substrings ----
def count_palindromes(s):
    count = 0
    def expand(l, r):
        nonlocal count
        while l >= 0 and r < len(s) and s[l] == s[r]:
            count += 1
            l -= 1; r += 1
    for i in range(len(s)):
        expand(i, i)         # Odd
        expand(i, i+1)       # Even
    return count

print(count_palindromes("aaa"))   # 6

# ---- Palindrome DP (for partitioning problems) ----
def build_palindrome_dp(s):
    n = len(s)
    dp = [[False]*n for _ in range(n)]
    for i in range(n):
        dp[i][i] = True                  # Single chars
    for i in range(n-1):
        dp[i][i+1] = (s[i] == s[i+1])   # Pairs
    for length in range(3, n+1):
        for i in range(n - length + 1):
            j = i + length - 1
            dp[i][j] = (s[i] == s[j]) and dp[i+1][j-1]
    return dp

# dp[i][j] = True means s[i..j] is a palindrome
```

### Arrays & Strings — Quick Complexity Cheatsheet

| Problem | Best Algorithm | Time | Space |
|---|---|---|---|
| Insert/Delete | Two-pointer / shift | O(n) | O(1) |
| Reversal | Two-pointer | O(n) | O(1) |
| 2D Rotation | Transpose + Reverse | O(n²) | O(1) |
| Pattern Matching | KMP | O(n+m) | O(m) |
| Array Rotation | Three-Reversal | O(n) | O(1) |
| Palindrome Check | Two-pointer | O(n) | O(1) |
| Longest Palindrome | Expand Center | O(n²) | O(1) |

### Arrays & Strings — Common Interview Traps & Tips

- **k % n** before any rotation — always normalize
- **Off-by-one** in 2D matrix bounds — draw it out first
- Palindrome: **even vs odd** length centers — two separate expand calls
- KMP LPS array — the fallback `length = lps[length-1]` line is what most candidates miss
- For 2D grid problems, always write the bounds check helper first
- Pattern matching: Python's `in` is O(n×m) worst case in CPython — always mention KMP as the theoretically optimal solution

---

# 2. Linked Lists

---

## Core Node Definitions

```python
# Singly linked list node
class ListNode:
    def __init__(self, val=0, next=None):
        self.val  = val
        self.next = next

# Doubly linked list node
class DListNode:
    def __init__(self, val=0, prev=None, next=None):
        self.val  = val
        self.prev = prev
        self.next = next

# Helper: build linked list from array
def build_list(arr):
    if not arr: return None
    head = ListNode(arr[0])
    cur  = head
    for val in arr[1:]:
        cur.next = ListNode(val)
        cur = cur.next
    return head

# Helper: convert linked list to array (for testing)
def to_array(head):
    result = []
    while head:
        result.append(head.val)
        head = head.next
    return result
```

---

## 2.1 Singly Linked List

### Key Concepts
- Each node holds **val + next pointer**
- No random access — must traverse from head: **O(n)**
- Insertion/deletion at known node: **O(1)** (once you have the pointer)
- Head pointer is the **only entry point** — never lose it

### Interview Details
- Always handle: **empty list**, **single node**, **head/tail operations**
- Most bugs come from losing the head reference or null pointer dereference
- **Dummy head node** pattern eliminates edge cases for head deletion/insertion
- Draw the pointer manipulation before coding

### Time/Space Complexity

| Operation | Time | Space |
|---|---|---|
| Access by index | O(n) | O(1) |
| Insert at head | O(1) | O(1) |
| Insert at tail | O(n) | O(1) |
| Insert at index | O(n) | O(1) |
| Delete by value | O(n) | O(1) |
| Search | O(n) | O(1) |

```python
class SinglyLinkedList:
    def __init__(self):
        self.head = None
        self.size = 0

    # ---- Insert ----
    def insert_at_head(self, val):
        node = ListNode(val)
        node.next = self.head
        self.head = node
        self.size += 1

    def insert_at_tail(self, val):
        node = ListNode(val)
        if not self.head:
            self.head = node
            self.size += 1
            return
        cur = self.head
        while cur.next:             # Walk to last node
            cur = cur.next
        cur.next = node
        self.size += 1

    def insert_at_index(self, index, val):
        if index < 0 or index > self.size:
            raise IndexError("Index out of bounds")
        if index == 0:
            self.insert_at_head(val)
            return
        dummy = ListNode(0)         # Dummy head simplifies edge cases
        dummy.next = self.head
        cur = dummy
        for _ in range(index):
            cur = cur.next
        node = ListNode(val)
        node.next = cur.next
        cur.next = node
        self.size += 1

    # ---- Delete ----
    def delete_by_value(self, val):
        if not self.head:
            return
        if self.head.val == val:    # Edge case: delete head
            self.head = self.head.next
            self.size -= 1
            return
        cur = self.head
        while cur.next:
            if cur.next.val == val:
                cur.next = cur.next.next    # Bypass the node
                self.size -= 1
                return
            cur = cur.next

    # Dummy head pattern — cleaner, handles head deletion uniformly
    def delete_by_value_dummy(self, val):
        dummy = ListNode(0)
        dummy.next = self.head
        cur = dummy
        while cur.next:
            if cur.next.val == val:
                cur.next = cur.next.next
                self.size -= 1
                break
            cur = cur.next
        self.head = dummy.next

    # ---- Search ----
    def search(self, val):
        cur, index = self.head, 0
        while cur:
            if cur.val == val:
                return index
            cur = cur.next
            index += 1
        return -1

    # ---- Get kth from end ----
    # Two-pointer: advance first pointer k steps, then move both
    def kth_from_end(self, k):
        fast = slow = self.head
        for _ in range(k):
            if not fast:
                return None          # k > length
            fast = fast.next
        while fast:
            fast = fast.next
            slow = slow.next
        return slow.val              # slow is now kth from end

# ---- Delete node given only that node (no head) ----
# Classic trick: copy next node's val, delete next node
def delete_node(node):
    # Assumption: node is NOT the tail
    node.val  = node.next.val
    node.next = node.next.next
```

---

## 2.2 Doubly Linked List

### Key Concepts
- Each node holds **val + prev + next pointers**
- Can traverse **both directions** — O(1) delete given a node pointer
- More memory per node (extra prev pointer)
- Basis for **LRU Cache** (dict + doubly linked list)

### Interview Details
- When inserting/deleting always update **both** prev and next pointers — most common bug
- Sentinel nodes (dummy head + dummy tail) eliminate ALL boundary conditions
- Know the LRU cache implementation — extremely common interview question

```python
class DoublyLinkedList:
    def __init__(self):
        # Sentinel nodes — never removed
        self.head = DListNode(0)     # Dummy head
        self.tail = DListNode(0)     # Dummy tail
        self.head.next = self.tail
        self.tail.prev = self.head
        self.size = 0

    def _insert_after(self, node, new_node):
        """Insert new_node immediately after node."""
        new_node.prev = node
        new_node.next = node.next
        node.next.prev = new_node    # Update successor's prev
        node.next = new_node         # Update node's next
        self.size += 1

    def _remove(self, node):
        """Remove an arbitrary node in O(1)."""
        node.prev.next = node.next
        node.next.prev = node.prev
        self.size -= 1

    def insert_at_front(self, val):
        self._insert_after(self.head, DListNode(val))

    def insert_at_back(self, val):
        self._insert_after(self.tail.prev, DListNode(val))

    def remove_front(self):
        if self.size == 0:
            return None
        node = self.head.next
        self._remove(node)
        return node.val

    def remove_back(self):
        if self.size == 0:
            return None
        node = self.tail.prev
        self._remove(node)
        return node.val

    def to_array(self):
        result = []
        cur = self.head.next
        while cur != self.tail:
            result.append(cur.val)
            cur = cur.next
        return result

# ---- LRU Cache (dict + doubly linked list) ----
# O(1) get and put
class LRUCache:
    def __init__(self, capacity):
        self.cap   = capacity
        self.cache = {}              # key → node
        self.head  = DListNode(0)    # Dummy: MRU side
        self.tail  = DListNode(0)    # Dummy: LRU side
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _insert_front(self, node):   # Insert as most recently used
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def get(self, key):
        if key not in self.cache:
            return -1
        node = self.cache[key]
        self._remove(node)
        self._insert_front(node)     # Mark as recently used
        return node.val

    def put(self, key, value):
        if key in self.cache:
            self._remove(self.cache[key])
        node = DListNode(value)
        node.key = key
        self.cache[key] = node
        self._insert_front(node)
        if len(self.cache) > self.cap:
            lru = self.tail.prev     # Least recently used
            self._remove(lru)
            del self.cache[lru.key]
```

---

## 2.3 Cycle Detection

### Key Concepts
- **Floyd's Tortoise and Hare** — two pointers, slow moves 1 step, fast moves 2
- If they meet → cycle exists
- To find **cycle start**: reset slow to head, advance both 1 step at a time — they meet at cycle entry
- **Proof**: if cycle length is C and pre-cycle length is F, when fast and slow meet inside cycle, slow has traveled F+a steps. Resetting slow to head and advancing both by 1 guarantees meeting at cycle start after F more steps

### Interview Details
- O(n) time, **O(1) space** — always mention this vs hash set O(n) space approach
- Hash set approach: store visited nodes, first revisit = cycle start (simpler, O(n) space)
- Always ask: just detect, or find the start node too?
- Edge cases: empty list, single node, cycle at head

```python
# ---- Detect cycle — Floyd's algorithm ----
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:            # Identity check, not equality
            return True
    return False

# ---- Find cycle start node ----
def detect_cycle_start(head):
    slow = fast = head

    # Phase 1: Detect meeting point inside cycle
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            break
    else:
        return None                  # No cycle

    # Phase 2: Find entry point
    slow = head                      # Reset slow to head
    while slow is not fast:          # Both move 1 step
        slow = slow.next
        fast = fast.next
    return slow                      # Cycle start node

# ---- Find cycle length ----
def cycle_length(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            # Count steps until they meet again
            count = 1
            fast = fast.next
            while slow is not fast:
                fast = fast.next
                count += 1
            return count
    return 0

# ---- Hash set approach (O(n) space, simpler) ----
def has_cycle_hashset(head):
    visited = set()
    cur = head
    while cur:
        if id(cur) in visited:
            return True
        visited.add(id(cur))
        cur = cur.next
    return False

# ---- Build a cycle for testing ----
def build_cycle(arr, pos):
    head = build_list(arr)
    if pos < 0:
        return head
    cycle_entry = head
    tail = head
    for _ in range(pos):
        cycle_entry = cycle_entry.next
    while tail.next:
        tail = tail.next
    tail.next = cycle_entry         # Create cycle
    return head
```

---

## 2.4 List Reversal

### Key Concepts
- **Iterative**: three pointers — prev, curr, next — O(n) time O(1) space
- **Recursive**: reverse rest of list, fix pointers on the way back — O(n) time O(n) stack space
- **Reverse sublist [l, r]**: reach position l-1, reverse k nodes, reconnect
- Group reverse (k-group) is a popular hard variant

### Interview Details
- Iterative is always preferred (O(1) space)
- Draw pointer arrows before coding — prevents bugs
- For sublist reversal: **track the tail of reversed section** for reconnection
- Don't forget to set the original head's next to None (or successor node)

```python
# ---- Iterative reversal — O(n) time, O(1) space ----
def reverse_list(head):
    prev = None
    curr = head
    while curr:
        next_node  = curr.next    # Save next
        curr.next  = prev         # Reverse pointer
        prev       = curr         # Advance prev
        curr       = next_node    # Advance curr
    return prev                   # New head

# ---- Recursive reversal — O(n) time, O(n) stack ----
def reverse_list_recursive(head):
    if not head or not head.next:
        return head
    new_head = reverse_list_recursive(head.next)  # Reverse rest
    head.next.next = head         # Make next node point back
    head.next = None              # Disconnect current
    return new_head

# ---- Reverse sublist from position l to r (1-indexed) ----
def reverse_between(head, l, r):
    if not head or l == r:
        return head

    dummy = ListNode(0)
    dummy.next = head
    prev = dummy

    for _ in range(l - 1):       # Move prev to node before l
        prev = prev.next

    curr = prev.next              # curr starts at position l
    for _ in range(r - l):
        next_node    = curr.next
        curr.next    = next_node.next
        next_node.next = prev.next
        prev.next    = next_node

    return dummy.next

# Example: 1→2→3→4→5, l=2, r=4 → 1→4→3→2→5

# ---- Reverse in k-groups ----
def reverse_k_group(head, k):
    # Check if k nodes remain
    count, node = 0, head
    while node and count < k:
        node = node.next
        count += 1
    if count < k:
        return head               # Not enough nodes, leave as-is

    # Reverse k nodes
    prev, curr = None, head
    for _ in range(k):
        nxt       = curr.next
        curr.next = prev
        prev      = curr
        curr      = nxt

    # head is now tail of reversed group; recurse for rest
    head.next = reverse_k_group(curr, k)
    return prev                   # New head of this group
```

---

## 2.5 Fast / Slow Pointers

### Key Concepts
- Also called **Floyd's two-pointer** or **hare & tortoise**
- Fast moves **2 steps**, slow moves **1 step** per iteration
- When fast reaches end, slow is at **middle**
- Used for: middle node, cycle detection, kth from end, palindrome list
- Can generalize: fast moves k steps per iteration for kth-from-end

### Interview Details
- Middle of even-length list: slow ends at **left-middle** by default; to get right-middle use `while fast.next` condition swap
- Extremely versatile — when you see "linked list + O(1) space", think fast/slow
- Palindrome linked list = find middle + reverse second half + compare

```python
# ---- Find middle node ----
def find_middle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow                   # Left-middle for even length

# 1→2→3→4→5 → returns node 3
# 1→2→3→4   → returns node 2  (left-middle)

def find_middle_right(head):     # Right-middle for even length
    slow = fast = head
    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next
    return slow

# ---- Kth node from end ----
def kth_from_end(head, k):
    fast = slow = head
    for _ in range(k):            # Advance fast by k steps
        if not fast:
            return None           # k > list length
        fast = fast.next
    while fast:                   # Move both until fast hits end
        fast = fast.next
        slow = slow.next
    return slow

# ---- Palindrome linked list — O(n) time, O(1) space ----
def is_palindrome_list(head):
    if not head or not head.next:
        return True

    # Step 1: Find middle
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    # Step 2: Reverse second half
    prev, curr = None, slow
    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt

    # Step 3: Compare both halves
    left, right = head, prev
    while right:                   # Right half is shorter or equal
        if left.val != right.val:
            return False
        left  = left.next
        right = right.next
    return True

# ---- Split list into two halves ----
def split_list(head):
    slow = fast = head
    prev = None
    while fast and fast.next:
        prev = slow
        slow = slow.next
        fast = fast.next.next
    if prev:
        prev.next = None          # Cut the list
    return head, slow             # (first_half, second_half)
```

---

## 2.6 Merge Lists

### Key Concepts
- **Merge two sorted lists**: compare heads, take smaller, advance that pointer — O(n+m)
- **Merge k sorted lists**: use a min-heap (priority queue) — O(N log k)
- **Divide and conquer** for k lists: merge pairs repeatedly — O(N log k)
- Always use **dummy head** to avoid special-casing the result head

### Interview Details
- Iterative merge is O(1) space vs recursive O(n) stack space
- For k lists: heap approach vs divide-and-conquer both O(N log k) — know both
- Merging is the backbone of **merge sort on linked lists**
- Edge cases: one or both lists empty, lists of different lengths

```python
import heapq

# ---- Merge two sorted lists — iterative ----
def merge_two_sorted(l1, l2):
    dummy = ListNode(0)
    cur = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            cur.next = l1
            l1 = l1.next
        else:
            cur.next = l2
            l2 = l2.next
        cur = cur.next
    cur.next = l1 if l1 else l2    # Attach remaining nodes
    return dummy.next

# ---- Merge two sorted lists — recursive ----
def merge_two_recursive(l1, l2):
    if not l1: return l2
    if not l2: return l1
    if l1.val <= l2.val:
        l1.next = merge_two_recursive(l1.next, l2)
        return l1
    else:
        l2.next = merge_two_recursive(l1, l2.next)
        return l2

# ---- Merge k sorted lists — min-heap ----
def merge_k_sorted(lists):
    heap = []
    for i, node in enumerate(lists):
        if node:
            # (val, index, node) — index breaks ties when vals equal
            heapq.heappush(heap, (node.val, i, node))

    dummy = cur = ListNode(0)
    while heap:
        val, i, node = heapq.heappop(heap)
        cur.next = node
        cur = cur.next
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))

    return dummy.next

# ---- Merge k sorted lists — divide & conquer ----
def merge_k_divide(lists):
    if not lists:
        return None
    while len(lists) > 1:
        merged = []
        for i in range(0, len(lists), 2):
            l1 = lists[i]
            l2 = lists[i+1] if i+1 < len(lists) else None
            merged.append(merge_two_sorted(l1, l2))
        lists = merged
    return lists[0]

# ---- Merge sort on linked list — O(n log n), O(1) space ----
def merge_sort_list(head):
    if not head or not head.next:
        return head

    # Split into halves using slow/fast pointers
    slow, fast = head, head.next
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    mid       = slow.next
    slow.next = None              # Cut list in half

    left  = merge_sort_list(head)
    right = merge_sort_list(mid)
    return merge_two_sorted(left, right)

# ---- Intersection of two linked lists ----
# Find the node where two lists converge — O(n+m), O(1) space
def get_intersection(headA, headB):
    a, b = headA, headB
    while a is not b:
        a = a.next if a else headB   # Switch to other list at end
        b = b.next if b else headA
    return a                         # None if no intersection
```

### Linked Lists — Quick Complexity Cheatsheet

| Operation | Algorithm | Time | Space |
|---|---|---|---|
| Insert / Delete at node | Pointer update | O(1) | O(1) |
| Search / Access | Linear scan | O(n) | O(1) |
| Reverse list | Iterative (3 ptrs) | O(n) | O(1) |
| Reverse list | Recursive | O(n) | O(n) |
| Cycle detection | Floyd's | O(n) | O(1) |
| Find middle | Fast/slow | O(n) | O(1) |
| Merge 2 sorted | Iterative | O(n+m) | O(1) |
| Merge k sorted | Min-heap | O(N log k) | O(k) |
| Merge sort list | Divide & conquer | O(n log n) | O(log n) |

### Linked Lists — Common Interview Traps & Tips

- **Never lose the head** — store it in a variable before any traversal
- **Dummy node** pattern — use it for any insert/delete that may affect head
- `slow is fast` not `slow == fast` — compare **identity**, not value
- Reversing: save `curr.next` before overwriting it — most common bug
- Fast/slow middle: `while fast and fast.next` gives **left-middle**; swap condition for right-middle
- Merge k lists: heap needs a **tiebreaker** (index) because `ListNode` is not comparable in Python
- Cycle start proof: F = pre-cycle length; meeting point inside cycle is at distance F from start — memorize this for interviews
- For LRU cache: **sentinel dummy nodes** at both ends make `_remove` and `_insert` unconditional — no null checks needed

---

# 3. Stacks & Queues

---

## Core Concepts Upfront

```python
# Python built-in options — know these cold
# Stack  → list (append/pop from right)
# Queue  → collections.deque (append/popleft)
# PQueue → heapq module

from collections import deque
import heapq

stack = []
stack.append(1)       # push  → O(1)
stack.pop()           # pop   → O(1)
stack[-1]             # peek  → O(1)

queue = deque()
queue.append(1)       # enqueue → O(1)
queue.popleft()       # dequeue → O(1)
queue[0]              # peek    → O(1)
```

---

## 3.1 Stack Implementation

### Key Concepts
- **LIFO** — Last In, First Out
- Core ops: `push`, `pop`, `peek`, `is_empty` — all **O(1)**
- Python `list` is a dynamic array — works perfectly as a stack
- Linked list implementation gives true O(1) with no resizing overhead
- **Monotonic stack** — stack that maintains increasing or decreasing order — critical interview pattern

### Interview Details
- Know array-based AND linked-list-based implementations
- **Min stack** (get minimum in O(1)) — very common interview question
- **Two-stack queue** — implement queue using two stacks
- Monotonic stack pattern: "next greater/smaller element" family of problems
- Always clarify: what happens on pop/peek of empty stack?

```python
# ---- Array-based stack ----
class ArrayStack:
    def __init__(self):
        self._data = []

    def push(self, val):
        self._data.append(val)          # O(1) amortized

    def pop(self):
        if self.is_empty():
            raise IndexError("Pop from empty stack")
        return self._data.pop()         # O(1)

    def peek(self):
        if self.is_empty():
            raise IndexError("Peek at empty stack")
        return self._data[-1]           # O(1)

    def is_empty(self):
        return len(self._data) == 0

    def size(self):
        return len(self._data)

# ---- Linked-list-based stack ----
class StackNode:
    def __init__(self, val, next=None):
        self.val  = val
        self.next = next

class LinkedStack:
    def __init__(self):
        self._top  = None
        self._size = 0

    def push(self, val):
        self._top  = StackNode(val, self._top)  # New node points to old top
        self._size += 1

    def pop(self):
        if not self._top:
            raise IndexError("Pop from empty stack")
        val       = self._top.val
        self._top = self._top.next              # Advance top
        self._size -= 1
        return val

    def peek(self):
        if not self._top:
            raise IndexError("Peek at empty stack")
        return self._top.val

    def is_empty(self):
        return self._top is None

# ---- Min Stack — O(1) get_min ----
# Key insight: store (val, current_min) pairs
class MinStack:
    def __init__(self):
        self._stack = []             # Stores (value, min_so_far) tuples

    def push(self, val):
        current_min = min(val, self._stack[-1][1]) if self._stack else val
        self._stack.append((val, current_min))

    def pop(self):
        if not self._stack:
            raise IndexError("Pop from empty stack")
        return self._stack.pop()[0]

    def peek(self):
        return self._stack[-1][0]

    def get_min(self):
        return self._stack[-1][1]    # Always O(1)

s = MinStack()
s.push(5); s.push(3); s.push(7); s.push(2)
print(s.get_min())   # 2
s.pop()
print(s.get_min())   # 3

# ---- Queue using two stacks ----
# Amortized O(1) enqueue and dequeue
class QueueViaStacks:
    def __init__(self):
        self._inbox  = []            # For enqueue
        self._outbox = []            # For dequeue

    def enqueue(self, val):
        self._inbox.append(val)      # Always O(1)

    def dequeue(self):
        if not self._outbox:         # Lazy transfer
            while self._inbox:
                self._outbox.append(self._inbox.pop())
        if not self._outbox:
            raise IndexError("Dequeue from empty queue")
        return self._outbox.pop()    # Amortized O(1)

    def peek(self):
        if not self._outbox:
            while self._inbox:
                self._outbox.append(self._inbox.pop())
        return self._outbox[-1]
```

---

## 3.2 Queue Implementation

### Key Concepts
- **FIFO** — First In, First Out
- Core ops: `enqueue`, `dequeue`, `peek`, `is_empty` — all **O(1)**
- Python `collections.deque` is a **doubly-ended queue** backed by doubly linked list — true O(1) both ends
- `list` as queue is **BAD** — `list.pop(0)` is O(n) due to shifting
- **Monotonic deque** — used in sliding window maximum problems

### Interview Details
- Always use `deque` not `list` for queue in Python
- Know **circular queue** (fixed capacity)
- **Stack using two queues** — mirror of two-stack queue
- Deque supports O(1) operations at both ends — useful for sliding window problems

```python
from collections import deque

# ---- Deque-based queue ----
class Queue:
    def __init__(self):
        self._data = deque()

    def enqueue(self, val):
        self._data.append(val)          # Add to right → O(1)

    def dequeue(self):
        if self.is_empty():
            raise IndexError("Dequeue from empty queue")
        return self._data.popleft()     # Remove from left → O(1)

    def peek(self):
        if self.is_empty():
            raise IndexError("Peek at empty queue")
        return self._data[0]

    def is_empty(self):
        return len(self._data) == 0

    def size(self):
        return len(self._data)

# ---- Stack using two queues ----
class StackViaQueues:
    def __init__(self):
        self._q1 = deque()
        self._q2 = deque()

    def push(self, val):
        self._q2.append(val)
        while self._q1:              # Drain q1 into q2
            self._q2.append(self._q1.popleft())
        self._q1, self._q2 = self._q2, self._q1   # Swap

    def pop(self):
        if not self._q1:
            raise IndexError("Pop from empty stack")
        return self._q1.popleft()

    def peek(self):
        return self._q1[0]

# ---- Sliding window maximum (monotonic deque) ----
def sliding_window_max(nums, k):
    dq     = deque()                 # Stores indices, decreasing values
    result = []

    for i, num in enumerate(nums):
        # Remove elements outside window
        while dq and dq[0] < i - k + 1:
            dq.popleft()
        # Maintain decreasing order — remove smaller elements from back
        while dq and nums[dq[-1]] < num:
            dq.pop()
        dq.append(i)
        if i >= k - 1:               # Window is full
            result.append(nums[dq[0]])
    return result

print(sliding_window_max([1,3,-1,-3,5,3,6,7], 3))
# [3, 3, 5, 5, 6, 7]
```

---

## 3.3 Parentheses Matching

### Key Concepts
- Classic stack application — push open brackets, pop and verify on close bracket
- Three variants: `()`, `[]`, `{}` — use a **mapping dict** for clean code
- Extended variants: minimum removals, minimum additions to make valid, score of parentheses
- Key rule: stack must be **empty** at end for full validity

### Interview Details
- Mapping close→open is cleaner than multiple `if` conditions
- Always handle: empty string (valid), only open brackets (invalid), only close brackets (invalid)
- **Minimum add/remove** variants use a counter approach — no stack needed
- Score of parentheses uses stack with integer values

```python
# ---- Basic parentheses validation ----
def is_valid_parens(s):
    stack   = []
    mapping = {')': '(', ']': '[', '}': '{'}
    for ch in s:
        if ch in mapping:                        # Closing bracket
            top = stack.pop() if stack else '#'
            if mapping[ch] != top:
                return False
        else:
            stack.append(ch)                     # Opening bracket
    return len(stack) == 0                       # Must be fully matched

print(is_valid_parens("()[]{}"))   # True
print(is_valid_parens("([)]"))     # False
print(is_valid_parens("{[]}"))     # True

# ---- Minimum removals to make valid ----
def min_remove_to_valid(s):
    open_count = close_count = 0
    for ch in s:
        if ch == '(':
            open_count += 1
        elif ch == ')':
            if open_count > 0:
                open_count -= 1      # Match with an open
            else:
                close_count += 1     # Unmatched close
    return open_count + close_count  # Total removals needed

# ---- Return valid string after minimum removals ----
def min_remove_valid_string(s):
    stack    = []                    # Stores indices of unmatched '('
    to_remove = set()

    for i, ch in enumerate(s):
        if ch == '(':
            stack.append(i)
        elif ch == ')':
            if stack:
                stack.pop()          # Matched
            else:
                to_remove.add(i)     # Unmatched ')'
    to_remove |= set(stack)          # Remaining unmatched '('

    return ''.join(ch for i, ch in enumerate(s) if i not in to_remove)

# ---- Longest valid parentheses substring ----
def longest_valid_parens(s):
    stack = [-1]                     # Sentinel for base
    max_len = 0
    for i, ch in enumerate(s):
        if ch == '(':
            stack.append(i)
        else:
            stack.pop()
            if not stack:
                stack.append(i)      # New base
            else:
                max_len = max(max_len, i - stack[-1])
    return max_len

# ---- Score of parentheses ----
# "()" = 1, "(A)" = 2*A, "AB" = A+B
def score_of_parens(s):
    stack = [0]                      # Current score at depth
    for ch in s:
        if ch == '(':
            stack.append(0)          # New depth level
        else:
            v = stack.pop()
            stack[-1] += max(2*v, 1) # "()" scores 1, else doubles
    return stack[0]

print(score_of_parens("(()(()))"))   # 6
```

---

## 3.4 Expression Evaluation

### Key Concepts
- **Infix** (normal): `3 + 4 * 2` — needs precedence/parentheses handling
- **Postfix / RPN** (Reverse Polish Notation): `3 4 2 * +` — no precedence needed, stack-based
- **Prefix**: `+ 3 * 4 2` — evaluate right to left
- **Shunting-yard algorithm** (Dijkstra): converts infix → postfix using operator stack
- Two-stack approach evaluates infix directly: **operand stack** + **operator stack**

### Interview Details
- Postfix evaluation is the cleanest — O(n) with one stack
- Shunting-yard needs precedence dict + handling of left/right associativity
- Basic calculator (LeetCode #224) uses stack to handle `+`, `-`, `(`, `)`
- Know how to handle **unary minus** and **multi-digit numbers**

```python
# ---- Postfix (RPN) evaluation ----
def eval_rpn(tokens):
    stack = []
    ops = {
        '+': lambda a, b: a + b,
        '-': lambda a, b: a - b,
        '*': lambda a, b: a * b,
        '/': lambda a, b: int(a / b)   # Truncate toward zero
    }
    for token in tokens:
        if token in ops:
            b = stack.pop()            # Right operand (popped first)
            a = stack.pop()            # Left operand
            stack.append(ops[token](a, b))
        else:
            stack.append(int(token))
    return stack[0]

print(eval_rpn(["2","1","+","3","*"]))   # 9  → (2+1)*3
print(eval_rpn(["4","13","5","/","+"]))  # 6  → 4+(13/5)

# ---- Infix to Postfix (Shunting-Yard) ----
def infix_to_postfix(expression):
    precedence = {'+': 1, '-': 1, '*': 2, '/': 2, '^': 3}
    output  = []
    op_stack = []

    tokens = expression.split()
    for token in tokens:
        if token.lstrip('-').isdigit():         # Operand
            output.append(token)
        elif token == '(':
            op_stack.append(token)
        elif token == ')':
            while op_stack and op_stack[-1] != '(':
                output.append(op_stack.pop())
            op_stack.pop()                       # Remove '('
        else:                                    # Operator
            while (op_stack and
                   op_stack[-1] != '(' and
                   precedence.get(op_stack[-1], 0) >= precedence.get(token, 0)):
                output.append(op_stack.pop())
            op_stack.append(token)

    while op_stack:
        output.append(op_stack.pop())
    return ' '.join(output)

print(infix_to_postfix("3 + 4 * 2"))         # "3 4 2 * +"
print(infix_to_postfix("( 3 + 4 ) * 2"))     # "3 4 + 2 *"

# ---- Basic Calculator — handles +, -, ( ) ----
def basic_calculator(s):
    stack   = []
    result  = 0
    sign    = 1                      # +1 or -1
    num     = 0

    for ch in s:
        if ch.isdigit():
            num = num * 10 + int(ch) # Build multi-digit number
        elif ch in '+-':
            result += sign * num
            num    = 0
            sign   = 1 if ch == '+' else -1
        elif ch == '(':
            stack.append(result)     # Save current result
            stack.append(sign)       # Save current sign
            result = 0               # Reset for sub-expression
            sign   = 1
        elif ch == ')':
            result  += sign * num
            num      = 0
            result  *= stack.pop()   # Apply sign before '('
            result  += stack.pop()   # Add result before '('

    return result + sign * num

# ---- Basic Calculator II — handles +, -, *, / (no parentheses) ----
def calculate_ii(s):
    stack   = []
    num     = 0
    op      = '+'                    # Default first operation is +

    for i, ch in enumerate(s):
        if ch.isdigit():
            num = num * 10 + int(ch)
        if (ch in '+-*/' or i == len(s) - 1) and ch != ' ':
            if op == '+':   stack.append(num)
            elif op == '-': stack.append(-num)
            elif op == '*': stack.append(stack.pop() * num)
            elif op == '/': stack.append(int(stack.pop() / num))
            op  = ch
            num = 0

    return sum(stack)

print(calculate_ii("3+2*2"))         # 7
```

---

## 3.5 Next Greater Element

### Key Concepts
- **Monotonic stack** pattern — maintain a stack of elements in decreasing order
- For each element: pop all smaller elements from stack; they found their "next greater"
- Process left→right; stack holds elements **waiting** for their next greater
- For circular arrays: process array **twice** (index `i % n`)
- Variants: next greater, next smaller, previous greater, previous smaller

### Interview Details
- Monotonic stack is O(n) — each element pushed and popped at most once
- Result array initialized to `-1` (no greater element found = -1 by convention)
- For previous greater element: traverse **right to left**
- **Daily temperatures** (LeetCode #739) and **next greater element** (LeetCode #496, #503) are must-know

```python
# ---- Next Greater Element (basic) ----
def next_greater(nums):
    n      = len(nums)
    result = [-1] * n
    stack  = []                      # Stores indices of elements awaiting NGE

    for i in range(n):
        # Pop all elements smaller than current — current is their NGE
        while stack and nums[stack[-1]] < nums[i]:
            idx          = stack.pop()
            result[idx]  = nums[i]
        stack.append(i)
    return result

print(next_greater([2, 1, 2, 4, 3]))   # [4, 2, 4, -1, -1]

# ---- Next Greater Element — circular array ----
# LeetCode #503
def next_greater_circular(nums):
    n      = len(nums)
    result = [-1] * n
    stack  = []

    for i in range(2 * n):           # Traverse twice
        while stack and nums[stack[-1]] < nums[i % n]:
            result[stack.pop()] = nums[i % n]
        if i < n:
            stack.append(i)          # Only push in first pass

    return result

print(next_greater_circular([1,2,1]))   # [2,-1,2]

# ---- Daily temperatures (next warmer day) ----
# LeetCode #739
def daily_temperatures(temps):
    n      = len(temps)
    result = [0] * n
    stack  = []                      # Indices of unresolved days

    for i, temp in enumerate(temps):
        while stack and temps[stack[-1]] < temp:
            j          = stack.pop()
            result[j]  = i - j       # Days to wait
        stack.append(i)

    return result

print(daily_temperatures([73,74,75,71,69,72,76,73]))
# [1, 1, 4, 2, 1, 1, 0, 0]

# ---- Largest rectangle in histogram (classic monotonic stack) ----
# LeetCode #84
def largest_rectangle(heights):
    stack   = []                     # Stores indices
    max_area = 0
    heights  = heights + [0]         # Sentinel to flush stack

    for i, h in enumerate(heights):
        start = i
        while stack and heights[stack[-1]] > h:
            idx      = stack.pop()
            width    = i - (stack[-1] + 1 if stack else 0)
            max_area = max(max_area, heights[idx] * width)
            start    = idx
        stack.append(i)

    return max_area

print(largest_rectangle([2,1,5,6,2,3]))   # 10
```

---

## 3.6 Circular Queue

### Key Concepts
- Fixed-size queue using an array with **head and tail pointers**
- `tail = (tail + 1) % capacity` — wrap around using modulo
- **Full** condition: `(tail + 1) % capacity == head`
- **Empty** condition: `head == tail` (or use a size counter)
- Also called **Ring Buffer** — used in OS, networking buffers, producer-consumer

### Interview Details
- Modulo arithmetic is the key — practice it without mistakes
- Two approaches to distinguish full vs empty: **leave one slot empty** OR **use a size counter** (cleaner, preferred)
- `deque(maxlen=k)` in Python automatically acts as circular — mention in interviews
- Know how circular queue underlies **BFS** in graph problems

```python
# ---- Circular Queue with size counter ----
class CircularQueue:
    def __init__(self, capacity):
        self.cap    = capacity
        self.queue  = [None] * capacity
        self.head   = 0                  # Dequeue from here
        self.tail   = 0                  # Enqueue here
        self.size   = 0

    def enqueue(self, val):
        if self.is_full():
            raise OverflowError("Queue is full")
        self.queue[self.tail] = val
        self.tail = (self.tail + 1) % self.cap   # Wrap around
        self.size += 1

    def dequeue(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        val       = self.queue[self.head]
        self.head = (self.head + 1) % self.cap   # Wrap around
        self.size -= 1
        return val

    def peek_front(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self.queue[self.head]

    def peek_rear(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self.queue[(self.tail - 1) % self.cap]

    def is_empty(self):
        return self.size == 0

    def is_full(self):
        return self.size == self.cap

# ---- Python deque as fixed circular buffer ----
from collections import deque

circular = deque(maxlen=3)
circular.append(1)
circular.append(2)
circular.append(3)
circular.append(4)         # Automatically drops leftmost (1)
print(circular)            # deque([2, 3, 4], maxlen=3)
```

### Stacks & Queues — Quick Complexity Cheatsheet

| Operation | Stack (list) | Queue (deque) | Circular Queue |
|---|---|---|---|
| Push / Enqueue | O(1) amortized | O(1) | O(1) |
| Pop / Dequeue | O(1) | O(1) | O(1) |
| Peek | O(1) | O(1) | O(1) |
| Search | O(n) | O(n) | O(n) |
| Min/Max stack | O(1) | — | — |
| NGE (mono stack) | O(n) total | — | — |
| Sliding window max | — | O(n) total | — |

### Stacks & Queues — Common Interview Traps & Tips

- **Never use `list` as a queue** — `pop(0)` is O(n); always use `deque`
- Min stack: store **(value, min_at_this_point)** tuples — cleaner than parallel stack
- Circular queue **full vs empty**: size counter is foolproof; one-slot-empty approach is error-prone
- Monotonic stack: decide **strictly greater** vs **greater or equal** based on problem — read carefully
- RPN evaluation: pop **b first, then a** — order matters for `-` and `/`
- Basic calculator: when hitting `)`, apply the **saved sign** from before `(`, not current sign
- Shunting-yard: `^` (power) is **right-associative** — use `>` not `>=` for precedence comparison

---

# 4. Hashing & Heaps

---

## Core Concepts Upfront

```python
from collections import defaultdict, Counter, OrderedDict
import heapq

# HashMap  → dict         O(1) avg get/set/delete
# HashSet  → set          O(1) avg add/remove/lookup
# Min Heap → heapq        O(log n) push/pop, O(1) peek
# Max Heap → heapq        negate values (heapq is min-only)
# Counter  → freq map     most_common(), arithmetic ops
```

---

## 4.1 HashMap / HashSet

### Key Concepts
- **HashMap**: key → value mapping, keys must be **hashable** (immutable)
- **HashSet**: unordered collection of unique hashable elements
- Hash function maps key → bucket index; **collision** handled by chaining or open addressing
- Python `dict` uses open addressing with pseudo-random probing
- **Load factor** = n/capacity; Python resizes at ~⅔ capacity → O(1) amortized
- Worst case O(n) for all ops (all keys collide) — rare, mention in interviews

### Interview Details
- `dict` preserves insertion order since Python 3.7+
- Unhashable types: `list`, `dict`, `set` — use `tuple` instead as key
- `defaultdict` avoids KeyError with default factory
- `Counter` is a specialized dict for frequency counting
- Two-sum, anagram detection, frequency problems → always think HashMap first

### Time/Space Complexity

| Operation | Average | Worst |
|---|---|---|
| Insert / Update | O(1) | O(n) |
| Delete | O(1) | O(n) |
| Lookup | O(1) | O(n) |
| Iteration | O(n) | O(n) |

```python
# ---- HashMap basics ----
freq = {}
freq['a']  = freq.get('a', 0) + 1      # Safe increment
freq['b']  = freq.setdefault('b', 0)    # Set if missing, return value

# defaultdict — cleaner default handling
graph   = defaultdict(list)             # Adjacency list
freq    = defaultdict(int)              # Frequency map
buckets = defaultdict(set)              # Bucket grouping

# Counter — frequency map with extras
words  = ["apple", "banana", "apple", "cherry", "banana", "apple"]
count  = Counter(words)
print(count.most_common(2))             # [('apple', 3), ('banana', 2)]
print(count['apple'])                   # 3
c1 = Counter("aab"); c2 = Counter("abb")
print(c1 & c2)                          # Intersection: Counter({'a': 1, 'b': 1})
print(c1 | c2)                          # Union:        Counter({'a': 2, 'b': 2})
print(c1 - c2)                          # Subtraction:  Counter({'a': 1})

# ---- HashSet basics ----
seen  = set()
seen.add(3)
seen.discard(3)                         # No error if missing (vs remove)

# Set operations
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
print(a & b)                            # Intersection: {3, 4}
print(a | b)                            # Union:        {1,2,3,4,5,6}
print(a - b)                            # Difference:   {1, 2}
print(a ^ b)                            # Symmetric diff: {1,2,5,6}

# Frozenset — hashable set (usable as dict key)
key = frozenset([1, 2, 3])
d   = {key: "value"}

# ---- Custom hash map implementation ----
class HashMap:
    def __init__(self, capacity=1024):
        self.capacity = capacity
        self.buckets  = [[] for _ in range(capacity)]  # Chaining

    def _hash(self, key):
        return hash(key) % self.capacity

    def put(self, key, val):
        bucket = self.buckets[self._hash(key)]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket[i] = (key, val)          # Update existing
                return
        bucket.append((key, val))               # Insert new

    def get(self, key):
        for k, v in self.buckets[self._hash(key)]:
            if k == key:
                return v
        return -1

    def remove(self, key):
        bucket = self.buckets[self._hash(key)]
        self.buckets[self._hash(key)] = [
            (k, v) for k, v in bucket if k != key
        ]
```

---

## 4.2 Coding Hash-Based Problems

### Key Patterns
- **Two Sum** → value-to-index map, one pass
- **Anagram grouping** → sorted string or char-count tuple as key
- **Subarray sum = k** → prefix sum + frequency map
- **Longest consecutive sequence** → set for O(1) lookup, only start sequences
- **Sliding window + hash** → character/element frequency window

```python
# ---- Two Sum — O(n) ----
def two_sum(nums, target):
    seen = {}                               # val → index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []

print(two_sum([2, 7, 11, 15], 9))           # [0, 1]

# ---- Group anagrams ----
def group_anagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = tuple(sorted(s))              # Sorted chars as key
        groups[key].append(s)
    return list(groups.values())

# Alternative key: character count tuple O(n) — no sorting
def group_anagrams_v2(strs):
    groups = defaultdict(list)
    for s in strs:
        count = [0] * 26
        for ch in s:
            count[ord(ch) - ord('a')] += 1
        groups[tuple(count)].append(s)
    return list(groups.values())

# ---- Subarray sum equals k — O(n) ----
def subarray_sum(nums, k):
    prefix_freq = defaultdict(int)
    prefix_freq[0] = 1                      # Empty prefix
    prefix = count = 0
    for num in nums:
        prefix += num
        count  += prefix_freq[prefix - k]   # How many subarrays end here
        prefix_freq[prefix] += 1
    return count

print(subarray_sum([1, 1, 1], 2))           # 2
print(subarray_sum([1, 2, 3], 3))           # 2

# ---- Longest consecutive sequence — O(n) ----
def longest_consecutive(nums):
    num_set   = set(nums)
    best      = 0
    for num in num_set:
        if num - 1 not in num_set:          # Only start a new sequence
            curr, length = num, 1
            while curr + 1 in num_set:
                curr   += 1
                length += 1
            best = max(best, length)
    return best

print(longest_consecutive([100,4,200,1,3,2]))  # 4 → [1,2,3,4]

# ---- Top K frequent elements — O(n log k) ----
def top_k_frequent(nums, k):
    count  = Counter(nums)
    return heapq.nlargest(k, count.keys(), key=count.get)

# Bucket sort approach — O(n)
def top_k_frequent_bucket(nums, k):
    count   = Counter(nums)
    buckets = [[] for _ in range(len(nums) + 1)]
    for num, freq in count.items():
        buckets[freq].append(num)
    result = []
    for i in range(len(buckets) - 1, 0, -1):
        result.extend(buckets[i])
        if len(result) >= k:
            return result[:k]
    return result

print(top_k_frequent([1,1,1,2,2,3], 2))    # [1, 2]

# ---- Longest substring without repeating characters — O(n) ----
def length_of_longest_substring(s):
    char_index = {}
    left = best = 0
    for right, ch in enumerate(s):
        if ch in char_index and char_index[ch] >= left:
            left = char_index[ch] + 1       # Shrink window
        char_index[ch] = right
        best = max(best, right - left + 1)
    return best

print(length_of_longest_substring("abcabcbb"))  # 3
```

---

## 4.3 Min Heap / Max Heap

### Key Concepts
- **Heap**: complete binary tree satisfying heap property
- **Min heap**: parent ≤ children — root is always minimum
- **Max heap**: parent ≥ children — root is always maximum
- Stored as array: parent of i = `(i-1)//2`, children = `2i+1`, `2i+2`
- Python `heapq` is **min heap only** — negate values for max heap

### Time/Space Complexity

| Operation | Time | Notes |
|---|---|---|
| heappush | O(log n) | Sift up |
| heappop | O(log n) | Sift down |
| heapify | O(n) | Bottom-up build |
| peek (heap[0]) | O(1) | Min element |
| nlargest / nsmallest | O(n + k log n) | Uses heap internally |

```python
import heapq

# ---- Min heap operations ----
min_heap = []
heapq.heappush(min_heap, 5)
heapq.heappush(min_heap, 2)
heapq.heappush(min_heap, 8)
heapq.heappush(min_heap, 1)

print(min_heap[0])                      # Peek min → 1  (O(1))
print(heapq.heappop(min_heap))          # Pop min  → 1  (O(log n))

# ---- Max heap — negate values ----
max_heap = []
for val in [5, 2, 8, 1]:
    heapq.heappush(max_heap, -val)      # Store negated

print(-max_heap[0])                     # Peek max → 8
print(-heapq.heappop(max_heap))         # Pop max  → 8

# ---- Min heap from scratch ----
class MinHeap:
    def __init__(self):
        self.heap = []

    def push(self, val):
        self.heap.append(val)
        self._sift_up(len(self.heap) - 1)

    def pop(self):
        if len(self.heap) == 1:
            return self.heap.pop()
        root = self.heap[0]
        self.heap[0] = self.heap.pop()  # Move last to root
        self._sift_down(0)
        return root

    def peek(self):
        return self.heap[0] if self.heap else None

    def _sift_up(self, i):
        parent = (i - 1) // 2
        if i > 0 and self.heap[i] < self.heap[parent]:
            self.heap[i], self.heap[parent] = self.heap[parent], self.heap[i]
            self._sift_up(parent)

    def _sift_down(self, i):
        n        = len(self.heap)
        smallest = i
        left     = 2 * i + 1
        right    = 2 * i + 2

        if left  < n and self.heap[left]  < self.heap[smallest]:
            smallest = left
        if right < n and self.heap[right] < self.heap[smallest]:
            smallest = right
        if smallest != i:
            self.heap[i], self.heap[smallest] = self.heap[smallest], self.heap[i]
            self._sift_down(smallest)
```

---

## 4.4 Priority Queue

### Key Concepts
- Abstract data type: always access/remove the **highest priority** element
- Implemented via heap — O(log n) push/pop, O(1) peek
- Python: `heapq` module (min priority) or `queue.PriorityQueue` (thread-safe)
- Custom priority: use tuples `(priority, item)` or define `__lt__`
- **K-way merge**, **Dijkstra's**, **task scheduling** all use priority queues

### Interview Details
- `heapq` is not thread-safe — use `queue.PriorityQueue` for concurrency
- For objects without natural ordering: add index as tiebreaker, or define `__lt__`
- `heapq.nlargest(k, iterable)` and `heapq.nsmallest(k, iterable)` — O(n log k)
- Know when heap beats sorting: streaming data, online algorithms, top-k problems

```python
# ---- Kth largest element — O(n log k) ----
def kth_largest(nums, k):
    heap = []
    for num in nums:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)         # Evict smallest
    return heap[0]                      # Root = kth largest

print(kth_largest([3,2,1,5,6,4], 2))   # 5

# ---- K closest points to origin — O(n log k) ----
def k_closest(points, k):
    heap = []
    for x, y in points:
        dist = x*x + y*y
        heapq.heappush(heap, (-dist, x, y))  # Max heap by negating
        if len(heap) > k:
            heapq.heappop(heap)
    return [[x, y] for (_, x, y) in heap]

# ---- Merge k sorted arrays — O(N log k) ----
def merge_k_sorted(arrays):
    heap   = []
    result = []
    for i, arr in enumerate(arrays):
        if arr:
            heapq.heappush(heap, (arr[0], i, 0))  # (val, arr_idx, elem_idx)

    while heap:
        val, i, j = heapq.heappop(heap)
        result.append(val)
        if j + 1 < len(arrays[i]):
            heapq.heappush(heap, (arrays[i][j+1], i, j+1))

    return result

print(merge_k_sorted([[1,4,7],[2,5,8],[3,6,9]]))
# [1,2,3,4,5,6,7,8,9]

# ---- Find median from data stream ----
# Two heaps: max heap (lower half) + min heap (upper half)
class MedianFinder:
    def __init__(self):
        self.lo = []    # Max heap (negate) — lower half
        self.hi = []    # Min heap — upper half

    def add_num(self, num):
        heapq.heappush(self.lo, -num)           # Push to lower
        heapq.heappush(self.hi, -heapq.heappop(self.lo))
        if len(self.hi) > len(self.lo):         # Keep lo >= hi in size
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def find_median(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2.0

mf = MedianFinder()
for n in [1, 2, 3, 4, 5]:
    mf.add_num(n)
    print(mf.find_median())             # 1, 1.5, 2, 2.5, 3
```

---

## 4.5 Heapify Operations

### Key Concepts
- `heapq.heapify(arr)` — converts list to heap **in-place** in **O(n)**
- Why O(n) not O(n log n)? Bottom-up build: leaf nodes are free, work decreases per level
- `heapreplace(heap, val)` — pop min then push new val in single O(log n) op
- `heappushpop(heap, val)` — push first then pop min — more efficient than two separate calls
- Manual heapify: start from last non-leaf `(n//2 - 1)` and sift down each node

```python
import heapq

# ---- heapify — O(n) ----
arr = [5, 3, 8, 1, 9, 2, 7]
heapq.heapify(arr)
print(arr)                              # Valid min heap
print(arr[0])                           # 1 — minimum guaranteed at root

# ---- heapreplace — pop then push, O(log n) ----
result = heapq.heapreplace(arr, 4)
print(result)                           # Returns old min

# ---- heappushpop — push then pop, O(log n) ----
result = heapq.heappushpop(arr, 0)
print(result)                           # Returns 0 (pushed value is still min)

# ---- Manual heapify from scratch ----
def heapify_manual(arr):
    n = len(arr)
    for i in range(n // 2 - 1, -1, -1):
        sift_down(arr, i, n)

def sift_down(arr, i, n):
    smallest = i
    left     = 2 * i + 1
    right    = 2 * i + 2
    if left  < n and arr[left]  < arr[smallest]: smallest = left
    if right < n and arr[right] < arr[smallest]: smallest = right
    if smallest != i:
        arr[i], arr[smallest] = arr[smallest], arr[i]
        sift_down(arr, smallest, n)

def sift_up(arr, i):
    while i > 0:
        parent = (i - 1) // 2
        if arr[i] < arr[parent]:
            arr[i], arr[parent] = arr[parent], arr[i]
            i = parent
        else:
            break

# ---- Why heapify is O(n) — key interview explanation ----
# Height of heap = log n
# Nodes at height h = ceil(n / 2^(h+1))
# Work per node at height h = O(h)
# Total = sum over h: n/2^(h+1) * h = O(n) * sum(h/2^h) = O(n) * O(2) = O(n)

# ---- When to use heapify vs repeated heappush ----
# heapify(arr)     → O(n)       build from existing list
# n x heappush     → O(n log n) build from scratch
# Rule: if you have all elements upfront → heapify
#       if elements arrive one by one    → heappush
```

---

## 4.6 Heap Sort

### Key Concepts
- **Phase 1**: Build max heap from array — O(n)
- **Phase 2**: Repeatedly extract max, place at end — O(n log n)
- **In-place**, **O(n log n)** worst case — unlike quicksort O(n²) worst
- **Not stable** (equal elements may reorder) — unlike merge sort
- Not cache-friendly (non-sequential memory access) — slower than quicksort in practice

```python
# ---- Heap Sort — in-place, O(n log n) ----
def heap_sort(arr):
    n = len(arr)

    # Phase 1: Build max heap (bottom-up)
    for i in range(n // 2 - 1, -1, -1):
        _sift_down_max(arr, i, n)

    # Phase 2: Extract elements one by one
    for end in range(n - 1, 0, -1):
        arr[0], arr[end] = arr[end], arr[0]   # Move max to end
        _sift_down_max(arr, 0, end)            # Restore heap on reduced array

    return arr

def _sift_down_max(arr, i, n):
    largest = i
    left    = 2 * i + 1
    right   = 2 * i + 2
    if left  < n and arr[left]  > arr[largest]: largest = left
    if right < n and arr[right] > arr[largest]: largest = right
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        _sift_down_max(arr, largest, n)

arr = [12, 11, 13, 5, 6, 7]
print(heap_sort(arr))                   # [5, 6, 7, 11, 12, 13]

# ---- Heap sort using Python heapq (not in-place) ----
def heap_sort_heapq(arr):
    heapq.heapify(arr)
    return [heapq.heappop(arr) for _ in range(len(arr))]

print(heap_sort_heapq([3, 1, 4, 1, 5, 9]))  # [1, 1, 3, 4, 5, 9]

# ---- Partial sort — get k sorted elements efficiently ----
def partial_sort(arr, k):
    # Only sort smallest k — O(n + k log n)
    return heapq.nsmallest(k, arr)

def partial_sort_largest(arr, k):
    # Only sort largest k — O(n + k log n)
    return heapq.nlargest(k, arr)
```

### Hashing & Heaps — Quick Complexity Cheatsheet

| Operation | HashMap/Set | Min/Max Heap | Priority Queue |
|---|---|---|---|
| Insert | O(1) avg | O(log n) | O(log n) |
| Delete | O(1) avg | O(log n) | O(log n) |
| Lookup / Peek min | O(1) avg | O(1) | O(1) |
| Build from list | O(n) | O(n) heapify | O(n log n) repeated push |
| Sort n elements | O(n log n) | O(n log n) | O(n log n) |
| k largest/smallest | O(n log k) | O(n log k) | O(n log k) |
| Heapify | — | O(n) | — |

### Hashing & Heaps — Common Interview Traps & Tips

- **Max heap**: always negate values — `heappush(h, -val)`, retrieve as `-heappop(h)`
- **Tuple tiebreaker**: if two tuples share priority, Python compares the second element — add a unique counter to avoid comparing non-comparable objects: `(priority, counter, item)`
- `heapify` is **O(n)**, not O(n log n) — be ready to explain why (bottom-up build proof)
- **Two-heap median**: lo is max heap (negated), hi is min heap; size invariant: `len(lo) >= len(hi)` and `len(lo) - len(hi) <= 1`
- **Subarray sum = k**: prefix sum + hashmap — NOT sliding window (negatives break sliding window)
- `Counter` subtraction ignores zero/negative counts — `Counter({'a':1}) - Counter({'a':2})` gives empty Counter
- `dict` in Python 3.7+ preserves insertion order but **set does not**
- For top-k frequent: if k is close to n, sorting beats heap; heap wins when k << n
- `heapreplace` is faster than `heappop` + `heappush` — use in hot loops
- Longest consecutive sequence: **only start counting from sequence beginnings** (`num-1 not in set`) — avoids O(n²) inner loop

---

*Generated for interview preparation — Python implementations throughout*