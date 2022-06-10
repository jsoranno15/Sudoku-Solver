# Sudoku-Solver

## How to run `main.py`
Mark `main.py` as an executable file in the terminal, and then input filenames after excetuable:
```
> chmod u+x main.py
> ./main.py Input1.txt
```
## Example Input/Output
### Input1.txt
```
0 0 0 5 6 0 7 0 4
6 8 0 0 7 0 0 9 0
4 9 0 0 0 1 2 0 0 
8 5 0 4 0 0 0 1 0
0 0 1 6 0 5 9 0 0 
0 2 0 0 0 3 0 5 8
0 0 9 3 0 0 0 7 1
0 1 0 0 2 0 0 3 6
7 0 3 0 4 8 0 0 0 
```
### Output1.txt
```
1 3 2 5 6 9 7 8 4 
6 8 5 2 7 4 1 9 3 
4 9 7 8 3 1 2 6 5 
8 5 6 4 9 2 3 1 7 
3 7 1 6 8 5 9 4 2 
9 2 4 7 1 3 6 5 8 
2 4 9 3 5 6 8 7 1 
5 1 8 9 2 7 4 3 6 
7 6 3 1 4 8 5 2 9 
```

## Code
```
#!/usr/bin/python3

"""
Juliana Soranno
CSUY 4613_A: Artifical Intelligence
Project 2: Sudoku
"""
import argparse


def mini_grid(csp, row, col):
    """ returns the 3x3 grid that csp[row][col] is located in """
    mini_grids = []
    j = 0
    h = 0

    # populate mini_grids with all 3x3 grids from left to right
    for k in range(0, 3):
        for i in range(0, 3):
            grid = []
            for row1 in range(h, h + 3):
                for col1 in range(j, j + 3):
                    grid.append(csp[row1][col1])
            j = j + 3
            mini_grids.append(grid)
        j = 0
        h = h + 3

    # return the grid that csp[row][col] is in
    row = row // 3
    col = col // 3
    for row2 in range(0, 3):
        if row2 == row:
            for col2 in range(0, 3):
                if col == col2:
                    return mini_grids[col + 3 * row]


def update_constraints(csp, row, col):
    """returns a list of numbers that csp[row][col] cannot be"""
    if csp[row][col] == 0:
        constraints = []

        # constraints for row
        for num in csp[row]:
            if num != 0:
                constraints.append(num)

        # constraints for column
        for line in csp:
            if line[col] != 0 and line[col] not in constraints:
                constraints.append(line[col])

        # constraints for 3 x 3 grid
        for num in mini_grid(csp, row, col):
            if num != 0 and num not in constraints:
                constraints.append(num)

        return constraints


def is_solution(csp):
    """checks to see if csp is the correct solution"""
    for row in csp:
        for num in row:
            if num == 0:
                return None
    return csp


def assignments(csp):
    """populates the list of assignments given a CSP"""
    assignment = []
    for row in range(0, 9):
        for col in range(0, 9):
            if csp[row][col] == 0:
                assignment.append([row, col, csp[row][col]])
    return assignment


def backtracking_search(csp):
    return backtrack(csp, [])


def backtrack(csp, assignment):
    """====BACKTRACKING ALGORITHM as descibed===="""
    # if assignment is complete then return assignment
    if is_solution(csp):
        return csp

    # list of assignments ex. [row,col,val] [s0,0,0]
    assignment = assignments(csp)

    # select the next optimal node to expand
    var = select_unassigned_variable(csp, assignment)
    for val in order_domain_values():
        # if var consistent with assignment
        if val in legal_values(update_constraints(csp, row=var[0], col=var[1])):
            # add{var = value} to assignment
            for elem in assignment:
                if elem[0] == var[0] and elem[1] == var[1]:
                    elem[2] = val
            csp[var[0]][var[1]] = val

            # result <- (backtrack(csp, assignment))
            result = backtrack(csp, assignment)

            # if result != failure, return result
            if result is not None:
                return result

            # remove {var = value from assignment} so var = 0
            for elem in assignment:
                if elem[0] == var[0] and elem[1] == var[1]:
                    elem[2] = 0
            csp[var[0]][var[1]] = 0
    # return failure
    return None


def legal_values(constraints):
    """ returns a list of legal values or domains given a constraint"""
    legal_values = [1, 2, 3, 4, 5, 6, 7, 8, 9]
    for num in constraints:
        if num in legal_values:
            legal_values.remove(num)
    return legal_values


def column(csp, col):
    """returns the column that node is in as list top to bottom"""
    column = []
    for row in csp:
        column.append(row[col])
    return column


def select_unassigned_variable(csp, assignment):
    """
    implements minimum_remaining_value() first
    if m_r_v() yields more than one value then apply
    degree heursitics, other wise, chooses arbitratily
    returns the optimal node as list ex. [2,7]
    """
    var_row = 0
    var_col = 0

    # minimum_remaining_value()
    min_legal_vals = 9
    num_with_min = 0
    MRVs = []
    for elem in assignment:
        row = elem[0]
        col = elem[1]
        val = elem[2]
        if val == 0:
            lvals = legal_values(update_constraints(csp, row, col))
            if len(lvals) < min_legal_vals:
                min_legal_vals = len(lvals)
                var_row = row
                var_col = col
            if len(lvals) == min_legal_vals:
                num_with_min = num_with_min + 1

    for elem in assignment:
        if elem[2] == 0:
            lvals = legal_values(update_constraints(csp, elem[0], elem[1]))
            if len(lvals) == min_legal_vals:
                MRVs.append([elem[0],elem[1],elem[2]])

    """
    J.S. NOTE: here is where i tried to implement
    max_constraints() however, i found that this both failed
    my algorithm, and also returned the same node as min remaining
    values, so i left it here commented out:
        min remaining vals = 9 - constraints
        max constraints = 9 + min remaining vals
    """

    most_unasign_neibrs = 0
    num_with_most_unassigned = 0
    if len(MRVs) > 1:
        # go through the list of minimum remaining values
        for elem in MRVs:
            row = elem[0]
            col = elem[1]
            val = elem[2]
            if val == 0:
                unasign_neibrs = csp[row].count(0) + \
                                 column(csp, col).count(0) + \
                                 mini_grid(csp, row, col).count(0)
                if unasign_neibrs > most_unasign_neibrs:
                    most_unasign_neibrs = unasign_neibrs
                    var_row = row
                    var_col = col
                if unasign_neibrs == most_unasign_neibrs:
                    num_with_most_unassigned += 1
        return [var_row, var_col]
    else:
        return [var_row, var_col]


def order_domain_values():
    """order the domain values from 1 9"""
    return [1, 2, 3, 4, 5, 6, 7, 8, 9]


def print_state(state):
    """formats the csp correctly
    and outputs to file"""
    file = open('output.txt', 'w')
    for line in state:
        for i in line:
            file.write(str(i) + " ")
        file.write("\n")
    file.close()


def convert_to_int(csp):
    """convert values from file to integer"""
    for line in csp:
        for i in range(len(line)):
            line[i] = int(line[i])
    return csp


def main():
    # read from the command line
    parser = argparse.ArgumentParser(description='Solve Sodoku Problem '
                                                 'using backtracking algorithm')
    parser.add_argument('filename', help='The .txt containing initial state')
    cmdline = parser.parse_args()

    # populate state with the contents of the file
    state = []
    with open(cmdline.filename) as f:
        for line in f:
            line = line.strip().split(" ")
            state.append(line)

    # convert the statespace from string to integer
    csp = convert_to_int(state)

    # call the backtracking algorithm on the initial state and print
    print_state(backtracking_search(csp))


if __name__ == '__main__':
    main()
```
