# Java Interview Notes
> Topics: Arrays & Strings · Linked Lists · Stacks & Queues · Hashing & Heaps

---

# Part 1 — Arrays & Strings

---

## 1. Array Insert / Delete

### Key Concepts
- Arrays in Java are **fixed size**. True insert/delete requires shifting elements.
- Use `ArrayList` for dynamic insert/delete; raw arrays require manual shifting.
- Time complexity: Insert/Delete at end → **O(1)**; at arbitrary index → **O(n)** due to shifting.

### Insert at Index (Raw Array)
```java
// Shift elements right, then insert
public static int[] insertAt(int[] arr, int index, int value, int size) {
    // arr has extra space at end (size < arr.length)
    for (int i = size - 1; i >= index; i--) {
        arr[i + 1] = arr[i];  // shift right
    }
    arr[index] = value;
    return arr;
}

// Example
int[] arr = new int[6];
arr = new int[]{1, 2, 4, 5, 0, 0};
insertAt(arr, 2, 3, 4); // → [1, 2, 3, 4, 5, 0]
```

### Delete at Index (Raw Array)
```java
public static int[] deleteAt(int[] arr, int index, int size) {
    for (int i = index; i < size - 1; i++) {
        arr[i] = arr[i + 1];  // shift left
    }
    arr[size - 1] = 0;  // clear last element
    return arr;
}
```

### With ArrayList (Preferred in interviews)
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
list.add(2, 99);     // Insert 99 at index 2 → [1, 2, 99, 3, 4, 5]
list.remove(3);      // Remove index 3 → [1, 2, 99, 4, 5]
list.remove(Integer.valueOf(99)); // Remove by value
```

### Interview Traps
- Shifting direction matters: **insert → shift right (from back)**, **delete → shift left (from front)**. Reversing this corrupts data.
- Off-by-one errors when boundary checking `index >= 0 && index < size`.
- Always clarify: "Is the array sorted? Does order need to be maintained?"

---

## 2. Array Reversal

### Key Concepts
- Two-pointer technique: swap `arr[left]` and `arr[right]`, move inward.
- In-place → **O(1)** space, **O(n)** time.

### Basic Reversal
```java
public static void reverse(int[] arr) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
```

### Reverse a Subarray (index l to r)
```java
public static void reverseRange(int[] arr, int l, int r) {
    while (l < r) {
        int temp = arr[l];
        arr[l] = arr[r];
        arr[r] = temp;
        l++; r--;
    }
}
```

### Reverse Using Java Built-ins
```java
// For Integer[] (not int[])
Integer[] arr = {1, 2, 3, 4, 5};
Collections.reverse(Arrays.asList(arr)); // → [5, 4, 3, 2, 1]

// StringBuilder reverse
String s = "hello";
String reversed = new StringBuilder(s).reverse().toString(); // "olleh"
```

### XOR Swap (No temp variable — common interview trick)
```java
arr[left] ^= arr[right];
arr[right] ^= arr[left];
arr[left] ^= arr[right];
// ⚠️ Fails when left == right (same reference)!
```

### Interview Traps
- Reversing a **String** in Java: Strings are immutable → must use `char[]` or `StringBuilder`.
- Don't use `Collections.reverse()` on `int[]` — it requires `Integer[]`.

---

## 3. 2D Arrays

### Key Concepts
- Declared as `int[rows][cols]`; each row can have different lengths (jagged arrays).
- Row-major traversal is cache-friendly (preferred).
- Matrix transpose: swap `[i][j]` with `[j][i]`.

### Declaration & Traversal
```java
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// Row-major traversal
for (int i = 0; i < matrix.length; i++) {
    for (int j = 0; j < matrix[0].length; j++) {
        System.out.print(matrix[i][j] + " ");
    }
}

// Enhanced for-loop
for (int[] row : matrix)
    for (int val : row)
        System.out.print(val + " ");
```

### Transpose a Matrix
```java
// Works for square (n x n) matrix
public static void transpose(int[][] matrix) {
    int n = matrix.length;
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++) {  // j starts at i+1!
            int temp = matrix[i][j];
            matrix[i][j] = matrix[j][i];
            matrix[j][i] = temp;
        }
}
```

### Rotate Matrix 90° Clockwise (Classic Interview Problem)
```java
// Step 1: Transpose
// Step 2: Reverse each row
public static void rotate90(int[][] matrix) {
    int n = matrix.length;
    // Transpose
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++) {
            int tmp = matrix[i][j];
            matrix[i][j] = matrix[j][i];
            matrix[j][i] = tmp;
        }
    // Reverse each row
    for (int[] row : matrix) {
        int l = 0, r = row.length - 1;
        while (l < r) {
            int tmp = row[l]; row[l] = row[r]; row[r] = tmp;
            l++; r--;
        }
    }
}
```

### Spiral Order Traversal
```java
public static List<Integer> spiralOrder(int[][] matrix) {
    List<Integer> result = new ArrayList<>();
    int top = 0, bottom = matrix.length - 1;
    int left = 0, right = matrix[0].length - 1;

    while (top <= bottom && left <= right) {
        for (int i = left; i <= right; i++) result.add(matrix[top][i]);
        top++;
        for (int i = top; i <= bottom; i++) result.add(matrix[i][right]);
        right--;
        if (top <= bottom)
            for (int i = right; i >= left; i--) result.add(matrix[bottom][i]);
        bottom--;
        if (left <= right)
            for (int i = bottom; i >= top; i--) result.add(matrix[i][left]);
        left++;
    }
    return result;
}
```

### Interview Traps
- Always check `matrix.length == 0` before accessing `matrix[0].length`.
- Jagged arrays: `matrix[i].length` differs per row — don't assume `matrix[0].length` is universal.
- 90° CCW = Transpose + Reverse each **column** (or reverse rows, then transpose).

---

## 4. Pattern Matching

### Key Concepts
- **Naive**: O(n * m) — slide pattern over text, check char by char.
- **KMP (Knuth-Morris-Pratt)**: O(n + m) — uses a failure function (LPS array) to skip re-comparisons.
- Java built-ins: `String.contains()`, `String.indexOf()`, `String.matches()` (regex).

### Naive Pattern Matching
```java
public static List<Integer> naiveSearch(String text, String pattern) {
    List<Integer> indices = new ArrayList<>();
    int n = text.length(), m = pattern.length();

    for (int i = 0; i <= n - m; i++) {
        int j = 0;
        while (j < m && text.charAt(i + j) == pattern.charAt(j)) j++;
        if (j == m) indices.add(i);
    }
    return indices;
}
```

### KMP Algorithm
```java
// Build LPS (Longest Proper Prefix which is also Suffix) array
public static int[] buildLPS(String pattern) {
    int m = pattern.length();
    int[] lps = new int[m];
    int len = 0, i = 1;

    while (i < m) {
        if (pattern.charAt(i) == pattern.charAt(len)) {
            lps[i++] = ++len;
        } else {
            if (len != 0) len = lps[len - 1]; // fallback
            else lps[i++] = 0;
        }
    }
    return lps;
}

