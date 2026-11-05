# VIVA VOCE PREPARATION GUIDE
## Design & Analysis of Algorithms (ETCCDA302) — Capstone Project
**Mini-Courier Planner: Intelligent Parcel Prioritization, Packing Optimization & Route Planning**  
**Department of Computer Science & Engineering, K.R. Mangalam University**

---

## Part 1: How the Entire Project Works (Step-by-Step Overview)

If the examiner asks: *"Explain how your project works from start to finish"*, use this clear 5-step response:

1. **Step 1: Parcel Intake & Management**:
   The system loads incoming parcels from `data/parcels.json`. Each parcel has a Parcel ID, Customer Name, Delivery Location, Weight, Priority (1-10), Delivery Deadline (hours), and Monetary Value (₹).
2. **Step 2: Prioritization using Merge Sort**:
   Because courier vans cannot carry every package at once, we prioritize parcels using our manual **Merge Sort** ($O(n \log n)$). Parcels are ordered by:
   - Higher Priority first
   - Earlier Delivery Deadline first
   - Parcel ID as a deterministic tie-breaker
3. **Step 3: Vehicle Packing Optimization using 0/1 Knapsack DP**:
   The vehicle has a gross payload limit $W$ (e.g., 35 kg). We solve the **0/1 Knapsack** problem using a manually computed 2D Dynamic Programming table ($O(n \cdot W)$). We backtrack through the table to identify the exact subset of parcels that maximizes monetary profit without exceeding weight capacity.
4. **Step 4: Shortest Road Paths using Dijkstra's Algorithm**:
   We model city locations and roads as a weighted undirected graph using an **Adjacency List**. We run our manual **Dijkstra's Algorithm** with a binary min-heap (`heapq`) in $O((V+E)\log V)$ time to find the true shortest road path from the Central Warehouse depot to all selected delivery destinations.
5. **Step 5: Delivery Tour Planning using Nearest Neighbor TSP**:
   Starting from the warehouse, the van uses the **Nearest Neighbor TSP** heuristic to choose the closest unvisited customer stop (based on Dijkstra shortest-path road distances). Once all customers receive their parcels, the van returns to the warehouse, forming an optimized closed loop tour.

---

## Part 2: 35 Comprehensive Viva Questions and Simple Answers

### Category 1: Merge Sort & Parcel Prioritization

#### Q1. What algorithmic design paradigm does Merge Sort use?
**Answer:** Merge Sort uses the **Divide-and-Conquer** paradigm. It recursively divides the input list into two equal halves until each sublist has only 1 element (the divide step), and then merges those sublists back together in sorted order (the conquer and combine step).

#### Q2. What is the time complexity of Merge Sort in Best, Average, and Worst cases?
**Answer:** The time complexity of Merge Sort is $\mathbf{O(n \log n)}$ in all three cases (Best, Average, and Worst). This is because it always divides the array into two halves ($\log n$ levels of recursion) and takes linear time $O(n)$ to merge the elements at each level.

#### Q3. What is the space complexity of Merge Sort, and why?
**Answer:** The auxiliary space complexity is $\mathbf{O(n)}$. During the merging step, temporary memory is required to store the elements of the left and right halves while comparing them before placing them back into the merged list.

#### Q4. Why did you choose Merge Sort over Quick Sort or Bubble Sort for prioritizing parcels?
**Answer:**
- Compared to Bubble Sort ($O(n^2)$), Merge Sort is drastically faster for large volumes of parcels.
- Compared to Quick Sort (which degrades to $O(n^2)$ in the worst case with poor pivot choices), Merge Sort **guarantees** $O(n \log n)$ performance.
- Furthermore, Merge Sort is **stable**, meaning packages with equal priority and deadlines retain their original relative arrival order.

#### Q5. How does your comparator handle ties between parcels?
**Answer:** Our comparator function checks three levels sequentially:
1. `priority` (descending: higher priority first).
2. If priorities are equal, `delivery_deadline` (ascending: sooner deadline first).
3. If both priority and deadline are identical, `parcel_id` (lexicographical ascending order) acts as a deterministic tie-breaker.

