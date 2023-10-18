---
layout: page
title: CS50 Search Week0
---
AI入门课程通关记录，和北大的人工智能选修课讲解的知识基本一致

# Note

*   在这门课程中，我们将会探索下面几个主题，来让AI变得可能：

    *   Search
    *   Knowledge
    *   Uncertainty
    *   Optimization
    *   Learning
    *   Neural Networks
    *   Language
*   搜索算法介绍

    *   Depth-First Search 深度优先搜索

        *   stack
    *   Breadth-First Search 广度优先搜索

        *   queue
    *   uninformed search：没有使用特定问题知识的搜索
    *   informed search：使用了特定知识的搜索
    *   greedy best-first search：贪心搜索，使用一个启发函数来评估接近目标程度
    *   A\* search：优先选择最低成本的节点，成本= g(n) + h(n)

        *   g(n)：到达节点的成本，即起点到该节点的距离
        *   h(n)：到达目标的预估成本
    *   Adversarial Search：对抗搜索/ 博弈搜索
    *   Minimax：最小最大算法，对抗搜索算法的一种；

        *   有两个玩家，一个追求得到最小分数 min，一个追求得到最高分数max
        *   两个玩家交替行动
        *   有一个递归推理过程，最大化玩家为每个状态生成可能由当前状态下产生的值。
    *   Alpha-Beta Pruning 剪枝：对最小最大算法的一种优化；
    *   Depth-Limited Minimax

        *   evaluation function：从某个状态评估期望实用性

# Project 0

*   Degrees

    *   写一个程序来确定最少多少次能让两个演员产生联系

        *   输入两个演员名字，两个人是同一部电影的演员，则认为他们有联系；联系可以传递
        *   使用广度优先算法找出最短路径(确保是最短路径！)，再用深度优先算法找出最短路径
        *   用递归来实现会让代码更加简洁。

```python
def draw_shortest_path(cur_node, target, path, explored_set, cur_degree, degree):
    # check terminal condition
    if cur_degree == degree:
        print("cur_degree: ", cur_degree, " cur_person_id: ", cur_node.state)
        if cur_node.state == target:
            path[cur_degree - 1] = [cur_node.action, cur_node.state]
            return True
        return False
    # add to path
    cur_person_id = cur_node.state
    if cur_degree > 0:
        path[cur_degree - 1] = [cur_node.action, cur_person_id]
    neighbors = neighbors_for_person(cur_person_id)
    explored_set.add(cur_person_id)
    for neighbor in neighbors:
        if neighbor[1] in explored_set:
            continue
        next_node = Node(neighbor[1], cur_person_id, neighbor[0])
        res = draw_shortest_path(next_node, target, path, explored_set, cur_degree + 1, degree)
        if res:
            return True
    return False


def shortest_path(source, target):
    """
    Returns the shortest list of (movie_id, person_id) pairs
    that connect the source to the target.

    If no possible path, returns None.
    """

    explored_set = set()
    frontier = QueueFrontier()
    frontier.add(source)
    # level --> [0] -- before [1] -- cur
    paths = list()
    found = False
    level = 1
    new_level = 0
    degree = 0
    # BFS could make sure that it is the shortest path
    while not frontier.empty():
        degree += 1
        while level > 0 :
            level -= 1
            person_id = frontier.remove()
            if person_id in explored_set:
                continue
            explored_set.add(person_id)
            neibors = neighbors_for_person(person_id)
            print(level, " neighbors len: ", len(neibors), " person_id: ", person_id)
            # check terminal conditions
            for pair in neibors:
                if pair[1] == person_id:  # skip redundent
                    continue
                if pair[1] in explored_set:
                    continue
                if target == pair[1]:
                    found = True
                    break
                frontier.add(pair[1])
                new_level += 1
            if found:
                break
        level = new_level
        new_level = 0
        if found:
            break
    if not found:
        return None
    path = [None] * degree
    # DFS print the shortest path; recursive way is much easier to implement
    explored_set.clear()
    cur_node = Node(source, None, None)
    res = draw_shortest_path(cur_node, target, path, explored_set, 0, degree)
    if res:
        return path
    return None
```

