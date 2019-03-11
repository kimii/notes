# Identifying influential nodes in complex networks
--------
## 摘要
- 旧方法及其局限
	- degree centrality: 简单但 little relevance
	- betweenness centrality and closeness centrality: 不适用于大规模网络由于计算复杂度 
- 新方法及特点
	- semi-local centrality: 在 low-relevant degree centrality 和其它 time-consuming measures 之间做权衡
- SIR( Susceptible–Infected–Recovered ) 模型
	- 使用 spreading rate 和 the number of infected nodes 评估性能
	- 4 个真实网络表明该方法能很好识别重要节点

## 节点重要度计算
### 半局部
- 3 step
  1. 计算相邻 + 相邻的相邻节点数，得 N(v)

  2. 计算 Q(v) = 相邻节点N(v)和
     $$
     Q(u) = \sum_{w\in \Gamma(u)}{N(w)}
     $$

  3. 计算 Cl(v) = 相邻节点Q(v)和
     $$
     C_{L}(u) = \sum_{u\in \Gamma(u)}{Q(u)}
     $$

- 对比
  - 比  degree centrality 信息多
  - 比  betweenness 和 closeness centralities 计算复杂度低

### 优点
- 计算简单
- 避免计算最短路径
  - 计算网络中所有最短路径复杂度：从 O(n^3) -(sparse graph稀疏图)-> O(n^2logn+nm)

  - 无权网络计算 betweenness centrality，<k> 是网络平均节点度
    - $$
      O(nm) = O(n^{2}<k>)
      $$

  - 该算法复杂度
    - $$
      O(n<k>^{2})
      $$



# ALGORITHM COMPAER
--------
## PageRank
- 算法简单
  - 基于 Markov 链，转移概率来自于在一网页上重新选择网页的概率(随机游走)，
    - $$
      1 - d = 1 - 0.85
      $$

  - PR值计算，N 指所有网页数量，n指网页j链出到网页i数量  
    - $$
      PR(i) = \frac{(1-d)}{N}+d\sum_{j=1}^{n}{\frac{PR(T_{j})}{C(T_{j})}}
      $$

    - 推导：当前网页 j，计算下一步浏览网页 i 的概率，可能性有两种(d 为选择当前网页上链接而不重新选择的概率，取0.85)
      1. 网页 i 被网页 j 链出
         $$
         p_{1} = \frac{d}{C_{j}} + \frac{1-d}{N}
         $$

      2. 网页 i 不被网页 j 链出
         $$
         p_{2} = \frac{1-d}{N}
         $$

      3. 令 g(ij) 为网页 i 被网页 j 链出概率，则网页 j 到 i 的转移概率
        - $$
          a_{ij} = g_{ij}p_{1}+(1-g_{ij})p_{2}=\frac{dg_{ij}}{C_{j}}+\frac{1-d}{N}
          $$

  - 计算：基于转移矩阵S(n*n)表示网页间出链，可以采用幂迭代法求解

    - $$
      A = dS + \frac{1-d}{N}ee^{T}
      $$

    - $$
      P_{n+1} = AP_{n}
      $$

- 局限性
  - 收敛性
  - 迭代次数
  - 大规模数据
  - 基于有向图

- 改进
  - TrustRank
  - HITS


# 对比/认识/问题
--------
- PageRank 算法甚至比计算 betweenness 和 closeness centrality 更加 time-consuming,而使用 local centrality 计算更容易
- PageRank 算法背后的思想是网页链接，是有向图，而 local centrality 计算无向图
- 指标计算或者说评价一个指标的好坏应当依赖模型，如 semi-local centrality 依赖 SIR model
- semi-local centrality 的计算考虑
	- 第一轮计算节点度只选取相邻+相邻的相邻？
	- 后面只迭代 2 轮？
- 指标比较
	- local centrality 比 betweenness cenntrality 和 degree centrality
	- 与 closeness centrality 几乎一样好，但计算更容易
- local centrality 适用网络结构
	- heterogenerous network(异构网络，即非均匀网络，特点是存在很多低度数点)
