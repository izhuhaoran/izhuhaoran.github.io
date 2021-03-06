---
title: 支持向量机SVM编程实战
date: 2021-05-08 19:51:53
tags:
  - 机器学习
  - 学习笔记
toc: true
declare: true
---

# TASK1:**分别用线性SVM和高斯核SVM预测对数据进行分类**

## 一、软件设计

### 1. 设计目标

#### ① 目标概述

**(1) 问题概述：**

task1_linear.mat中有一批数据点，试用线性SVM对他们进行分类，并在图中画分出决策边界。task1_gaussian中也有一批数据点，试用高斯核SVM对他们进行分类，并在图中画出决策边界。

<!--more-->

**(2) 步骤提示：**

1.实验所需函数的代码在SVM_Functions.py中已经给出，请尝试读懂代码，理解算法思想与步骤，并自行读入数据，执行所需函数，观察实验结果。（也可以自己编写实验代码）

2..task1_linear.mat和task1_gaussian.mat图像如下所示：

<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/%E6%95%B0%E6%8D%AE%E5%9B%BE1.png?raw=true" alt="数据图1" style="zoom: 67%;" />

4.训练过程：

（a）加载与可视化数据

```python
loadData()，plotData()
```

（b）训练模型

```python
 model = SVM_Functions.svmTrain_SMO(X, y, C=1, max_iter=20)
 model=SVM_Functions.svmTrain_SMO(X,y,C=1,kernelFunction='gaussian',K_matrix=s.gaussianKernel(X,sigma=0.1))
```

（c）决策边界可视化

```python
 SVM_Functions.visualizeBoundaryLinear(X, y, model)
```

#### ② 目标要求

任务一需要编写实验报告进行简述其原理，代码步骤，并根据训练结果在图中画出决策边界。



### 2. 算法设计

> 所用项目环境	IDE：VS code		Python版本：python3.7.5

#### ① SVM支持向量机

给定数据样本集
$D=\left\{ \left( x_1,y_1 \right) ,\left( x_2,y_2 \right) ,...,\left( x_m,y_m \right) \right\} ,y_i\in \left\{ -1,+1 \right\} $

- **基本概念**：

  划分超平面的**线性方程**：$w^Tx+b=0,法向量w=(w_1;w_2;...;w_d)，位移项b$

  **间隔**$\gamma =\frac{2}{||w||}$,欲找到具有最大间隔的超平面，问题转化为下列问题，也即SVM基本型：
$$
\underset{w,b}{\min}\ \frac{1}{2}||w||^2
$$

$$
s.t.\ y_i\left( w^Tx_i+b \right) \ge 1,i=1,2,...,m
$$

&emsp;&emsp;使用拉格朗日乘子法可得到 “**对偶问题**”：

$$
L\left( w,b,\alpha \right) =\frac{1}{2}||w||^2+\sum_{i=1}^m{\alpha _i\left( 1-y_i\left( w^Tx_i+b \right) \right)}
$$
&emsp;&emsp;可转化为：
$$
\underset{\alpha}{\max}\ \sum_{i=1}^m{\alpha _i-\frac{1}{2}\sum_{i=1}^m{\sum_{j=1}^m{\alpha _i\alpha _jy_iy_jx_{i}^{T}x_j}}}
$$
$$
s.t.\ \sum_{i=1}^m{\alpha _iy_i=0，\alpha _i\ge 0,i=1,2,...,m}
$$

