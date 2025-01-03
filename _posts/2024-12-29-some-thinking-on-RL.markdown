MDP = <S, A, R, P0, P>



在MDP理论下，强化学习最优策略必定存在（最优策略的定义为最大化期望回报）

一点思考：

将最大化期望回报作为最优策略的定义是否合理，毕竟期望本身就带有不确定性，或者说将最大化<期望>回报作为理想行动方针是否合理，能否有某种确定性目标（不以期望为目标）定义的最优策略使得我们以及可以逼近最大期望回报的最优策略，因为在这种确定性目标下，到达该定义下的最优策略可能非常容易到达。
或许精炼地说，以某种确定性目标作为最大化期望回报目标的替代，而在该目标下较易得到的最优策略的最大化期望回报非常接近于最大化期望回报目标的最优策略。



根据Bellman期望方程和Bellman最优方程，用探索充分的数据进行更新，Q函数可以逼近当前策略Q函数和最优策略Q函数。
一次采样数据的更新相当于bellman期望方程和最有方程的一次算子运算，理论上可以证明当进行足够多算子运算后，Q函数到达不定点，即此时满足bellman期望方程或bellman最优方程。



--以下情况均以Bellman最优方程作为背景

在DQN中，将多个样本同时用于Q网络更新，并视作对相应状态动作对的Q函数做一次算子运算，那么在经过多次Q网络更新后，理论上Q网络可以逼近最优策略Q函数。
然而，DQN中Q网络本身是一个非线性函数估计器，这使得即便数据足够多，更新次数足够多，都无法保证所有状态动作对的Q值同时满足最优策略Q函数值，即可能有特定的状态动作对的Q值存在误差。

那很显然，我们应该在每次更新时，使得更新前后的误差尽可能的小，这就得到了Q网络参数的更新方向。



现在我们看看更新过程，算子运算即为一次采样估计，而此时会用到当前Q网络，而Q网络本身就有误差，更新方向就会因当前Q网络的误差而逐次产生误差，那么多次更新的误差就会累计得到离谱的结果，所以DQN采用目标网络的方法，隔一段时间将Q网络和目标网络对齐，那么再这段时间内的误差就不再是逐次的，而是隔段的，在一定程度上减少了得到离谱结果的可能性。



注意最优方程的一次算子运算是在s，a条件下s', max_a'{Q(s', a')}的估计，所以我们可以保存（s, a, r, s'）的数据,然后利用Q网络得到max_a'{Q(s', a')}，所以出现了经验回放的机制。当然也可以直接利用在线策略得到的数据进行更新，但是由于决策的连贯性，可能有很多状态动作对无法得到充分采样，这会使得最后的Q网络的性能。当有了经验回放机制，某些稀有状态动作对亦可以用于Q网络的更新，直觉上确实可以有助于算法的有效性。如果没有经验重放机制，为了算法有效性，我们应该通过在线策略采样尽可能多的数据才能达到拥有经验重放机制的算法才能达到的效果，从这一角度看，经验重放机制同时提高了样本效率即减少了与环境交互的次数。



综上，Bellman最优方程是理论基础，一次更新视作一次算子运算，并且足够多次算子运算后，Q网络逼近最优策略Q函数，是整体思路。

存在的挑战是：

1.Q网络本身的架构。是否存在更有的架构，使得在每次更新后，对于所有的状态动作对，都更逼近于之前Q网络的Q值的估计？

2.用于更新的数据的充分性。如何在每次更新时得到充分的数据？

3.更新过程中的误差累积问题。是否存在其他创新方法，使得当前Q网络的误差不会因为更新目标而传递到更新后的Q网络？