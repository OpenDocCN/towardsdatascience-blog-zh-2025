# 使用优化来解决对抗性问题

> 原文：[`towardsdatascience.com/using-optimization-to-solve-adversarial-problems-99943614dde8/`](https://towardsdatascience.com/using-optimization-to-solve-adversarial-problems-99943614dde8/)

在之前关于优化的文章中，我探讨了寻找[Betting on the World Series 问题](https://medium.com/towards-data-science/solving-the-classic-betting-on-the-world-series-problem-using-hill-climbing-5e9766e1565d)的优化策略。在这篇文章中，我探讨了一个更困难的问题，为棋盘游戏中的两个对抗性玩家制定策略。

我特别关注猫和老鼠游戏，在这个游戏中，有两个玩家，猫和老鼠，被放置在一个 n x m 的网格上，类似于棋盘，但可以有任意数量的行和列。猫或老鼠可以先移动，然后轮流移动，直到有胜者。在每一步，他们必须向左、右、上或下移动；他们不能停留在同一个单元格中，也不能斜着移动。因此，他们通常有四种可能的移动，但如果在边缘，则只有三种，如果在角落，则只有两种。

如果猫能够捕获老鼠，它就赢了，这是通过移动到老鼠所在的同一天格实现的。老鼠通过足够长时间地躲避捕获而获胜。我们将探讨定义这一点的几种方法。

每个玩家都从对角线位置开始，猫在左下角，老鼠在右上角，因此一开始（如果我们有 8 行和 7 列），棋盘看起来像：

![猫和老鼠游戏的 8 x 7 棋盘起始位置](img/62ced31bca68d0ac74ceee0ae3bbf1d9.png)

猫和老鼠游戏的 8 x 7 棋盘起始位置。

训练猫和老鼠在这款游戏中玩得好的方法有很多。从高层次来看，我们可以将这些方法分为两大类：1) 玩家在游戏中各自决定自己的移动；2) 玩家在游戏开始前，为每种情况确定一个完整的策略，关于他们将如何在每种情况下移动。

第一种方法更普遍地用于游戏，通常更可扩展和稳健，至少对于更复杂的游戏来说是这样。然而，对于这个问题，由于它相对简单，我将探讨学习每个玩家完整、确定性的策略的方法，他们可以在游戏中简单地执行这些策略。

## 游戏中确定每个移动

因此，虽然为猫和老鼠游戏提前确定完整的策略是可行的，但这在更复杂的游戏中并不适用。例如，一个训练来玩棋类的应用程序，无法在实际游戏中提前开发出一种策略来指示如何处理它可能遇到的所有情况；棋类游戏过于复杂，不允许这样做。

在未来的文章中，我将探讨允许玩家在每一步评估他们当前情况的技巧，并根据他们在游戏中的评估进行移动。但是，在这里，我将只提供一个关于确定游戏过程中应采取的走法的技巧的快速概述。

开发例如国际象棋应用程序的方法有很多，但传统的方法是构建所谓的*游戏树*。此外，强化学习通常用于开发游戏应用程序，也可以结合使用。

游戏树描述了每个玩家从当前棋盘开始可以进行的所有可能的走法序列。例如，在国际象棋中，可能是黑方的回合，黑方可能有 8 种合法的走法。游戏树的根节点将代表当前棋盘。由黑方当前可能的走法产生的棋盘是树的下一层，因此树的第二层将有 8 个可能的棋盘。对于这些棋盘中的每一个，白方都有一些合法的走法。如果第二层的 8 个棋盘每个都有白方 10 种合法的回应，那么第三层将有 80 个棋盘。第四层是所有可能的走法产生的棋盘，这些走法是基于第三层的棋盘，依此类推。这样，树的每一层都比前一层大得多。

对于像井字棋或四子棋这样的简单游戏，可以创建完整的游戏树并在每一步确定最佳走法。但对于像跳棋、国际象棋、围棋等更复杂的游戏，这是不可能的。然而，我们可以只为有限步数构建游戏树，并在每个叶节点估计当前玩家的棋盘质量。

因此，如果树扩展到五层深度，我们将在树的第五层拥有一些棋盘，但其中很少是任何一方的胜利。然而，我们必须评估棋盘对一方或另一方是否更有利。对于跳棋或国际象棋等类似游戏，这可以通过简单地计算棋子数量来完成。一种更有效但更慢的方法是同时观察棋盘位置。例如，在国际象棋中，我们可以评估每个棋子的前进程度、暴露程度等。

还可以使用所谓的 _ 蒙特卡洛树搜索 _。在这里，通过玩出每个棋盘的完整游戏来扩展树叶，但通过一系列随机游戏（其中双方完全随机地玩）。这是一种评估每个棋盘的方法，但不需要分析棋盘本身。因此，如果树扩展到 5 层，那么在该层可能有 1000 个棋盘。为了评估这些棋盘中的每一个，我们可以从每个棋盘开始进行一些固定的随机游戏，并计算每个玩家获胜的次数，这为棋盘对双方玩家的强度提供了一个合理的估计。

## 猫捉老鼠游戏的基于策略的解决方案

猫和老鼠游戏相当直接，为猫制定一个完全定义的策略，并为老鼠制定另一个策略（允许两者在游戏过程中以确定性的方式简单地遵循这些策略）实际上是可行的。这在我们所考虑的第一个案例中尤其如此，其中我们假设棋盘是一个已知且固定的大小，并且相对较小。我们稍后会考虑更困难的案例，其中棋盘大小非常大。

![3x3 棋盘上的猫和老鼠游戏](img/85fa2df626f59ba66ddb613996e7824d.png)

3×3 棋盘上的猫和老鼠游戏

在这里展示的图像中，有一个非常小的 3×3 棋盘。在这种情况下，很容易看出猫可以制定一个完全定义的策略，指定当它在任何 9 个方格中的任何一个方格时，以及老鼠在任何一个 9 个方格中的任何一个方格时，它会做什么。考虑到猫和老鼠可能出现的所有组合，总共有 81 种组合，并且可以在这些 81 种场景中训练一个策略以最佳方式玩游戏。也就是说，在猫的策略案例中，在 81 种情况中，我们有一个方向（要么是左、右、上或下），猫将移动，对老鼠的策略也是如此。

