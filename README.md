# Java Collections Cheatsheet for DSA

```java
import java.util.*;
import java.util.stream.*;
```

---

## 1. Which structure do I need?

| Need | Use | Key ops |
|---|---|---|
| Dynamic array / index access | `ArrayList` | `get`/`set` O(1), `add` amortized O(1) |
| Stack (LIFO) | `ArrayDeque` | `push` / `pop` / `peek` O(1) |
| Queue (FIFO), BFS | `ArrayDeque` | `offer` / `poll` / `peek` O(1) |
| Deque, sliding-window max | `ArrayDeque` | `offerFirst/Last`, `pollFirst/Last` O(1) |
| Min/max element repeatedly, top-K, Dijkstra | `PriorityQueue` | `offer`/`poll` O(log n), `peek` O(1) |
| Key → value lookup | `HashMap` | `get`/`put` O(1) avg |
| Membership / dedupe | `HashSet` | `add`/`contains` O(1) avg |
| Sorted keys, floor/ceiling, ranges | `TreeMap` | O(log n) |
| Sorted unique values, floor/ceiling | `TreeSet` | O(log n) |
| Insertion-order map, LRU cache | `LinkedHashMap` | O(1) |

> Avoid `Stack` and `Vector` (legacy, synchronized). Prefer `ArrayDeque` over `LinkedList` for stacks/queues — but `ArrayDeque` does **not** allow `null`.

---

## 2. Arrays (`java.util.Arrays`)

```java
int[] a = new int[n];                 // zero-filled
int[] b = {3, 1, 2};
int[][] grid = new int[r][c];

Arrays.fill(a, -1);                   // fill 1D
for (int[] row : grid) Arrays.fill(row, -1);   // fill 2D

Arrays.sort(a);                       // ascending, primitives
Arrays.sort(a, from, to);             // sort subrange [from, to)
Arrays.toString(a);                   // "[1, 2, 3]"
Arrays.deepToString(grid);            // for 2D
Arrays.copyOf(a, newLen);
Arrays.copyOfRange(a, from, to);      // [from, to)
Arrays.equals(a, b);
Arrays.binarySearch(a, key);          // a must be sorted; returns -(insertionPoint)-1 if absent

int sum = Arrays.stream(a).sum();
int max = Arrays.stream(a).max().getAsInt();
```

### Sorting with a comparator

```java
// Comparators need objects: int[] can't take one, Integer[] can.
Integer[] boxed = {3, 1, 2};
Arrays.sort(boxed, Collections.reverseOrder());        // descending

// 2D arrays (intervals, pairs): each row is an object
Arrays.sort(intervals, (x, y) -> Integer.compare(x[0], y[0]));
Arrays.sort(intervals, Comparator.comparingInt(x -> x[0]));

// Sort by col 0 asc, then col 1 desc
Arrays.sort(pairs, (x, y) -> x[0] != y[0]
        ? Integer.compare(x[0], y[0])
        : Integer.compare(y[1], x[1]));

// Strings by length, then lexicographic
Arrays.sort(words, Comparator.comparingInt(String::length)
                             .thenComparing(Comparator.naturalOrder()));
```

---

## 3. ArrayList / List

```java
List<Integer> list = new ArrayList<>();
List<Integer> list2 = new ArrayList<>(List.of(1, 2, 3));   // mutable copy

list.add(x);              // append
list.add(i, x);           // insert at index — O(n)
list.get(i);
list.set(i, x);
list.remove(i);           // by INDEX (int)
list.remove(Integer.valueOf(x));   // by VALUE
list.size();
list.isEmpty();
list.contains(x);         // O(n)
list.indexOf(x);          // O(n)
list.clear();
list.subList(from, to);   // view, [from, to)

Collections.sort(list);                       // or list.sort(null)
list.sort(Comparator.reverseOrder());
Collections.reverse(list);
Collections.swap(list, i, j);
Collections.max(list); Collections.min(list);
Collections.frequency(list, x);

// List of lists (results in backtracking, adjacency lists)
List<List<Integer>> res = new ArrayList<>();
res.add(new ArrayList<>(path));   // copy! don't add the live reference
```