public static List<Integer> kmpSearch(String text, String pattern) {
    List<Integer> result = new ArrayList<>();
    int n = text.length(), m = pattern.length();
    int[] lps = buildLPS(pattern);
    int i = 0, j = 0;

    while (i < n) {
        if (text.charAt(i) == pattern.charAt(j)) { i++; j++; }
        if (j == m) {
            result.add(i - j);   // match found
            j = lps[j - 1];
        } else if (i < n && text.charAt(i) != pattern.charAt(j)) {
            if (j != 0) j = lps[j - 1];
            else i++;
        }
    }
    return result;
}
```

### Java Built-ins
```java
String text = "hello world hello";
// Index of first occurrence
int idx = text.indexOf("hello");          // 0
int lastIdx = text.lastIndexOf("hello");  // 12

// Check if contains
boolean has = text.contains("world");    // true

// All occurrences
int pos = 0;
while ((pos = text.indexOf("hello", pos)) != -1) {
    System.out.println("Found at: " + pos);
    pos++;
}

// Regex matching
boolean match = "abc123".matches("[a-z]+\\d+"); // true
```

### Interview Traps
- KMP LPS array is 0-indexed; understand why `len = lps[len-1]` and not `len--`.
- `String.matches()` matches the **entire** string, not a substring — use `Pattern.matcher().find()` for substring.
- Edge case: empty pattern or pattern longer than text.

---

## 5. Array Rotation

### Key Concepts
- **Left rotation by k**: element at index `i` moves to `i - k` (mod n).
- **Right rotation by k**: element at index `i` moves to `i + k` (mod n).
- Best approach: **Reversal algorithm** — O(n) time, O(1) space.
- Always normalize: `k = k % n` to handle `k >= n`.

### Left Rotation — Reversal Algorithm
```java
public static void leftRotate(int[] arr, int k) {
    int n = arr.length;
    k = k % n;
    reverseRange(arr, 0, k - 1);      // reverse first k elements
    reverseRange(arr, k, n - 1);      // reverse remaining
    reverseRange(arr, 0, n - 1);      // reverse entire array
}

private static void reverseRange(int[] arr, int l, int r) {
    while (l < r) {
        int tmp = arr[l]; arr[l] = arr[r]; arr[r] = tmp;
        l++; r--;
    }
}

// Example: [1,2,3,4,5], k=2 → [3,4,5,1,2]
```

### Right Rotation — Reversal Algorithm
```java
public static void rightRotate(int[] arr, int k) {
    int n = arr.length;
    k = k % n;
    reverseRange(arr, 0, n - 1);      // reverse entire
    reverseRange(arr, 0, k - 1);      // reverse first k
    reverseRange(arr, k, n - 1);      // reverse remaining
}
// Example: [1,2,3,4,5], k=2 → [4,5,1,2,3]
```

### Using Extra Array (Simple but O(n) space)
```java
public static int[] leftRotateExtraSpace(int[] arr, int k) {
    int n = arr.length;
    k = k % n;
    int[] result = new int[n];
    for (int i = 0; i < n; i++)
        result[i] = arr[(i + k) % n];
    return result;
}
```

### Check if Array is a Rotation of Another
```java
public static boolean isRotation(int[] arr1, int[] arr2) {
    if (arr1.length != arr2.length) return false;
    int[] doubled = new int[arr1.length * 2];
    for (int i = 0; i < arr1.length; i++) {
        doubled[i] = arr1[i];
        doubled[i + arr1.length] = arr1[i];
    }
    return Arrays.toString(doubled).contains(Arrays.toString(arr2)
        .replace("[", "").replace("]", ""));
}
```

### Interview Traps
- Always `k = k % n` first — rotating by `n` is a no-op.
- Left rotate by `k` = Right rotate by `n - k`.
- Reversal algorithm order matters: remember which segment you reverse first.

---

## 6. Palindrome Check

### Key Concepts
- **String palindrome**: same forwards and backwards (`"racecar"`).
- **Array palindrome**: symmetric around center.
- Two-pointer: compare `arr[left]` and `arr[right]`.
- Ignore case/non-alphanumeric for interview "valid palindrome" variants.

### String Palindrome — Two Pointer
```java
public static boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) return false;
        left++; right--;
    }
    return true;
}
```

### Valid Palindrome (Ignore Non-Alphanumeric, Case-Insensitive)
```java
// LeetCode 125 variant
public static boolean isValidPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) left++;
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) right--;
        if (Character.toLowerCase(s.charAt(left)) !=
            Character.toLowerCase(s.charAt(right))) return false;
        left++; right--;
    }
    return true;
}
```

### Palindrome Using StringBuilder
```java
public static boolean isPalindromeSimple(String s) {
    String rev = new StringBuilder(s).reverse().toString();
    return s.equals(rev);  // O(n) space
}
```

### Integer Palindrome (No String Conversion)
```java
public static boolean isPalindromeInt(int x) {
    if (x < 0 || (x % 10 == 0 && x != 0)) return false;
    int reversed = 0;
    while (x > reversed) {
        reversed = reversed * 10 + x % 10;
        x /= 10;
    }
    return x == reversed || x == reversed / 10;
}
```

### Array Palindrome
```java
public static boolean isArrayPalindrome(int[] arr) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        if (arr[left] != arr[right]) return false;
        left++; right--;
    }
    return true;
}
```

### Longest Palindromic Substring — Expand Around Center
```java
public static String longestPalindrome(String s) {
    if (s == null || s.length() < 1) return "";
    int start = 0, end = 0;

    for (int i = 0; i < s.length(); i++) {
        int len1 = expand(s, i, i);      // odd length
        int len2 = expand(s, i, i + 1);  // even length
        int len = Math.max(len1, len2);

        if (len > end - start) {
            start = i - (len - 1) / 2;
            end = i + len / 2;
        }
    }
    return s.substring(start, end + 1);
}

private static int expand(String s, int left, int right) {
    while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
        left--; right++;
    }
    return right - left - 1;
}
```

### Interview Traps
- `"A man, a plan, a canal: Panama"` → must strip spaces/punctuation + lowercase before checking.
- Even-length palindromes: `"abba"` has no single center — expand-around-center handles both cases.
- Integer palindrome: negative numbers are **never** palindromes.
- `StringBuilder.reverse()` is O(n) space — two-pointer is preferred for optimal space.

---

### Arrays & Strings — Complexity Reference

| Operation | Time | Space |
|---|---|---|
| Insert/Delete (array) | O(n) | O(1) |
| Reversal (two-pointer) | O(n) | O(1) |
| Matrix Transpose | O(n²) | O(1) |
| Spiral Traversal | O(n*m) | O(1) |
| Naive Pattern Match | O(n*m) | O(1) |
| KMP Pattern Match | O(n+m) | O(m) |
| Array Rotation (reversal) | O(n) | O(1) |
| Palindrome Check | O(n) | O(1) |
| Longest Palindrome Substring | O(n²) | O(1) |

---

# Part 2 — Linked Lists

---

## 1. Singly Linked List

### Key Concepts
- Each node holds **data + one pointer** (`next`).
- Head pointer is the only access point — lose it, lose the list.
- No random access — must traverse from head → **O(n)** lookup.
- Insert/Delete at head → **O(1)**; at tail/middle → **O(n)**.

### Node Definition
```java
class ListNode {
    int val;
    ListNode next;

    ListNode(int val) {
        this.val = val;
        this.next = null;
    }
}
```

### Build a Linked List
```java
public class SinglyLinkedList {
    ListNode head;

    // Insert at head — O(1)
    public void insertAtHead(int val) {
        ListNode node = new ListNode(val);
        node.next = head;
        head = node;
    }