## 猫和老鼠游戏中的最佳策略

根据棋盘的大小和形状以及哪个玩家先走，在完美的游戏中（即没有玩家犯错误），猫和老鼠游戏实际上有一个已知的胜者。

为了形象化这一点，考虑一个 3×3 棋盘的案例。与井字棋类似，猫（以及老鼠）可以创建一个游戏树，覆盖它可以做的每一个移动，老鼠可以做出的每一个反应，猫的每一个下一步，等等，直到游戏结束（要么猫抓住老鼠，要么老鼠足够长时间地逃脱）。这样做，鉴于游戏有一个有限且相对较小的长度，可以考虑到所有可能的移动序列，并确定每个可能场景中的理想移动。

然而，我们还想支持更大的棋盘，例如 8×8 的棋盘，考虑到所有可能的移动序列可能是不切实际的。在这里，游戏树可能增长到巨大的大小。因此，如上所述，开发部分游戏树并在叶节点评估棋盘是相当可能的。但是，开发完整的游戏树是不切实际的。

在这些情况下，尽管如此，仍然可以使用爬山优化技术为两个玩家制定完整的策略。以下是一个示例，我们为一个 8×7 的棋盘进行操作。这里棋盘上有 56 个方格，所以猫可能有 56 个位置，老鼠也可能有 56 个位置（实际上有 55 个，因为如果它们在同一方格上，游戏就结束了，猫已经赢了，但我们将通过假设每个玩家可能出现在任何方格上来简化这一点）。

然后他们位置的可能组合有 3,136 种，因此为每个玩家制定策略（至少在使用这里描述的第一个、最简单的方法时——每个玩家都有一个定义好的移动，无论是左、右、上还是下，对于猫和老鼠位置的每一种组合）需要制定一个大小为 3,136 的策略。

这不会扩展到更大的棋盘（我们将在后面介绍一个类似的方法，该方法可以覆盖任意大小的棋盘），但它很好地覆盖了中等大小的棋盘，是一个很好的起点。

## 猫什么时候会赢？

在我们查看这个问题的算法解决方案之前，猫和老鼠的游戏本身就是一个有趣的问题，并且可能在这里稍作停顿，思考一下：猫什么时候能赢，什么时候不能赢（这样老鼠就能赢）？在给出答案之前，我会给你一点时间来思考这个问题。

. . .

思考中…

. . .

再多思考一下…

. . .

等到你准备好看到答案时再来看…

. . .

好的，我会至少介绍一种看待这个问题的方法。像很多类似的问题一样，这可以通过棋盘上的颜色来理解。方格在黑色和白色之间交替。每次移动，老鼠都会从一个颜色移动到另一个颜色，老鼠也是如此。

两位玩家都从某个颜色的方格开始。比如说，猫（在左下角）站在一个黑色方格上。如果行数和列数都是偶数（比如一个 8x8 的棋盘），那么老鼠开始的地方（在右上角）也将是黑色的。

如果猫先走，它会从一个黑色方格移动到白色方格。然后老鼠也会这样做。所以每次猫和老鼠移动后，他们都会处于相同的颜色（要么都在黑色，要么都在白色）。这意味着，当猫移动时，它会移动到与老鼠当前所在颜色相反的方格。猫永远也抓不到老鼠。

在这个例子中，实际上老鼠本身并没有可赢的策略：它只能随机移动，只要它不移动到猫那里（本质上，这是一种自杀性的移动）。但是，猫确实需要一个良好的策略（否则它将无法在允许的步数内捕捉到老鼠），随机移动不太可能赢得比赛（尽管在足够小的棋盘上，这种情况可能会经常发生）。

对于老鼠来说，尽管在这种情况下没有可赢的策略，但仍然有一种最优的玩法感——那就是它至少可以避免被捕捉到尽可能长的时间。

另一方面，如果行数或列数是奇数，或者老鼠先动，猫就可以捕捉到老鼠。要做到这一点，它只需要靠近老鼠，使其与老鼠呈对角线相邻，这将迫使老鼠向两个可能的方向之一远离猫。然后猫可以跟随老鼠，保持与老鼠的对角线相邻，直到老鼠最终被逼入角落并被捕捉。

在这种情况下，当猫采取最佳策略时，老鼠无法获胜，但如果猫没有采取最佳策略，老鼠仍然可以获胜，因为它只需要避免被捕获一定数量的步数。

下一个图像展示了猫斜着移动到老鼠旁边，迫使老鼠走向一个角落（在这个例子中，是右上角）。

![猫斜着移动到老鼠旁边，留给老鼠只有两个合法的移动（向上和向右），这两个移动都不会移动到靠近猫的单元格，而且都会让它更接近角落。如果老鼠选择向左或向下移动，将自己置于猫的旁边，猫就会在下一步简单地将它捕获。](img/4b2c78877f4ddf6265032264db0d85c0.png)

猫在老鼠的斜对面，留给老鼠只有两个合法的移动（向上和向右），这两个移动都不会移动到靠近猫的单元格，而且都会让它更接近角落。如果老鼠选择向左或向下移动，将自己置于猫的旁边，猫就会在下一步简单地将它捕获。

一旦老鼠被困在角落（如下所示），如果猫在老鼠的斜对面，老鼠将失去下一步的行动机会。它只有两个合法的移动，都是向靠近猫的方格移动，猫可以在下一步移动到老鼠所在的位置。

![老鼠现在被困在角落，只能向左或向下移动。这两个移动都允许猫在下一步捕获它。](img/867f0dbddad100c3384792c1c1aa17af.png)

老鼠现在被困在角落，只能向左或向下移动。这两个移动都允许猫在下一步捕获它。

## 训练两种不同的策略

那么，问题是如何从零知识开始训练两位玩家，以玩出完美的一局游戏，这意味着两位玩家都采取最佳策略，并且没有人犯错。

