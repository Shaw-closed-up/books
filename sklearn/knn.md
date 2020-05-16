# sklearn K近邻(KNN)

本章将帮助您了解Sklearn中最接近的邻居方法。

基于邻居的学习方法有两种类型，即有**监督的**和**无监督的。**基于监督的基于邻居的学习既可以用于分类预测问题，也可以用于回归预测问题，但是它主要用于行业中的分类预测问题。

基于邻居的学习方法没有专门的训练阶段，而是在分类时将所有数据用于训练。它还不假定有关基础数据的任何信息。这就是它们本质上是惰性和非参数化的原因。

最近邻居方法背后的主要原理是-

- 查找距离新数据点最近的壁橱中预定数量的训练样本
- 从这些数量的训练样本中预测标签。

在这里，样本数可以是用户定义的常数，例如在K近邻学习中，也可以根据点的局部密度而变化，例如在基于半径的邻居学习中。

### sklearn.neighbors模块

Scikit-learn具有**sklearn.neighbors**模块，该模块为无监督和受监督的基于邻居的学习方法提供功能。作为输入，此模块中的类可以处理NumPy数组或**scipy.sparse**矩阵。

## 算法类型

可以在基于邻居的方法的实现中使用的不同类型的算法如下-

### 蛮力

数据集中所有点对之间距离的强力计算提供了最幼稚的邻居搜索实现。从数学上来说，对于D个维度上的N个样本，蛮力方法的缩放比例为***0 [DN 2 ]\***

对于小数据样本，此算法可能非常有用，但是随着样本数量的增加，它变得不可行。可以通过编写关键字**algorithm ='brute'**来启用蛮力邻居搜索。

### KD树

为了解决暴力破解方法的计算效率低下而发明的基于树的数据结构之一就是KD树数据结构。基本上，KD树是一种二叉树结构，称为K维树。它通过将参数空间划分为嵌套的正交区域来填充数据点，从而沿数据轴递归划分参数空间。

### 优点

以下是KD树算法的一些优点-

**构造速度快** -由于仅沿数据轴执行分区，因此KD树的构造速度非常快。

**更少的距离计算** -该算法只需很少的距离计算即可确定查询点的最近邻居。它只需要**𝑶[𝐥𝐨𝐠（𝑵）]个**距离计算。

### 缺点

**仅对低维邻居搜索**快速-对低维（D <20）邻居搜索非常快，但是随着D的增长，它变得无效。由于仅沿数据轴执行分区，

可以通过编写关键字**algorithm ='kd_tree'**来启用KD树邻居搜索。

### 球树

众所周知，KD树在高维方面效率低下，因此，为了解决KD树的这种低效率问题，开发了球树数据结构。在数学上，它将数据递归地分为质心C和半径r定义的节点，以使节点中的每个点都位于质心**C**和半径**r**定义的超球面内。它使用下面给出的三角形不等式，从而减少了邻居搜索的候选点数

$$
\arrowvert X+Y\arrowvert\leq \arrowvert X\arrowvert+\arrowvert Y\arrowvert
$$

### 优点

以下是Ball Tree算法的一些优点-

**高效处理高度结构化的数据** -由于球形树将数据划分为一系列嵌套的超球体，因此对高效处理高度结构化的数据非常有效。

**表现优于KD树** -球树在高维方面表现优于KD树，因为它具有球树节点的球形几何形状。

### 缺点

成本**高昂** -将数据划分为一系列嵌套的超球体，使其构建成本很高。

可以通过编写关键字**algorithm ='ball_tree'**来启用球树邻居搜索。

## 选择最近邻居算法

给定数据集的最佳算法的选择取决于以下因素-

### 样本数（N）和维数（D）

这些是选择最近邻居算法时要考虑的最重要因素。这是由于以下原因-

- 蛮力算法的查询时间随着O [DN]的增长而增加。
- 球树算法的查询时间随着O [D log（N）]而增长。
- KD树算法的查询时间随D的变化而变化，这很难描述。当D <20时，成本为O [D log（N）]，该算法非常有效。另一方面，在D> 20的情况下效率低下，因为成本增加到接近O [DN]。

### 数据结构

影响这些算法性能的另一个因素是数据的固有维数或数据的稀疏性。这是因为球树和KD树算法的查询时间会受到很大的影响。而蛮力算法的查询时间在数据结构上是不变的。通常，当植入具有较小固有维数的稀疏数据时，球树和KD树算法会产生更快的查询时间。

### 邻居数（k）

请求一个查询点的邻居数（k）影响球树和KD树算法的查询时间。随着邻居数（k）的增加，查询时间变慢。而蛮力的查询时间将不受k值的影响。

### 查询点数

因为它们需要构造阶段，所以如果存在大量查询点，则KD树算法和Ball树算法都将有效。另一方面，如果查询点数量较少，则蛮力算法的性能要优于KD树和Ball树算法。



## KNN实现

k-NN（k最近邻）是最简单的机器学习算法之一，本质上是非参数的和惰性的。非参数意味着没有基础数据分布的假设，即从数据集中确定了模型结构。懒惰或基于实例的学习意味着，出于模型生成的目的，它不需要任何训练数据点，并且在测试阶段将使用整个训练数据。