    // Insert at tail — O(n)
    public void insertAtTail(int val) {
        ListNode node = new ListNode(val);
        if (head == null) { head = node; return; }
        ListNode curr = head;
        while (curr.next != null) curr = curr.next;
        curr.next = node;
    }

    // Insert at index — O(n)
    public void insertAt(int index, int val) {
        if (index == 0) { insertAtHead(val); return; }
        ListNode curr = head;
        for (int i = 0; i < index - 1; i++) {
            if (curr == null) throw new IndexOutOfBoundsException();
            curr = curr.next;
        }
        ListNode node = new ListNode(val);
        node.next = curr.next;
        curr.next = node;
    }

    // Delete by value — O(n)
    public void delete(int val) {
        if (head == null) return;
        if (head.val == val) { head = head.next; return; }

        ListNode curr = head;
        while (curr.next != null) {
            if (curr.next.val == val) {
                curr.next = curr.next.next;
                return;
            }
            curr = curr.next;
        }
    }

    // Search — O(n)
    public boolean search(int val) {
        ListNode curr = head;
        while (curr != null) {
            if (curr.val == val) return true;
            curr = curr.next;
        }
        return false;
    }
}
```

### Interview Traps
- **Null check before accessing `.next`** — most NullPointerExceptions come from forgetting this.
- When deleting, you need the node **before** the target — track `prev` or use `curr.next` lookahead.
- Always check empty list (`head == null`) as a base case.

---

## 2. Doubly Linked List

### Key Concepts
- Each node holds **data + `next` + `prev`** pointers.
- Allows **O(1)** insert/delete if you already have the node reference.
- Used internally by Java's `LinkedList` class.

### Node Definition
```java
class DListNode {
    int val;
    DListNode next;
    DListNode prev;

    DListNode(int val) {
        this.val = val;
        this.next = null;
        this.prev = null;
    }
}
```

### Core Operations
```java
public class DoublyLinkedList {
    DListNode head, tail;

    // Insert at head — O(1)
    public void insertAtHead(int val) {
        DListNode node = new DListNode(val);
        if (head == null) { head = tail = node; return; }
        node.next = head;
        head.prev = node;
        head = node;
    }

    // Insert at tail — O(1)
    public void insertAtTail(int val) {
        DListNode node = new DListNode(val);
        if (tail == null) { head = tail = node; return; }
        tail.next = node;
        node.prev = tail;
        tail = node;
    }

    // Delete a node (given direct reference) — O(1)
    public void deleteNode(DListNode node) {
        if (node == null) return;
        if (node.prev != null) node.prev.next = node.next;
        else head = node.next;

        if (node.next != null) node.next.prev = node.prev;
        else tail = node.prev;

        node.next = node.prev = null;
    }
}
```

### LRU Cache (Classic DLL Interview Problem)
```java
class LRUCache {
    private final int capacity;
    private final Map<Integer, DListNode> map = new HashMap<>();
    private final DListNode head = new DListNode(0); // dummy head
    private final DListNode tail = new DListNode(0); // dummy tail

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        DListNode node = map.get(key);
        remove(node);
        insertFront(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) remove(map.get(key));
        if (map.size() == capacity) {
            map.remove(tail.prev.val);
            remove(tail.prev);
        }
        DListNode node = new DListNode(value);
        map.put(key, node);
        insertFront(node);
    }

    private void remove(DListNode node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void insertFront(DListNode node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
```

### Interview Traps
- **Always update BOTH `next` and `prev`** on every insert/delete.
- Dummy head/tail nodes eliminate null checks and simplify boundary cases.

---

## 3. Cycle Detection

### Key Concepts
- **Floyd's Tortoise and Hare**: slow moves 1 step, fast moves 2 steps.
- If a cycle exists, fast and slow **will meet** inside the cycle.
- Time: **O(n)**, Space: **O(1)**.

### Detect Cycle (Boolean)
```java
public static boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

### Find Cycle Start Node
```java
public static ListNode detectCycleStart(ListNode head) {
    ListNode slow = head, fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) break;
    }

    if (fast == null || fast.next == null) return null;

    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next;
    }
    return slow;
}
```

### Find Cycle Length
```java
public static int cycleLength(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            int length = 1;
            ListNode curr = slow.next;
            while (curr != slow) { curr = curr.next; length++; }
            return length;
        }
    }
    return 0;
}
```

### HashSet Approach (Simpler, O(n) space)
```java
public static boolean hasCycleHashSet(ListNode head) {
    Set<ListNode> visited = new HashSet<>();
    ListNode curr = head;
    while (curr != null) {
        if (visited.contains(curr)) return true;
        visited.add(curr);
        curr = curr.next;
    }
    return false;
}
```

### Interview Traps
- Check `fast != null && fast.next != null` — skipping either causes NPE.
- Meeting point ≠ cycle start — must do Phase 2 to find the actual entry node.

---

## 4. List Reversal

### Key Concepts
- Iterative uses 3 pointers: `prev`, `curr`, `next` — **O(1) space**.
- Recursive uses call stack — **O(n) space**.

### Iterative Reversal
```java
public static ListNode reverse(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;

    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;  // prev is now the new head
}
```

### Recursive Reversal
```java
public static ListNode reverseRecursive(ListNode head) {
    if (head == null || head.next == null) return head;

    ListNode newHead = reverseRecursive(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}
```

### Reverse Between Positions (l to r) — LeetCode 92
```java
public static ListNode reverseBetween(ListNode head, int l, int r) {
    if (head == null) return null;
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode prev = dummy;

    for (int i = 1; i < l; i++) prev = prev.next;

    ListNode curr = prev.next;
    for (int i = 0; i < r - l; i++) {
        ListNode next = curr.next;
        curr.next = next.next;
        next.next = prev.next;
        prev.next = next;
    }
    return dummy.next;
}
```

### Reverse in K-Groups — LeetCode 25
```java
public static ListNode reverseKGroup(ListNode head, int k) {
    ListNode curr = head;
    int count = 0;
    while (curr != null && count < k) { curr = curr.next; count++; }
    if (count < k) return head;

    ListNode prev = null; curr = head;
    for (int i = 0; i < k; i++) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    head.next = reverseKGroup(curr, k);
    return prev;
}
```

### Interview Traps
- After iterative reversal, `curr` is `null` — return `prev`, not `curr`.
- Forgetting `head.next = null` in recursive reversal creates a cycle.
- Dummy node is essential for `reverseBetween` to handle `l = 1` cleanly.

---

## 5. Fast / Slow Pointers

### Key Concepts
- Fast moves 2x speed of slow.
- Useful for: cycle detection, finding middle, kth from end, palindrome check.
- All solutions: **O(n)** time, **O(1)** space.

### Find Middle of Linked List
```java
public static ListNode findMiddle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

### Find Kth Node from End
```java
public static ListNode kthFromEnd(ListNode head, int k) {
    ListNode fast = head, slow = head;
    for (int i = 0; i < k; i++) {
        if (fast == null) return null;
        fast = fast.next;
    }
    while (fast != null) {
        slow = slow.next;
        fast = fast.next;
    }
    return slow;
}
```

### Delete Kth Node from End — LeetCode 19
```java
public static ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode fast = dummy, slow = dummy;

    for (int i = 0; i <= n; i++) fast = fast.next;

    while (fast != null) {
        slow = slow.next;
        fast = fast.next;
    }
    slow.next = slow.next.next;
    return dummy.next;
}
```

### Check Linked List Palindrome
```java
public static boolean isPalindrome(ListNode head) {
    if (head == null || head.next == null) return true;

    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    ListNode secondHalf = reverse(slow);

    ListNode first = head;
    ListNode second = secondHalf;
    while (second != null) {
        if (first.val != second.val) return false;
        first = first.next;
        second = second.next;
    }
    return true;
}
```

### Interview Traps
- `fast.next != null` check is **mandatory** before `fast.next.next`.
- For even-length lists, two middles exist — clarify which one is expected.
- `k > length` edge case: guard against `fast == null` during initial advance.

---

## 6. Merge Lists

### Key Concepts
- **Merge two sorted lists**: O(m + n), use dummy node.
- **Merge k sorted lists**: use min-heap O(N log k) or divide-and-conquer.

### Merge Two Sorted Lists — Iterative
```java
public static ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode curr = dummy;

    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) {
            curr.next = l1; l1 = l1.next;
        } else {
            curr.next = l2; l2 = l2.next;
        }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

