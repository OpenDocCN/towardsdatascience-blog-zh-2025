# 回溯法的温和介绍

> 原文：[`towardsdatascience.com/a-gentle-introduction-to-backtracking/`](https://towardsdatascience.com/a-gentle-introduction-to-backtracking/)

<mdspan datatext="el1751200321414" class="mdspan-comment">回溯</mdspan>是一种通用的技术，用于探索各种类型的数据科学问题的解空间，并逐步构建候选解决方案——有点像在迷宫中导航。在这篇文章中，我们将在深入研究用 Python 编写的几个直观的、实践示例之前，简要介绍回溯法的基本概念。

***注意：**以下各节中的所有示例代码都是由本文作者创建的。*

## 概念概述

在高层次上，回溯技术涉及对计算问题（通常是可以表述为约束满足或组合优化的一个问题）的解空间的逐步探索。在探索的每一步中，我们沿着不同的路径前进，检查在前进过程中问题约束是否得到满足。

如果我们在探索过程中发现了一个有效解决方案，我们将记录下来。在这种情况下，如果我们的问题只需要找到一个有效解决方案，我们就可以结束搜索。如果问题要求找到多个（或所有）可能的解决方案，我们可以继续探索先前发现的解决方案的扩展。

然而，如果在任何时刻违反了问题约束，我们将进行回溯；这意味着回到我们搜索中构建部分解决方案的最后一个点（并且那里仍然可能存在有效解决方案），并从那里沿着不同的路径继续搜索。这种向前和向后的探索过程可以根据需要继续进行，直到整个解空间被探索并且所有有效解决方案都被探索。

## 实践示例

### 解决数独问题

数独谜题是约束满足问题的经典示例，它在从[运筹学](https://arxiv.org/pdf/1210.2584)到[密码学](https://www.techscience.com/cmc/v77n1/54507)等不同领域的实际应用中都有应用。谜题的标准版本由一个 9x9 的网格组成，由九个不重叠的 3x3 子网格（或块）组成。在谜题的起始配置中，网格中的 81 个单元格中的一些预先填充了从 1 到 9 的数字。为了完成谜题，剩余的单元格必须填充 1 到 9 的数字，同时遵守以下约束：没有一行、一列或 3x3 的块可以包含重复的数字。

下面的 Python 代码展示了如何使用回溯法实现数独解算器，以及一个用于格式化打印网格的便利函数。请注意，解算器期望空单元格用零表示（或初始化）。

```py
from copy import deepcopy

def is_valid(board, row, col, num):
    # Check if num is not in the current row or column
    for i in range(9):
        if board[row][i] == num or board[i][col] == num:
            return False
    # Check if num is not in the 3-by-3 block
    start_row, start_col = 3 * (row // 3), 3 * (col // 3)
    for i in range(start_row, start_row + 3):
        for j in range(start_col, start_col + 3):
            if board[i][j] == num:
                return False
    return True

def find_empty_cell(board):
    # Find the next empty cell (denoted by 0)
    # Return (row, col) or None if puzzle is complete
    for row in range(9):
        for col in range(9):
            if board[row][col] == 0:
                return row, col
    return None

def solve_board(board):
    empty = find_empty_cell(board)
    if not empty:
        return True  # Solved
    row, col = empty
    for num in range(1, 10):
        if is_valid(board, row, col, num):
            board[row][col] = num
            if solve_board(board):
                return True
            board[row][col] = 0  # Backtrack
    return False

def solve_sudoku(start_state):
    board_copy = deepcopy(start_state)  # Avoid overwriting original puzzle
    if solve_board(board_copy):
        return board_copy
    else:
        raise ValueError("No solution exists for the given Sudoku puzzle")

def print_board(board):
    for i, row in enumerate(board):
        if i > 0 and i % 3 == 0:
            print("-" * 21)
        for j, num in enumerate(row):
            if j > 0 and j % 3 == 0:
                print("|", end=" ")
            print(num if num != 0 else ".", end=" ")
        print()
```

现在，假设我们输入一个数独谜题，用零初始化空单元格，并按以下方式运行解算器：

```py
puzzle = [
    [5, 0, 0, 0, 3, 0, 0, 0, 7],
    [0, 0, 0, 4, 2, 7, 0, 0, 0],
    [0, 2, 0, 0, 6, 0, 0, 4, 0],
    [0, 1, 0, 0, 9, 0, 0, 2, 0],
    [0, 7, 0, 0, 0, 0, 0, 5, 0],
    [4, 0, 6, 0, 0, 0, 7, 0, 1],
    [0, 4, 2, 0, 7, 0, 6, 1, 0],
    [0, 0, 0, 0, 4, 0, 0, 0, 0],
    [7, 0, 0, 9, 5, 6, 0, 0, 2],
]

solution = solve_sudoku(puzzle)
print_board(solution)
```

解算器将在毫秒内生成以下解决方案：

```py
5 6 4 | 1 3 9 | 2 8 7 
1 9 8 | 4 2 7 | 5 6 3 
3 2 7 | 8 6 5 | 1 4 9 
---------------------
8 1 5 | 7 9 4 | 3 2 6 
2 7 9 | 6 1 3 | 8 5 4 
4 3 6 | 5 8 2 | 7 9 1 
---------------------
9 4 2 | 3 7 8 | 6 1 5 
6 5 3 | 2 4 1 | 9 7 8 
7 8 1 | 9 5 6 | 4 3 2
```

### 破解数学奥林匹克问题

[数学奥林匹克竞赛](https://en.wikipedia.org/wiki/International_Mathematical_Olympiad)是面向预大学学生的竞赛，包括必须在不使用计算器的情况下在规定时间内解决的困难数学问题。由于通常无法系统地探索此类问题的完整解决方案空间，成功的解决方案方法往往依赖于分析推理和数学创造力，利用从问题陈述中获得的显式和隐式约束来简化解决方案空间的搜索。一些问题与约束满足和组合优化有关，我们在工业界的数据科学问题中也会遇到（例如，检查是否存在通往给定目的地的路径，找到通往目的地的所有可能路径，找到通往目的地的最短路径）。因此，即使对于特定的奥林匹克问题存在巧妙的数学解决方案方法，调查其他可推广的方法（如回溯法）也是有益的，这些方法利用了今天计算机的力量，并可用于实际中解决一系列类似问题的广泛问题。

例如，考虑以下问题，该问题出现在 2018 年 11 月英国数学奥林匹克竞赛第一轮中：*“在黑板上按升序写下五个两位正整数。这五个整数都是 3 的倍数，且黑板上每个数字 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 都恰好出现一次。以这种方式可以有多少种做法？请注意，两位数不能以数字 0 开头。”*

恰好，上述问题的解是 288。下面的视频解释了一种巧妙利用特定问题陈述中一些关键显式和隐式特征的解决方案方法（例如，解决方案必须以有序列表的形式呈现，一个数字是 3 的倍数，如果其各位数字之和也是 3 的倍数）。

下面的 Python 代码展示了如何使用回溯法来解决该问题：

```py
def is_valid_combination(numbers):
    # Checks if each digit from 0-9 appears exactly once in a list of numbers
    digits = set()
    for number in numbers:
        digits.update(str(number))
    return len(digits) == 10

def find_combinations():
    multiples_of_3 = [i for i in range(12, 100) \
                        if i % 3 == 0 and '0' not in str(i)[0]]
    valid_combinations = []
    def backtrack(start, path):
        if len(path) == 5:
            if is_valid_combination(path):
                valid_combinations.append(tuple(path))
            return
        for i in range(start, len(multiples_of_3)):
            backtrack(i + 1, path + [multiples_of_3[i]])
    backtrack(0, [])
    return valid_combinations

print(f"Solution: {len(find_combinations())} ways")
```

函数 `is_valid_combination()` 指定了在搜索空间探索过程中发现的每个有效 5 数列表必须满足的关键约束。列表 `multiples_of_3` 包含可能出现在有效 5 数列表中的候选数字。函数 `find_combinations()` 应用回溯法，有效地尝试 `multiples_of_3` 中的所有独特的 5 数组合。

函数 `is_valid_combination()` 和用于生成 `multiples_of_3` 的列表推导式可以修改，以解决一系列类似的问题。

## 超越回溯法

正如我们所见，回溯法是一种简单而强大的技术，用于解决不同类型的约束满足和组合优化问题。然而，其他技术，如深度优先搜索（DFS）和动态规划（DP）也存在，并且表面上可能看起来相似——那么何时使用回溯法而不是这些其他技术是有意义的呢？

回溯法可以被视为一种更策略性的深度优先搜索（DFS）形式，其中约束检查是每个决策步骤的核心特性，无效路径可以提前放弃。同时，动态规划（DP）可能用于具有两个特性的问题：*重叠子问题*和*最优子结构*。如果一个问题的子问题在解决更大问题时需要多次解决，则该问题具有重叠子问题；存储和重用重复子问题的结果（例如，使用记忆化）是动态规划的关键特性。此外，如果一个问题的最优解可以通过构建其子问题的最优解来构造，则该问题具有最优子结构。

现在，考虑 N 皇后问题，该问题研究如何在 N×N 的国际象棋棋盘上放置 N 个皇后，使得没有两个皇后可以相互攻击；这是一个经典问题，在需要找到无冲突解决方案的几个现实场景中有应用（例如，资源分配、调度、电路设计和机器人的路径规划）。N 皇后问题本身不具有重叠子问题或最优子结构，因为子问题可能不需要重复解决以解决整体问题，并且棋盘某一部分的皇后放置并不能保证整个棋盘的最优放置。因此，N 皇后问题的固有复杂性使其不太适合利用动态规划的优势，而回溯法与问题的结构更自然地吻合。
