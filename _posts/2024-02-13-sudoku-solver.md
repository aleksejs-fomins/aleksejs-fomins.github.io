---
title: "Optimizing an automated sudoku solver in Python"
categories:
  - Meta
tags:
  - Sudoku
  - Python
  - Optimization
author_profile: false
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/bozhin-karaivanov-yqcWOAHH1Rs-unsplash.jpg
  actions:
    - label: "Learn More"
      url: "/terms/"
  caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
---

As an avid sudoku enthusiast, and a fan of [Cracking The Cryptic](https://www.youtube.com/c/CrackingTheCryptic), I frequently find myself needing a little boost from my external brain. So, why not use this as an opportunity to exercise some coding optimization skills as well. In this post, I will cover the generic sudoku solver. In the future posts, I will cover using quick code snippets to brute force through some smaller sub-problems.

## The Setting

Sudoku is an [NP-complete](http://www.cs.ox.ac.uk/people/paul.goldberg/FCS/sudoku.html) problem, which means that general-purpose solvers have to resort to brute-force search to solve them. Since the goal of this post is to practice some optimization, we will write a sudoku solver from scratch, and attempt to optimize it along the way.

## Input Data

In order to test the performance of our solver, I have copied 5 examples of classical sudoku from sudoku.com website, and stored them in a basic text files. As we will later see, we would optimally need far more examples to make sure that our solvers work well on average, but this will do for now. Our text files are simple 9x9 grids, where unfilled spaces are marked with a space character, and filled spaces are digits. We can write a simple reader for such files as follows

```python
with open('examples/grid_easy.dat', 'r') as f:
    lines = f.readlines()
    lines = [l.strip('\n').replace(' ', '0') for l in lines]
    grid = np.array([[int(c) for c in l] for l in lines])
```
In the end we get a 2D Numpy array of integers, where empty cells are marked as zeroes.

## Constraints

The family of sudoku puzzles are all about filling in missing digits in a grid, while keeping a certain set of constraints satisfied. In a classical sudoku, the digits in any row, column or any of the nine 3x3 boxes may not repeat. While it is easily possible to hard-code these constraints, it is my dream to ultimately extend this solver to be able to handle a wide variety of different sudoku challenges, and thus I would like to keep it as generic as possible. Thus, we will define a generic constraint class.

```python
from typing import Protocol

class GenericConstraint(Protocol):
    def test(self, grid: np.ndarray) -> bool:
      ...

    def idxs_affected(self) -> np.ndarray:
      ...
```

Here, the `test` function would take a grid, and tell us if the constraint is satisfied, whereas `idxs_affected` would tell us the indices of the grid cells that the current constraint is going to use for its test. We don't need the latter just yet, but may come in handy later.

For example, the row constraint from classical sudoku could look something like this

```python
class HorizontalSudokuConstraint(GenericConstraint):
    def __init__(self, idxRow: int):
        self.idxRow = idxRow

    def idxs_affected(self) -> [np.array, np.array]:
        return np.full(9, self.idxRow), np.arange(9)

    def test(self, grid: np.array) -> bool:
        digits = grid[self.idxRow]
        return are_numbers_unique(digits[digits > 0])
```
We would be initializing the constraint by telling it which row it applies to. The test would extract all of the digits of that row from the grid and check if they are all different. The `idxs_affected` would simply return the indices of all cells in that row.

We can now mix and match different constraints in order to define different sudoku problems. Here, we will consider constructing a classical 9x9 sudoku problem. However, it is easy to see that this can be generalized to other types of sudoku, such as [killer sudoku](https://sudoku.com/killer).

```python
class ClassicalProblem:
    def __init__(self):
        self.constraints = []
        for iRow in range(9):
            self.constraints += [cs.VerticalSudokuConstraint(iRow)]
        for iCol in range(9):
            self.constraints += [cs.HorizontalSudokuConstraint(iCol)]
        for iRow in [0, 3, 6]:
            for iCol in [0, 3, 6]:
                self.constraints += [cs.BoxSudokuConstraint(iRow, iCol)]
```
So, our problem class is nothing more than a collection of different constraints that apply to this problem. For example, in a classical sudoku we have 9 horizontal constraints, 9 vertical constraints, and 9 box constraints.


## Initial algorithm

The most naive solver would go over all possible combinations of digit positioning in the grid, and test if it matches. The number of attempts such a solver would do is $$9^m$$, where $$m$$ is the number of empty grid cells. Since the number of empty cells can be as large as 70, we are looking at $$9^{70} \approx 10^{67}$$ iterations! Even if we do something smarter and only go over valid permutations of the available digits, this will still require years to complete. We have to take a different approach from the start.

One important observation is that it is not necessary to see all of the digits to prove that the solution does not work. Finding just one wrong digit (such as two of the same digit in the same row) is sufficient to discard the solution, regardless of the values of all the other digits. Thus, we will be filling in digits one at a time. The generic algorithm would be as follows


1. Find all unfilled digits and enumerate them
2. Start with the first unfilled digit, setting the current digit index to 0
3. If the current digit index is -1, exit the loop, we are done
4. Increment the current digit by 1, or set it to 1 if it is unfilled
5. If the current digit is 10, then unset the current digit, decrease the digit index by 1, and go back to 3.
6. Test all constraints. If any constraints fail, continue to 4.
7. If all constraints pass, increment the digit index by 1
8. If the digit index is equal to the number of unfilled digits, we have found a fitting solution. Output the solution, and decrease the digit index by 1.
9. Return to 3.
{: .notice--primary }

The example solver class would look somethinkg like the example below. The initializer finds the empty cells and arranges them in a 1D array to be filled in a sequence later. It also constructs a mapping between the 1D and 2D representations of the missing cells. The helper function `test` tests all of the constraints of the problem. The helper function `set` fills in a digit into the 1D array of the missing cells, and also in the corresponding missing cell in the 2D grid.

```python
class BruteSolverV1:
    def __init__(self, grid: np.array, problem: ClassicalProblem):
        self.grid = np.copy(grid)
        self.problem = problem

        # Step 1: Identify coordinates of all unfilled digits
        self.idxsRow, self.idxsCol = np.where(self.grid == 0)
        self.nEmpty = len(self.idxsRow)
        self.empty1D = np.zeros_like(self.idxsRow)

    def test(self) -> bool:
        for constr in self.problem.constraints:
            if not constr.test(self.grid):
                return False
        return True

    def set(self, idxEmpty1D: int, val: int) -> None:
        self.empty1D[idxEmpty1D] = val
        self.grid[self.idxsRow[idxEmpty1D], self.idxsCol[idxEmpty1D]] = val

    def solve(self):
        idxEmpty1D = 0
        while True:
            if idxEmpty1D == -1:
                # First termination condition: if we are pointing at an index -1, finish. The loop has been exhausted
                break
            if idxEmpty1D == self.nEmpty:
                # We have reached a plausible solution. Print it, and continue the loop
                print(self.grid)
                idxEmpty1D -= 1
                continue

            # Progressively increment the value of this cell, until a fitting value is found or all have failed
            fits = False
            for newVal in range(self.empty1D[idxEmpty1D] + 1, 10):
                # Set this value and test fitness
                self.set(idxEmpty1D, newVal)
                fits = self.test()

                if fits:
                    # If this value fits, keep it, and continue fitting the next cell
                    idxEmpty1D += 1
                    break

            if not fits:
                # If no remaining values fit, clear this square and revert to increasing the previous one
                self.set(idxEmpty1D, 0)
                idxEmpty1D -= 1

```

## Putting it all together

Now that we have implemented all of the parts in their corresponding classes, our control panel class is going to be tiny

```python
from readers import classical_reader
from problems import ClassicalProblem
from solvers import BruteSolverV1

grid = classical_reader('examples/grid_easy.dat')
problem = ClassicalProblem()

bs = BruteSolverV1(grid, problem)
bs.solve()
```

## Optimization

The vigilant reader certainly would have spotted an obvious optimization by now. Not all constraints apply to all digit. For example, in a classical sudoku, exactly 3 constraints apply to each cell - the corresponding row, column and box constraints. If we can figure out what constraints apply to what cells, we can test only those constraints when changing any one digit. So, instead of 27 constraints, we now have 3 - a savings of a factor of 9 on the iteration part! For this, we need two things.

First, we need to find what the constraint indices that affect each cell. For this, we will use the `idxs_affected` function that we implemented before. We will find the cells affected by each constraint, then "invert" that list to find what constraints is each of the cells affected by. We add the following to the solver initializer

```python
# For each constraint, get cell indices that are affected by it
# Make a temporary map from 2D index to 1D index
idxs2Dto1D = {(iRow, iCol): i for i, (iRow, iCol) in enumerate(zip(self.idxsRow, self.idxsCol))}

constr1Didxs = []
for constr in self.problem.constraints:
    xIdxs, yIdxs = constr.idxs_affected()
    constr1Didxs += [[idxs2Dto1D[(iRow, iCol)] for iRow, iCol in zip(xIdxs, yIdxs) if (iRow, iCol) in idxs2Dto1D]]

# For each empty cell, find indices of constraints that affect it
self.empty1DconstrIdxs = []
for iCell in range(self.nEmpty):
    self.empty1DconstrIdxs += [[iConstr for iConstr, idxs in enumerate(constr1Didxs) if iCell in idxs]]
```


Second, the `test` function is now uses the cell index as an argument, and tests only the relevant cells

```python
def test(self, idxEmpty1D: int) -> bool:
    # Only test the constraints that are relevant for this cell
    for idxConstr in self.empty1DconstrIdxs[idxEmpty1D]:
        if not self.problem.constraints[idxConstr].test(self.grid):
            return False
    return True
```

## Experiment

There is another optimization I would like to do, but I am not quite sure how to go about it. It surely matters in what order our solver attempts to fill in the digits, but it is not obvious to me what the optimal order would be, or whether there is an optimal order. I am convinced that some heuristics can do better than others, and even if we cannot arrive at an optimal solution, we can try to get a good one. Thus, we will conduct an experiment.

Firstly, we will attempt to fill the digits in a random order. The order of the digits will be selected at the start for the solver, and kept throughout the solution. We will run this a few times and see what solution times we get for different sudoku examples. The implementation of this solution can be found in the full code at the end of this post. In short, we will require an additional mapping between the original 1D sequence of empty cells, and the order in which we will actually visit them. In this case, the new order is a random permutation.

Secondly, I want to try a heuristic which naively seems like it could work. Specifically, we can initially go over all empty cells in the grid and check how many options satisfy the constraints. Then visit the cells in the order of increasing number of options (visiting the most constrained cells first). On the first sight it makes sense - we start with less possible branches, and visit the less constrained cells once a large fraction of the grid has already been filled in. It is what a human would do. Again, the implementation of this is quite simple, compared to what we have already done. It can be found in the full code.

## Results

So, we have 4 different algorithms
* **BruteSolverV1** - the original solver
* **BruteSolverV2** - like V1, but using less constraints
* **BruteSolverV3** - like V2, but filling numbers in a random order
* **BruteSolverV4** - like V2, but filling numbers in the order of increasing number of options (in the initial puzzle).

We also have 5 different example sudoku puzzles
* `grid_easy.dat`
* `grid_easy2.dat`
* `grid_medium.dat`
* `grid_medium2.dat`
* `grid_hard.dat`

In the below table I show the average runtime in seconds for each solver and each problem. For the V3, I give the 90% confidence interval out of 50 different runs.

We can make the following observations:
* V2 indeed performs better than V1, but the improvement factor is less than 9, since we also spend time on reading the puzzle from file and initializing the solver.
* Somewhat surprisingly, V3 performs orders of magnitude worse than V2, or even V1. So the initial arrangement of doing empty cells in sequence is not random at all. It seems that the proximity of cells matters for the order the are filled in.
* V4 is far better than V2 for the hard problem, but far worse for the medium problem. We are clearly onto something with this heuristic, but it is also clear that this heuristic is not the right one.

## Conclusion

We were able to write a sudoku solver can solve a hard classical sudoku puzzles in 10s, which is feasible for daily use. Yay!

The job is certainly not finished yet. We need to figure out the secret of the order of brute-forcing, try some other techniques, and maybe some other sudoku like killer sudoku! Here are the a few ideas I would try from the top of my head:
* Try an adaptive ordering, where, after filling each digit, we find the new most constrained digit and fill that one next. That is as human as it gets
* Try an adaptive ordering for constraints. Choose the constraint with the least unfilled digits, fill it in completely, then select the next constraint with the least unfilled digits.
* Try looking ahead: whenever a digit is filled, check what are the remaining possibilities for other digits. If no possibilities remain for at least one other digit, then this arrangement is already wrong.

The code for the current solution can be found in [this](https://github.com/aleksejs-fomins/SudokuSolver) repository.

Thanks to everybody who read and enjoyed this brief execrise. Stay tuned for more sudoku posts in the future.