---

### Category 2: Dynamic Programming & 0/1 Knapsack

#### Q6. What is Dynamic Programming, and what two properties must a problem have to be solved by DP?
**Answer:** Dynamic Programming is an optimization technique that solves complex problems by breaking them down into simpler overlapping subproblems, solving each subproblem once, and storing the results in a table (memoization or tabulation) to avoid redundant recalculation.  
The two necessary properties are:
1. **Optimal Substructure**: An optimal solution to the problem contains within it optimal solutions to its subproblems.
2. **Overlapping Subproblems**: The same smaller subproblems are solved repeatedly if solved recursively.

#### Q7. What is the difference between the 0/1 Knapsack and the Fractional Knapsack problem?
**Answer:**
- In **Fractional Knapsack**, items can be broken into parts (e.g., liquids, grains). It can be solved greedily by sorting by value-to-weight ratio in $O(n \log n)$ time.
- In **0/1 Knapsack**, items are indivisible (you either pack a whole parcel or leave it behind). Greedy strategies fail to guarantee optimality, so **Dynamic Programming** is required.

#### Q8. What is the recurrence relation for the 0/1 Knapsack DP table?
**Answer:**
For item $i$ with weight $w_i$ and value $v_i$, and current weight capacity $w$:
$$\text{dp}[i][w] = \begin{cases} \text{dp}[i-1][w] & \text{if } w_i > w \\ \max(\text{dp}[i-1][w], \text{dp}[i-1][w - w_i] + v_i) & \text{if } w_i \le w \end{cases}$$

#### Q9. How do you find which exact parcels were selected from the filled DP table?
**Answer:** We perform **backtracking** starting from the bottom-right cell `dp[n][W]`:
- If `dp[i][w] != dp[i-1][w]`, it means parcel $i-1$ was included in the optimal pack. We add it to our selected list, reduce remaining capacity by its weight ($w \leftarrow w - w_i$), and move to row $i-1$.
- If `dp[i][w] == dp[i-1][w]`, parcel $i-1$ was excluded, and we simply move to row $i-1$.

#### Q10. What is the time and space complexity of your 0/1 Knapsack implementation?
**Answer:**
- **Time Complexity**: $O(n \cdot W)$, where $n$ is the number of parcels and $W$ is the maximum vehicle capacity.
- **Space Complexity**: $O(n \cdot W)$ to store the 2D table of size $(n+1) \times (W+1)$.

#### Q11. Why is 0/1 Knapsack referred to as having "pseudo-polynomial" time complexity?
**Answer:** The time complexity $O(n \cdot W)$ is polynomial with respect to the *numeric value* of $W$, but exponential with respect to the *number of bits* required to represent $W$ ($b = \log_2 W$). If $W$ is an astronomically large number, the algorithm becomes slow and memory-intensive.

#### Q12. How does your Knapsack code handle edge cases such as zero vehicle capacity or a parcel heavier than the vehicle?
**Answer:**
- If capacity $W \le 0$ or the parcel list is empty, the function returns immediately with total value 0 and empty selections.
- If an individual parcel has $w_i > W$, the recurrence branch `dp[i][w] = dp[i-1][w]` safely ignores the parcel without array out-of-bounds errors.

---

### Category 3: Graph Representation & Data Structures

#### Q13. How did you represent the delivery road network in code?
**Answer:** We represented the road network using an **Adjacency List**, implemented as a Python dictionary where each key is a location name, and its value is a list of tuples containing `(neighbor_location, road_distance_km)`.

