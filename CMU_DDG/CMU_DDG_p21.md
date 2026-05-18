# CMU_DDG_p21

> 主题：拉普拉斯算子（$\Delta$）与拉普拉斯-贝尔特拉米算子（$\Delta_g$）——从欧氏空间到流形的推广，及其几何、物理与变分含义。

## 一、为什么 Laplacian 重要

### 1. 历史与物理起源

由拉普拉斯在研究引力时提出。它是少数几个几乎渗透所有物理基础线性方程的算子：

| 方程 | 形式 | 含义 |
| :--- | :--- | :--- |
| 拉普拉斯方程 | $\Delta u = 0$ | 势场平衡分布、稳态 |
| 泊松方程 | $\Delta u = f$ | 带源/汇的稳态 |
| 热方程 | $\partial_t u = \alpha\Delta u$ | 温度扩散 |
| 波动方程 | $\partial_t^2 u = c^2 \Delta u$ | 位移传播 |

更复杂的弹性力学、量子力学等系统在这些线性核心之上加非线性项构成。

### 2. 计算价值

将许多几何处理 / 物理仿真 / 图论 / 机器学习问题归结为**稀疏线性系统**——这是已经被高度优化的基础任务，现成库丰富。

### 3. 几何价值

- 提供曲面**平均曲率**的统一表达。
- **等距不变**——曲面在保距弯曲下，$\Delta$ 不变。
- 在任意紧致流形上提供一组**频率基**（特征函数），把"傅里叶分析"推广到任意几何。

> 符号约定：本课程用 $\Delta$ 表示 Laplacian，$\nabla^2$ 留给 Hessian（注意有些物理书将两者混用）。

---

## 二、Laplacian 的多种等价定义

下列定义在 $\mathbb{R}^n$ 上等价；多数可推广到流形（少数不行）。

### 1. 偏导数之和（仅 $\mathbb{R}^n$）

$$\Delta u = \sum_{i=1}^n \frac{\partial^2 u}{\partial x_i^2}$$

二维例：$u(x,y) = -x^2 - 2y^2$，$\Delta u = -2 + (-4) = -6$；$u = x^3 - 3x^2$ 在全平面 $\Delta u = 6x - 6 + 0 \ne 0$（非调和）。

### 2. Hessian 的迹

$$\Delta u = \mathrm{tr}(H(u)), \qquad H(u)_{ij} = \frac{\partial^2 u}{\partial x_i \partial x_j}$$

- Hessian 是**最佳二次逼近**，捕获**方向性**曲率（各向异性）。
- 拉普拉斯只保留各向同性的标量总和——信息更少，但更易处理。
- Hessian 作为双线性形式：$X^T H(u) Y$ = 沿 $X$ 方向对梯度求方向导数后再与 $Y$ 内积。
- 牛顿法、各向异性扩散等需要 Hessian；扩散/平滑只用 Laplacian 就够。

### 3. 梯度的散度（最通用）

$$\Delta u = \nabla \cdot (\nabla u)$$

这是推广到流形最自然的形式。只要黎曼度量 $g$ 给出内积与体积形式，就能定义 $\nabla_M$（梯度）和 $\mathrm{div}_M$（散度），从而：

$$\Delta_g u = \mathrm{div}_g(\nabla_g u)$$

### 4. 局部坐标显式公式（Laplace-Beltrami）

$$\Delta_g u = \frac{1}{\sqrt{\det g}}\sum_{i,j}\frac{\partial}{\partial x^i}\left(\sqrt{\det g}\, g^{ij}\, \frac{\partial u}{\partial x^j}\right)$$

笛卡尔坐标下 $g = I$、$g^{ij} = \delta_{ij}$、$\det g = 1$，退化为偏导数之和。

### 5. 外微分形式

$$\Delta u = \delta\, d\, u = -\star\, d\, \star\, d\, u$$

- $d$：外微分（拓扑性，与几何无关）。
- $\star$：Hodge 星算子（承载几何——面积、共形结构）。
- 该公式还隐含 **sharp ($\sharp$) / flat ($\flat$) 算子**——把向量场和 1-form 互转的"音乐同构"，由黎曼度量决定：梯度 $\nabla u = (du)^\sharp$，散度 $\mathrm{div}\,X = \star d\star X^\flat$，组合即得 $\Delta u = \star d\star d u$。
- 优势：**几何与拓扑解耦**。在 DEC（离散外微积分）框架中：
  - $D$：稀疏矩阵表示离散 $d$。
  - $S$：对角矩阵表示离散 $\star$。
  - 离散 Laplacian $\approx S^{-1} D^T S D$。

