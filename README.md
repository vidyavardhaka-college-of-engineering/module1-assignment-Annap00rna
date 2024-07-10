
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define N 3 // Size of the puzzle (N x N)

// Structure to represent a state of the puzzle
typedef struct {
    int puzzle[N][N];
    int blank_row;
    int blank_col;
    int cost; // Total cost (g + h)
} PuzzleState;

// A* Node structure
typedef struct {
    PuzzleState state;
    int priority; // Priority in the priority queue
} AStarNode;

// Priority queue functions (min-heap)
void swap(AStarNode *a, AStarNode *b);
void heapify(AStarNode arr[], int n, int i);
AStarNode extract_min(AStarNode arr[], int *n);
void insert(AStarNode arr[], int *n, AStarNode key);

// Helper functions
void print_puzzle(PuzzleState state);
bool is_goal_state(PuzzleState state);
bool is_valid_move(int row, int col);
void move(int row1, int col1, int row2, int col2, PuzzleState *state);
int calculate_heuristic(PuzzleState state);

// A* search function
void solve_puzzle(PuzzleState initial_state);

// Main function
int main() {
    PuzzleState initial_state = {
        {{1, 2, 3}, {4, 5, 6}, {7, 8, 0}}, // Initial configuration of the puzzle
        2, 2, // Position of the blank space (row, col)
        0 // Cost of initial state
    };

    solve_puzzle(initial_state);

    return 0;
}

void solve_puzzle(PuzzleState initial_state) {
    int max_nodes = 1000; // Maximum number of nodes to consider
    AStarNode priority_queue[max_nodes];
    int queue_size = 0;

    // Insert the initial state into the priority queue
    AStarNode initial_node = { initial_state, initial_state.cost + calculate_heuristic(initial_state) };
    insert(priority_queue, &queue_size, initial_node);

    while (queue_size > 0) {
        // Extract the node with the minimum priority
        AStarNode current_node = extract_min(priority_queue, &queue_size);
        PuzzleState current_state = current_node.state;

        // If current state is the goal state, print and return
        if (is_goal_state(current_state)) {
            printf("Solution found!\n");
            print_puzzle(current_state);
            return;
        }

        // Generate all possible moves (up, down, left, right)
        int drow[] = {-1, 1, 0, 0};
        int dcol[] = {0, 0, -1, 1};

        for (int i = 0; i < 4; ++i) {
            int new_row = current_state.blank_row + drow[i];
            int new_col = current_state.blank_col + dcol[i];

            if (is_valid_move(new_row, new_col)) {
                // Create a new state after moving the blank space
                PuzzleState new_state = current_state;
                move(current_state.blank_row, current_state.blank_col, new_row, new_col, &new_state);

                // Calculate the cost and priority for the new state
                new_state.cost = current_state.cost + 1;
                int priority = new_state.cost + calculate_heuristic(new_state);

                // Insert the new state into the priority queue
                AStarNode new_node = { new_state, priority };
                insert(priority_queue, &queue_size, new_node);
            }
        }
    }

    // If no solution found
    printf("No solution found.\n");
}

void print_puzzle(PuzzleState state) {
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            printf("%d ", state.puzzle[i][j]);
        }
        printf("\n");
    }
}

bool is_goal_state(PuzzleState state) {
    // Check if the puzzle is in the goal state
    int count = 1;
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            if (state.puzzle[i][j] != count % (N * N)) {
                return false;
            }
            count++;
        }
    }
    return true;
}

bool is_valid_move(int row, int col) {
    // Check if the move is within bounds of the puzzle
    return (row >= 0 && row < N && col >= 0 && col < N);
}

void move(int row1, int col1, int row2, int col2, PuzzleState *state) {
    // Move the blank space from (row1, col1) to (row2, col2)
    int temp = state->puzzle[row1][col1];
    state->puzzle[row1][col1] = state->puzzle[row2][col2];
    state->puzzle[row2][col2] = temp;
    state->blank_row = row2;
    state->blank_col = col2;
}

int calculate_heuristic(PuzzleState state) {
    // Calculate the heuristic (Manhattan distance)
    int heuristic = 0;
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            int value = state.puzzle[i][j];
            if (value != 0) {
                int target_row = (value - 1) / N;
                int target_col = (value - 1) % N;
                heuristic += abs(i - target_row) + abs(j - target_col);
            }
        }
    }
    return heuristic;
}

void swap(AStarNode *a, AStarNode *b) {
    AStarNode temp = *a;
    *a = *b;
    *b = temp;
}

void heapify(AStarNode arr[], int n, int i) {
    int smallest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;

    if (left < n && arr[left].priority < arr[smallest].priority)
        smallest = left;

    if (right < n && arr[right].priority < arr[smallest].priority)
        smallest = right;

    if (smallest != i) {
        swap(&arr[i], &arr[smallest]);
        heapify(arr, n, smallest);
    }
}

AStarNode extract_min(AStarNode arr[], int *n) {
    AStarNode min_node = arr[0];
    arr[0] = arr[*n - 1];
    (*n)--;
    heapify(arr, *n, 0);
    return min_node;
}

void insert(AStarNode arr[], int *n, AStarNode key) {
    (*n)++;
    int i = *n - 1;
    arr[i] = key;

    while (i > 0 && arr[(i - 1) / 2].priority > arr[i].priority) {
        swap(&arr[i], &arr[(i - 1) / 2]);
        i = (i - 1) / 2;
    }
}