k-NN算法包括以下两个步骤-

### 步骤1

在此步骤中，它计算并存储训练集中每个样本的k个最近邻居。

### 第2步

在此步骤中，对于未标记的样本，它将从数据集中检索k个最近的邻居。然后，在这些k近邻中，它通过投票来预测班级（多数票的班级获胜）。

实现k近邻算法的模块**sklearn.neighbors**提供了**无监督**以及基于**监督**的基于邻居的学习方法的功能。

无监督的最近邻居实施不同的算法（BallTree，KDTree或蛮力）以找到每个样本的最近邻居。此无监督版本基本上只是上面讨论的第一步，并且是需要邻居搜索的许多算法（KNN和K-means是著名的算法）的基础。简而言之，它是用于实施邻居搜索的无监督学习者。

另一方面，基于监督的基于邻居的学习可用于分类和回归。

## 无监督的KNN学习

如讨论的那样，存在许多算法，例如KNN和K-Means，它们需要最近邻居搜索。这就是Scikit-learn决定将邻居搜索部分实现为自己的“学习者”的原因。进行邻居搜索作为单独的学习者的原因是，计算所有成对距离来查找最近的邻居显然不是很有效。让我们看一下Sklearn用于实现无监督的最近邻居学习的模块以及示例。

### Scikit学习模块

**sklearn.neighbors.NearestNeighbors**是用于实施无监督的最近邻居学习的模块。它使用名为BallTree，KDTree或蛮力的特定最近邻居算法。换句话说，它充当这三种算法的统一接口。

**实施实例**

下面的示例将使用**sklearn.neighbors.NearestNeighbors**模块在两组数据之间找到最接近的邻居。

首先，我们需要导入所需的模块和软件包-

```python
from sklearn.neighbors import NearestNeighbors
import numpy as np
```

现在，在导入包之后，在我们要查找最近的邻居之间定义数据集-

```python
Input_data = np.array([[-1, 1], [-2, 2], [-3, 3], [1, 2], [2, 3], [3, 4],[4, 5]])
```

接下来，应用无监督学习算法，如下所示：

```python
nrst_neigh = NearestNeighbors(n_neighbors = 3, algorithm = 'ball_tree')
```

接下来，使用输入数据集拟合模型。

```python
nrst_neigh.fit(Input_data)
```

现在，找到数据集的K邻居。它将返回每个点的邻居的索引和距离。

```python
distances, indices = nrst_neigh.kneighbors(Input_data)
indices
```

上面的输出显示每个点的最近邻居是该点本身，即零。这是因为查询集与训练集匹配。

我们还可以通过生成如下的稀疏图来显示相邻点之间的连接-

```python
nrst_neigh.kneighbors_graph(Input_data).toarray()
```

一旦我们拟合了无监督的**NearestNeighbors**模型，数据将基于为参数**'algorithm'**设置的值存储在数据结构中。在此之后，我们可以利用这个无人监督学习者的**kneighbors**在需要邻居的搜索的模式。

**完整的工作/可执行程序**

```python
from sklearn.neighbors import NearestNeighbors
import numpy as np
Input_data = np.array([[-1, 1], [-2, 2], [-3, 3], [1, 2], [2, 3], [3, 4],[4, 5]])
nrst_neigh = NearestNeighbors(n_neighbors = 3, algorithm='ball_tree')
nrst_neigh.fit(Input_data)
distances, indices = nrst_neigh.kneighbors(Input_data)
indices
distances
nrst_neigh.kneighbors_graph(Input_data).toarray()
```

## 监督KNN学习

基于监督的基于邻居的学习用于-

- 分类，用于带有离散标签的数据
- 回归，用于带有连续标签的数据。

### 最近邻分类器

我们可以借助以下两个特征来了解基于邻居的分类-

- 它是根据每个点的最近邻居的简单多数票计算得出的。
- 它只是存储培训数据的实例，这就是为什么它是一种非一般性学习的原因。

### Sklearn模块

以下是scikit-learn使用的两种不同类型的最近邻居分类器-

| 序号 | 分类器和说明                                                 |
| ---- | ------------------------------------------------------------ |
| 1。  | KNeighborsClassifier该分类器名称中的K代表k个最近邻居，其中k是用户指定的整数值。因此，顾名思义，该分类器基于k个最近的邻居实现学习。k值的选择取决于数据。 |
| 2。  | RadiusNeighborsClassifier此分类器名称中的半径表示指定半径r内的最近邻居，其中r是用户指定的浮点值。因此，顾名思义，该分类器基于每个训练点的固定半径r内的邻居数实现学习。 |

## 最近邻居回归

它在数据标签本质上是连续的情况下使用。分配的数据标签是基于其最近邻居标签的平均值计算的。

以下是scikit-learn使用的两种不同类型的最近邻居回归器-

## KNeighborsRegressor

此回归器名称中的K表示k个最近的邻居，其中**k**是用户指定的**整数值**。因此，顾名思义，该回归器基于最近的k个邻居实现学习。k值的选择取决于数据。让我们借助一个实现示例来进一步了解它。