我们可以考虑两种情况：

1.  在游戏中猫可以获胜的情况下。在这种情况下，我们希望确保无论老鼠如何行动，我们都能训练猫可靠地获胜。这意味着学习如何靠近老鼠，并将其逼入角落。此外，我们希望确保老鼠能够尽可能长时间地逃脱捕获。

1.  在游戏中猫无法获胜的情况下。在这种情况下，我们希望确保我们训练老鼠足够好，使其能够长时间逃脱捕获并宣布获胜。同时，我们希望确保猫训练得足够好，以便在老鼠犯错时能够捕获它。

显然，猫和老鼠的游戏远比象棋、跳棋、围棋或奥赛罗等游戏简单，但它确实有一个难点，那就是游戏是不对称的，两位玩家必须各自发展不同的策略。

在只需要一种策略的游戏中，可以允许两位玩家相互对战，并随着时间的推移逐渐发展出更好的策略。这里我们将采取同样的方法，但与训练生成对抗网络时通常的做法类似，代码实际上在训练猫和训练老鼠之间交替。也就是说，它训练猫直到它能够获胜，然后训练老鼠直到它能够获胜，然后是猫，以此类推。

## 确定最优策略的方法

由于目标是开发一个最优策略，因此使用优化技术是相当自然的，我们在这里就是这样做的。这种选择包括爬山、模拟退火、遗传算法和群体智能算法。