&emsp;&emsp;上述过程需要满足**KKT条件**：
$$
\left\{ \begin{array}{l}
  	\alpha _i\ge 0;\\
  	y_if\left( x_i \right) -1\ge 0;\\
  	\alpha _i\left( y_if\left( x_i \right) -1 \right) =0;\\
  \end{array} \right.
$$

&emsp;&emsp;此时<b>划分方程f(x)</b>转化为:
$$
f\left( x \right) =\sum_{i=1}^m{\alpha _iy_iK\left( x_i*x_j \right)}+b
$$



-----------------

> 1. 需要特别说明的是，以上的推导都是建立在“硬间隔”的基础上，“硬间隔”要求样本集中每一个样本都满足约束条件。
>
> 2. 在现实中往往很难确定合适的核函数使得训练样本在特征空间中是线性可分的，缓解该问题的一个办法是允许支持向量机在一些样本上出错，为此引入“软间隔”的概念。
>
> 3. 具体来说，“硬间隔”要求所有参与训练的样本都必须满足SVM的约束条件，而“软间隔”允许有部分样本不满足这样的约束。

-------------------------------------------------



- **软间隔支持向量机**

  **对偶问题**转化为：
$$
\underset{\alpha}{\max}\ \sum_{i=1}^m{\alpha _i-\frac{1}{2}\sum_{i=1}^m{\sum_{j=1}^m{\alpha _i\alpha _jy_iy_jx_{i}^{T}x_j}}}
$$

$$
s.t.\ \sum_{i=1}^m{\alpha _iy_i=0，\alpha _i\ge 0,i=1,2,...,m}
$$

&emsp;&emsp;**KKT条件**转化为：
$$
\left\{ \begin{array}{l}
	\alpha _i\ge 0;\mu _i\ge 0;\\
	y_if\left( x_i \right) -1+\xi _i\ge 0;\\
	\alpha _i\left( y_if\left( x_i \right) -1+\xi _i \right) =0;\\
	\xi _i\ge 0;\mu _i\xi _i=0;\\
\end{array} \right. 
$$

- **核函数**

  令X为输入空间，k(·,·)是定义在X*X上的对称函数，则k是核函数当且仅当对于任意数据D，“核矩阵”K总是半正定的：
  
  <img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/K%E7%9F%A9%E9%98%B5.png?raw=true" alt="K矩阵" style="zoom: 67%;" />
  
  常用核函数：
  
  <img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/%E6%A0%B8%E5%87%BD%E6%95%B0.png?raw=true" alt="核函数" style="zoom: 80%;" />

#### ② SMO算法

SMO的基本思路是先固定$\alpha_i$之外的所有参数,然后求$\alpha_i$上的极值.由于存在约束$\sum_{i=1}^m{\alpha _iy_i=0}$,若固定αi之外的其他变量,.则αi可由其他变量导出.于是,SMO每次选择两个变量ai和aj,并固定其他参数.这样,在参数初始化后,SMO不断执行如下两个步骤直至收敛:

- 选取一对需更新的变量$\alpha_i和\alpha_j;$
- 固定$\alpha_i和\alpha_j$以外的参数，求解转化的对偶问题公式获得更新后的$\alpha_i和\alpha_j;$

由约束$\sum_{i=1}^m{\alpha _iy_i=0}$,可以得到$\alpha _1y_1+\alpha _2y_2=C\rightarrow \alpha _1=\left( C-\alpha _2y_2 \right) y_1$

> 具体对偶问题求解过程省略，主要为求偏导，下面简单介绍求解结果
> <span id = "jump1"> </span> 	[跳转回SMO算法代码实现](#jump2)	[跳转回TASK2](#jump3)	[跳转回TASK3](#jump4)

**（1）决策误差**

已知$f\left( x \right) =\sum_{i=1}^m{\alpha _iy_iK\left( x_i*x_j \right)}+b$,得出误差Ei为：
$$
E_i=f(x_i)-y_i,i=1,2
$$
当i=1,2时，Ei表示函数f(x)对输入xi的预测值与真实值yi之间的差值

**（2）$\alpha$修改时的上下界L、H**

- $y_1\ne y_2$时：

$$
L=\max \left( 0,\alpha _{2}^{old}-\alpha _{1}^{old} \right) ;\ \ H=\min \left( C,C+\alpha _{2}^{old}-\alpha _{1}^{old} \right)
$$

<div align="center"><img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/%E5%8E%9F%E7%90%861.png?raw=true" alt="原理1" style="zoom: 67%" /></div>

- $y_1=y_2$时：

$$
L=\max \left( 0,\alpha _{2}^{old}-\alpha _{1}^{old}-C \right) ;\ \ H=\min \left( C,\alpha _{2}^{old}-\alpha _{1}^{old} \right)
$$

<img src="F:\own_file\课程文件\机器学习导论\支持向量机\报告\img\原理1.png" alt="原理1" style="zoom: 67%;" />



**（3）α更新公式**

- 未剪辑前：

$$
\alpha _{2}^{new,nuc}=\alpha _{2}^{old}+\frac{y_2\left( E_1-E_2 \right)}{\eta},\text{其中}\eta =K_{11}+K_{22}-2K_{12}
$$

- 剪辑后：

$$
\alpha _{2}^{new}=\left\{ \begin{array}{l}
	H,\alpha _{2}^{new,unc}>H\\
	\alpha _{2}^{new,unc},L\le \alpha _{2}^{new,unc}\le H\\
	L,\alpha _{2}^{new,unc}<L\\
\end{array} \right.
$$

由$\alpha_2^{new}$求得$\alpha_1^{new}$为：
$$
\alpha_1^{new}=\alpha_1^{old}+y_1y_2(\alpha_2^{old}-\alpha_2^{new})
$$
**（4）阈值/偏差 b更新公式**
$$
b_1^{new}=b^{old}-E_1-y_1K_{11}(\alpha_1^{new}-\alpha_1^{old})-y_2K_{21}(\alpha_2^{new}-\alpha_2^{old})
$$
$$
b_2^{new}=b^{old}-E_1-y_1K_{12}(\alpha_1^{new}-\alpha_1^{old})-y_2K_{22}(\alpha_2^{new}-\alpha_2^{old})
$$

$$
b^{new}=\left\{ \begin{array}{l}
	b_{1}^{new},\ 0\le \alpha _{1}^{new}\le C\\
	b_{2}^{new},\ 0\le \alpha _{2}^{new}\le C\\
	\frac{b_1+b_2}{2},\ otherwise\\
\end{array} \right.
$$

**（5）w*计算公式**
$$
w^*=\sum_{i=1}^{m}\alpha_i^*y_ix_i
$$

#### ③ 代码实现

> **具体代码已经已经给出.py文件，只需写好调用函数即可，下面粗略分析给出的SMO算法，部分代码省略**

##### (1)数据预处理

**算法介绍**：对提供的训练文件进行读取为矩阵，并将标签提取出来

> **数据整理**：对于缺失值的情况，作业中的数据已经将缺失值全部补0了，根据设计目标中的数据集讲解可得，设为0即可不影响后续。


- **数据读取函数：**

```python
"""
输入:
  数据集路径
输出:
  numpy.array格式的X, y数据array
  X为m×n的数据array, m为样例数, n为特征维度
  y为m×1的标签array, 1表示正例, 0表示反例
"""
def loadData(filename):
    dataDict = loadmat(filename)
    return dataDict['X'], dataDict['y']
```

- - **代码细节**： 

1. from scipy.io import loadmat,使用loadmat读取.mat文件

2. loadmat返回值是一个字典形式，使用dataDict['X'], dataDict['y']返回特征和标签
<br>
- **数据可视化函数：**

```python
def plotData(X, y, title=None):
    plt.rcParams['font.sans-serif'] = ['KaiTi']     # 指定默认字体
    plt.rcParams['axes.unicode_minus'] = False      # 解决保存图像是负号'-'显示为方块的问题
    ·······
    plt.show()
```

- - **代码细节：**

1. import matplotlib.pyplot as plt，使用pyplot库函数将读取的数据绘制即可

2. plt.rcParams['font.sans-serif'] = ['KaiTi']  是为了解决图像无法正常显示中文，指定默认字体为楷体
plt.rcParams['axes.unicode_minus'] = False  是为了解决保存图像是负号'-'显示为方块的问题



##### (2)SMO算法实现SVM

> 具体代码解析，参考作业给出的ppt文件，下列只叙述自己认为的细节之处

**算法介绍**:使用SMO算法实现SVM，对处理好的数据，计算出超平面相关参数


- **sigmoid函数**<span id = "jump2"> </span>

```python
"""
利用简化版的SMO算法训练SVM
输入：
X, y为loadData函数的返回值   C为惩罚系数
kernelFunction表示核函数类型, 对于非线性核函数，也可直接输入核函数矩阵K
tol为容错率  max_iter为最大迭代次数
输出：
model['kernelFunction']为核函数类型  model['X']为支持向量
model['y']为对应的标签  model['alpha']为对应的拉格朗日参数
model['w'], model['b']为模型参数
"""
def svmTrain_SMO(X, y, C, kernelFunction='linear', tol=1e-3, max_iter=5, **kargs):
    start = time.clock()

   ···········

    idx = np.where(alphas > 0)
    model = {'X': X[idx[0], :], 'y': y[idx], 'kernelFunction': str(kernelFunction),
             'b': b, 'alphas': alphas[idx], 'w': (np.multiply(alphas, y). T * X).T}
    return model
```

- - **代码细节：**

1. 代码实现下列的伪代码，具体公式参考①②中叙述的算法原理公式，这里不再叙述，
[跳转回SMO算法原理](#jump1)
主要是 对偶问题的求解，并根据KKT条件更新参数
<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/%E5%8E%9F%E7%90%863.png?raw=true" alt="原理3" style="zoom: 40%;" />

2. 使用time计时程序运行时间，同时使用print打印“···”进度条，避免程序运行过长，终端难以看出运行进度。

3. while iters < max_iter: 使用while循环进行不断迭代更新参数，同时对于不满足KKT条件等参数不需要更新的循环直接使用continue结束内循环。



- **线性SVM边界可视化函数**：

```python
def visualizeBoundaryLinear(X, y, model, title=None):
    plt.rcParams['font.sans-serif'] = ['KaiTi']     # 指定默认字体
    plt.rcParams['axes.unicode_minus'] = False      # 解决保存图像是负号'-'显示为方块的问题
    ······

    plt.show()
```

- - **代码细节**： 

1. import matplotlib.pyplot as plt，使用pyplot库函数将读取的数据绘制即可

2. 根据$y=w^Tx+b$  计算边界代码如下：
&emsp;xp = np.linspace(min(X[:, 0]), max(X[:, 0]), 100)
&emsp;yp = np.squeeze(np.array(-(w[0] * xp + b) / w[1]))  
<br>
- **高斯核计算函数：**

  > 对于非线性高斯核函数的SVM，编写高斯核，同时计算训练集在高斯核函数下的矩阵值

```python
def gaussianKernelSub(x1, x2, sigma):
    x1 = np.mat(x1).reshape(-1, 1)
    x2 = np.mat(x2).reshape(-1, 1)
    n = -(x1 - x2).T * (x1 - x2) / (2 * sigma**2)
    return np.exp(n)

def gaussianKernel(X, sigma):
    start = time.clock()
    ······
    for i in range(m):
        if dots % 10 == 0:
            print('.', end='')
        dots = dots + 1
        if dots > 780:
            dots = 0
            print()
        for j in range(m):
            K[i, j] = gaussianKernelSub(X[i, :].T, X[j, :].T, sigma)
            K[j, i] = K[i, j].copy()
	·····
    return K
```

- - **代码细节**： 

1. 高斯核直接用代码实现公式即可n = -(x1 - x2).T * (x1 - x2) / (2 * sigma**2)

2. 计算高斯核的矩阵使用公式K[i, j] = gaussianKernelSub(X[i, :].T, X[j, :].T, sigma)
    调用高斯核数学公式计算对应位置的值即可

<br>

- **结果预测分类器函数：**

```python
"""
利用得到的model, 计算给定X的模型预测值
输入：
model为svmTrain_SMO返回值
X为待预测数据
sigma为训练参数
"""
def svmPredict(model, X, *arg):
    ······
    if model['kernelFunction'] == 'linear':
        p = X * model['w'] + model['b']
    else:
        for i in range(m):
            prediction = 0
            for j in range(model['X'].shape[0]):
                prediction += model['alphas'][:, j] * model['y'][:, j] * \
                    gaussianKernelSub(X[i, :].T, model['X'][j, :].T, *arg)
            p[i] = prediction + model['b']
    pred[np.where(p >= 0)] = 1
    pred[np.where(p < 0)] = 0
    return pred
```

- - **代码细节**： 

1. 根据传入的SVM模型的核函数类型进行不同的计算(线性或者高斯型)
线性$y=w^Tx+b$ ，高斯核$f\left( x \right) =\sum_{i=1}^m{\alpha _iy_iK\left( x_i*x_j \right)}+b$
代码如下：
```python
线性：p = X * model['w'] + model['b']
高斯：prediction += model['alphas'][:, j] * model['y'][:, j] * \
                          gaussianKernelSub(X[i, :].T, model['X'][j, :].T, *arg)
```
2. pred[np.where(p >= 0)] = 1大于零的位置为正例设为1；	pred[np.where(p < 0)] = 0,小于零的位置为反例设为0
<br>
- **高斯SVM分类边界可视化函数：**

```python
def visualizeBoundaryGaussian(X, y, model, sigma, title=None):

	·······

    for i in range(X1.shape[1]):
        print('.', end='')
        dots += 1
        if dots == 78:
            dots = 0
            print()
        this_X = np.concatenate((X1[:, i], X2[:, i]), axis=1)
        vals[:, i] = svmPredict(model, this_X, sigma)
    print('Done')

    ·········

    plt.show()

```

- - **代码细节**： 

  (1) 调用分类器函数将分类边界计算出来： vals[:, i] = svmPredict(model, this_X, sigma)
  
  (2)  import matplotlib.pyplot as plt，使用pyplot库函数将读取的数据绘制即可

<br>

##### (3)main函数

**算法介绍：**调用上述编写的算法函数

```python
import SVM_Functions as s

if __name__ == "__main__":
    X_lin, y_lin = s.loadData("task1/task1_linear.mat")
    s.plotData(X_lin, y_lin, title="linear图像")
    model1 = s.svmTrain_SMO(X_lin, y_lin, C=1, max_iter=20)
    s.visualizeBoundaryLinear(X_lin, y_lin, model1, title='linear_SVM边界图')

    X_gauss, y_gauss = s.loadData("task1/task1_gaussian.mat")
    s.plotData(X_gauss, y_gauss, title="gaussian图像")
    model2 = s.svmTrain_SMO(X_gauss, y_gauss, C=1, kernelFunction='gaussian', K_matrix=s.gaussianKernel(X_gauss, sigma=0.1))
    s.visualizeBoundaryGaussian(X_gauss, y_gauss, model2, sigma=0.1, title='gaussian_SVM边界图')
```

- **代码细节**： 

  (1)  调用给出的SVM_Functions.py中的函数，注意函数参数的对应，线性和高斯核对应参数不一致




## 二、运行结果

#### 截图一_运行结果，终端显示

<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/task1/5.png?raw=true" alt="5" style="zoom: 50%;" />

#### 截图一_运行结果，线性核

<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/task1/1.png?raw=true" alt="1" style="zoom: 35%;" /><img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/task1/2.png?raw=true" alt="2" style="zoom: 35%;" />

- 图片分析：上图为线性核情况下原始数据图以及边界可视化图，由上图显示，边界正确

#### 截图二_运行结果，高斯核

<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/task1/3.png?raw=true" alt="3" style="zoom:35%;" /><img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/task1/4.png?raw=true" alt="4" style="zoom:35%;" />

- 图片分析：上图为高斯核情况下原始数据图以及边界可视化图，由上图显示，边界正确



## 三、总结体会

####  程序设计心得:

1. 本次算法过程中，遇到程序错误在正常不过，知错能改，善莫大焉，一个完整的工程背后一定是一次次的debug，自身犯错最多的地方在于差错处理，可以多考虑极端情况，多进行程序的调试，对每一种特殊情况进行处理，避免程序运行的自行结束。通过这次软件课设，自身养成了不惧困难，不退缩，提高思维能力

2. 软件工程量较大，建议采用模块化设计，同时不同的内容可以分为不同的文件，优先设计函数头，确定参数及返回值后具体设计逻辑

3. SVM算法的构建，重点在于对于数学公式的理解，遇到难以解决的问题可以先单步调试，找到问题一步步修改



# TASK2: 使用高斯核SVM对给定数据集进行分类

## 一、软件设计

### 1. 设计目标

#### ① 数据集讲解：

给定数据集（文件task2.mat）, 参考task1的代码, 编程实现一个高斯核SVM进行分类。
输出训练参数C, sigma分别取0.01, 0.03, 0.1, 0.3, 1, 3, 10, 30时(共64组参数组合)的训练集上的准确率。
（程序运行时间8mins左右，准确率 = 预测正确样本数/样本总数 ）

#### ② 设计提示思路

**1.** **数据可视化：**
<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/task2/1.png?raw=true" style="zoom: 67%;" />

**2.** **程序运行结果举例**：由于SMO算法的随机性，你的结果应该跟下面的例子不完全相同：
<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/task2/2.png?raw=true" alt="2" style="zoom: 80%;" />

#### ③ **目标要求**

- 编写实验报告，报告中需要包含：实验设置，编程思路，实验结果展示以及自己的思考等。
- 其中实验结果展示按照上图的格式在实验报告中给出。



### 2. 算法设计分析

> 所用项目环境	IDE：VS code		Python版本：python3.7.5		

#### ①SVM/SMO算法原理

这部分的原理及代码实现在task1中已经详细描述过，这里不再重复叙述
<span id = "jump3"> </span> 	[跳转回SMO算法代码实现](#jump1)

#### ②代码实现

> SVM的详细实现代码跳转到task1阅读即可，下列是基于task1中的SMO算法使用

##### （1）子模块

- **SVM模型精度计算算法**:对于参数c与sigma不同的组合，得出不同的SVM模型model并计算精度

```python
def predict_rate(sigmaList, CList):
    # error_num = 0
    index = 0
    X_gauss, y_gauss = s.loadData("task2/task2.mat")
    rate_list = [[] for i in range(len(CList))]
    # s.plotData(X_gauss, y_gauss, title="gaussian图像")
    for c in CList:
        for Sigma in sigmaList:
            model = s.svmTrain_SMO(X_gauss, y_gauss, C=c, kernelFunction='gaussian', K_matrix=s.gaussianKernel(X_gauss, sigma=Sigma))
            pre = s.svmPredict(model, X_gauss, Sigma)
            rate = 1 - (np.sum(abs(np.array(pre) - np.array(y_gauss))) / float(len(y_gauss)))
            rate_list[index].append(('%.3f' % rate))
        # print(rate_list[index])
        index += 1
    return rate_list
```

- - **代码细节**：

1. 使用两层for循环，对于参数c与sigma不同的组合，调用SVM算法

2. 标签只有1,0两种，计算精度使用下列代码：
    rate = 1 - (np.sum(abs(np.array(pre) - np.array(y_gauss))) / float(len(y_gauss)))
    计算预测值与真实值两个数组之间的差的绝对值并求和得出，预测值与真实值不同的个数，
    求和个数与总标签数相除得到错误率，1-错误率=精度
    <br>

- **列表打印算法**:将传入的列表打印成任务要求中的对齐格式

```python
def List_print(List):
    for i in range(len(List)):
        if i == (len(List) - 1):
            print(List[i])
        else:
            print(List[i], end='\t')
```

- - **代码细节**： 代码简单，使用for循环，每一个print使用参数end=‘\t’,当打印列表最后一个元素的时候默认以换行结束



##### （2）main函数

```python
if __name__ == "__main__":

    sigmaList = [0.01, 0.03, 0.1, 0.3, 1, 3, 10, 30]
    CList = [0.01, 0.03, 0.1, 0.3, 1, 3, 10, 30]
    rate_list = predict_rate(sigmaList, CList)
    print('Training Accuracy:\n' + 'sigma', end='\t')
    List_print(sigmaList)
    print('c')
    for i in range(len(rate_list)):
        print(CList[i], end='\t')
        List_print(rate_list[i])
```

- - **代码细节**： 

1. 调用之前设计的数个函数即可
  
2. 开始打印print('Training Accuracy:\n' + 'sigma', end='\t')；  List_print(sigmaList)打印最开始的提示信息；对于返回的二维精度列表，循环打印每一行。

## 二、运行结果

- **采用Google colab运行代码**

<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/task2/4.png?raw=true" alt="4" style="zoom:60%;" />



- **自己电脑运行代码：**

<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/task2/5.png?raw=true" alt="5" style="zoom:60%;" />




- - 实验结果分析：

  **由上面两个运行结果截图，与任务要求中的示例(下)对比知：程序设计成功，输出结果大致是相同的**
  <img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/task2/2.png?raw=true" alt="2" style="zoom:75%;" />

  **具体分析知**：不同的sigma和c，精度结果会出现较大差异，下面四行的精度明显比上四行好很多；
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;对于参数c，随着图中8个c的不同取值增大，精度也会越来越好；
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;对于参数sigma，随着值的增大，大概呈一种先上升后下降的趋势。

## 三、总结体会

####  程序设计心得:

1. 本次算法过程中，遇到程序错误在正常不过，知错能改，善莫大焉，一个完整的工程背后一定是一次次的debug，自身犯错最多的地方在于差错处理，可以多考虑极端情况，多进行程序的调试，对每一种特殊情况进行处理，避免程序运行的自行结束。通过这次软件课设，自身养成了不惧困难，不退缩，提高思维能力
2. 本次代码编写过程中，由于task1中已经实现SMO算法，故重点在于对于该算法函数的调用，注意参数的设置，编写对应的格式化输出函数即可

# TASK3: 使用线性SVM实现对垃圾邮件分类

## 一、软件设计

### 1. 设计目标

#### ① 数据集讲解：

- 编程实现一个垃圾邮件SVM线性分类器，分别在训练集和测试集上计算准确率。
- 其中训练数据文件：task3_train.mat，要求导入数据时输出样本数和特征维度。
- 测试数据文件：task3_test.mat，要求导入数据时输出样本数和特征维度，测试数据标签未给出。（程序运行时间10mins左右）

#### ② 设计提示思路

**1.** **对SMO算法的实现进行举例说明:**
<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/%E5%8E%9F%E7%90%863.png?raw=true" alt="原理3" style="zoom: 50%;" />

#### ③ **目标要求**

- 编写实验报告，报告中需要包含：实验设置，编程思路等
- 将测试数据预测结果按顺序存储为txt文件，每一行为一个样本的标签，上传到系统对应作业栏。



### 2. 算法设计分析

> **所用项目环境	IDE：VS code		Python版本：python3.7.5		** 

#### ①SVM/SMO算法原理

这部分的原理及代码实现在task1中已经详细描述过，这里不再重复叙述
<span id = "jump4"> </span> 	[跳转回SMO算法代码实现](#jump1)

#### ②代码实现

> SVM的详细实现代码跳转到task1阅读即可，下列是基于task1中的SMO算法具体使用代码

##### （1）子模块

- **数据读取函数_修改**:task1中的数据读取返回特征和标签，而测试集中不含有标签故修改读取函数

```python
def loadData(filename):
    dataDict = loadmat(filename)
    if 'y' in dataDict.keys():
        return dataDict['X'], dataDict['y']
    else:
        return dataDict['X']
```

- - **代码细节**:判断是否存在标签y，若存在返回特征x和标签y；不存在则只返回特征x

  

<br>
- **分类器预测算法：**

```python
# 调用sklearn库函数
def predict_sklearn(x, y, x_test):
    x_train, x_validation, y_train, y_validation = train_test_split(x, y, test_size=0.2)
    svm_clf = svm.LinearSVC()
    svm_clf.fit(x_train, y_train)
    acc = svm_clf.score(x_validation, y_validation)
    print('验证集精度为: %.4f' % acc)
    
    # svm_clf.fit(x, y)
    dec = svm_clf.predict(x_test)
    np.savetxt('task3.txt', dec, fmt="%d")

# 调用task1中的算法，并划分验证集
def predict_rate_valid(x, y, x_test):
    x_train, x_validation, y_train, y_validation = train_test_split(x, y, test_size=0.2)
    model = s.svmTrain_SMO(x_train, y_train, C=1, tol=1e-3, max_iter=20)

    pre_train = s.svmPredict(model, x_train)
    pre_validation = s.svmPredict(model, x_validation)
    y_test = s.svmPredict(model, x_test)

    rate1 = 1 - (np.sum(abs(np.array(pre_train) - np.array(y_train))) / float(len(y_train)))
    rate2 = 1 - (np.sum(abs(np.array(pre_validation) - np.array(y_validation))) / float(len(y_validation)))
    # rate1 = ('%.4f' % rate1)
    # rate2 = ('%.4f' % rate2)
    print('用训练集20%%划分为验证集, 训练集模型准确度为：%.4f, 验证集准确度为：%.4f' % (rate1, rate2))

    txtsave('task3/task3_testlabels2.txt', y_test)
    return rate1, rate2

#调用task1中的算法，不划分验证集
def predict_rate(x, y, x_test):
    model = s.svmTrain_SMO(x, y, C=1, tol=1e-3, max_iter=20)
    pre_train = s.svmPredict(model, x)
    y_test = s.svmPredict(model, x_test)

    rate = 1 - (np.sum(abs(np.array(pre_train) - np.array(y))) / float(len(y)))
    # rate = ('%.4f' % rate)
    print('训练集精度为：%.4f' % rate)

    txtsave('task3/task3_testlabels1.txt', y_test)
    return rate
```

- - **代码细节**：

1. 含有三个函数,分别作用如下：
predict_sklearn():  调用sklearn库函数svm;  
predict_rate_valid():  调用task1中的svm算法，并划分验证集 ; 
predict_rate():调用task1中的svm算法，不划分验证集

2. 采用rate = 1 - (np.sum(abs(np.array(pre_train) - np.array(y))) / float(len(y)))计算精度，与task2中方法相同
sklearn中采用acc = svm_clf.score(x_validation, y_validation)计算精度

3. 采用from sklearn.model_selection import train_test_split划分数据集


<br>

- **数据保存算法**:将预测的标签值保存为txt文件

```python
def txtsave(filename, data):  # filename为写入txt文件的路径，data为要写入数据列表.
    file = open(filename, 'w')
    for i in range(len(data)):
        s = str(data[i]).replace('[', '').replace(']', '').replace(',', '\t')  # 去除[]
        s = s.replace("'", '').replace(',', '').replace(',', '\t') + '\n'  # 去除单引号，逗号，每行末尾追加换行符
        file.write(s)
    file.close()
    print(filename, "保存文件成功")
```

- - **代码细节**： 代码简单，使用python文件操作，注意使用.replace()去除不需要的符号


<br>

##### （2）main函数

```python
if __name__ == '__main__':
    xTrain, yTrain = s.loadData('task3/task3_train.mat')
    xTest = s.loadData('task3/task3_test.mat')
    # train_len = len(xTrain)
    # train_feat_len = len(xTrain[0])
    print('训练集样本数为：%d, 特征维度为：%d' % (len(xTrain), len(xTrain[0])))
    print('测试集样本数为：%d, 特征维度为：%d' % (len(xTest), len(xTest[0])))
    rate = predict_rate(xTrain, yTrain, xTest)
    rate1, rate2 = predict_rate_valid(xTrain, yTrain, xTest)
    predict_sklearn(xTrain, yTrain, xTest)

```

- - **代码细节**： 调用之前设计的数个函数即可

  

## 二、运行结果

- **采用Google colab运行代码**(调用手写的svm算法)
  

  <img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/9.png?raw=true" alt="9" style="zoom: 55%;" />
  <img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/8.png?raw=true" alt="8" style="zoom: 50%;" />



- **自己电脑运行代码：**（sklearn库函数）

<img src="https://github.com/izhuhaoran/hexo_imags/blob/main/ML_tasks/svm/img/10.png?raw=true" alt="10" style="zoom: 67%;" />




- - **实验结果分析：上图结果都是多次调参运行后的最好结果**

1. 调用手写的SVM算法运行时间较长大概10分钟左右，精度为0.96左右（会随着参数改变而改变
2. 调用sklearn库函数运行时间很短(几秒钟)，精度大约为0.97（会随着参数改变而改变
  
   

## 三、总结体会

####  程序设计心得:

1. 本次算法过程中，遇到程序错误在正常不过，知错能改，善莫大焉，一个完整的工程背后一定是一次次的debug，自身犯错最多的地方在于差错处理，可以多考虑极端情况，多进行程序的调试，对每一种特殊情况进行处理，避免程序运行的自行结束。通过这次软件课设，自身养成了不惧困难，不退缩，提高思维能力
2. 本次代码编写过程中，由于task1中已经实现SMO算法，故重点在于对于该算法函数的调用，注意参数的设置，编写对应的格式化输出函数即可；也可以调用sklearn库函数中的svm相关包实现。