以下是scikit-learn使用的两种不同类型的最近邻居回归器-

### 实施实例

在此示例中，我们将使用scikit-learn **KNeighborsRegressor**在名为Iris Flower数据集的数据集上实现KNN 。

首先，按以下方式导入IRIS数据集-

```python
from sklearn.datasets import load_iris
iris = load_iris()
```

现在，我们需要将数据分为训练和测试数据。我们将使用Sklearn **train_test_split**函数将数据分成70（训练数据）和20（测试数据）的比率-

```python
X = iris.data[:, :4]
y = iris.target
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.20)
```

接下来，我们将在Sklearn预处理模块的帮助下进行数据缩放-

```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
scaler.fit(X_train)
X_train = scaler.transform(X_train)
X_test = scaler.transform(X_test)
```

接下来，从Sklearn 导入**KNeighborsRegressor**类，并按如下所示提供邻居的值。

```python
import numpy as np
from sklearn.neighbors import KNeighborsRegressor
knnr = KNeighborsRegressor(n_neighbors = 8)
knnr.fit(X_train, y_train)
```

现在，我们可以找到MSE（均方误差），如下所示：

```python
print("The MSE is:",format(np.power(y-knnr.predict(X),4).mean()))
```

现在，使用它来预测值，如下所示：

```python
X = [[0], [1], [2], [3]]
y = [0, 0, 1, 1]
from sklearn.neighbors import KNeighborsRegressor
knnr = KNeighborsRegressor(n_neighbors = 3)
knnr.fit(X, y)
print(knnr.predict([[2.5]]))
```


### 完整示例

```python
from sklearn.datasets import load_iris
iris = load_iris()
X = iris.data[:, :4]
y = iris.target
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.20)
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()

scaler.fit(X_train)
X_train = scaler.transform(X_train)
X_test = scaler.transform(X_test)

import numpy as np
from sklearn.neighbors import KNeighborsRegressor
knnr = KNeighborsRegressor(n_neighbors=8)
knnr.fit(X_train, y_train)

print("The MSE is:",format(np.power(y-knnr.predict(X),4).mean()))

X = [[0], [1], [2], [3]]
y = [0, 0, 1, 1]
from sklearn.neighbors import KNeighborsRegressor
knnr = KNeighborsRegressor(n_neighbors=3)
knnr.fit(X, y)
print(knnr.predict([[2.5]]))
```

## RadiusNeighborsRegressor

此回归器名称中的半径表示指定半径r内最近的邻居，其中r是用户指定的浮点值。因此，顾名思义，该回归器基于每个训练点的固定半径r内的邻居数实现学习。让我们在一个实现示例的帮助下更加了解它-

### 实施实例

在这个例子中，我们将会对数据实施KNN集使用scikit学习命名的鸢尾花数据集**RadiusNeighborsRegressor** -

首先，按以下方式导入IRIS数据集-

```python
from sklearn.datasets import load_iris
iris = load_iris()
```

现在，我们需要将数据分为训练和测试数据。我们将使用Sklearn train_test_split函数将数据分成70（训练数据）和20（测试数据）的比率-

```python
X = iris.data[:, :4]
y = iris.target
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.20)
```

接下来，我们将在Sklearn预处理模块的帮助下进行数据缩放-

```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
scaler.fit(X_train)
X_train = scaler.transform(X_train)
X_test = scaler.transform(X_test)
```

接下来，从Sklearn 导入**RadiusneighborsRegressor**类，并提供radius的值，如下所示：

```python
import numpy as np
from sklearn.neighbors import RadiusNeighborsRegressor
knnr_r = RadiusNeighborsRegressor(radius=1)
knnr_r.fit(X_train, y_train)
```

现在，我们可以找到MSE（均方误差），如下所示：

```python
print("The MSE is:",format(np.power(y-knnr_r.predict(X),4).mean()))
```

现在，使用它来预测值，如下所示：

```python
X = [[0], [1], [2], [3]]
y = [0, 0, 1, 1]
from sklearn.neighbors import RadiusNeighborsRegressor
knnr_r = RadiusNeighborsRegressor(radius=1)
knnr_r.fit(X, y)
print(knnr_r.predict([[2.5]]))
```


### 完整的工作/可执行程序

```python
from sklearn.datasets import load_iris

iris = load_iris()

X = iris.data[:, :4]
y = iris.target
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.20)
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
scaler.fit(X_train)
X_train = scaler.transform(X_train)
X_test = scaler.transform(X_test)
import numpy as np
from sklearn.neighbors import RadiusNeighborsRegressor
knnr_r = RadiusNeighborsRegressor(radius = 1)
knnr_r.fit(X_train, y_train)
print("The MSE is:",format(np.power(y-knnr_r.predict(X),4).mean()))
X = [[0], [1], [2], [3]]
y = [0, 0, 1, 1]
from sklearn.neighbors import RadiusNeighborsRegressor
knnr_r = RadiusNeighborsRegressor(radius = 1)
knnr_r.fit(X, y)
print(knnr_r.predict([[2.5]]))
```