每种方法都有其优点，但在这篇文章中，正如在[Betting on the World Series](https://medium.com/towards-data-science/solving-the-classic-betting-on-the-world-series-problem-using-hill-climbing-5e9766e1565d)文章中一样，我们将探讨爬山方法来为两位玩家开发策略。爬山可能是上述优化技术中最简单的一种，但足以处理这个问题。在未来的文章中，我将涵盖更困难的问题和更复杂的解决方案。

对于像这样的游戏中的任何一位玩家，爬山算法都是从某种策略（通常是随机创建的，或者初始化为某种合理的东西）开始的，然后通过反复尝试小的变化，选择其中最好的，对那个进行小的变化，以此类推，直到最终达到看似最优的解决方案。

## 使用一个小型、固定的棋盘大小

如指示，在第一种方法中，我们以尽可能简单的方式开发策略：猫的策略指定了猫的位置和老鼠的位置的每种组合所应采取的具体移动。同样，对于老鼠的策略也是如此。

对于一个 8×7 的棋盘，这需要为每位玩家设置一个大小为 3,136 的策略。最初，策略将被设置为非常糟糕：在这个例子中，我们指定两位玩家简单地总是向上移动，除非在顶层，在这种情况下，他们向下移动。但是，随着时间的推移，爬山过程将逐渐向两位玩家越来越强的策略移动。

与本文相关的代码托管在 github 上，在[CatAndMouseGame](https://github.com/Brett-Kennedy/CatAndMouseGame/tree/main)仓库中。我们现在考虑的是 version_1 笔记本。

第一个单元格包含一些选项，您可以调整这些选项以查看如何使用不同的值进行训练过程。

```py
NUM_ROWS = 8
NUM_COLS = 7

FIRST_MOVE = "cat"           # Set to "cat" or "mouse"
INITIAL_CAT_POLICY = "WEAK"  # Set to "WEAK" or "STRONG"
ANIMATION_TIME_PER_MOVE = 0.5
```

为了简洁起见，我不会涵盖这个 INITIAL_CAT_POLICY，并假设它被设置为‘弱’，但如果设置为‘强’，猫将被初始化为总是向老鼠移动（如果设置为‘弱’，它必须学会这一点）。

代码从初始化棋盘开始，使得两个玩家位于对角位置。它还初始化了两个玩家的策略（如上所述——所以两个玩家总是向上移动，除非在顶层，在这种情况下，他们向下移动）。

然后，它进行一场游戏。由于游戏是确定性的，因此只需要进行一场游戏就可以确定给定猫策略和给定老鼠策略的获胜者。第一场比赛的结果是猫随后试图不断改进的基准，直到它能够击败老鼠。然后我们反复改进老鼠，直到它能够成为猫，依此类推。

这被设置为执行 100,000 次迭代，这需要几分钟的时间，足以让两个玩家都建立起相当好的玩法。

## 评估每场比赛

当猫学习时，它在任何时刻都拥有当前策略：迄今为止发现的最佳策略。然后它在这个策略的基础上创建出多个变体，每个变体都对当前最佳策略进行少量修改（例如，改变猫在策略的 3,126 个单元格中移动的方向）。接着，它通过让这些变体与当前最佳策略的猫进行对弈来评估每个变体。然后猫会选取这些候选新策略中表现最好的一个（除非没有比当前最佳策略更好的策略，在这种情况下，它会继续生成随机变体，直到至少发现一个比当前策略更好的为止）。

为了使爬山法有效，它需要能够检测从一种策略到另一种策略的微小改进。因此，如果玩家只知道在每场比赛后他们是否获胜，那么实施这一点将会很困难。

相反，在每场比赛后，我们报告获胜前的移动次数以及比赛结束时两个玩家之间的距离。当猫获胜时，这将为零。然而，当老鼠获胜时，猫希望最小化这个距离：它希望至少接近老鼠。而老鼠则希望最大化这个距离：它希望远离猫。

通常情况下，对于猫来说，如果先前的最佳策略导致老鼠获胜，而新的策略导致猫获胜，那么就会找到一种改进。但是，如果老鼠在先前的最佳策略中获胜，并且老鼠仍然获胜，但猫最终的位置更接近老鼠。或者，如果猫在先前的最佳策略中获胜，并且仍然获胜，但移动次数更少。

有趣的是，对于猫来说，奖励更长的游戏也是有益的，至少在一定程度上。这鼓励猫更多地移动，不要停留在同一个区域。但我们必须小心，因为我们不希望鼓励猫在能够捕捉到老鼠时比必要的速度更慢。

对于鼠标来说，如果先前的最佳策略导致猫获胜而新的策略导致鼠标获胜，则可以找到一种改进。同样，如果猫在先前的最佳策略中获胜，但仍然获胜，但游戏更长，则也是一种改进。如果鼠标在先前的最佳策略中获胜，但仍然获胜，但距离猫更远，则也是一种改进。

## 版本 1 的代码

完整的代码在此提供，同时在 GitHub 上也有。

在每次迭代中，猫或鼠标中有一个在学习，这里的“学习”意味着尝试新的策略并选择其中最好的。

```py
def init_board():
    # The cat starts in the bottom-left corner; the mouse in the upper-right.
    # The y values start with 0 at the bottom, with the top row being NUM_ROWS-1
    board = {'cat_x': 0, 
             'cat_y': 0, 
             'mouse_x': NUM_COLS-1, 
             'mouse_y': NUM_ROWS-1}
    return board

def draw_board(board, round_idx):
    clear_output(wait=True)
    s = sns.scatterplot(x=[], y=[])
    for i in range(NUM_ROWS):
        s.axhline(i, linewidth=0.5)
    for i in range(NUM_COLS):
        s.axvline(i, linewidth=0.5)    
    s.set_xlim(0, NUM_COLS)
    s.set_ylim(0, NUM_ROWS)
    offset = 0.1 
    size = 250 / max(NUM_ROWS, NUM_COLS)
    plt.text(board['cat_x'] + offset, board['cat_y'] + offset, '🐱 ', size=size, color='brown') 
    plt.text(board['mouse_x'] + offset, board['mouse_y'] + offset, '🐭 ', size=size, color='darkgray')
    s.set_xticks([])
    s.set_yticks([])
    plt.title(f"Round: {round_idx}")
    plt.show()
    time.sleep(ANIMATION_TIME_PER_MOVE)

def set_initial_cat_policy():
    # Initially, the cat is set to simply move towards the mouse
    policy = np.zeros([NUM_COLS, NUM_ROWS, NUM_COLS, NUM_ROWS]).tolist()
    for cat_x in range(NUM_COLS):
        for cat_y in range(NUM_ROWS):
            for mouse_x in range(NUM_COLS):
                for mouse_y in range(NUM_ROWS):

                    if INITIAL_CAT_POLICY == 'WEAK':
                        if cat_y == NUM_ROWS-1:
                            policy[cat_x][cat_y][mouse_x][mouse_y] = 'D'
                        else:
                            policy[cat_x][cat_y][mouse_x][mouse_y] = 'U'                            

                    else: # STRONG
                        dist_x = abs(cat_x - mouse_x)
                        dist_y = abs(cat_y - mouse_y)
                        if dist_x > dist_y:
                            if mouse_x > cat_x:
                                policy[cat_x][cat_y][mouse_x][mouse_y] = 'R'
                            else:
                                policy[cat_x][cat_y][mouse_x][mouse_y] = 'L'
                        else:
                            if mouse_y > cat_y:
                                policy[cat_x][cat_y][mouse_x][mouse_y] = 'U'
                            else:
                                policy[cat_x][cat_y][mouse_x][mouse_y] = 'D'                        
    return policy

def set_initial_mouse_policy():  
    # Intially, the mouse is set to simply move up, unless it is in the top row,
    # in which case it moves down. This will initially cause it to oscillate between
    # the top-right corner and the cell immediately below this.
    policy = np.zeros([NUM_COLS, NUM_ROWS, NUM_COLS, NUM_ROWS]).tolist()
    for cat_x in range(NUM_COLS):
        for cat_y in range(NUM_ROWS):
            for mouse_x in range(NUM_COLS):
                for mouse_y in range(NUM_ROWS):
                    if mouse_y == NUM_ROWS-1:
                        policy[cat_x][cat_y][mouse_x][mouse_y] = 'D'
                    else:
                        policy[cat_x][cat_y][mouse_x][mouse_y] = 'U'
    return policy

def convert_board_to_tuple(board):
    """
    Used to create a dictionary key, which tracks which board positions have
    been seen before. 
    """
    return tuple((board['cat_x'], board['cat_y'], 
                  board['mouse_x'], board['mouse_y']))

def execute(cat_policy, mouse_policy, draw_execution=False, round_idx=None):
    """
    Execute a game given a cat policy and a mouse policy. Return the winner
    as well as stats regarding the number of moves and their distance apart
    at the end of the game. 
    """

    def check_winner(board):
        """
        Determine if either player has won.
        """
        if convert_board_to_tuple(board) in board_history:
            return 'mouse'
        if (board['cat_x'] == board['mouse_x']) and (board['cat_y'] == board['mouse_y']):
            return 'cat'
        return None

    def move_cat(board, cat_policy): 
        """
        Move the cat from one position on the board to another, given the 
        current cat position and mouse position and the cat's policy.
        """
        move = cat_policy[board['cat_x']] 
                         [board['cat_y']] 
                         [board['mouse_x']] 
                         [board['mouse_y']]
        if move == 'R':
            board['cat_x'] += 1
        elif move == 'L':
            board['cat_x'] -= 1
        elif move == 'U':
            board['cat_y'] += 1
        elif move == 'D':
            board['cat_y'] -= 1
        else:
            assert "Invalid move type"                        
        return board

    def move_mouse(board, mouse_policy):
        """
        Move the mouse from one position on the board to another, given the 
        current cat position and mouse position and the mouse's policy.
        """        
        move = mouse_policy[board['cat_x']] 
                           [board['cat_y']] 
                           [board['mouse_x']] 
                           [board['mouse_y']]
        if move == 'R':
            board['mouse_x'] += 1
        elif move == 'L':
            board['mouse_x'] -= 1
        elif move == 'U':
            board['mouse_y'] += 1
        elif move == 'D':
            board['mouse_y'] -= 1
        else:
            assert "Invalid move type"
        return board

    def get_distance(board):
        """
        Return the distance between the cat and mouse.
        """
        return abs(board['cat_x'] - board['mouse_x']) + abs(board['cat_y'] - board['mouse_y'])

    board = init_board()
    board_history = {convert_board_to_tuple(board): True}  

    if FIRST_MOVE == 'cat':
        board = move_cat(board, cat_policy)

    # Execute for at most the possible number of unique board positions. 
    # After this, there must be a cycle if there is no capture.
    for move_number in range(NUM_ROWS * NUM_COLS * NUM_ROWS * NUM_COLS + 1):
        # Move the mouse
        board = move_mouse(board, mouse_policy)
        if draw_execution: 
            draw_board(board, round_idx)
        winner = check_winner(board)
        if winner:
            return winner, move_number, get_distance(board)
        board_history[convert_board_to_tuple(board)] = True

        # Move the cat
        board = move_cat(board, cat_policy)
        if draw_execution: 
            draw_board(board, round_idx)
        winner = check_winner(board)
        if winner:
            return winner, move_number, get_distance(board)
        board_history[convert_board_to_tuple(board)] = True

    # If the mouse evades capture for the full execution, it is the winner
    assert False, "Executed maximum moves without a capture or repeated board"
    return 'mouse', move_number, get_distance(board)

def get_variations(policy, curr_player):
    """
    For a given policy, return a set of similar, random policies.
    """
    num_changes = np.random.randint(1, 11)
    new_policies = []

    for _ in range(num_changes):
        cat_x = np.random.randint(NUM_COLS)
        cat_y = np.random.randint(NUM_ROWS)
        mouse_x = np.random.randint(NUM_COLS)
        mouse_y = np.random.randint(NUM_ROWS)
        direction = np.random.choice(['R', 'L', 'U', 'D'])

        # Skip this variation if the move is illegal (going outside the grid)
        if (curr_player == 'cat') and (cat_x == (NUM_COLS-1)) and (direction == 'R'):
            continue
        if (curr_player == 'cat') and (cat_x == 0) and (direction == 'L'):
            continue
        if (curr_player == 'cat') and (cat_y == (NUM_ROWS-1)) and (direction == 'U'):
            continue
        if (curr_player == 'cat') and (cat_y == 0) and (direction == 'D'):
            continue

        if (curr_player == 'mouse') and (mouse_x == (NUM_COLS-1)) and (direction == 'R'):
            continue
        if (curr_player == 'mouse') and (mouse_x == 0) and (direction == 'L'):
            continue
        if (curr_player == 'mouse') and (mouse_y == (NUM_ROWS-1)) and (direction == 'U'):
            continue
        if (curr_player == 'mouse') and (mouse_y == 0) and (direction == 'D'):
            continue            

        p = copy.deepcopy(policy)
        p[cat_x][cat_y][mouse_x][mouse_y] = direction
        new_policies.append(p)
    return new_policies

np.random.seed(0)
cat_policy = set_initial_cat_policy()
mouse_policy = set_initial_mouse_policy()
winner, num_moves, distance = execute(cat_policy, mouse_policy, draw_execution=True, round_idx="Initial Policies")
prev_winner, prev_num_moves, prev_distance = winner, num_moves, distance

game_stats_winner = []
game_stats_num_moves = []
game_stats_distance = []

# Execute 100,000 iterations. Each iteration we attempt to improve either
# the cat's or the mouse's policy, depending which is weaker at that time.
for round_idx in range(100_000):

    # Display progress as the two players learn
    if (((round_idx % 1000) == 0) and (round_idx > 0)) or 
        (prev_winner != winner) or (prev_num_moves != num_moves) or (prev_distance != distance):
        print(f"Iteration: {round_idx:>6,}, Current winner: {winner:<5}, number of moves until a win: {num_moves:>2}, distance: {distance}")
        prev_winner, prev_num_moves, prev_distance = winner, num_moves, distance

    if winner == 'cat':
        # Improve the mouse
        best_p = copy.deepcopy(mouse_policy)
        best_num_moves = num_moves
        best_distance = distance
        policy_variations = get_variations(mouse_policy, curr_player='mouse')        
        for p in policy_variations:
            p_winner, p_num_moves, p_distance = execute(cat_policy, p)

            # The mouse's policy improves if it starts winning, the execution takes longer, or the execution takes
            # the same number of time, but the mouse ends farther from the cat
            if ((winner == 'cat') and (p_winner == 'mouse')) or 
               ((winner == 'mouse') and (p_winner == 'mouse') and (p_num_moves > best_num_moves)) or 
               ((winner == 'cat') and (p_winner == 'cat') and (p_num_moves > best_num_moves)) or 
               ((winner == 'cat') and (p_winner == 'cat') and (p_num_moves == best_num_moves) and (p_distance > best_distance)):
                winner = p_winner
                best_p = copy.deepcopy(p)
                best_num_moves = p_num_moves
                best_distance = p_distance

        mouse_policy = copy.deepcopy(best_p)
        num_moves = best_num_moves
        distance = best_distance

    else:
        # Improve the cat
        best_p = copy.deepcopy(cat_policy)
        best_num_moves = num_moves
        best_distance = distance
        policy_variations = get_variations(cat_policy, curr_player='cat')
        for p in policy_variations:
            p_winner, p_num_moves, p_distance = execute(p, mouse_policy)

            # The cat's policy improves if it starts winning, or it wins in fewer moves, or it still loses, but 
            # after more moves, or if it still loses in the same number of moves, but it's closer to the mouse
            if ((winner == 'mouse') and (p_winner == 'cat')) or 
               ((winner == 'mouse') and (p_winner == 'mouse') and (p_distance < best_distance)) or 
               ((winner == 'mouse') and (p_winner == 'mouse') and (p_distance == best_distance) and (p_num_moves > best_num_moves)) or 
               ((winner == 'cat') and (p_winner == 'cat') and (p_num_moves < best_num_moves)):
                winner = p_winner
                best_p = copy.deepcopy(p)
                best_num_moves = p_num_moves
                best_distance = p_distance

        cat_policy = copy.deepcopy(best_p)
        num_moves = best_num_moves
        distance = best_distance

    game_stats_winner.append(winner)
    game_stats_num_moves.append(num_moves)
    game_stats_distance.append(distance)

    draw_execution = (round_idx % 10_000 == 0) and (round_idx > 0)
    if draw_execution:
        execute(cat_policy, mouse_policy, draw_execution=True, round_idx=round_idx)

winner, num_moves, distance = execute(cat_policy, mouse_policy, draw_execution=True, round_idx="Final Policies")            
print(f"The {winner} wins in {num_moves} moves.")
```

## 动画化移动

随着笔记本的执行，每 10,000 次迭代，它就会根据猫和鼠标（当时）的策略进行一次游戏。随着时间的推移，我们看到两位玩家都在进行越来越合理的游戏。为此，它调用 clear_output()（由 IPython.display 提供），因为它绘制每个移动，所以清除笔记本的输出单元格并重新绘制两位玩家的当前棋盘位置，从而产生动画效果。

此外，还有一些打印语句描述了两位玩家学习更好策略的进度。

## 版本 1 代码的局限性

版本 1 的笔记本演示了为游戏中的玩家开发完整策略的基本思想，但并未处理我们在一个生产质量系统中希望解决的问题。这是可以接受的，因为这仅仅是一个简单示例，但我会在这里列出一些可以改进的地方（尽管会使代码变得稍微复杂一些），在另一个环境中进行改进。

首先，对于这个版本，作为一个简化，我们宣布如果玩家重复相同的棋盘位置，则鼠标获胜。这并不理想，但在这种情况下有些道理，因为两位玩家的策略都是确定性的——如果他们进入相同的位置两次，我们知道模式将继续重复，猫不会捕捉到鼠标。

代码还可以改进以检测在一段时间内两位玩家都没有改进的情况，以便允许提前停止，或者允许类似模拟退火（允许移动到看似较弱的策略，以跳出局部最优），或者允许测试新的策略，这些策略不仅仅是当前最佳策略的小幅修改，而是更大的修改。

当一位玩家无法击败另一位玩家时，仍然可以允许另一位玩家继续学习，以开发一个越来越强大的策略。

这里采取的另一个简化是，玩家各自尝试简单地击败对方的当前策略。这效果相当不错（因为对方玩家也在不断改进），但为了创建更稳健的玩家，最好是根据策略如何与其他玩家当前策略的表现来评估每个策略，而不是仅基于它如何表现。然而，这是一个更慢的过程，所以在这个笔记本中跳过了。

其中一些在第二个解决方案中得到了解决（该解决方案侧重于处理更大的、未知的棋盘大小，但也提供了一些其他改进），以下将进行介绍。

## 对版本 1 流程的观察

这种学习类型可以被称为协同进化，其中两个智能体一起学习。当一个变得更强大时，它帮助另一个也变得更强大。在这种情况下，两个玩家在训练过程中最终赢得大约一半的游戏。在笔记本的最后打印出每个玩家的总胜利次数，我们有：

```py
mouse    54760
cat      45240
```

在训练过程中，双方可能会有一些意外的走法。这些不太可能是类似于 AlphaGo 对阵李世石（一位非常强大但新且意外的走法）的 Move 78。这些通常只是策略尚未定义合理走法的组合（猫的位置和鼠标的位置）。随着训练的继续，这些组合的数量会减少。

## 使用更通用的策略，允许玩家在任意大小的棋盘上游戏

上述方法在棋盘相对较小的情况下可以满意地工作，但如果棋盘很大，比如说 20 x 20，或者 30 x 35，这就不实用了。例如，对于 30 x 35，我们会有 30 x 35 x 30 x 35 的大小的策略，这需要调整超过一百万个参数。

这是非常不必要的，因为当鼠标相对较远时，猫的移动方式可能无论它们在棋盘上的具体位置如何都是相同的；而当鼠标非常接近且不在任何边缘附近时，猫的移动方式同样可能相同，无论确切位置如何，以及其他类似场景。

可以定义一个策略，用更通用的术语描述如何玩游戏，而不涉及棋盘的具体单元格。

可以为猫（鼠标的策略将是类似的）定义一个策略，从它们位置的性质来看相当通用，但足以详细描述它们的位置，从而可以开发出良好的策略。

这可以包括，例如：

+   鼠标是在猫的上方、下方，还是在同一行

+   鼠标是在猫的左边、右边，还是在同一列

+   鼠标在垂直方向上离猫更近还是水平方向上更近

+   鼠标是否在角落

+   鼠标是否在边缘

+   鼠标不在顶部/底部/左边/右边边缘上，但靠近这些边缘

我们还可以考虑鼠标在每个维度上离边缘是奇数还是偶数个空间，以及它离每个边缘是奇数还是偶数个空间——因为猫试图避免与鼠标形成一个圆形。

另一种解决这个问题的方法不是开发一个单一的策略，而是一系列小的子策略，每个子策略都与第一种方法中开发的类似。在那里，如果我们有一个 5 x 6 的棋盘，我们会开发一个 5 x 6 x 5 x 6 的策略。但也可以定义一系列 3 x 3 策略，为每个玩家确定他们在两个玩家彼此靠近的各种场景中应该如何移动（他们也会有一个或多个子策略来描述当玩家相距较远时玩家应该如何移动）。

例如，我们可以为当两个玩家都在棋盘左上角的 3 x 3 区域内时猫应该如何移动定义一个 3 x 3 策略，当他们在棋盘右上角的 3 x 3 区域内时定义另一个策略，当他们在顶部边缘（但不在角落）时，以及如此等等。

为了简化这一点，我们实际上可以只为角落定义一个策略（根据玩家所在的角落进行旋转）。同样，对于四个边缘，只需要一个策略（而不是四个），以此类推。

下图显示了猫和老鼠彼此靠近且都位于右上角区域的情况，并且可能使用与此情况相关的子策略，仅考虑这里概述的 3 x 3 区域。

![在这里，猫和老鼠都位于右上角的 3 x 3 区域内](img/dac15737f0efc206cf5b069a7f065e94.png)

在这里，猫和老鼠都位于右上角的 3 x 3 区域内

下图显示了这个 3 x 3 区域。这个区域的子策略可以优化并成为玩家完整策略的一部分。优化这个棋盘区域是一个较小、可管理的难题，可以与其他区域的关注点分开。

![一个 3 x 3 的区域，在这种情况下具体代表整个棋盘右上角的 3 x 3 区域。一个子策略可能专门为此区域优化。](img/60530cf7e4ef150b286926eefeacce19.png)

一个 3 x 3 的区域，在这种情况下具体代表整个棋盘右上角的 3 x 3 区域。可以针对这个区域专门优化一个子策略。

如所示，只需要优化一个这样的 3 x 3 策略来处理四个角落，因为一个策略可以通过旋转来适应所有四种情况，以匹配整个棋盘。

## 版本 2 笔记本

在这个（用第二个笔记本[version_2](https://github.com/Brett-Kennedy/CatAndMouseGame/blob/main/version_2.ipynb)编写的）版本 2 中，我们采用了一种通用的方法来训练策略，即在训练策略时，我们不假设任何特定的棋盘大小。这实际上与前面描述的通用解决方案的一些方法略有不同，但方向相同，展示了另一种可能的方法。

这又是基于定义猫和老鼠彼此靠近时的策略，这又定义为在某个共同的 3 x 3 空间内。在这种情况下，我们保持与整个棋盘相同的方向，但向策略中添加其他维度，以指示我们是否位于棋盘的一个或多个边缘上。

因此，这使用了一个 3 x 3 x 3 x 3 x 3 x 3（大小为 729）的策略。前四个元素代表猫和老鼠的 x 和 y 位置。下一个元素有三个维度，指定老鼠相对于棋盘左右边缘的位置。这可以是以下之一：

+   两侧都没有边缘

+   左侧棋盘边缘位于 3×3 区域的左侧

+   右侧棋盘边缘位于 3×3 区域的右侧。

对于最后一个维度也是如此，但这与整个棋盘的上下边缘有关。

即，我们为每种组合都有一个特定的策略：

+   这个 3 x 3 区域内猫的 x 位置

+   这个 3 x 3 区域内猫的 y 位置

+   这个 3 x 3 区域内老鼠的 x 位置

+   这个 3 x 3 区域内老鼠的 y 位置

+   如果这个 3 x 3 区域的左侧位于整个空间的最左侧边缘，如果这个 3 x 3 区域的右侧位于整个空间的最右侧边缘，或者两者都不是这种情况。

+   如果这个 3 x 3 区域的底部位于整个空间的底部，如果这个 3 x 3 区域的顶部位于整个空间的顶部，或者两者都不是这种情况。

例如，当猫和老鼠在棋盘上的任何 3 x 3 网格中时，我们可以执行这个策略，这也会考虑 3 x 3 区域是否与整个棋盘的边缘相邻（如图中所示为粗线）

![当猫和老鼠相对彼此位于 3 x 3 的空间内时，我们可以制定一个与此情况相关的子策略](img/821b0570a6573787c503a14a4c4f3ebd.png)

当猫和老鼠相对彼此位于 3 x 3 的空间内时，我们可以制定一个与此情况相关的子策略

下图显示了它们可能被观察到的 3 x 3 空间。为这个简单情况制定子策略使我们能够忽略棋盘的其他部分，只需关注它们的相对位置以及附近是否有边缘。

![猫和老鼠可能被投影到的 3 x 3 空间。](img/5df76358c62eaef9b3ddc85ae7734bec.png)

猫和老鼠可能被投影到的 3 x 3 空间

因此，这些子策略仅在两个玩家彼此靠近时使用。

为了简化这个笔记本，如果猫和老鼠之间的距离不足以投影到 3 x 3 的子区域，那么猫会简单地移动以减少到老鼠的欧几里得距离。这也可以同样容易地学习，但为了保持这个例子简单，它只涵盖了当它们距离较近时的策略学习。

作为另一种简化，这个笔记本只训练猫。鼠标随机移动，这使得猫能够发展出更稳健的策略，因为它可以被训练到在任意数量的游戏中，无论鼠标如何移动，都能以 100%的频率持续击败鼠标。

鼠标也可以简单地通过使用前一个笔记本中显示的过程进行训练。但，对于这个例子，我主要想专注于扩展上面的例子，以定义可以处理任何棋盘大小的策略。

由于这个笔记本专注于训练猫，我们用一个案例来展示游戏对猫来说是可赢的。

## 版本 2 的训练过程

猫的训练是通过保持与版本 1 一样的一个当前最佳策略来进行的。每次迭代它都会生成这个策略的 10 个随机的小变化，并确定是否有任何比之前的版本更好（如果是这样，就选择这些中的最佳者作为它的新当前最佳策略）。

为了评估每个候选策略，我们使用该候选策略与鼠标进行 1000 场比赛。策略的比较主要基于在 1000 场比赛中击败随机移动的鼠标的比赛数量。它还考虑了（以便过程可以选出稍微更好的策略给猫，即使两者在 1000 场比赛中的获胜次数相同），直到获胜的平均步数（越低越好），以及所有游戏中的平均距离（所有移动的总和）从鼠标（在这里也是一样，越低越好）。

代码实际上分为两个步骤。在第一步中，猫与一个纯粹随机移动的鼠标进行游戏，直到它能够持续击败这个鼠标。然后它开始与一个稍微聪明一点的鼠标进行游戏，即鼠标不会移动到猫旁边的方格，除非没有其他合法的选择。

将训练分为两个步骤并不是绝对必要的。在更大版本的此版本中（目前不在 github 上可用，但可能很快会添加——它包含一些进一步的微小改进，包括训练两个玩家），这被简化为只有一个训练循环，对训练猫所需的时间影响很小。

但它确实提出了一个重要的想法，即 Hill Climbing：创建一个情况，其中策略的小幅改进可以被检测到，在这种情况下，允许猫在 1000 场比赛中获得更多的胜利（因为它最初在一个胜利相当可能的情况下进行游戏）。

运行版本 2 的笔记本，猫需要 30 次迭代才能在所有 1000 场比赛中击败鼠标。然后它开始与更聪明的鼠标进行游戏。最初它只能赢得 821 场中的 1000 场，但在额外的 17 次迭代后，能够持续击败它所有的 1000 场比赛。在这一点上，它试图减少直到获胜所需的步数。

以下展示了切换到更智能的鼠标后前 16 次迭代的输出：

```py
Iteration:      1, Number of wins: 821, avg. number of moves until a win: 8.309, avg_distance: 2.241806490961094
Iteration:      2, Number of wins: 880, avg. number of moves until a win: 8.075, avg_distance: 2.2239653936929944
Iteration:      3, Number of wins: 902, avg. number of moves until a win: 9.143, avg_distance: 2.2353713664032475
Iteration:      4, Number of wins: 950, avg. number of moves until a win: 7.371, avg_distance: 2.1287877056217774
Iteration:      5, Number of wins: 957, avg. number of moves until a win: 7.447, avg_distance: 2.1256372455916117
Iteration:      7, Number of wins: 968, avg. number of moves until a win: 7.433, avg_distance: 2.129003455466747
Iteration:      8, Number of wins: 979, avg. number of moves until a win: 7.850, avg_distance: 2.167468227927774
Iteration:      9, Number of wins: 992, avg. number of moves until a win: 7.294, avg_distance: 2.1520372286793874
Iteration:     10, Number of wins: 993, avg. number of moves until a win: 7.306, avg_distance: 2.15156512341623
Iteration:     11, Number of wins: 994, avg. number of moves until a win: 7.263, avg_distance: 2.1409090350777533
Iteration:     13, Number of wins: 997, avg. number of moves until a win: 7.174, avg_distance: 2.137799442343003
Iteration:     15, Number of wins: 998, avg. number of moves until a win: 7.125, avg_distance: 2.128880373673454
Iteration:     16, Number of wins: 999, avg. number of moves until a win: 7.076, avg_distance: 2.1214920528568437
```

使用 1000 场比赛足够评估猫的表现，并且也能检测到猫的策略中甚至相对较小的改进，例如，当它从赢得 678 场中的 679 场游戏时。尽管这只是微小的改进，但仍然是一个改进。

总的来说，只需要大约 200 次迭代就可以训练出一个强大的猫的策略。

在笔记本中，使用 5 x 5 的棋盘进行训练，因为这允许快速执行游戏，并允许为四个角落中的每一个以及角落之间的边缘开发单独的策略。笔记本的最后一个单元在 15 x 15 的棋盘上执行策略，这证明了发现的策略可以应用于任何棋盘大小。

对于版本 2，我们将老鼠获胜定义为至少逃避捕捉指定次数的移动，其中这个次数是：(NUM_ROWS + NUM_COLS) * 2。这是猫应该能够捕捉到老鼠的移动次数（实际上稍微长于必要的次数，给猫一些灵活性）。这是定义老鼠获胜的更可取的方式，并且在这里是可能的，因为老鼠的移动是非确定性的。

与版本 1 相比，这也更新了猫的适应度函数，以便查看每一步与老鼠的平均距离，而不是游戏结束时的距离。这也允许一旦猫能够可靠地赢得所有 1000 场比赛，游戏就能稳步、逐渐地改进。

## 训练玩家距离较远时的策略

这个例子仅涵盖了处理玩家距离较近时的子策略开发，但完整的解决方案还需要一个子策略来处理玩家距离较远的情况。这可以通过硬编码实现，就像在这个笔记本中一样，但通常更倾向于让优化过程（或游戏树、蒙特卡洛游戏树或其他类似方法）来发现这一点。

为了模拟玩家距离较远时的选择，我们想要捕捉它们位置的相关属性，但不要额外的属性（因为这将需要更多的优化工作）。但是，选择参数可能是一个先入为主的问题——也就是说，提前确定最佳策略，然后简单地定义捕捉该策略所需的参数。

例如，我们可以假设当玩家距离较远时，猫的最佳策略是使到达老鼠的旅行距离最小化（即曼哈顿距离）。因此，我们可以将它们的棋盘位置简单地表示为一个具有四个可能值的单变量，表示老鼠是否最接近左、右、上或下。但是，使用欧几里得距离实际上可能比曼哈顿距离更适合猫和老鼠游戏。此外，捕捉棋盘边缘的信息可能也很有用，这样猫就可以把老鼠推向最近的角落。

也就是说，从最佳策略的假设开始，只捕捉执行该策略所需的棋盘属性，可能会阻止我们找到真正最优的解决方案。

我们可能想要包括一些额外的参数来捕捉那些即使我们怀疑它们可能不相关的因素。一个潜在的集合是，如之前所述：

+   鼠标是在猫的上方、下方还是在同一行？

+   鼠标是在猫的左边、右边还是在同一列？

+   鼠标在垂直方向上离猫更近还是水平方向上离猫更近？

+   鼠标是否在角落？

+   鼠标是否在边缘？

+   鼠标不是完全在顶部/底部/左边/右边边缘，而是接近边缘

与任何建模案例一样，这是一个在捕捉过多细节（训练速度慢，解释困难）和太少（导致次优表现）之间的平衡行为。

## 结论

这里展示的两个示例是解决这个问题的可行选项，并且是这种方法的 useful 例子：基于优化算法（在这种情况下，是爬山法）定义游戏策略的完整策略。

这里提出的思想相当简单，并且并不直接扩展到更复杂的游戏，但可以很好地处理类似甚至稍微复杂一些的问题。例如，它们可以处理在猫捉老鼠游戏中添加的合理数量的复杂性，例如在棋盘上放置障碍物，一个玩家能够以不同的速度移动，存在多个猫或多个老鼠等。

此外，在特定条件下生效的明确子策略是一个有用的想法，可以将其纳入甚至更复杂问题的解决方案中，这些可以通过游戏树或如这里所示的最优化技术来定义。

我将在未来的文章中介绍更高级的方法，但这是一个在适用的情况下有用的方法。

所有图像均由作者提供。
