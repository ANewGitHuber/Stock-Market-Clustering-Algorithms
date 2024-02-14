## Stock-Market-Clustering-Algorithms

Datasets:
SP-500_firms.csv: contains firm and ticker names
SP_500_close_2015.csv: contains stock price data for 2015

### Part 1 - Some Effective Tools from pandas Library 

#### Returns

We calculated stock price _returns_ over a period of time. The return is defined as the percentage change, so the return between periods $t-1$ and $t$ for stock price $p$ would be

$$
x_t = \frac{p_t - p_{t-1}}{p_{t-1}}.
$$

Calculate the returns in `pandas` for all the stocks in `price_data`.

#### Correlations

Correlations

Analysts often care about the _correlation_ of stock prices between firms. Correlation measures the statistical similarity between the two prices' movements. If the prices move very similarly, the correlation of their _returns_  is close to 1. If they tend to make exactly the opposite movements (ie one price moves up and the other one down), the correlation is close to -1. If there is no clear statistical relationship between the movements of two stock prices, the correlation in their returns is close to zero.

For a sample of stock price returns $x,y$ with observations for $n$ days, the correlation $r_{xy}$ between $x$ and $y$ can be calculated as:

$$
r_{xy} = \frac{\sum x_i y_i - n \bar{x}\bar{y}}{ns_x s_y} = {\frac {n\sum x_{i}y_{i}-\sum x_{i}\sum y_{i}}{{\sqrt {n\sum x_{i}^{2}-(\sum x_{i})^{2}}}~{\sqrt {n\sum y_{i}^{2}-(\sum y_{i})^{2}}}}}.
$$

Here $\bar{x}$ refers to the average value of $x$ over the $n$ observations, and $s_x$ to its standard deviation.

Based on time series of the stock returns we just computed, we can calculate a  correlation value for each pair of stocks, for example between MSFT (Microsoft) and AAPL (Apple). This gives us a measure of the similarity between the two stocks in this time period.

### Part 2 - Clustering (Kruskal's Algorithm)

Usually, Kruskal's algorithm is used to find a minimum spanning spanning forest of an undirected edge-weighted graph. In our case, rather than connecting nodes considering the minimum distance between them we evaluate the correlation between companies (connecting the highest first). Similarly to Kruskal's algorithm we also connect all nodes although in our scenario we have directed edges which leads us to have a botttom node.

What is a good measure for stocks "performing similarly" to use for clustering. Let's use the one we just calculated: correlations in their returns. How can we use this similarity information for clustering? We now have access to all correlations between stock returns in S&P 500. We can think of this as a _graph_ as follows. The _nodes_ of the graph are the stocks (eg MSFT and AAPL). The _edges_ between them are the correlations, which we have just calculated between each stock, where the value of the correlation is the edge weight. Notice that since we have the correlations between all companies, this is a _dense_ graph, where all possible edges exist.

We thus have a graph representing pairwise "similarity" scores in correlations, and we want to divide the graph into clusters. There are many possible ways to do this, but here we'll use a _greedy_ algorithm design. The algorithm is as follows:

1. Sort the edges in the graph by their weight (ie the correlation), pick a number $k$ for the number of iterations of the algorithm
2. Create single-node sets from each node in the graph
3. Repeat $k$ times:
	1. Pick the graph edge with the highest correlation
	2. Combine the two sets containing the source and the destination of the edge
	3. Repeat with the next-highest weight edge
4. Return the remaining sets after the $k$ iterations 

![cluster0](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/71daf924-4990-483f-b982-c4a7ab754f73)

Suppose we pick $k=3$ and have sorted the edge list in step 1 of the algorithm. How should we represent the clusters in step 2? One great way is to use a dictionary where each _key_ is a node, and each _value_ is another node that this node "points to". A cluster is then a chain of these links, which we represent as a dictionary.

In step 2 of the algorithm, we start with four nodes that point to themselves, ie the dictionary `{'A':'A','B':'B','C':'C','D':'D'}`. When a node points to itself, it ends the chain. Here the clusters are thus just the nodes themselves, as in the figure below.

![cluster1](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/21c0823d-43e1-458b-bc6d-c51927cefa9e)

Let's walk through the algorithm's next steps. We first look at the highest-weight edge, which is between C and D. These clusters will be combined. In terms of the dictionary, this means that one of them will not point to itself, but to the other one (here it does not matter which one). So we make the dictionary at `C` point to `D`. The dictionary becomes `{'A':'A','B':'B','C':'D','D':'D'}`.

![cluster2](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/8d9955af-2c2f-4322-8341-0972bd04f7ff)

The next highest correlation is between A and B, so these clusters would be combined. The dictionary becomes `{'A':'B','B':'B','C':'D','D':'D'}`.