### Merge Two Sorted Lists — Recursive
```java
public static ListNode mergeTwoListsRecursive(ListNode l1, ListNode l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;

    if (l1.val <= l2.val) {
        l1.next = mergeTwoListsRecursive(l1.next, l2);
        return l1;
    } else {
        l2.next = mergeTwoListsRecursive(l1, l2.next);
        return l2;
    }
}
```

### Merge K Sorted Lists — Min-Heap
```java
public static ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);

    for (ListNode node : lists)
        if (node != null) pq.offer(node);

    ListNode dummy = new ListNode(0);
    ListNode curr = dummy;

    while (!pq.isEmpty()) {
        ListNode node = pq.poll();
        curr.next = node;
        curr = curr.next;
        if (node.next != null) pq.offer(node.next);
    }
    return dummy.next;
}
```

### Merge Sort a Linked List
```java
public static ListNode mergeSort(ListNode head) {
    if (head == null || head.next == null) return head;

    ListNode slow = head, fast = head.next;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    ListNode mid = slow.next;
    slow.next = null;

    ListNode left = mergeSort(head);
    ListNode right = mergeSort(mid);
    return mergeTwoLists(left, right);
}
```

### Intersection of Two Lists
```java
public static ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    ListNode a = headA, b = headB;
    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }
    return a;
}
```

### Interview Traps
- `curr.next = (l1 != null) ? l1 : l2` — just attach the rest, don't loop.
- PriorityQueue comparator: use `Integer.compare(a.val, b.val)` to avoid overflow.
- Merge sort: **cut the link** (`slow.next = null`) before recursing.

---

### Linked Lists — Complexity Reference