`List.of(...)` and `Arrays.asList(...)` are **immutable / fixed-size** — wrap in `new ArrayList<>(...)` to modify.

---

## 4. Stack, Queue, Deque — `ArrayDeque`

```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(x);        // add to top
stack.pop();          // remove top (throws if empty)
stack.peek();         // view top (null if empty)
stack.isEmpty();

Queue<Integer> q = new ArrayDeque<>();
q.offer(x);           // enqueue at tail
q.poll();             // dequeue head (null if empty)
q.peek();             // view head (null if empty)

Deque<Integer> dq = new ArrayDeque<>();
dq.offerFirst(x);  dq.offerLast(x);
dq.pollFirst();    dq.pollLast();
dq.peekFirst();    dq.peekLast();
```

| Throws on empty/fail | Returns null/false |
|---|---|
| `add`, `remove`, `element` | `offer`, `poll`, `peek` |

---

## 5. PriorityQueue (Heap)

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

pq.offer(x);      // O(log n)
pq.poll();        // remove min (or max) — O(log n)
pq.peek();        // O(1)
pq.size();
pq.remove(x);     // O(n) — avoid in hot loops

// Pairs / arrays: e.g. {node, dist}
PriorityQueue<int[]> pq = new PriorityQueue<>((x, y) -> Integer.compare(x[1], y[1]));

// Objects by field
PriorityQueue<Node> pq2 = new PriorityQueue<>(Comparator.comparingInt(n -> n.cost));

// Map entries by frequency (top-K frequent)
PriorityQueue<Map.Entry<Integer,Integer>> heap =
    new PriorityQueue<>((e1, e2) -> e1.getValue() - e2.getValue());
```

Iterating a `PriorityQueue` (for-each / `toString`) is **not** in sorted order — only `poll()` gives sorted output.

---

## 6. HashMap

```java
Map<String, Integer> map = new HashMap<>();

map.put(k, v);
map.get(k);                       // null if absent
map.getOrDefault(k, 0);
map.containsKey(k);
map.containsValue(v);             // O(n)
map.remove(k);
map.putIfAbsent(k, v);
map.size(); map.isEmpty();

// Counting
map.put(k, map.getOrDefault(k, 0) + 1);
map.merge(k, 1, Integer::sum);           // same thing, shorter

// Decrement and remove at zero (sliding window)
map.merge(k, -1, Integer::sum);
if (map.get(k) == 0) map.remove(k);

// Grouping / adjacency
Map<Integer, List<Integer>> graph = new HashMap<>();
graph.computeIfAbsent(u, key -> new ArrayList<>()).add(v);

// Iteration
for (Map.Entry<String, Integer> e : map.entrySet()) {
    String key = e.getKey(); int val = e.getValue();
}
for (String key : map.keySet()) { }
for (int val : map.values()) { }
```

---

## 7. HashSet

```java
Set<Integer> set = new HashSet<>();
set.add(x);          // returns false if already present — handy for "seen" checks
set.contains(x);
set.remove(x);
set.size();

Set<Integer> s = new HashSet<>(list);   // dedupe a list

// Set operations (mutate a)
a.retainAll(b);   // intersection
a.addAll(b);      // union
a.removeAll(b);   // difference
```

---

## 8. TreeMap & TreeSet (sorted, O(log n))

```java
TreeMap<Integer, Integer> tm = new TreeMap<>();
tm.firstKey();  tm.lastKey();
tm.floorKey(x);     // largest  ≤ x  (null if none)
tm.ceilingKey(x);   // smallest ≥ x
tm.lowerKey(x);     // largest  <  x
tm.higherKey(x);    // smallest >  x
tm.firstEntry();  tm.lastEntry();  tm.floorEntry(x);  tm.ceilingEntry(x);
tm.pollFirstEntry();  tm.pollLastEntry();
tm.headMap(x);          // keys < x   (view)
tm.tailMap(x);          // keys ≥ x   (view)
tm.subMap(lo, hi);      // lo ≤ key < hi
tm.descendingMap();

