## 2021本科毕业设计

[toc]

### 任务书：

#### 一、题目

面向多体博弈的有人/无人机编队智能空战决策方法

Intelligent air combat decision method for MAV-UAV formation based on multi-agents game

#### 二、研究主要内容

1. 选题来源：纵向科研课题。

2. 研究内容：

（1）基于博弈理论的有人/无人机编队智能空战决策模型；

（2）基于强化学习的有人/无人机编队智能决策优化算法。

3. 研究方法：

（1）多智能体博弈理论；

（2）强化学习理论。

4. 研究目标：

以异构协同空战为背景，研究有人/无人机编队作战行动、目标打击等方面的协同决策能力，提出一套完整的智能空战决策方法。作战想定为红蓝双方均为1架有人机与4架无人机编队进行自由空战，首先建立空战态势优势评价体系，其次基于博弈理论建立有人/无人机编队目标分配模型和机动决策模型，然后建立有人/无人机编队空战智能决策模型，最后结合强化学习理论优化模型算法，提高其在动态对抗环境下的适应性和鲁棒性。

#### 三、主要技术指标

1. 空战智能决策胜率不低于80%；

2. 决策具备实时性，决策时长不高于1秒；

3. 强化学习下的空战决策胜率提高5%以上。

### 第一阶段：查找资料、提出问题

#### 一、对研究内容的解读

>1. 基于博弈理论的有人/无人机编队智能空战决策模型；
>2. 基于强化学习的有人/无人机编队智能决策优化算法。

#### 二、提出对研究内容的问题

##### 1. 什么是博弈理论？

* Player：我方战斗机

  > 利益完全一致的多个参与者应视为同一个 Player

* Strategy：机动、攻击等等，一般是一个集合

* Payment：每一次决策得到的分数，一般是矩阵形式

  > 支付矩阵设置具有主观性，需要引入专家知识

* Situation：每一次决策后形成的局势

1. 博弈模型：完全信息静态博弈——纳什均衡
2. 红蓝两方对决可以简化为两人博弈吗？
3. 现在的问题是参与者要独立选择自己的策略，且在不知道对方策略的情况下使得自己得到最大的利益
4. **思路：**每一时间节点检查当前局势，通过局势信息确定支付矩阵，进行博弈选择

##### 2. 什么是空战决策？

根据《多对多无人机空战的智能决策研究》，空战决策可以分为以下两个部分：

* 目标分配：即考虑各种约束进行敌我的匹配以实现某种尺度下的群体最优

* 机动决策

##### 3. 强化学习如何应用？

#### 三、现存有关模型、方法

##### 1. 几何空间关系矩阵

对于某有人/无人机 $i$，其他某飞机 $j$ 与其之间几何关系以下考虑 5 个参数：

* 两机之间距离 $d^i_j$
* 该飞机对某飞机的滞后角 $\alpha^i_j$
* 该飞机对某飞机的超前角 $\beta^i_j$

则有：
$$
\left\{\begin{aligned}

& d^i_j = \abs{\vec d^i_j} = \sqrt{(x_i - x_j)^2 + (y_i - y_j)^2 + (z_i - z_j)^2}\\

& \alpha^i_j = \arccos{\frac{\vec v_i \cdot \vec d^i_j}{v_i \cdot d^i_j}}\\

& \beta^i_j = \arccos{\frac{\vec v_j \cdot \vec d^i_j}{v_j \cdot d^i_j}}\\

\end{aligned}\right.
$$

##### 2. 局势优势评价函数

> 表示某架有/无人机对另一架有/无人机相对优势的大小。优势函数越大，我方无人机生存并击毁敌机的可能性越大
>
> 包含：角度优势、距离优势和能量优势
>
> 感觉可以作为支付函数矩阵的计算依据

###### 角度优势函数

$$
G_A = 1 - \frac{\alpha^i_j+\beta^i_j}{180}
$$

###### 距离优势函数

$$
G_D = 

\left\{\begin{aligned}

& 1 & d \le R_0\\
& e^{- \frac{d-R_0}{k}} & d >R_0\\

\end{aligned}\right.
$$

其中，其中 $d$ 是我机与敌机之间的距离；$R_0$ 是机载武器最佳射程；$k$ 是无量纲参数

###### 能量优势函数

$$
G_E = 

\left\{\begin{aligned}

& 1 & \eta\geq 2\\

& 0.6 \eta -0.2 & 0.5\le\eta < 2\\

& 0.1 & \eta < 0.5

\end{aligned}\right.
$$

其中 $\eta = E_B/E_R$，表示我方和敌方无人机能量之比，其能量定义为：
$$
E = H+\frac{v^2}{2g}
$$
其中，$H$ 为飞机的高度，$g$ 为重力加速度

###### 优势函数和优势矩阵

$$
G = \omega_1G_A\times G_D+\omega_2G_E\\
G = \begin{bmatrix}
G_{11} & G_{12} & \cdots & G_{1n}\\
G_{21} & G_{22} & \cdots & G_{2n}\\
\vdots & \vdots & \ddots & \vdots\\
G_{m1} & G_{m2} & \cdots & G_{mn}\\
\end{bmatrix}
$$

其中，$\omega_1$ 和 $\omega_2$ 为各优势函数的权重，$G_{ij}$ 表示我方第 $i$ 架无人机对敌方第 $j$ 架无人机的优势函数的值，对于对抗双方均有优势矩阵

##### 3. 决策方法

首先利用证据理论对优势矩阵进行多级融合，最终获得双方的总优势矩阵 $G_R$ 和 $G_B$

构建目标函数，指导混合策略纳什均衡向最优解方向搜索进行
$$
\max = (Mu_R-Mu_B)
$$
即使得 R 方的在对抗过程中相对于 B 方的对抗优势越大
