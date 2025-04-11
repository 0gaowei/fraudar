# FRAUDAR: Bounding Graph Fraud in the Face of Camouflage

**作者：** Bryan Hooi, Hyun Ah Song, Alex Beutel, Neil Shah, Kijung Shin, Christos Faloutsos

FRAUDAR 是一款软件，旨在以抗伪装的方式检测图数据集（如评论图、Twitter 关注图等）中的欺诈块。它能够识别异常密集且列度较低的块。

## 许可协议

本软件采用 [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0) 许可。除非适用法律要求或书面同意，根据许可分发的软件按“原样”提供，不附带任何明示或暗示的担保或条件。具体许可权限和限制请参阅许可证。

## 版本与联系方式

- **版本：** 1.1
- **日期：** 2018年6月12日
- **主要联系人：** Bryan Hooi (bhooi@andrew.cmu.edu)

## 在测试示例上运行 FRAUDAR

运行 FRAUDAR 的最简单方法是使用 `run_greedy.py`，例如：

```bash
python run_greedy.py data/example.txt out/out
```

此命令在不使用节点可疑值的情况下运行贪婪算法，处理一个示例数据集（一个稀疏数据集，其中前 20 个索引注入了一个 20 x 20 的团）。输出将保存为 `out/out.rows` 和 `out/out.cols`，分别包含检测到的块的行索引和列索引。

**注意：** 示例数据集包含一个稀疏数据集，其中前 20 个索引注入了一个 20 x 20 的团，贪婪算法应能检测到此块。

### 可选：包含节点可疑值

如果您想包含节点可疑值（从外部来源获得），可以在第三个参数中添加其路径：

```bash
python run_greedy.py data/example.txt out/out data/susp
```

在这种情况下，可疑值应存储在 `data/susp.rows` 和 `data/susp.cols` 中。

**提示：** 如果默认检测返回的块过大，可以通过传递常数正向量作为节点可疑值，以获得更小且更密集的块。

### 依赖项

- `numpy`
- `scipy`

这些依赖可以单独安装，也可以通过 Anaconda 等捆绑包安装。

### 公共数据集

- 论文中使用的 Amazon 数据集 `arts.csv` 可从 [SNAP](https://snap.stanford.edu/data/) 获取。
- 论文中使用的 Twitter 数据集可从 [KAIST](http://an.kaist.ac.kr/traces/WWW2010.html) 获取。

## `greedy.py` 中的函数

`greedy.py` 中可用的主要函数包括：

- **`readData`**：读取文件并将其转换为 Python 稀疏矩阵。输入文件应为以空格分隔的格式，每行表示一条边，格式如下：
  ```
  row_index col_index [weight]
  ```
  `weight` 为可选值，若存在应为 1，但会被忽略，因为假设图是无权的。

- **`aveDegree`**, **`sqrtWeightedAveDegree`**, **`logWeightedAveDegree`**：这些函数使用不同的加权方案检测块。
  - **输入：** Python 稀疏矩阵（由 `readData` 返回）以及一个可选的元组 `(row_susp, col_susp)`，表示节点可疑值向量。若未提供，默认值为 `None`，表示不使用节点可疑值。
  - **输出：** 一个元组 `((row_indices, col_indices), score)`，表示检测到的块的行和列索引及其得分。
  - **加权方案：**
    - `aveDegree`：无加权（所有边权重为 1）。
    - `sqrtWeightedAveDegree`：每条边的权重为 \( \frac{1}{\sqrt{\text{column degree}}} \)。
    - `logWeightedAveDegree`：每条边的权重为 \( \frac{1}{\log(\text{column degree})} \)。

### 辅助函数

- **`subsetAboveDegree`**：移除低于某个度数的所有边。
- **`detectMultiple`**：检测多个块。
- **`shuffleMatrix`**：随机打乱矩阵的行和列。
- **`injectCliqueCamo`**：注入团及其各种伪装。
- **`jaccard`**, **`getPrecision`**, **`getRecall`** 等：根据真实值元组测量检测到的 `(row_indices, col_indices)` 元组的准确性。