TreeSet<Integer> ts = new TreeSet<>();
ts.first(); ts.last();
ts.floor(x); ts.ceiling(x); ts.lower(x); ts.higher(x);
ts.pollFirst(); ts.pollLast();
ts.headSet(x); ts.tailSet(x); ts.subSet(lo, hi);
ts.descendingSet();

// Multiset trick (TreeSet has no duplicates): TreeMap<value, count>
tm.merge(x, 1, Integer::sum);
if (tm.merge(x, -1, Integer::sum) == 0) tm.remove(x);
```

---

## 9. LinkedHashMap — LRU Cache

```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private final int cap;
    LRUCache(int cap) { super(cap, 0.75f, true); this.cap = cap; }  // true = access order
    public int get(int key) { return super.getOrDefault(key, -1); }
    public void put(int key, int value) { super.put(key, value); }
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > cap;
    }
}
```

---

## 10. Strings & Characters

```java
char[] cs = s.toCharArray();
String t = new String(cs);
String.valueOf(cs);  String.valueOf(42);
s.charAt(i);  s.length();  s.substring(i, j);   // [i, j)
s.indexOf("ab");  s.equals(t);  s.compareTo(t);
s.split(" ");  s.split("\\s+");  s.trim();
String.join(",", list);

int[] freq = new int[26];
for (char c : s.toCharArray()) freq[c - 'a']++;

Character.isDigit(c); Character.isLetter(c); Character.isLetterOrDigit(c);
Character.toLowerCase(c); Character.isUpperCase(c);
int d = c - '0';

StringBuilder sb = new StringBuilder();
sb.append(x);  sb.insert(0, x);  sb.deleteCharAt(sb.length() - 1);
sb.setCharAt(i, c);  sb.reverse();  sb.length();  sb.toString();

// Anagram key
char[] k = s.toCharArray(); Arrays.sort(k); String key = new String(k);
```

---

## 11. Conversions

```java
// int[] → List<Integer>
List<Integer> l = Arrays.stream(arr).boxed().collect(Collectors.toList());
List<Integer> l2 = Arrays.stream(arr).boxed().toList();     // Java 16+, immutable

// List<Integer> → int[]
int[] arr2 = list.stream().mapToInt(Integer::intValue).toArray();

// List<int[]> → int[][]  (e.g. merge intervals result)
int[][] out = res.toArray(new int[0][]);

// List<String> → String[]
String[] sa = strList.toArray(new String[0]);

// Set → List
List<Integer> fromSet = new ArrayList<>(set);
```

---

## 12. Common DSA Patterns

```java
// BFS on grid
int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
Queue<int[]> q = new ArrayDeque<>();
q.offer(new int[]{sr, sc});
boolean[][] seen = new boolean[R][C];
seen[sr][sc] = true;
while (!q.isEmpty()) {
    int[] cur = q.poll();
    for (int[] d : dirs) {
        int nr = cur[0] + d[0], nc = cur[1] + d[1];
        if (nr < 0 || nc < 0 || nr >= R || nc >= C || seen[nr][nc]) continue;
        seen[nr][nc] = true;
        q.offer(new int[]{nr, nc});
    }
}

// Graph adjacency list (n nodes)
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
for (int[] e : edges) { adj.get(e[0]).add(e[1]); adj.get(e[1]).add(e[0]); }

// Dijkstra
int[] dist = new int[n]; Arrays.fill(dist, Integer.MAX_VALUE); dist[src] = 0;
PriorityQueue<int[]> pq = new PriorityQueue<>((x, y) -> Integer.compare(x[1], y[1]));
pq.offer(new int[]{src, 0});
while (!pq.isEmpty()) {
    int[] cur = pq.poll();
    int u = cur[0], du = cur[1];
    if (du > dist[u]) continue;                       // stale entry
    for (int[] e : wadj.get(u)) {                     // e = {v, w}
        if (du + e[1] < dist[e[0]]) {
            dist[e[0]] = du + e[1];
            pq.offer(new int[]{e[0], dist[e[0]]});
        }
    }
}