![cluster3](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/60598d8b-b2f0-4cd7-a186-3adfe62ebb7b)

The third highest correlation is between C and B. Let's think about combining these clusters using the dictionary we have. Looking up `B`, we get `B`: the node B is in the _bottom_ of the chain representing its cluster. But when we look up `C`, it points to `D`. Should we make `C` point to `B`? No - that would leave nothing  pointing at `D`, and `C` and `D` should remain connected! We could perhaps have `C` somehow point at both nodes, but that could become complicated, so we'll do the following instead. We'll follow the chain to the bottom. In the dictionary, we look up `C` and see that it points to `D`. We then look up `D` which points to itself, so `D` is the _bottom_ node. We then pick one of the bottom nodes `B` and `D`, and make it point to the other. We then have the dictionary `{'A':'B','B':'B','C':'D','D':'B'}`, and the corresponding clustering in the figure below.

![cluster4](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/833a30e6-3a1e-496d-89e8-1e209d61454e)

In other words, we'll keep track of clusters in a dictionary such that **each cluster has exactly one bottom node**. To do this, we need a mechanism for following a cluster to the bottom. You'll implement this in the function `find_bottom` below. The function takes as input a node and a dictionary, and runs through the "chain" in the dictionary until it finds a bottom node pointing to itself.

(See Jupyter Notebook "Stock Market Clustering" for full description)

#### Result after 20 Iterations

![Company Correlation Network for 20 Iterations](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/f0642e5b-d894-4d6a-ba18-9beff3dd5425)

![Number of Values per Key (20 Iterations)](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/29210c7b-536b-49b3-ad03-1021e4c44eec)

#### Result after 2000 Iterations

![Company Correlation Network for 2000 Iterations](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/d30671e6-f5c7-41c2-a628-8529b10e6805)

![Number of Values per Key (2000 Iterations)](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/249df462-5349-4edb-b9c2-cc7a9415c9a9)

#### Result after 20000 Iterations

![Company Correlation Network for 20000 Iterations](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/1ed116ed-df4a-4d7d-bffa-6e4642f259a8)

![Number of Values per Key (20000 Iterations)](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/ec9abce6-68e5-48d7-8707-3b6bd8420f2d)

### Part 3 - In-depth Analysis & Other Clustering Methods

#### Pairs Trading

Pairs trading is a popular trading strategy in financial markets that involves the simultaneous buying and selling (or shorting) of two related or correlated financial instruments, typically stocks. The goal of pairs trading is to profit from the relative price movements between the two instruments.

- **Pair Selection**: Traders identify a pair of financial instruments that historically have exhibited a high degree of correlation. This correlation indicates that the prices of the two instruments tend to move together in a similar direction.

![Step 1](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/537037ae-5274-4608-94ea-84a6661375f4)


- **Calculation of Spread**: The historical price relationship between the two instruments is analyzed, and a spread is calculated based on the price ratio or the price difference between the two instruments. This spread is used to determine potential trading opportunities.

![Step 2](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/985711bd-b288-49d3-b3a3-cf270ac34283)

![Step 2 - 2](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/8865debf-8008-4efc-afce-0f93f865d453)


- **Establishing Positions**: If the spread deviates from its historical average or a predetermined threshold, traders take positions. If the spread widens beyond the historical norm, they go long on the underperforming instrument (buy the instrument), expecting it to increase in value. Simultaneously, they go short on the outperforming instrument (sell the instrument), expecting it to decrease in value.

![Step 3](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/8d845774-7e45-48db-8f12-a4c2ef0aef6b)

![Step 3 - 2](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/926dc410-00be-4406-b594-7de7cbc8a69c)


- **Monitoring and Reversion to Mean**: Traders monitor the positions and wait for the spread to revert to its historical average or mean. When the spread narrows, they can close their positions, aiming to make a profit. The strategy is based on the belief that the prices of the two instruments will converge over time.

we do the same thing as before but for 2017 values.

![2017 - 2000 Iterations](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/104ca3f1-3a1d-49f1-91df-80ea79a79170)

![2017 - 2000 Iterations(2)](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/7905b9c0-b769-4480-8460-8ba0d12e416e)

![2017 - 100000 Iterations](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/7475e43b-dc13-4536-81d7-7b2e81dd1756)

![2017 - 100000 Iterations(2)](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/730ab496-5b02-4577-93f2-00640eb02e81)

#### K-means

##### Two Dimensional K-means Clustering

![Two Dimensional K-mean Clustering](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/fad33af0-aa65-4fa5-b5cc-afe857591560)

#### Three Dimensional K-means Clustering

![Three Dimensional K-mean Clustering](https://github.com/ANewGitHuber/Stock-Market-Clustering-Algorithms/assets/88078123/b17ea46b-2b34-485a-9d36-8b5550e732c0)