### 6. Dirichlet 能量的变分

$$E(u) = \frac{1}{2}\int_M \|\nabla u\|^2\, dV$$

- 衡量函数的**粗糙度**——梯度大处能量大。
- $\Delta$ 是 $E$ 的（负）梯度：$\delta E = \int (\Delta u)\,\delta u\,dV$。
- **调和函数**（$\Delta u = 0$）就是 $E$ 的临界点。
- 内积形式：$E(u) = \langle u, \Delta u\rangle_{L^2}$（自伴性的直接体现）。

### 7. 布朗运动的无穷小生成元

设 $X_t$ 是从 $x$ 出发的布朗粒子，$u(t,x) = \mathbb{E}[\varphi(X_t)]$ 解 $\partial_t u = \Delta u$。则：

$$\Delta\varphi(x) = \lim_{t\to 0}\frac{\mathbb{E}[\varphi(X_t)] - \varphi(x)}{t}$$

**直觉**：$\Delta u(x)$ ≈ "把 $u$ 模糊一下" 减 "原 $u$"，再除以 $t$。

---

## 三、核心几何直觉：偏离局部平均

无论从哪个定义出发，$\Delta u(x)$ 都可以读作：**该点函数值与其无穷小邻域平均值的偏差**。

### 1. 图 Laplacian（最直观的离散类比）

设图顶点 $i$、邻居 $j\in\mathrm{adj}(i)$、值 $u_i$：

$$\Delta u_i = \frac{1}{\deg(i)}\sum_{j\in\mathrm{adj}(i)} u_j - u_i$$

- 第一项 = 邻居平均；第二项 = 中心值；差就是"我比邻居高多少"。

### 2. 一维极限

中心点 $x_0$ 取邻居 $x_0\pm\varepsilon$：

$$\Delta u(x_0) \propto \lim_{\varepsilon\to 0}\left[\frac{u(x_0+\varepsilon)+u(x_0-\varepsilon)}{2} - u(x_0)\right]$$

恢复二阶导数。

### 3. 多维（球面平均）

$$\Delta u(x_0) = \lim_{\varepsilon\to 0}\frac{C}{\varepsilon^2}\left(\frac{1}{|S|}\oint_S u\,dS - u(x_0)\right)$$

**统一图像**：$\Delta$ = "比邻居有多特别"。

### 4. 与曲率的关系（一维）