| Operation | Time | Space |
|---|---|---|
| SLL insert at head | O(1) | O(1) |
| SLL insert/delete at index | O(n) | O(1) |
| DLL delete given node ref | O(1) | O(1) |
| Cycle detection (Floyd's) | O(n) | O(1) |
| List reversal (iterative) | O(n) | O(1) |
| List reversal (recursive) | O(n) | O(n) |
| Find middle (fast/slow) | O(n) | O(1) |
| Merge two sorted lists | O(m+n) | O(1) |
| Merge k sorted lists (heap) | O(N log k) | O(k) |
| Merge sort linked list | O(n log n) | O(log n) |

---

# Part 3 — Stacks & Queues

---

## 1. Stack Implementation

### Key Concepts
- **LIFO** — Last In, First Out.
- Core ops: `push`, `pop`, `peek`, `isEmpty` — all **O(1)**.
- Java: prefer `Deque` (`ArrayDeque`) over legacy `Stack` class.

### Array-Based Stack
```java
public class ArrayStack {
    private int[] data;
    private int top;
    private int capacity;

    public ArrayStack(int capacity) {
        this.capacity = capacity;
        data = new int[capacity];
        top = -1;
    }

    public void push(int val) {
        if (top == capacity - 1) throw new RuntimeException("Stack overflow");
        data[++top] = val;
    }

    public int pop() {
        if (isEmpty()) throw new RuntimeException("Stack underflow");
        return data[top--];
    }

    public int peek() {
        if (isEmpty()) throw new RuntimeException("Stack is empty");
        return data[top];
    }

    public boolean isEmpty() { return top == -1; }
    public int size()        { return top + 1; }
}
```

### LinkedList-Based Stack (Dynamic)
```java
public class LinkedStack {
    private ListNode top;
    private int size;

    public void push(int val) {
        ListNode node = new ListNode(val);
        node.next = top;
        top = node;
        size++;
    }

    public int pop() {
        if (isEmpty()) throw new RuntimeException("Stack underflow");
        int val = top.val;
        top = top.next;
        size--;
        return val;
    }

    public int peek()        { return top.val; }
    public boolean isEmpty() { return top == null; }
    public int size()        { return size; }
}
```

### Java Built-in (Interview Preferred)
```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.push(2);
stack.push(3);
int top = stack.peek();    // 3
int val = stack.pop();     // 3
boolean empty = stack.isEmpty();
```

### Min Stack — O(1) getMin
```java
class MinStack {
    private Deque<Integer> stack    = new ArrayDeque<>();
    private Deque<Integer> minStack = new ArrayDeque<>();

    public void push(int val) {
        stack.push(val);
        if (minStack.isEmpty() || val <= minStack.peek())
            minStack.push(val);
    }

    public void pop() {
        int val = stack.pop();
        if (val == minStack.peek()) minStack.pop();
    }

    public int top()    { return stack.peek(); }
    public int getMin() { return minStack.peek(); }
}
```

### Interview Traps
- `top = -1` sentinel is critical — forgetting it causes wrong `isEmpty()` logic.
- MinStack: push to minStack when `val <= min` (not just `<`) to handle **duplicate minimums**.
- `ArrayDeque` is NOT thread-safe — use `ConcurrentHashMap` in multithreaded contexts.

---

## 2. Queue Implementation

### Key Concepts
- **FIFO** — First In, First Out.
- Core ops: `enqueue`, `dequeue`, `peek` — all **O(1)**.

### LinkedList-Based Queue
```java
public class LinkedQueue {
    private ListNode front, rear;
    private int size;

    public void enqueue(int val) {
        ListNode node = new ListNode(val);
        if (rear != null) rear.next = node;
        rear = node;
        if (front == null) front = node;
        size++;
    }

    public int dequeue() {
        if (isEmpty()) throw new RuntimeException("Queue underflow");
        int val = front.val;
        front = front.next;
        if (front == null) rear = null;
        size--;
        return val;
    }

    public int peek()        { return front.val; }
    public boolean isEmpty() { return front == null; }
    public int size()        { return size; }
}
```

### Java Built-in Queue
```java
Queue<Integer> queue = new ArrayDeque<>();
queue.offer(1);
queue.offer(2);
queue.offer(3);
int front = queue.peek(); // 1
int val   = queue.poll(); // 1

// PriorityQueue — min-heap by default
Queue<Integer> pq = new PriorityQueue<>();
pq.offer(5); pq.offer(1); pq.offer(3);
pq.poll();   // returns 1

// Max-heap
Queue<Integer> maxPQ = new PriorityQueue<>(Collections.reverseOrder());
```

### Queue Using Two Stacks
```java
class QueueUsingStacks {
    private Deque<Integer> inbox  = new ArrayDeque<>();
    private Deque<Integer> outbox = new ArrayDeque<>();

    public void enqueue(int val) { inbox.push(val); }

    public int dequeue() {
        refill();
        return outbox.pop();
    }

    public int peek() {
        refill();
        return outbox.peek();
    }

    private void refill() {
        if (outbox.isEmpty())
            while (!inbox.isEmpty())
                outbox.push(inbox.pop());
    }

    public boolean isEmpty() { return inbox.isEmpty() && outbox.isEmpty(); }
}
// Amortized O(1) per operation
```

### Stack Using Two Queues
```java
class StackUsingQueues {
    private Queue<Integer> q1 = new ArrayDeque<>();
    private Queue<Integer> q2 = new ArrayDeque<>();

    public void push(int val) {
        q2.offer(val);
        while (!q1.isEmpty()) q2.offer(q1.poll());
        Queue<Integer> tmp = q1; q1 = q2; q2 = tmp;
    }

    public int pop()  { return q1.poll(); }
    public int peek() { return q1.peek(); }
    public boolean isEmpty() { return q1.isEmpty(); }
}
// Push: O(n), Pop/Peek: O(1)
```

### Interview Traps
- `offer` vs `add`: `offer` returns `false` on failure; `add` throws — use `offer`.
- `poll` vs `remove`: `poll` returns `null` on empty; `remove` throws — use `poll`.
- Two-stack queue: only refill outbox when it's **empty**.

---

## 3. Parentheses Matching

### Key Concepts
- Push open brackets, pop and verify on closing brackets.
- Time: **O(n)**, Space: **O(n)**.

### Basic Parentheses Matching
```java
public static boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();

    for (char c : s.toCharArray()) {
        if (c == '(' || c == '{' || c == '[') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            char top = stack.pop();
            if (c == ')' && top != '(') return false;
            if (c == '}' && top != '{') return false;
            if (c == ']' && top != '[') return false;
        }
    }
    return stack.isEmpty();
}
```

### Cleaner with Map
```java
public static boolean isValidMap(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> map = Map.of(')', '(', '}', '{', ']', '[');

    for (char c : s.toCharArray()) {
        if (map.containsValue(c)) {
            stack.push(c);
        } else if (map.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != map.get(c))
                return false;
        }
    }
    return stack.isEmpty();
}
```

### Minimum Removals to Make Valid
```java
public static int minRemoveToMakeValid(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    Set<Integer> remove = new HashSet<>();

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (c == '(') {
            stack.push(i);
        } else if (c == ')') {
            if (stack.isEmpty()) remove.add(i);
            else stack.pop();
        }
    }
    while (!stack.isEmpty()) remove.add(stack.pop());

    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < s.length(); i++)
        if (!remove.contains(i)) sb.append(s.charAt(i));
    return remove.size();
}
```

### Longest Valid Parentheses Substring
```java
public static int longestValidParentheses(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(-1);
    int maxLen = 0;

    for (int i = 0; i < s.length(); i++) {
        if (s.charAt(i) == '(') {
            stack.push(i);
        } else {
            stack.pop();
            if (stack.isEmpty()) stack.push(i);
            else maxLen = Math.max(maxLen, i - stack.peek());
        }
    }
    return maxLen;
}
```

### Interview Traps
- Empty string `""` → return `true`.
- `"([)]"` → **invalid** — nesting order matters.
- After loop, forgetting `stack.isEmpty()` misses unclosed openers like `"((("`.
- Push index (not char) when position info is needed.

---

## 4. Expression Evaluation

### Key Concepts
- **Infix**: `3 + 4 * 2` | **Postfix (RPN)**: `3 4 2 * +` | **Prefix**: `+ 3 * 4 2`.
- Postfix is stack-evaluable without precedence rules.
- Two-stack approach evaluates infix directly.

### Evaluate Postfix Expression
```java
public static int evalPostfix(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();

    for (String token : tokens) {
        switch (token) {
            case "+": stack.push(stack.pop() + stack.pop()); break;
            case "*": stack.push(stack.pop() * stack.pop()); break;
            case "-": {
                int b = stack.pop(), a = stack.pop();
                stack.push(a - b); break;
            }
            case "/": {
                int b = stack.pop(), a = stack.pop();
                stack.push(a / b); break;
            }
            default: stack.push(Integer.parseInt(token));
        }
    }
    return stack.pop();
}
// ["2","1","+","3","*"] → (2+1)*3 = 9
```

### Infix → Postfix (Shunting Yard Algorithm)
```java
public static String infixToPostfix(String expr) {
    Deque<Character> opStack = new ArrayDeque<>();
    StringBuilder output = new StringBuilder();

    Map<Character, Integer> precedence = Map.of(
        '+', 1, '-', 1, '*', 2, '/', 2, '^', 3
    );

    for (char c : expr.toCharArray()) {
        if (Character.isDigit(c)) {
            output.append(c).append(' ');
        } else if (c == '(') {
            opStack.push(c);
        } else if (c == ')') {
            while (!opStack.isEmpty() && opStack.peek() != '(')
                output.append(opStack.pop()).append(' ');
            opStack.pop();
        } else if (precedence.containsKey(c)) {
            while (!opStack.isEmpty() && opStack.peek() != '(' &&
                   precedence.getOrDefault(opStack.peek(), 0) >= precedence.get(c))
                output.append(opStack.pop()).append(' ');
            opStack.push(c);
        }
    }
    while (!opStack.isEmpty())
        output.append(opStack.pop()).append(' ');

    return output.toString().trim();
}
// "3+4*2" → "3 4 2 * +"
```

### Evaluate Infix Directly — Two Stacks
```java
public static int evalInfix(String s) {
    Deque<Integer> nums = new ArrayDeque<>();
    Deque<Character> ops = new ArrayDeque<>();
    Map<Character, Integer> prec = Map.of('+', 1, '-', 1, '*', 2, '/', 2);

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (c == ' ') continue;

        if (Character.isDigit(c)) {
            int num = 0;
            while (i < s.length() && Character.isDigit(s.charAt(i)))
                num = num * 10 + (s.charAt(i++) - '0');
            i--;
            nums.push(num);
        } else if (c == '(') {
            ops.push(c);
        } else if (c == ')') {
            while (ops.peek() != '(') apply(nums, ops);
            ops.pop();
        } else if (prec.containsKey(c)) {
            while (!ops.isEmpty() && ops.peek() != '(' &&
                   prec.getOrDefault(ops.peek(), 0) >= prec.get(c))
                apply(nums, ops);
            ops.push(c);
        }
    }
    while (!ops.isEmpty()) apply(nums, ops);
    return nums.pop();
}

private static void apply(Deque<Integer> nums, Deque<Character> ops) {
    int b = nums.pop(), a = nums.pop();
    char op = ops.pop();
    switch (op) {
        case '+': nums.push(a + b); break;
        case '-': nums.push(a - b); break;
        case '*': nums.push(a * b); break;
        case '/': nums.push(a / b); break;
    }
}
```

### Interview Traps
- For `-` and `/`, order matters: pop `b` first, then `a` → compute `a op b`.
- Multi-digit numbers: accumulate digits in a loop.
- Right-associative operators (like `^`): change `>=` to `>` in precedence comparison.

---

## 5. Next Greater Element

### Key Concepts
- **Monotonic stack**: maintains elements in increasing or decreasing order.
- For NGE: maintain a **decreasing stack** of indices.
- Time: **O(n)** — each element pushed and popped at most once.

### Next Greater Element — Basic
```java
public static int[] nextGreaterElement(int[] arr) {
    int n = arr.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && arr[stack.peek()] < arr[i])
            result[stack.pop()] = arr[i];
        stack.push(i);
    }
    return result;
}
// [2,1,2,4,3] → [4,2,4,-1,-1]
```

### Next Greater Element in Circular Array — LeetCode 503
```java
public static int[] nextGreaterCircular(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < 2 * n; i++) {
        int idx = i % n;
        while (!stack.isEmpty() && nums[stack.peek()] < nums[idx])
            result[stack.pop()] = nums[idx];
        if (i < n) stack.push(idx);
    }
    return result;
}
// [1,2,1] → [2,-1,2]
```

### Next Smaller Element
```java
public static int[] nextSmallerElement(int[] arr) {
    int n = arr.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && arr[stack.peek()] > arr[i])
            result[stack.pop()] = arr[i];
        stack.push(i);
    }
    return result;
}
```

### Largest Rectangle in Histogram
```java
public static int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>();
    int maxArea = 0;
    int n = heights.length;

    for (int i = 0; i <= n; i++) {
        int currHeight = (i == n) ? 0 : heights[i];
        while (!stack.isEmpty() && heights[stack.peek()] > currHeight) {
            int height = heights[stack.pop()];
            int width  = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
// [2,1,5,6,2,3] → 10
```

### Interview Traps
- Store **indices** in stack, not values.
- `Arrays.fill(result, -1)` before loop — elements with no NGE stay `-1`.
- Circular array: push indices only on first pass (`i < n`).
- Largest Rectangle: append sentinel `0` to flush remaining stack elements.

---

## 6. Circular Queue

### Key Concepts
- Fixed-size array where `rear` wraps using **modulo arithmetic**.
- Avoids "false full" problem of linear array queues.
- All operations: **O(1)**.

### Circular Queue — Size Counter Approach
```java
public class CircularQueue {
    private int[] data;
    private int front, rear, size, capacity;

    public CircularQueue(int capacity) {
        this.capacity = capacity;
        data  = new int[capacity];
        front = 0; rear = 0; size = 0;
    }

    public boolean enqueue(int val) {
        if (isFull()) return false;
        data[rear] = val;
        rear = (rear + 1) % capacity;
        size++;
        return true;
    }

    public int dequeue() {
        if (isEmpty()) throw new RuntimeException("Queue is empty");
        int val = data[front];
        front = (front + 1) % capacity;
        size--;
        return val;
    }

    public int peek()        { return data[front]; }
    public boolean isEmpty() { return size == 0; }
    public boolean isFull()  { return size == capacity; }
    public int size()        { return size; }
}
```

### Circular Deque (Double-Ended Queue)
```java
public class CircularDeque {
    private int[] data;
    private int front, rear, size, capacity;

    public CircularDeque(int capacity) {
        this.capacity = capacity;
        data  = new int[capacity];
        front = 0; rear = 0; size = 0;
    }

    public void addFront(int val) {
        if (isFull()) throw new RuntimeException("Deque full");
        front = (front - 1 + capacity) % capacity;  // wrap backward
        data[front] = val;
        size++;
    }

    public void addRear(int val) {
        if (isFull()) throw new RuntimeException("Deque full");
        data[rear] = val;
        rear = (rear + 1) % capacity;
        size++;
    }

    public int removeFront() {
        if (isEmpty()) throw new RuntimeException("Deque empty");
        int val = data[front];
        front = (front + 1) % capacity;
        size--;
        return val;
    }

    public int removeRear() {
        if (isEmpty()) throw new RuntimeException("Deque empty");
        rear = (rear - 1 + capacity) % capacity;
        int val = data[rear];
        size--;
        return val;
    }

    public boolean isEmpty() { return size == 0; }
    public boolean isFull()  { return size == capacity; }
}
```

### Sliding Window Maximum using Deque
```java
public static int[] maxSlidingWindow(int[] nums, int k) {
    Deque<Integer> deque = new ArrayDeque<>();
    int[] result = new int[nums.length - k + 1];
    int ri = 0;

    for (int i = 0; i < nums.length; i++) {
        while (!deque.isEmpty() && deque.peekFirst() < i - k + 1)
            deque.pollFirst();

        while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i])
            deque.pollLast();

        deque.offerLast(i);

        if (i >= k - 1) result[ri++] = nums[deque.peekFirst()];
    }
    return result;
}
// [1,3,-1,-3,5,3,6,7], k=3 → [3,3,5,5,6,7]
```

### Interview Traps
- Moving `front` backward: `(front - 1 + capacity) % capacity` — `+ capacity` prevents negative modulo in Java.
- `isFull` with no size counter: `(rear + 1) % capacity == front`.
- Sliding window: store **indices** not values in deque.

---

### Stacks & Queues — Complexity Reference

| Operation | Array Stack | Linked Stack | Circular Queue | Priority Queue |
|---|---|---|---|---|
| Push / Enqueue | O(1) | O(1) | O(1) | O(log n) |
| Pop / Dequeue | O(1) | O(1) | O(1) | O(log n) |
| Peek | O(1) | O(1) | O(1) | O(1) |
| Search | O(n) | O(n) | O(n) | O(n) |
| NGE (monotonic) | O(n) total | — | — | — |
| Sliding Window Max | O(n) | — | — | — |

---

# Part 4 — Hashing & Heaps

---

## 1. HashMap / HashSet

### Key Concepts
- **HashMap**: key-value pairs, keys unique, one `null` key allowed, **O(1)** average get/put.
- **HashSet**: unique elements, backed by HashMap, **O(1)** average add/contains.
- Java 8+: bucket chains → **Red-Black Tree** when chain length > 8 → worst case **O(log n)**.
- Default load factor: **0.75** — resizes when 75% full.

### HashMap Core Operations
```java
Map<String, Integer> map = new HashMap<>();

map.put("apple", 3);
map.put("banana", 5);
map.put("apple", 7);                            // overwrites

int val = map.get("apple");                     // 7
int def = map.getOrDefault("cherry", 0);        // 0

boolean hasKey = map.containsKey("apple");      // true
map.remove("banana");

// Iterate — entrySet is most efficient
for (Map.Entry<String, Integer> entry : map.entrySet())
    System.out.println(entry.getKey() + " = " + entry.getValue());

map.putIfAbsent("apple", 99);                   // no-op
map.compute("apple", (k, v) -> v == null ? 1 : v + 1);
map.merge("apple", 1, Integer::sum);            // apple += 1
```

### HashSet Core Operations
```java
Set<Integer> set = new HashSet<>();
set.add(1); set.add(2); set.add(2);  // duplicate ignored
set.contains(2);  // true
set.remove(1);

// Set operations
Set<Integer> a = new HashSet<>(Arrays.asList(1, 2, 3));
Set<Integer> b = new HashSet<>(Arrays.asList(2, 3, 4));

Set<Integer> intersect = new HashSet<>(a); intersect.retainAll(b);  // {2,3}
Set<Integer> union = new HashSet<>(a);     union.addAll(b);          // {1,2,3,4}
Set<Integer> diff = new HashSet<>(a);      diff.removeAll(b);        // {1}
```

### LinkedHashMap — Insertion Order Preserved
```java
Map<String, Integer> linked = new LinkedHashMap<>();
linked.put("c", 3); linked.put("a", 1); linked.put("b", 2);
// Iterates as: c, a, b

// Access-order mode (LRU behavior)
Map<String, Integer> lru = new LinkedHashMap<>(16, 0.75f, true);
```

### TreeMap — Sorted by Key
```java
TreeMap<String, Integer> sorted = new TreeMap<>();
sorted.put("banana", 2); sorted.put("apple", 1); sorted.put("cherry", 3);
// Iterates: apple, banana, cherry

sorted.firstKey();          // "apple"
sorted.lastKey();           // "cherry"
sorted.floorKey("b");       // "banana"
sorted.ceilingKey("b");     // "banana"
sorted.headMap("b");        // {apple}
sorted.tailMap("b");        // {banana, cherry}
```

### Interview Traps
- `HashMap` is **not thread-safe** — use `ConcurrentHashMap` for multithreading.
- Custom key objects must override both `hashCode()` and `equals()`.
- `map.get(key)` returns `null` if absent — unboxing to `int` throws NPE.
- Modifying map during for-each throws `ConcurrentModificationException`.

---

## 2. Coding Hash-Based Problems

### Frequency Counting
```java
public static Map<Character, Integer> charFrequency(String s) {
    Map<Character, Integer> freq = new HashMap<>();
    for (char c : s.toCharArray())
        freq.merge(c, 1, Integer::sum);
    return freq;
}

// Top K frequent elements — LeetCode 347
public static int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);

    List<Integer>[] buckets = new List[nums.length + 1];
    for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
        int f = e.getValue();
        if (buckets[f] == null) buckets[f] = new ArrayList<>();
        buckets[f].add(e.getKey());
    }

    int[] result = new int[k];
    int idx = 0;
    for (int i = buckets.length - 1; i >= 0 && idx < k; i--)
        if (buckets[i] != null)
            for (int num : buckets[i])
                if (idx < k) result[idx++] = num;
    return result;
}
```

### Two Sum — HashMap Classic
```java
public static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement))
            return new int[]{seen.get(complement), i};
        seen.put(nums[i], i);
    }
    return new int[]{};
}
```

### Anagram Check & Grouping
```java
public static boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    Map<Character, Integer> map = new HashMap<>();
    for (char c : s.toCharArray()) map.merge(c, 1, Integer::sum);
    for (char c : t.toCharArray()) {
        map.merge(c, -1, Integer::sum);
        if (map.get(c) < 0) return false;
    }
    return true;
}

// Group anagrams — LeetCode 49
public static List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
```

### Subarray Sum Equals K — Prefix Sum + HashMap
```java
// LeetCode 560
public static int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0, 1);  // empty subarray
    int count = 0, sum = 0;

    for (int num : nums) {
        sum += num;
        count += prefixCount.getOrDefault(sum - k, 0);
        prefixCount.merge(sum, 1, Integer::sum);
    }
    return count;
}
```

### Longest Consecutive Sequence — O(n)
```java
// LeetCode 128
public static int longestConsecutive(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int n : nums) set.add(n);

    int longest = 0;
    for (int n : set) {
        if (!set.contains(n - 1)) {  // only start from sequence beginners
            int curr = n, length = 1;
            while (set.contains(curr + 1)) { curr++; length++; }
            longest = Math.max(longest, length);
        }
    }
    return longest;
}
```

### Interview Traps
- Prefix sum: initialize `prefixCount.put(0, 1)` — missing this skips subarrays from index 0.
- Consecutive sequence: only begin counting when `n - 1` is absent — else O(n²) worst case.
- `computeIfAbsent` is cleaner than manual null check for map of lists.

---

## 3. Min Heap / Max Heap

### Key Concepts
- **Min-Heap**: parent ≤ children → root is minimum.
- **Max-Heap**: parent ≥ children → root is maximum.
- Array representation: left child = `2i+1`, right child = `2i+2`, parent = `(i-1)/2`.
- Insert (sift-up): **O(log n)**. Extract root (sift-down): **O(log n)**. Build heap: **O(n)**.

### Manual Min Heap Implementation
```java
public class MinHeap {
    private int[] data;
    private int size, capacity;

    public MinHeap(int capacity) {
        this.capacity = capacity;
        data = new int[capacity];
        size = 0;
    }

    private int parent(int i)    { return (i - 1) / 2; }
    private int leftChild(int i) { return 2 * i + 1; }
    private int rightChild(int i){ return 2 * i + 2; }

    private void swap(int i, int j) {
        int tmp = data[i]; data[i] = data[j]; data[j] = tmp;
    }

    public void insert(int val) {
        if (size == capacity) throw new RuntimeException("Heap full");
        data[size] = val;
        siftUp(size);
        size++;
    }

    private void siftUp(int i) {
        while (i > 0 && data[parent(i)] > data[i]) {
            swap(i, parent(i));
            i = parent(i);
        }
    }

    public int extractMin() {
        if (size == 0) throw new RuntimeException("Heap empty");
        int min = data[0];
        data[0] = data[--size];
        siftDown(0);
        return min;
    }

    private void siftDown(int i) {
        while (true) {
            int smallest = i;
            int l = leftChild(i), r = rightChild(i);
            if (l < size && data[l] < data[smallest]) smallest = l;
            if (r < size && data[r] < data[smallest]) smallest = r;
            if (smallest == i) break;
            swap(i, smallest);
            i = smallest;
        }
    }

    public int peek() { return data[0]; }

    // Build heap from array — O(n)
    public void buildHeap(int[] arr) {
        data = Arrays.copyOf(arr, capacity);
        size = arr.length;
        for (int i = size / 2 - 1; i >= 0; i--)
            siftDown(i);
    }
}
```

### Interview Traps
- `buildHeap` starts at `size/2 - 1` (last non-leaf).
- `buildHeap` is **O(n)**, not O(n log n) — most nodes are leaves and cost O(1).
- Index formula: 0-indexed → parent `(i-1)/2`; 1-indexed → parent `i/2`.

---

## 4. Priority Queue

### Key Concepts
- Java's `PriorityQueue` is a **min-heap** by default.
- **O(log n)** offer/poll, **O(1)** peek, **O(n)** contains.
- Does **not** guarantee order on iteration — only `poll()` gives sorted order.

### Basic Usage
```java
// Min-heap
PriorityQueue<Integer> minPQ = new PriorityQueue<>();
minPQ.offer(5); minPQ.offer(1); minPQ.offer(3);
minPQ.peek();  // 1
minPQ.poll();  // 1

// Max-heap
PriorityQueue<Integer> maxPQ = new PriorityQueue<>(Collections.reverseOrder());
maxPQ.offer(5); maxPQ.offer(1); maxPQ.offer(3);
maxPQ.poll();  // 5

// Custom comparator
PriorityQueue<String> pq = new PriorityQueue<>((a, b) -> a.length() - b.length());
pq.offer("banana"); pq.offer("fig"); pq.offer("apple");
pq.poll();  // "fig"
```

### Kth Largest Element — LeetCode 215
```java
public static int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minPQ = new PriorityQueue<>();
    for (int num : nums) {
        minPQ.offer(num);
        if (minPQ.size() > k) minPQ.poll();
    }
    return minPQ.peek();
}
```

### K Closest Points to Origin — LeetCode 973
```java
public static int[][] kClosest(int[][] points, int k) {
    PriorityQueue<int[]> pq = new PriorityQueue<>(
        (a, b) -> (b[0]*b[0] + b[1]*b[1]) - (a[0]*a[0] + a[1]*a[1])
    );
    for (int[] p : points) {
        pq.offer(p);
        if (pq.size() > k) pq.poll();
    }
    return pq.toArray(new int[k][]);
}
```

### Median from Data Stream — LeetCode 295
```java
class MedianFinder {
    private PriorityQueue<Integer> lower = new PriorityQueue<>(Collections.reverseOrder());
    private PriorityQueue<Integer> upper = new PriorityQueue<>();

    public void addNum(int num) {
        lower.offer(num);
        upper.offer(lower.poll());
        if (lower.size() < upper.size())
            lower.offer(upper.poll());
    }

    public double findMedian() {
        return lower.size() > upper.size()
            ? lower.peek()
            : (lower.peek() + upper.peek()) / 2.0;
    }
}
```

### Interview Traps
- Kth largest → **min-heap of size k** (evict smaller candidates).
- Kth smallest → **max-heap of size k**.
- Comparator overflow: use `Integer.compare(b, a)` not `b - a`.
- `PriorityQueue` does not support `decreaseKey` — remove and re-insert (O(n) remove).

---

## 5. Heapify Operations

### Key Concepts
- **Sift-down (heapify)**: fix one node downward — used in build-heap and extract.
- **Sift-up**: fix upward — used after insertion.
- **Build-heap**: apply sift-down bottom-up — **O(n)**.

### Sift-Down — Iterative
```java
public static void heapifyDown(int[] arr, int n, int i) {
    while (true) {
        int smallest = i;
        int l = 2 * i + 1, r = 2 * i + 2;
        if (l < n && arr[l] < arr[smallest]) smallest = l;
        if (r < n && arr[r] < arr[smallest]) smallest = r;
        if (smallest == i) break;
        int tmp = arr[i]; arr[i] = arr[smallest]; arr[smallest] = tmp;
        i = smallest;
    }
}
```

### Sift-Up
```java
public static void heapifyUp(int[] arr, int i) {
    while (i > 0) {
        int parent = (i - 1) / 2;
        if (arr[parent] <= arr[i]) break;
        int tmp = arr[i]; arr[i] = arr[parent]; arr[parent] = tmp;
        i = parent;
    }
}
```

### Build Heap — O(n)
```java
public static void buildMinHeap(int[] arr) {
    int n = arr.length;
    for (int i = n / 2 - 1; i >= 0; i--)
        heapifyDown(arr, n, i);
}
```

### Why BuildHeap is O(n) — Interview Explanation
```
Height h has at most n/2^(h+1) nodes, each costing O(h) to sift down.
Total cost = Σ (n/2^(h+1)) * h  for h = 0 to log(n)
           = n * Σ h/2^(h+1)
           = n * 2  (geometric series sum)
           = O(n)

Key insight: Most nodes are near the bottom (leaves) and cost O(1) to heapify.
```

### Interview Traps
- `n/2 - 1` is last non-leaf — heapifying leaves is a no-op.
- Heapify direction: **insert → sift-up**; **extract / build → sift-down**.
- Forgetting `break` in iterative sift-down causes infinite loop.
- `buildHeap` O(n) vs repeated insert O(n log n) — know the difference.

---

## 6. Heap Sort

### Key Concepts
- Phase 1: **Build max-heap** O(n). Phase 2: **Repeatedly extract max** O(n log n).
- Total: **O(n log n)** time, **O(1)** space — in-place.
- **Not stable** — equal elements may change relative order.

### Heap Sort Implementation
```java
public static void heapSort(int[] arr) {
    int n = arr.length;

    // Phase 1: Build max-heap
    for (int i = n / 2 - 1; i >= 0; i--)
        heapifyDownMax(arr, n, i);

    // Phase 2: Extract max one by one
    for (int i = n - 1; i > 0; i--) {
        int tmp = arr[0]; arr[0] = arr[i]; arr[i] = tmp;
        heapifyDownMax(arr, i, 0);
    }
}

private static void heapifyDownMax(int[] arr, int n, int i) {
    while (true) {
        int largest = i;
        int l = 2 * i + 1, r = 2 * i + 2;
        if (l < n && arr[l] > arr[largest]) largest = l;
        if (r < n && arr[r] > arr[largest]) largest = r;
        if (largest == i) break;
        int tmp = arr[i]; arr[i] = arr[largest]; arr[largest] = tmp;
        i = largest;
    }
}

// Trace for [4, 10, 3, 5, 1]:
// After buildMaxHeap:  [10, 5, 3, 4, 1]
// i=4: swap 10↔1 →    [1, 5, 3, 4, | 10], heapify → [5,4,3,1,|10]
// i=3: swap 5↔1 →     [1, 4, 3, | 5, 10], heapify → [4,1,3,|5,10]
// i=2: swap 4↔3 →     [3, 1, | 4, 5, 10], heapify → [3,1,|4,5,10]
// i=1: swap 3↔1 →     [1, | 3, 4, 5, 10]
// Sorted:              [1, 3, 4, 5, 10]
```

### Sort K-Sorted (Nearly Sorted) Array
```java
public static void sortKSorted(int[] arr, int k) {
    PriorityQueue<Integer> pq = new PriorityQueue<>();
    int idx = 0;

    for (int i = 0; i <= Math.min(k, arr.length - 1); i++)
        pq.offer(arr[i]);

    for (int i = k + 1; i < arr.length; i++) {
        arr[idx++] = pq.poll();
        pq.offer(arr[i]);
    }

    while (!pq.isEmpty()) arr[idx++] = pq.poll();
}
// O(n log k) — better than O(n log n) when k is small
```

### Merge K Sorted Arrays Using Heap
```java
public static int[] mergeKSortedArrays(int[][] arrays) {
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    int totalLen = 0;

    for (int i = 0; i < arrays.length; i++) {
        totalLen += arrays[i].length;
        if (arrays[i].length > 0)
            pq.offer(new int[]{arrays[i][0], i, 0});
    }

    int[] result = new int[totalLen];
    int idx = 0;

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        result[idx++] = curr[0];
        int ai = curr[1], ei = curr[2];
        if (ei + 1 < arrays[ai].length)
            pq.offer(new int[]{arrays[ai][ei + 1], ai, ei + 1});
    }
    return result;
}
// O(N log k) where N = total elements, k = number of arrays
```

### Interview Traps
- Heap sort uses **max-heap** to sort ascending.
- Pass heap **size `i`** (not `n`) to `heapifyDown` in phase 2.
- Heap sort is **not stable**.
- K-sorted array: heap size is `k+1` not `k`.
- Comparator overflow: use `Integer.compare(a[0], b[0])` not `a[0] - b[0]`.

---

### Hashing & Heaps — Complexity Reference

| Operation | HashMap | HashSet | Min/Max Heap | PriorityQueue |
|---|---|---|---|---|
| Insert | O(1) avg | O(1) avg | O(log n) | O(log n) |
| Delete | O(1) avg | O(1) avg | O(log n) | O(n) by value |
| Search / Contains | O(1) avg | O(1) avg | O(n) | O(n) |
| Get min/max | O(n) | O(n) | O(1) | O(1) |
| Build from array | O(n) | O(n) | O(n) | O(n) |
| Heap sort | — | — | O(n log n) | — |
| K-sorted sort | — | — | O(n log k) | O(n log k) |