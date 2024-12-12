import java.util.*;

class test {

    static class Node implements Comparable<Node> {
        int level;          // Level of the node in the tree
        int[] assignment;   // Partial assignment
        int cost;           // Cost of this node
        int bound;          // Lower bound for this node

        Node(int level, int[] assignment, int cost, int bound) {
            this.level = level;
            this.assignment = assignment.clone();
            this.cost = cost;
            this.bound = bound;
        }

        @Override
        public int compareTo(Node other) {
            return Integer.compare(this.bound, other.bound);
        }
    }

    // Calculate lower bound for a node
    static int calculateBound(int[][] costMatrix, int level, int[] assignment) {
        int bound = 0;
        boolean[] clubAssigned = new boolean[costMatrix.length];

        // Include the cost of already assigned clubs
        for (int i = 0; i <= level; i++) {
            bound += costMatrix[i][assignment[i]];
            clubAssigned[assignment[i]] = true;
        }

        // Add the minimum cost for remaining students
        for (int i = level + 1; i < costMatrix.length; i++) {
            int minCost = Integer.MAX_VALUE;
            for (int j = 0; j < costMatrix.length; j++) {
                if (!clubAssigned[j]) {
                    minCost = Math.min(minCost, costMatrix[i][j]);
                }
            }
            bound += minCost;
        }
        return bound;
    }