// Top-K largest: min-heap of size K
PriorityQueue<Integer> heap = new PriorityQueue<>();
for (int x : nums) { heap.offer(x); if (heap.size() > k) heap.poll(); }

// Monotonic stack: next greater element (store indices)
int[] ans = new int[n]; Arrays.fill(ans, -1);
Deque<Integer> st = new ArrayDeque<>();
for (int i = 0; i < n; i++) {
    while (!st.isEmpty() && nums[st.peek()] < nums[i]) ans[st.pop()] = nums[i];
    st.push(i);
}

// Sliding window maximum: monotonic deque of indices
Deque<Integer> dq = new ArrayDeque<>();
for (int i = 0; i < n; i++) {
    if (!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst();
    while (!dq.isEmpty() && nums[dq.peekLast()] <= nums[i]) dq.pollLast();
    dq.offerLast(i);
    if (i >= k - 1) result.add(nums[dq.peekFirst()]);
}

// Prefix sum + HashMap (count subarrays with sum k)
Map<Integer, Integer> seenSums = new HashMap<>(); seenSums.put(0, 1);
int run = 0, count = 0;
for (int x : nums) {
    run += x;
    count += seenSums.getOrDefault(run - k, 0);
    seenSums.merge(run, 1, Integer::sum);
}
```

---

## 13. Complexity Reference

| Structure | Access | Search | Insert | Delete | Notes |
|---|---|---|---|---|---|
| `ArrayList` | O(1) | O(n) | O(1)* end / O(n) mid | O(n) | *amortized |
| `LinkedList` | O(n) | O(n) | O(1) at ends | O(1) at ends | rarely the best pick |
| `ArrayDeque` | — | O(n) | O(1) ends | O(1) ends | no nulls |
| `PriorityQueue` | O(1) peek | O(n) | O(log n) | O(log n) poll, O(n) remove(x) | not iteration-sorted |
| `HashMap`/`HashSet` | — | O(1) avg | O(1) avg | O(1) avg | no ordering |
| `LinkedHashMap` | — | O(1) avg | O(1) avg | O(1) avg | insertion/access order |
| `TreeMap`/`TreeSet` | — | O(log n) | O(log n) | O(log n) | sorted, floor/ceiling |
| `Arrays.sort` primitives | | | O(n log n) avg | | dual-pivot quicksort |
| `Arrays.sort` / `Collections.sort` objects | | | O(n log n) | | TimSort, stable |

---

## 14. Pitfalls That Cost Test Cases

1. **`Integer` equality** — `a == b` on `Integer` objects only works reliably for −128..127. Use `a.equals(b)` or unbox: `map.get(x).intValue() == map.get(y)`.
2. **`list.remove(i)`** removes by *index* when `i` is an `int`. Use `list.remove(Integer.valueOf(x))` to remove a value.
3. **Comparator overflow** — `(a, b) -> a - b` breaks for large/negative values. Use `Integer.compare(a, b)`.
4. **`Arrays.asList(intArray)`** gives `List<int[]>` with one element, not a list of ints. Use streams + `boxed()`.
5. **Array keys in HashMap/HashSet** — `int[]` uses identity hashing. Use `List<Integer>`, a `String` key (`r + "," + c`), or encode as `long`/`int` (`r * C + c`).
6. **ConcurrentModificationException** — don't add/remove from a collection while iterating it with for-each; use an `Iterator` with `it.remove()` or collect changes first.
7. **Backtracking** — add `new ArrayList<>(path)` to results, not `path` itself.
8. **`Math.abs(Integer.MIN_VALUE)`** is still negative; use `long` when values can hit the edges.
9. **Null returns** — `map.get`, `pq.peek`, `q.poll`, `tm.floorKey` return `null` when empty/absent; unboxing that `null` into `int` throws `NullPointerException`.
10. **`Arrays.sort(int[])`** can be pushed to O(n²) by adversarial inputs on some judges; shuffle first or sort an `Integer[]`/`long[]` copy if that's a concern.