二阶导数描述凹凸；曲率公式 $k = \frac{|u''|}{(1+(u')^2)^{3/2}}$。**在极值点**（$u'=0$）二阶导等于曲率。一般情况下 $\Delta$ 是各方向"平均凸凹性"的多维推广。

---

## 四、热方程与波动方程的物理诠释

### 热方程 $\partial_t u = \Delta u$

- $\Delta u > 0$（值低于邻居均值）→ 热量流入，温度上升。
- $\Delta u < 0$（值高于邻居均值）→ 热量流出，温度下降。
- $\Delta u = 0$ → 局部平衡。
- 长期演化：边界固定时，每点都收敛到局部均值——即**调和函数**。

> 类比：社交网络上每人 IQ 不断取朋友平均，最终所有连通分量内每人和朋友"一样聪明"——非常和谐的稳态。

### 波动方程 $\partial_t^2 u = \Delta u$

- 高于邻居均值的点受向下力；低于的受向上力。
- $\Delta$ 在这里扮演"恢复力"，把波形维持。

**核心结论**：$\Delta$ 量化"格格不入"的程度，是把系统推向均匀化的根本驱动力。

---

## 五、Laplacian 的代数/谱性质

### 1. 五大基本性质

| 性质 | 说明 | 备注 |
| :--- | :--- | :--- |
| 常数函数在核中 | $\Delta c = 0$ | 总是成立 |
| 线性函数在核中 | $\Delta(a\cdot x + b) = 0$ | **仅限欧氏空间**；流形上一般无定义 |
| 刚性运动不变 | 平移、旋转后 $\Delta$ 不变 | 计算结果不依赖坐标系 |
| 等距不变 | 保距形变下 $\Delta$ 不变 | 弯手肘、折地图后无需重算特征向量 |
| 自伴 | $\int v\,\Delta u = \int u\,\Delta v$ | 分部积分直接得；保证实特征值与正交特征基 |

### 2. 类比正半定矩阵

- $\langle u, \Delta u\rangle = \int \|\nabla u\|^2 \ge 0$，能量景观是**凸碗**。
- 沿碗下滑就能最小化，可逆性强（核只含常数）。
- $\Delta$ 是椭圆算子；在紧致域上是自伴椭圆算子。

### 3. 谱定理

紧致域上自伴椭圆算子有：
- 离散特征值 $\lambda_1\le\lambda_2\le\cdots$；
- 正交特征函数 $\phi_1, \phi_2, \ldots$。

**实例：圆上 $d^2/dx^2$**——特征函数是 $\cos(nx), \sin(nx)$，特征值 $-n^2$；构成函数空间正交基（Fourier 级数）。

### 4. 在任意曲面上的推广

球面上 → 球谐函数（化学中的轨道理论）。任意紧致曲面 → 一组离散频率基函数，是几何处理"谱方法"的基础（信号分解、滤波、形状描述符）。

---

## 六、调和函数与 Dirichlet 问题

### 1. 调和函数的核心刻画

$\Delta u = 0$ 的解称为调和函数。等价描述：

- **能量极小**：在给定边界值下使 Dirichlet 能量 $E(u) = \tfrac{1}{2}\int|\nabla u|^2$ 最小（Dirichlet 原理）。
- **均值性质**：对任意完全位于域内的球，
  $$u(x) = \frac{1}{|B(x,r)|}\int_{B(x,r)} u\,dy = \frac{1}{|\partial B(x,r)|}\oint_{\partial B(x,r)} u\,dS$$
  对**任意**半径 $r$ 都成立。
- **最大原理**：调和函数在内部不存在严格的极大/极小；极值必然出现在边界。
  - 若内部某点是局部最大，它不可能等于周围的平均（因为均值小于此最大），矛盾。
  - 在环域等多连通拓扑下，极大与极小可能位于不同边界分量上。

### 2. 推导：能量最小化 → 拉普拉斯方程

$E(u) = \tfrac{1}{2}\int|\nabla u|^2$，凸 → 局部极值就是全局极值。

变分法：扰动 $u\to u + \varepsilon\eta$（$\eta$ 在边界为 0），

$$\delta E = \int \nabla u\cdot\nabla\eta = -\int (\Delta u)\,\eta = 0\quad\forall\eta \;\Rightarrow\; \Delta u = 0$$

边界条件 $u|_{\partial\Omega} = g$。

**Dirichlet 原理**：极小化 Dirichlet 能量 ⇔ 解 Laplace 方程。

> 历史插曲：这个原理曾因 Weierstrass 指出"极限可能取不到"而被搁置数十年，直到 Hilbert 用泛函分析严格证明。教训：好的几何直觉不应因暂缺严格证明就抛弃；离散设定也常能为光滑设定提供新直觉。

### 3. 物理对应

- 稳态热分布：边界温度固定，长时间演化的解。
- 肥皂膜：给定边界框，最小表面（线性化为最小 Dirichlet 能量）。
- Thin Plate Spline：基于类似能量的插值。
- 半监督学习：在图 Laplacian 上扩散标签。

### 4. 曲面上的推广

把 $\Delta$ 换成 $\Delta_M$，把 $\Omega$ 换成曲面 $M$，所有结论不变：
- 在曲面少数顶点固定值（如 $\pm 1$），求 $\Delta_M u = 0$ + 边界，得到全曲面的平滑插值——**几何处理中常用的"曲面数据传播"工具**。

---

## 七、Poisson 方程

### 1. 定义与物理意义

$$\Delta u = -f \;\;\text{在}\;\Omega,\qquad u = g \;\;\text{在}\;\partial\Omega$$

- $f = 0$ 退化为 Laplace 方程。
- $f \ne 0$ 表示有热源 / 汇——稳态热分布。
- 也来自变分：给向量场 $X$ 找最佳标量势 $u$ 使 $\nabla u$ 拟合 $X$，得到 $\Delta u = \mathrm{div}\,X$。这是从向量场重构标量势的标准操作（图像处理、几何处理常用）。

### 2. Green 函数与卷积解

Green 函数 $G(x,y)$ 解 $\Delta G(x,y) = -\delta(x-y)$——单位点源在 $x$ 处的响应。性质：源点处奇异，远处平滑衰减。

线性叠加 → 任意 $f$ 的解：

$$u(x) = \int_\Omega G(x,y)\,f(y)\,dy$$

> **核心观点**：解线性 PDE = 与基本解（Green 函数）卷积。

---

## 八、边界条件与可解性

边界条件**完全决定**解。代码里最易出错的地方。

### 1. 三类常见条件

| 类型 | 形式 | 含义 |
| :--- | :--- | :--- |
| Dirichlet | $u = g$ on $\partial\Omega$ | 固定边界值 |
| Neumann | $\partial u/\partial n = h$ on $\partial\Omega$ | 固定边界法向导数（通量） |
| Robin | $\alpha u + \beta\partial u/\partial n = h$ | 混合 |

### 2. 一维：仿射函数

$d^2u/dx^2 = 0$ 解全是 $u(x) = ax + b$（直线）。

- **Dirichlet** 总有解：直线过任意两点。
- **Neumann** 一般无解：直线只有一个斜率，要求两端斜率不同时矛盾——必须**一致**。
- **混合**（一端值 + 另一端导数）：有解。

### 3. 二维 Dirichlet：通常有解

物理直观：边界值视为热源温度，让热扩散达稳态——这个稳态分布就是解。少量病态例子（极端边界）可让解不存在，工程问题中通常无忧。

### 4. 二维 Neumann：必须满足相容性

由 $\Delta u = 0$ 与散度定理：

$$0 = \int_\Omega \Delta u = \int_\Omega \nabla\cdot(\nabla u) = \oint_{\partial\Omega}\frac{\partial u}{\partial n}\,dS = \oint_{\partial\Omega} h\,dS$$

> **Neumann 可解性条件**：边界数据 $h$ 必须满足 $\oint_{\partial\Omega} h\,dS = 0$——"进去的一定要出来"。

否则**无解**。这是工程上最容易踩的坑：求解器不会告诉你问题定义不合法，它会"盲目地"输出一个无意义的伪解。**写求解器之前应预先检查可解性**。

---

## 九、关键术语速查

| 术语 | 含义 |
| :--- | :--- |
| Laplacian $\Delta$ | $\nabla\cdot\nabla$；$\sum \partial^2/\partial x_i^2$；衡量与局部均值的偏差 |
| Laplace-Beltrami $\Delta_g$ | Laplacian 在黎曼流形上的推广，由度量 $g$ 决定 |
| Hessian | 二阶偏导数矩阵；其迹 = $\Delta u$；保留方向性曲率 |
| 度量张量 $g$ | 给出切向量内积；流形上一切几何量的来源 |
| 调和函数 | $\Delta u = 0$ 的解；Dirichlet 能量的临界点 |
| Dirichlet 能量 | $\tfrac12\int\|\nabla u\|^2$；衡量函数粗糙度 |
| 均值性质 | 调和函数 = 任意内含球上的均值 |
| 最大原理 | 调和函数极值在边界 |
| Poisson 方程 | $\Delta u = -f$；带源稳态 |
| Green 函数 | 点源解；卷积构造任意 $f$ 的解 |
| Hodge 星 $\star$ | 微分形式间的几何对偶；承载度量信息 |
| 外微分 $d$ | 拓扑性的微分；$d^2 = 0$ |
| Dirichlet 条件 | 边界固定函数值 |
| Neumann 条件 | 边界固定法向导数；需满足通量为零 |
| Neumann 相容性 | $\oint h = 0$，否则无解 |
| 谱定理 | 自伴椭圆算子有离散实特征值与正交特征基 |
| 球谐函数 | 球面上 $\Delta_g$ 的特征函数 |
| 等距不变 | 保距弯曲下 $\Delta$ 不变；用于形状分析 |
| 自伴 | $\int v\Delta u = \int u\Delta v$；保证谱定理成立 |
| Brownian 生成元 | $\Delta\varphi = \lim_{t\to 0}(\mathbb{E}[\varphi(X_t)] - \varphi)/t$ |