#### Q14. What are the advantages of an Adjacency List over an Adjacency Matrix?
**Answer:**
1. **Space Efficiency**: Road networks are sparse graphs where each intersection typically connects to only 2 to 4 roads ($E \ll V^2$). An adjacency list takes $O(V + E)$ space, whereas an adjacency matrix consumes $O(V^2)$ space (mostly storing zeros or infinities).
2. **Faster Neighbor Iteration**: Finding the connected neighbors of a node takes $O(\text{degree}(u))$ time in an adjacency list compared to $O(V)$ time in a matrix.

#### Q15. Is your delivery network graph directed or undirected?
**Answer:** It is a **weighted undirected graph**, because standard two-way urban roads allow vehicles to travel in both directions between locations with the same physical distance.

---

### Category 4: Dijkstra's Shortest Path Algorithm

#### Q16. What is the purpose of Dijkstra's algorithm in your courier system?
**Answer:** It calculates the shortest driving distance and the exact sequence of road segments from the central logistics warehouse depot to every customer delivery location.

#### Q17. Why does Dijkstra's algorithm require non-negative edge weights?
**Answer:** Dijkstra is a greedy algorithm that finalizes the shortest distance to a vertex the moment it is popped from the priority queue. If negative edge weights were present, a previously finalized path could be shortened later, causing the greedy assumption to fail. Urban road distances in kilometers are always positive, making Dijkstra valid.

#### Q18. Why did you use a binary min-heap (`heapq`) in Dijkstra?
**Answer:** A binary min-heap allows us to extract the unvisited node with the smallest tentative distance in $\mathbf{O(\log V)}$ time, rather than scanning the entire list of vertices in $O(V)$ time. This reduces overall time complexity from $O(V^2)$ to $\mathbf{O((V + E) \log V)}$.

#### Q19. How does your Dijkstra implementation reconstruct the actual turn-by-turn road path?
**Answer:** We maintain a `predecessors` dictionary. Whenever we relax an edge $(u, v)$ such that distance to $v$ improves, we set `predecessors[v] = u`. Once Dijkstra completes, we backtrack from the destination node through its predecessors until reaching the warehouse, and then reverse the path.

#### Q20. How does your algorithm handle a disconnected node in the graph?
**Answer:** Any disconnected node remains at its initialized distance of $\infty$ (`float('inf')`) and its predecessor remains `None`. The function checks this and reports the location as "Unreachable" without crashing.

---

### Category 5: Traveling Salesperson Problem (TSP) & Nearest Neighbor

#### Q21. What is the Traveling Salesperson Problem (TSP)?
**Answer:** The TSP asks: Given a list of cities and the distances between each pair of cities, what is the shortest possible route that visits each city exactly once and returns to the origin city?

#### Q22. What is the computational complexity class of TSP?
**Answer:** TSP is **NP-Hard** (and its decision version is NP-Complete). An exact brute-force search takes $O(n!)$ time, and dynamic programming (Held-Karp) takes $O(2^n n^2)$ time, which becomes impossible to compute in real-time as the number of stops grows.

#### Q23. How does the Nearest Neighbor heuristic work?
**Answer:**
1. Start at the Central Warehouse depot.
2. Look at all unvisited customer delivery stops and pick the one with the shortest road distance from the current location.
3. Travel to that location, mark it as visited, and repeat step 2.
4. Once all customer stops have been visited, travel back to the warehouse depot to complete the closed tour.

#### Q24. Does the Nearest Neighbor algorithm guarantee the global optimal TSP tour?
**Answer:** No. Nearest Neighbor is a **greedy heuristic**, not an exact algorithm. Because it only makes locally optimal choices at each step, it can be forced into taking a very long, expensive return trip at the end. However, it runs in fast polynomial time ($O(k^2)$) and gives a practical, good-quality route for real-world delivery logistics.

#### Q25. Why did you combine Dijkstra with Nearest Neighbor in your project?
**Answer:** Customer delivery locations do not always have a direct, single road connecting them. To accurately determine the distance between two stops, we use Dijkstra's algorithm to calculate the true shortest road path across intermediate intersections.

---

### Category 6: Complexity & Performance Analysis