*   Tictactoe

    *   Alpha-beta pruning可以让程序更高效！
    *   注意player的计算，要根据board实时算出来，通过比较X和O的数量实现

```python
def player(board):
    """
    Returns player who has the next turn on a board.
    """
    x_count = 0
    o_count = 0
    for i in range(3):
        for j in range(3):
            if board[i][j] == "X":
                x_count += 1
            elif board[i][j] == "O":
                o_count += 1
    if x_count == o_count:
        return "X"
    else:
        return "O"

def actions(board):
    """
    Returns set of all possible actions (i, j) available on the board.
    """
    res = set()
    for i in range(3):
        for j in range(3):
            if board[i][j] == EMPTY:
                res.add((i, j))
    return res


def result(board, action):
    """
    Returns the board that results from making move (i, j) on the board.
    """
    if action is None:
        return board
    new_board = list()
    p = player(board)
    for i in range(3):
        new_board.append([j for j in board[i]])
    new_board[action[0]][action[1]] = p
    return new_board


def winner(board):
    """
    Returns the winner of the game, if there is one.
    """
    for i in range(3):
        if board[i][0] != EMPTY and board[i][0] == board[i][1] and board[i][1] == board[i][2]:
            return board[i][0]
    for i in range(3):
        if board[0][i] != EMPTY and board[0][i] == board[1][i] and board[1][i] == board[2][i]:
            return board[0][i]
    if board[1][1] == EMPTY:
        return None
    if board[0][0] != EMPTY and board[0][0] == board[1][1] and board[1][1] == board[2][2]:
        return board[0][0]
    if board[0][2] != EMPTY and board[0][2] == board[1][1] and board[1][1] == board[2][0]:
        return board[0][2]
    return None


def terminal(board):
    """
    Returns True if game is over, False otherwise.
    """
    # there is a winner or there is no empty in board
    if winner(board) != None:
        return True
    for i in range(3):
        for j in range(3):
            if board[i][j] == EMPTY:
                return False
    return True

def utility(board):
    """
    Returns 1 if X has won the game, -1 if O has won, 0 otherwise.
    """
    who = winner(board)
    if who == "X":
        return 1
    elif who == "O":
        return -1
    else:
        return 0

def maxvalue(board):
    if terminal(board):
        return utility(board)
    action_set = actions(board)
    value = -2
    for action in action_set:
        new_value = minvalue(result(board, action))
        if new_value > value:
            value = new_value
        if value == 1:
            break
    return value

def minvalue(board):
    if terminal(board):
        return utility(board)
    action_set = actions(board)
    value = 2
    for action in action_set:
        new_value = maxvalue(result(board, action))
        if new_value < value:
            value = new_value
        if value == -1:
            break
    return value

def minimax(board):
    """
    Returns the optimal action for the current player on the board.
    """
    if terminal(board):
        return None
    # find the optimal move
    p = player(board)
    action_set = actions(board)
    if len(action_set) == 0:
        return None
    # traverse all the possible move
    action = None
    max_value = -2
    min_value = 2
    for pair in action_set:
        if p == "X":
            new_value = minvalue(result(board, pair))
            if new_value > max_value or (max_value == new_value and winner(board) == "X"):
               max_value = new_value
               action = pair
            if max_value == 1:  # max value reached
               break
        elif p == "O":
            new_value = maxvalue(result(board, pair))
            # print("minimizing player new_value: ", new_value, " value: ", min_value)
            if min_value > new_value:
               min_value = new_value 
               action = pair
            if min_value == -1:
               break
    return action
```

有一个地方不太理解，这里X玩家可以直接下在\[0][0]的位置，AI会多下一步，不知道是我实现有问题还是算法本来就是这样。

|      |  X   |  X   |
| :--: | :--: | :--: |
|  O   |      |      |
|  X   |  O   |  O   |

# Reference

*   <https://cs50.harvard.edu/ai/2023/>
*   <https://cdn.cs50.net/ai/2023/x/projects/0/degrees.zip>
*   <https://cdn.cs50.net/ai/2023/x/projects/0/tictactoe.zip>