    // Solve the club assignment problem
    static void solveClubAssignment(int[][] costMatrix) {
        int n = costMatrix.length;
        PriorityQueue<Node> pq = new PriorityQueue<>();
        int[] bestAssignment = new int[n];
        int minCost = Integer.MAX_VALUE;

        // Start with an empty assignment
        Node root = new Node(-1, new int[n], 0, calculateBound(costMatrix, -1, new int[n]));
        pq.add(root);

        while (!pq.isEmpty()) {
            Node current = pq.poll();

            // If we have a complete assignment
            if (current.level == n - 1) {
                if (current.cost < minCost) {
                    minCost = current.cost;
                    bestAssignment = current.assignment.clone();
                }
                continue;
            }

            // Generate child nodes
            int nextLevel = current.level + 1;
            for (int j = 0; j < n; j++) {
                boolean alreadyAssigned = false;

                // Check if this club is already assigned
                for (int i = 0; i <= current.level; i++) {
                    if (current.assignment[i] == j) {
                        alreadyAssigned = true;
                        break;
                    }
                }

                if (!alreadyAssigned) {
                    int[] newAssignment = current.assignment.clone();
                    newAssignment[nextLevel] = j;
                    int newCost = current.cost + costMatrix[nextLevel][j];
                    int newBound = calculateBound(costMatrix, nextLevel, newAssignment);

                    // Only add the child if the bound is promising
                    if (newBound < minCost) {
                        pq.add(new Node(nextLevel, newAssignment, newCost, newBound));
                    }
                }
            }
        }

        // Print the result
        System.out.println("Minimum cost: " + minCost);
        System.out.print("Best assignment: ");
        for (int i = 0; i < n; i++) {
            System.out.print("Student " + i + " -> Club " + bestAssignment[i] + ", ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        // Example cost matrix
        int[][] costMatrix = {
            {9, 2, 7, 8},
            {6, 4, 3, 7},
            {5, 8, 1, 8},
            {7, 6, 9, 4}
        };
        solveClubAssignment(costMatrix);
    }
}





// Problem Statement
// We have:

// 𝑁
// N students.
// 𝑁
// N clubs.
// A cost matrix 
// 𝐶
// [
// 𝑁
// ]
// [
// 𝑁
// ]
// C[N][N], where 
// 𝐶
// [
// 𝑖
// ]
// [
// 𝑗
// ]
// C[i][j] tells us the cost of assigning Student 
// 𝑖
// i to Club 
// 𝑗
// j.
// Our goal:

// Assign each student to one unique club in such a way that the total assignment cost is minimized.
// Approach: Branch and Bound
// This approach uses a state space tree where:

// Each level in the tree represents assigning a student to a club.
// Each node represents a specific partial assignment of students to clubs.
// We calculate a bound for each node (a lower estimate of the total cost if we complete the assignments starting from this node).
// Why bounds?: Bounds help us decide if a node is worth exploring further. If a node has a bound worse than the current best solution, we ignore it (pruning).
// Step-by-Step Procedure
// 1. Start with the root node
// The root node represents no assignments made yet. Its cost is 0.
// Calculate its bound using the formula:
// Add the minimum possible cost for each unassigned student.
// 2. Expand the root node
// Assign Student 0 to each club, creating child nodes.
// For each child:
// Compute the cost of assigning Student 0 to a club.
// Compute the bound for this child node by estimating the minimum costs for unassigned students.
// 3. Use a priority queue
// Add all child nodes to a priority queue, sorted by their bounds (smallest bound first).
// Pick the node with the smallest bound to explore next. This ensures we explore the most promising assignments first.
// 4. Generate child nodes
// For the current node:

// Assign the next student (e.g., Student 1) to a club that hasn't been assigned yet.
// Compute the new cost and the new bound.
// Add the new nodes to the priority queue if their bound is less than the current best solution.
// 5. Continue until all students are assigned
// Repeat the above steps until you reach a node where all students have been assigned to clubs.
// Check if the total cost of this complete assignment is less than the current minimum cost. If yes, update the best solution.
// 6. Pruning
// Nodes with bounds worse than the current best solution are ignored. This drastically reduces the number of nodes we need to explore.
// Detailed Example
// Cost Matrix
// Let’s say we have 4 students and 4 clubs with the following cost matrix:

// Club 0	Club 1	Club 2	Club 3
// Student 0	9	2	7	8
// Student 1	6	4	3	7
// Student 2	5	8	1	8
// Student 3	7	6	9	4
// Our goal is to assign each student to a unique club such that the total cost is minimized.

// Step-by-Step
// Step 1: Start with the Root Node
// No assignments yet: [].
// Cost: 
// 0
// 0 (no students assigned yet).
// Bound:
// Add the minimum cost for each student:
// Student 0 → Min = 2 (Club 1).
// Student 1 → Min = 3 (Club 2).
// Student 2 → Min = 1 (Club 2).
// Student 3 → Min = 4 (Club 3).
// Bound = 
// 0
// +
// 2
// +
// 3
// +
// 1
// +
// 4
// =
// 10
// 0+2+3+1+4=10.
// Step 2: Expand Root Node
// Assign Student 0 to each club. Create 4 child nodes:

// Assign Student 0 to Club 0:

// Partial Assignment: [0 → 0].
// Cost: 
// 9
// 9.
// Bound:
// Student 1 → Min = 3 (Club 2).
// Student 2 → Min = 1 (Club 2).
// Student 3 → Min = 4 (Club 3).
// Bound = 
// 9
// +
// 3
// +
// 1
// +
// 4
// =
// 17
// 9+3+1+4=17.
// Assign Student 0 to Club 1:

// Partial Assignment: [0 → 1].
// Cost: 
// 2
// 2.
// Bound:
// Student 1 → Min = 3 (Club 2).
// Student 2 → Min = 1 (Club 2).
// Student 3 → Min = 4 (Club 3).
// Bound = 
// 2
// +
// 3
// +
// 1
// +
// 4
// =
// 10
// 2+3+1+4=10.
// Assign Student 0 to Club 2:

// Partial Assignment: [0 → 2].
// Cost: 
// 7
// 7.
// Bound = 
// 7
// +
// 3
// +
// 1
// +
// 4
// =
// 15
// 7+3+1+4=15.
// Assign Student 0 to Club 3:

// Partial Assignment: [0 → 3].
// Cost: 
// 8
// 8.
// Bound = 
// 8
// +
// 3
// +
// 1
// +
// 4
// =
// 16
// 8+3+1+4=16.
// Step 3: Use Priority Queue
// Add all child nodes to the queue: [10, 17, 15, 16].
// Pick the smallest bound: Node with bound 
// 10
// 10.
// Step 4: Expand Node with 
// 10
// 10
// From [0 → 1] (Student 0 → Club 1), assign Student 1 to all remaining clubs:

// Student 1 → Club 0:

// Partial Assignment: [0 → 1, 1 → 0].
// Cost: 
// 2
// +
// 6
// =
// 8
// 2+6=8.
// Bound = 
// 8
// +
// 1
// +
// 4
// =
// 13
// 8+1+4=13.
// Student 1 → Club 2:

// Partial Assignment: [0 → 1, 1 → 2].
// Cost: 
// 2
// +
// 3
// =
// 5
// 2+3=5.
// Bound = 
// 5
// +
// 1
// +
// 4
// =
// 10
// 5+1+4=10.
// Student 1 → Club 3:

// Partial Assignment: [0 → 1, 1 → 3].
// Cost: 
// 2
// +
// 7
// =
// 9
// 2+7=9.
// Bound = 
// 9
// +
// 1
// +
// 4
// =
// 14
// 9+1+4=14.
// Repeat Until All Students Are Assigned
// Continue expanding nodes with the smallest bound until all students are assigned.
// The best solution is:
// Student 0 → Club 1, Student 1 → Club 2, Student 2 → Club 0, Student 3 → Club 3.
// Total Cost = 
// 2
// +
// 3
// +
// 5
// +
// 4
// =
// 13
// 2+3+5+4=13.
// Final Output
// Minimum Cost: 
// 13
// 13.
// Assignment:
// Student 0 → Club 1.
// Student 1 → Club 2.
// Student 2 → Club 0.
// Student 3 → Club 3.
// Why Branch and Bound Works Well?
// Pruning: Nodes with large bounds are ignored.
// Optimality: Guarantees the minimum cost by exploring all possibilities systematically.
// Efficiency: Explores only promising nodes, saving time