#### Q26. Summarize the time and space complexity of all major algorithms in your project.
**Answer:**
- **Merge Sort**: Time $O(n \log n)$, Space $O(n)$.
- **0/1 Knapsack DP**: Time $O(n \cdot W)$, Space $O(n \cdot W)$.
- **Dijkstra's Algorithm**: Time $O((V + E) \log V)$, Space $O(V + E)$.
- **Nearest Neighbor TSP**: Time $O(k^2 + k(V+E)\log V)$ where $k$ is delivery stops, Space $O(V + E)$.

#### Q27. How did you measure algorithm execution times in your project?
**Answer:** We used Python's `time.perf_counter()`, which is a high-resolution, monotonic timer unaffected by system clock updates, ideal for measuring microsecond and millisecond algorithmic runtimes.

#### Q28. What empirical trends did your performance benchmarks show when scaling from 10 to 500 parcels?
**Answer:**
- Merge Sort scaled smoothly from $0.015 \text{ ms}$ for 10 items to $2.12 \text{ ms}$ for 500 items, closely following $O(n \log n)$.
- 0/1 Knapsack scaled from $0.13 \text{ ms}$ ($10 \times 50$) to $70.13 \text{ ms}$ ($500 \times 500$), demonstrating the $O(n \cdot W)$ 2D matrix fill time.
- Dijkstra remained under $0.31 \text{ ms}$ even on a 100-node graph due to min-heap efficiency.

#### Q29. How does increasing vehicle capacity $W$ affect Knapsack runtime when item count $N$ is kept constant?
**Answer:** As capacity $W$ increases linearly, runtime increases linearly because the inner loop runs from $0$ to $W$. Our sensitivity test ($W = 20$ to $1500$) confirmed this linear scaling.

---

### Category 7: Project Architecture & Engineering

#### Q30. How are the individual algorithms integrated into one complete system?
**Answer:**
1. Incoming parcels $\to$ **Merge Sort** prioritizes them by importance and deadline.
2. Prioritized parcels $\to$ **0/1 Knapsack DP** selects the highest-value subset that fits into the delivery van.
3. Destination locations of the selected parcels $\to$ **Dijkstra** finds road distances and paths from the depot.
4. Destination locations $\to$ **Nearest Neighbor TSP** links them into an ordered delivery tour that starts and finishes at the depot.

#### Q31. Did you use NetworkX or built-in functions to compute shortest paths or TSP?
**Answer:** No. Dijkstra's algorithm and the Nearest Neighbor TSP solver were implemented completely manually from scratch. NetworkX was used solely for rendering 2D node coordinates in Matplotlib visualizations.

#### Q32. What testing methodology did you implement?
**Answer:** We developed automated unit tests using Python's `unittest` framework covering 28 test scenarios, including edge cases: empty inputs, single-item lists, duplicate priorities, zero vehicle capacity, parcels heavier than capacity, disconnected graph nodes, and full pipeline integration.

#### Q33. What are the key limitations of your current system?
**Answer:**
1. The TSP tour is an approximation and can be suboptimal.
2. The 0/1 Knapsack table requires integer weights.
3. Road edge weights are static distances without dynamic live traffic data.

#### Q34. How could this project be extended for industrial use (Future Scope)?
**Answer:**
1. Integrate the **2-Opt** algorithm to remove crossed paths and refine the TSP tour.
2. Support **Capacitated Vehicle Routing Problem (CVRP)** for a fleet of multiple vehicles.
3. Incorporate customer delivery time windows (**VRPTW**).
4. Integrate live GPS traffic data from OpenStreetMap.

#### Q35. What was the most challenging part of this capstone assignment, and how did you solve it?
**Answer:** Ensuring seamless mathematical and data integration between disparate algorithmic paradigms—specifically mapping the discrete subset of parcels chosen by Knapsack DP to geographic graph nodes, and using Dijkstra shortest-path sub-routes to calculate accurate edge costs for the Nearest Neighbor TSP tour without duplicate stops or missing return legs.
