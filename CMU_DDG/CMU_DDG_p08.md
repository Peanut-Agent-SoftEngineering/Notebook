# CMU_DDG_p08

> 主题：外微分 $d$ —— 把梯度、旋度、散度统一为一种与度量无关的、可推广到任意维度和弯曲流形的"导数"，并铺垫斯托克斯定理与 DEC。

## 一、为什么需要外微分

### 1. 经典向量微积分的局限

| 问题 | 经典视角 | 外微分视角 |
| :--- | :--- | :--- |
| 三个不同算子（grad/curl/div） | 各有定义、彼此独立 | 统一为一个 $d$ |
| 依赖内积、长度、角度 | 必须先有度量 | **完全不依赖度量**，纯拓扑/微分结构 |
| 在弯曲空间不易推广 | 需要局部坐标技巧 | 自然推广到任意维度、任意流形 |
| 一般体积变化的测量 | 只能算具体的面/体积分 | 给出统一框架 |
| 不同物理量的几何区分 | 速度/动量都是"向量" | 速度=向量，动量=1-形式（余向量） |

### 2. 外微分的目标

- **统一**：grad、curl、div 是 $d$ 在不同阶上的体现。
- **推广**：把"导数"提升为"$k$-形式 → $(k+1)$-形式"的算子。
- **离散化**：在网格上 $d$ 精确等于**边界算子**，是 DEC 的代数基石。
- **拓扑性**：建立 de Rham 上同调，连接微分与拓扑。

### 3. 重新审视"导数"

| 层次 | 视角 | 形式 |
| :--- | :--- | :--- |
| 一维微积分 | 变化率 / 斜率 | $f'(x) = \lim_{\epsilon\to 0}\frac{f(x+\epsilon)-f(x)}{\epsilon}$ |
| 多变量 | 最佳线性逼近（雅可比） | 切平面 |
| 微分几何（核心） | **推前（push-forward）**：把切向量从源映到目标 | $df: T_p M \to T_{f(p)}N$ |

> **关键转变**：不要把 $f$ 当作"图"，而当作**映射**。导数 $df$ 衡量"输入向量被拉伸/旋转成什么样"——这就是雅可比矩阵的几何本质，也是后续推广的统一起点。

### 4. 向量 vs 余向量：必须区分

| | 向量（vector） | 余向量 / 1-形式（covector） |
| :--- | :--- | :--- |
| 类比 | 箭头 | 测量工具（标尺、测速仪） |
| 物理量 | 速度、位移 | 动量、力（须经度量转换） |
| 行为 | 被移动 / 被测量 | 收缩（吃）向量 → 标量 |
| 数学位置 | $T_pM$（切空间） | $T_p^*M$（余切空间） |

在 $\mathbb{R}^n$ 中由于天然内积存在，二者常被混为一谈（梯度被当作向量）；但在弯曲流形上必须刻意分清——这正是外代数的价值所在。

---

## 二、经典 grad / div / curl 速览

### 1. 直觉

| 算子 | 输入 | 输出 | 几何含义 |
| :--- | :--- | :--- | :--- |
| 梯度 $\nabla\phi$ | 标量场 $\phi$ | 向量场 | 最陡上升方向 + 速率 |
| 散度 $\nabla\cdot X$ | 向量场 $X$ | 标量场 | 源/汇强度（流入/流出净通量） |
| 旋度 $\nabla\times X$ | 向量场 $X$ | 向量场（3D）/ 标量（2D） | 局部旋转/涡旋强度 |

### 2. 笛卡尔坐标公式（$X = (u, v, w)$）

$$\nabla\phi = \left(\tfrac{\partial\phi}{\partial x}, \tfrac{\partial\phi}{\partial y}, \tfrac{\partial\phi}{\partial z}\right)$$

$$\nabla\cdot X = \tfrac{\partial u}{\partial x} + \tfrac{\partial v}{\partial y} + \tfrac{\partial w}{\partial z}$$

$$\nabla\times X = \left(\tfrac{\partial w}{\partial y} - \tfrac{\partial v}{\partial z},\ \tfrac{\partial u}{\partial z} - \tfrac{\partial w}{\partial x},\ \tfrac{\partial v}{\partial x} - \tfrac{\partial u}{\partial y}\right)$$

### 3. 经典恒等式（即将统一为 $d^2 = 0$）

$$\nabla\times(\nabla\phi) = 0,\qquad \nabla\cdot(\nabla\times F) = 0$$

> 直觉重于公式：先记几何含义，再认公式形态。

---

## 三、$k$-形式的回顾与外积反对称性

| 阶 | 对象 | 几何 | 积分 |
| :--- | :--- | :--- | :--- |
| 0 | 标量函数 | 点上的值 | — |
| 1 | 余向量场 | 沿曲线测量 | $\int_C \omega$ |
| 2 | 面积元 | 穿过曲面 | $\int_S \alpha$ |
| 3 | 体积元 | 内部体积 | $\int_V \beta$ |

楔积 $\wedge$ 把 $k$-形式与 $l$-形式合成 $(k+l)$-形式，满足
$$\omega \wedge \eta = (-1)^{kl}\, \eta \wedge \omega$$
对应"交换两条边 → 取向反转"的几何事实。

---

## 四、外微分 $d$ 的公理化定义

**定义**：$d$ 是一族线性算子 $d: \Omega^k \to \Omega^{k+1}$，满足以下三条公理（且**唯一**）：

### 公理 1：作用于 0-形式 = 方向导数测量器

对任意标量函数（0-形式）$\phi$ 和任意向量场 $X$：
$$d\phi(X) = \mathcal{L}_X \phi = \nabla_X \phi$$

即 $d\phi$ 是一个 1-形式，它在向量 $X$ 上的"读数"等于 $\phi$ 沿 $X$ 的方向导数。

### 公理 2：广义莱布尼茨法则（带符号）

对 $k$-形式 $\alpha$ 与任意形式 $\beta$：
$$d(\alpha \wedge \beta) = d\alpha \wedge \beta + (-1)^k \alpha \wedge d\beta$$

符号 $(-1)^k$ 来自楔积的反对称性——保证代数自洽。

### 公理 3：幂零性（exactness）

$$d(d\omega) = 0\qquad\text{（即 } d^2 = 0\text{）}$$

对任意微分形式都成立。

### 几何动机汇总

- **公理 1**：捕捉所有方向上的瞬时变化率，把局部斜率拼成全局 1-形式。
- **公理 2**：保证导数的代数一致性，符号项追踪取向。
- **公理 3**：编码"**边界的边界为空**"这一拓扑事实——是 de Rham 上同调与斯托克斯定理的基石。

---

## 五、深入理解公理 1：方向导数与梯度的区别

### 1. 方向导数的极限定义

$$D_X\phi(p) = \lim_{\epsilon\to 0}\frac{\phi(p + \epsilon X) - \phi(p)}{\epsilon}$$

- $X$ 不仅是方向，还包含**速度大小**：走得越快、单位时间变化越大。
- 在向量场上逐点定义：$D_X\phi$ 是一个标量函数（0-形式）。

### 2. 梯度（坐标定义）

$$\nabla\phi = \sum_{i} \frac{\partial\phi}{\partial x^i}\,\mathbf{e}_i$$

在欧氏空间中通过点积联系方向导数：
$$D_X\phi = \langle \nabla\phi, X\rangle$$
（前提是 $X$ 为单位向量；否则乘以模长。）

### 3. 梯度的无坐标定义

> $\nabla\phi$ 是**唯一**的向量场，使得对任意向量场 $X$：
> $$\langle \nabla\phi, X\rangle = X[\phi]$$

- **依赖内积**：换一个内积，梯度也跟着变。
- **应用价值**：在网格几何处理中，选合适的内积让离散梯度更好地匹配曲率；在优化中，换内积可以**显著加速算法**（预条件、自然梯度）。
- **超越欧氏**：定义在无穷维函数空间中也成立——是泛函分析中梯度概念的基石。

### 4. 微分 $d\phi$（外微分版的"梯度"）

| 概念 | 类型 | 坐标公式（$\mathbb{R}^n$） | 作用方式 |
| :--- | :--- | :--- | :--- |
| 梯度 $\nabla\phi$ | **向量场** | $\sum_i \tfrac{\partial\phi}{\partial x^i}\,\mathbf{e}_i$ | $\langle\nabla\phi, X\rangle = D_X\phi$ |
| 微分 $d\phi$ | **1-形式** | $\sum_i \tfrac{\partial\phi}{\partial x^i}\,dx^i$ | $d\phi(X) = D_X\phi$ |

**本质区别**：
- 梯度组合**基向量场** → 是箭头；**依赖内积**。
- 微分组合**基 1-形式** → 是测量器；**只依赖偏导数（微分结构），不依赖度量**。
- 二者通过 $\sharp/\flat$ 桥接：$d\phi^\sharp = \nabla\phi$，$(\nabla\phi)^\flat = d\phi$。

> **工程师视角**：在网格上只需**连通性**就能定义 $d$（拓扑）；要算梯度才需要**顶点位置/边长**（几何）。代码上把 $d$ 与具体度量解耦，能极大复用。

---

## 六、深入理解公理 2：莱布尼茨法则的几何来源

### 1. 一维直觉：矩形面积

考虑 $f(x) g(x)$ 视为矩形面积，边长分别 $f$、$g$。$x$ 增加 $h$ 时：

$$f(x+h)g(x+h) - f(x)g(x) = \underbrace{\Delta f \cdot g}_{\text{底带}} + \underbrace{f \cdot \Delta g}_{\text{侧带}} + \underbrace{\Delta f \cdot \Delta g}_{\text{角块}}$$

- 前两项各只含一个 $h$ 因子，除以 $h$ 取极限得 $f'g$、$fg'$。
- **角块** $\Delta f\cdot\Delta g$ 含两个 $h$ → **高阶无穷小**，极限消失。

得到经典乘积法则：
$$\frac{d}{dx}(fg) = f'g + f g'$$

### 2. 推广：微分形式的"体积变化"

把一维矩形换成 $(k+l)$ 维"平行多面体" $\alpha\wedge\beta$。同时给 $\alpha$、$\beta$ 各加一个微小扰动：

- $d\alpha\wedge\beta$ ≈ "底带"
- $\alpha\wedge d\beta$ ≈ "侧带"
- $d\alpha\wedge d\beta$ ≈ "角块"，是高阶无穷小，不出现于线性化结果

得到广义乘积法则：
$$d(\alpha\wedge\beta) = d\alpha\wedge\beta + (-1)^k \alpha\wedge d\beta$$

### 3. 符号 $(-1)^k$ 的由来

把 $d$ "穿过" $\alpha$（一个 $k$-形式）时，需要利用楔积反对称性把 $d$ 与 $\alpha$ 对调位置；每对调一个 1-形式因子产生一个负号 → 总共 $(-1)^k$。

> **结论**：莱布尼茨法则不是约定，而是"忽略高阶小量"的几何线性化结果；符号 $(-1)^k$ 来自楔积反对称性，保证整套代数一致。

---

## 七、计算外微分：递归算法

### 1. 基础情形：0-形式（标量函数）

$$d\phi = \frac{\partial\phi}{\partial x^1}\,dx^1 + \cdots + \frac{\partial\phi}{\partial x^n}\,dx^n$$

把偏导作系数，组合基 1-形式。这是所有外微分计算的递归终点。

### 2. 递归规则

任意 $k$-形式可写成"标量函数 × 基楔积"。配合公理：

- **乘积法则**：$d(f\cdot\omega) = df \wedge \omega + f\, d\omega$（注意 $f$ 为 0-形式，$(-1)^0 = 1$）。
- **基的导数**：$d(dx^i) = 0$（由 $d^2 = 0$）。
- **楔积分配**。

### 3. 算法流程

1. 把形式按基楔积展开。
2. 对每个标量系数取 $d$（变成 1-形式）。
3. 利用 $d(dx^i) = 0$ 让"基微分"项消失。
4. 用反对称性 $dx^i\wedge dx^i = 0$、$dx^j\wedge dx^i = -dx^i\wedge dx^j$ 化简、合并同类项。

### 4. 工作示例（递归到底）

设 $\alpha = u\,dx$、$\beta = v\,dy$、$\gamma = w\,dz$，$\omega = \alpha\wedge\beta = uv\,dx\wedge dy$。

计算 $d(\omega\wedge\gamma)$：
- $d(\omega\wedge\gamma) = d\omega\wedge\gamma + (-1)^2 \omega\wedge d\gamma = d\omega\wedge\gamma + \omega\wedge d\gamma$
- $d\omega = d\alpha\wedge\beta - \alpha\wedge d\beta$（$\alpha$ 是 1-形式，符号是 $(-1)^1$）
- $d\alpha = du\wedge dx + u\,d(dx) = du\wedge dx$（因 $d(dx)=0$）
- 同理 $d\beta = dv\wedge dy$、$d\gamma = dw\wedge dz$
- 最终递归归约到 $du, dv, dw$ 这三个 0-形式的微分。

### 5. 高斯函数实例

$\phi(x, y) = \tfrac{1}{2}\, e^{-(x^2+y^2)}$：

$$d\phi = \frac{\partial\phi}{\partial x}\,dx + \frac{\partial\phi}{\partial y}\,dy = -2x\phi\,dx -2y\phi\,dy$$

---

## 八、$d^2 = 0$ 的代数验证与拓扑意义

### 1. 对 0-形式的代数验证

$$d(d\phi) = \sum_{i,j}\frac{\partial^2\phi}{\partial x^j\partial x^i}\,dx^j\wedge dx^i$$

- **混合偏导对称**：$\partial^2\phi/\partial x^i\partial x^j = \partial^2\phi/\partial x^j\partial x^i$。
- **楔积反对称**：$dx^i\wedge dx^j = -dx^j\wedge dx^i$；同因子 $dx^i\wedge dx^i = 0$。
- 对称量乘反对称基 → 全部抵消 → 结果 $= 0$。

### 2. 经典对应表

| 向量微积分 | 外微分 |
| :--- | :--- |
| $\nabla\times(\nabla\phi) = 0$（梯度无旋） | $d(d\phi) = 0$ on 0-form |
| $\nabla\cdot(\nabla\times F) = 0$（旋度无散） | $d(d\omega) = 0$ on 1-form |
| 梯度 (grad) | $d$：0-形式 → 1-形式 |
| 旋度 (curl) | $d$：1-形式 → 2-形式 |
| 散度 (div) | $d$：2-形式 → 3-形式 |

### 3. 拓扑解释（先放下，第 11 节展开）

$d^2 = 0$ 等价于**"边界的边界为空"** —— 这是斯托克斯定理与 de Rham 上同调的基础。

### 4. 实用价值

- **简化计算**：所有 $d(dx^i)$ 立即 = 0，是最强的化简武器。
- **正确性自检**：复杂算子链经常用 $d^2 = 0$ 校验，是"心理安全网"。
- **理论基石**：精确形式（exact）$\subset$ 闭形式（closed），de Rham 上同调度量"闭但不精确"的差距。

---

## 九、外微分实例：与 grad / curl / div 的精确对应

### 1. 例 A：对 1-形式 $\alpha = x\,dx + y\,dy$ 的外微分

$$d\alpha = d(x)\wedge dx + d(y)\wedge dy = dx\wedge dx + dy\wedge dy = 0$$

——保守场（来自势函数 $\tfrac{1}{2}(x^2+y^2)$ 的梯度），无旋。

### 2. 例 B：余微分 $d{\star}\alpha$（同样 $\alpha = x\,dx + y\,dy$）

第一步：$\star\alpha = x\,dy - y\,dx$（用 2D 规则 $\star dx = dy$、$\star dy = -dx$）。

第二步：
$$d(\star\alpha) = dx\wedge dy - dy\wedge dx = 2\,dx\wedge dy$$

——非零的 2-形式，散度 = 2，正是位置场 $(x, y)$ 的散度。

### 3. 例 C：3D 一形式的外微分（推出旋度）

设 $\alpha = u\,dx + v\,dy + w\,dz$。

$$d\alpha = du\wedge dx + dv\wedge dy + dw\wedge dz$$

展开 $du = u_x dx + u_y dy + u_z dz$ 等，利用 $dx^i\wedge dx^i = 0$ 与反对称性合并同类项：

$$d\alpha = \left(\tfrac{\partial v}{\partial x} - \tfrac{\partial u}{\partial y}\right)dx\wedge dy + \left(\tfrac{\partial w}{\partial y} - \tfrac{\partial v}{\partial z}\right)dy\wedge dz + \left(\tfrac{\partial u}{\partial z} - \tfrac{\partial w}{\partial x}\right)dz\wedge dx$$

——三个系数恰好是 $\nabla\times(u, v, w)$ 的三个分量。

### 4. 例 D：3D 一形式的"先星后微"（推出散度）

$\star\alpha = u\,dy\wedge dz + v\,dz\wedge dx + w\,dx\wedge dy$（用 3D 规则）。

$$d(\star\alpha) = \left(\tfrac{\partial u}{\partial x} + \tfrac{\partial v}{\partial y} + \tfrac{\partial w}{\partial z}\right)dx\wedge dy\wedge dz$$

——系数正是 $\nabla\cdot(u, v, w)$。

### 5. 三大算子的统一公式

把向量场 $X$ 与其 1-形式 $X^\flat$ 对应起来，则：

| 经典算子 | 微分几何公式 | 流程 |
| :--- | :--- | :--- |
| $\mathrm{grad}\,\phi$ | $(d\phi)^\sharp$ | 0-形式 $\to$ 1-形式 $\to$ 向量场 |
| $\mathrm{curl}\,X$ | $\bigl(\star\, d\, X^\flat\bigr)^\sharp$ | 向量场 $\to$ 1-形式 $\to$ 2-形式 $\to^\star$ 1-形式 $\to$ 向量场 |
| $\mathrm{div}\,X$ | $\star\, d\, \star\, X^\flat$ | 向量场 $\to$ 1-形式 $\to^\star$ 2-形式 $\to^d$ 3-形式 $\to^\star$ 0-形式 |

流程图（旋度示例）：
```
X  --♭-->  X^♭  --d-->  d X^♭  --★-->  ★ d X^♭  --♯-->  curl X
```

> **核心洞察**：grad/curl/div 都只是"$d$ 与 $\star$ 在不同阶停留时的投影"。$d$ 不依赖度量，$\star$ 引入度量。它们的**组合**编码了所有三大经典算子，并自然推广到弯曲流形。

### 6. 余微分 $\delta$（codifferential）

定义为
$$\delta := \star\, d\, \star\quad (\text{含符号因子，依阶与维度})$$

完整定义：$\delta = (-1)^{n(k+1)+1}\,\star d\,\star$，作用在 $k$-形式上。

- **降阶**：$\delta: \Omega^k \to \Omega^{k-1}$。
- **散度本质**：$\mathrm{div}\,X = \star d\star X^\flat = -\delta X^\flat$（差个符号约定）。
- **类比**：grad $\leftrightarrow$ $d$（升阶），div $\leftrightarrow$ $\delta$（降阶），是"伴随对"。
- **性质**：$\delta^2 = 0$，与 $d^2 = 0$ 对偶。

---

## 十、维度图：算子作用全景

### 1. 一维（$n = 1$）

仅有 0-形式与 1-形式：
- $d$：0-形式 $\to$ 1-形式（一维"梯度"）
- $\star$：0-形式 $\leftrightarrow$ 1-形式

### 2. 二维（$n = 2$）

- $\star$：0 $\leftrightarrow$ 2，1 $\leftrightarrow$ 1（在 1-形式上 $\star$ 等于"逆时针旋 90°"）
- $d$ 链：0 $\to$ 1 $\to$ 2
- $\delta$ 链：2 $\to$ 1 $\to$ 0
- 关系图是**交换图**：不同路径组合最终结果一致。

### 3. 三维（$n = 3$）

- $\star$：0 $\leftrightarrow$ 3，1 $\leftrightarrow$ 2（1-形式不再映回自身）
- $d$ 链：0 $\to$ 1 $\to$ 2 $\to$ 3
- $\delta$ 链：3 $\to$ 2 $\to$ 1 $\to$ 0

### 4. 维度奇偶差异

- **偶数维**：$\star$ 把 $k$-形式映入"对称类"（如 2D 中 1-形式 $\to$ 1-形式）。
- **奇数维**：$\star$ 永不把 $k$-形式映回同度数空间。

### 5. 总结表

| $n$ | $\star$ 作用 | $d$ 链 | $\delta$ 链 |
| :--- | :--- | :--- | :--- |
| 1 | $0\leftrightarrow 1$ | $0\to 1$ | $1\to 0$ |
| 2 | $0\leftrightarrow 2$，$1\leftrightarrow 1$ | $0\to 1\to 2$ | $2\to 1\to 0$ |
| 3 | $0\leftrightarrow 3$，$1\leftrightarrow 2$ | $0\to 1\to 2\to 3$ | $3\to 2\to 1\to 0$ |

### 6. 重要恒等式

- **拉普拉斯-德拉姆算子**：$\Delta = d\delta + \delta d$，作用于 $k$-形式仍得 $k$-形式。
  - 作用于 0-形式：$\Delta\phi = \delta d\phi$（第二项消失），等价于 $-\nabla\cdot\nabla\phi$ 或 $\star d\star d\phi$（含符号约定）。
  - 这是热扩散、振动模态、几何流的统治算子；在弯曲流形上自然推广为 Laplace-Beltrami。
- **庞加莱引理**：星形区域上，闭形式都是精确形式。
- **离散化**：在三角网格上把 $\star$ 替换为质量矩阵（顶点对偶面积），$d$ 替换为离散边界算子，即得 DEC 中的离散 Laplace。

### 7. 记忆三件套

- $d$：上升一级。
- $\delta$：下降一级。
- $\star$：在 $k\leftrightarrow n-k$ 之间跳跃。

---

## 十一、$d^2 = 0$ 的最深刻理由：斯托克斯定理与"边界的边界为空"

### 1. 斯托克斯定理

> **微积分基本定理的终极推广**：对一个 $k+1$ 维区域 $\Omega$、其边界 $\partial\Omega$、定义在其上的 $k$-形式 $\omega$：
> $$\int_\Omega d\omega = \int_{\partial\Omega} \omega$$

它统一了经典 Green、Gauss（散度）、Stokes（曲面）三大定理。一维特例：
$$\int_a^b f'(x)\,dx = f(b) - f(a)$$

### 2. 用斯托克斯解释 $d^2 = 0$

对任意 $\Omega$ 与 $\omega$，连续两次套用斯托克斯：
$$\int_\Omega d(d\omega) = \int_{\partial\Omega} d\omega = \int_{\partial(\partial\Omega)} \omega = \int_\emptyset \omega = 0$$

**关键事实**：**边界的边界 $\partial(\partial\Omega) = \emptyset$**——
- 二维圆盘的边界是闭合圆环，圆环没有起点终点。
- 三维球的边界是球面，球面没有边界。

由于这对**所有** $\Omega$ 与所有 $\omega$ 成立，必然 $d(d\omega) = 0$。

### 3. 因此 $d^2 = 0$ 不是代数巧合

它源自：
1. **微分**与**积分**互为对偶（斯托克斯）。
2. **拓扑**层面的"边界算子平方为零"。

这条恒等式联通了：
- 微分代数（$d^2 = 0$）。
- 拓扑（$\partial^2 = 0$）。
- 积分（斯托克斯定理）。

是现代几何与物理（电磁、流体、规范场）的共同基础。

---

## 十二、终极图谱：DEC 的方向

外微分提供了一个**完全离散化友好**的导数：在网格上 $d$ = 边界算子。这构成 **DEC（离散外微积分）** 的核心：

- **形式离散化**：$k$-形式离散为网格 $k$-单元（顶点、边、面、体）上的实数赋值。
- **$d$ 离散化**：$d$ = 单元的（带符号）邻接矩阵（边界算子的转置）——纯组合的，**不需要顶点位置**。
- **$\star$ 离散化**：通过对偶网格、对偶单元面积/体积构造（对角矩阵或更精细形式）；引入度量。
- **$\delta$、$\Delta$**：由 $d$ 与 $\star$ 组合得到。
- **$d^2 = 0$**：精确成立——纯拓扑事实，离散后也保留。
- **斯托克斯定理**：在网格上是单元加法的恒等式，自动满足。

> **一句话**：经典向量微积分 = 欧氏空间的特例；外微分 = 任意流形的统一框架；DEC = 它在网格上的精确离散映射，是几何处理与物理仿真的现代数学基础。

---

## 十三、关键术语速查

| 术语 | 含义 |
| :--- | :--- |
| 外微分 $d$ | $\Omega^k \to \Omega^{k+1}$ 的线性算子，统一 grad/curl/div |
| 公理化定义 | (1) $d\phi(X) = \nabla_X\phi$；(2) $d(\alpha\wedge\beta) = d\alpha\wedge\beta + (-1)^{|\alpha|}\alpha\wedge d\beta$；(3) $d^2 = 0$ |
| 推前 (push-forward) | 把切向量从源映到目标的导数视角 |
| 微分 $d\phi$ | 0-形式的外微分，1-形式版的"梯度"；不依赖度量 |
| 梯度 $\nabla\phi$ | 向量场版的"梯度"；依赖内积 |
| 方向导数 $\nabla_X\phi$ | $\phi$ 沿 $X$ 的瞬时变化率（含速度大小） |
| Sharp $\sharp$ / Flat $\flat$ | 1-形式 ↔ 向量场的转换 |
| 莱布尼茨法则（外微分版） | $d(\alpha\wedge\beta) = d\alpha\wedge\beta + (-1)^k\alpha\wedge d\beta$ |
| $d^2 = 0$ | 外微分幂零；对应"梯度无旋、旋度无散"，也对应"边界的边界为空" |
| 闭形式 / 精确形式 | $d\omega = 0$ vs $\omega = d\eta$；二者关系由 de Rham 上同调度量 |
| 霍奇星 $\star$ | $k$-形式 ↔ $(n-k)$-形式；引入度量信息 |
| 余微分 $\delta = \star d\star$ | $\Omega^k\to\Omega^{k-1}$，外微分的伴随；$\delta^2 = 0$ |
| Laplace-de Rham $\Delta = d\delta + \delta d$ | 作用于任意阶形式的拉普拉斯算子 |
| 斯托克斯定理 | $\int_\Omega d\omega = \int_{\partial\Omega}\omega$，统一 Green/Gauss/Stokes |
| 边界的边界 = $\emptyset$ | $\partial^2 = 0$，是 $d^2=0$ 的拓扑根源 |
| DEC（离散外微积分） | 在网格上以组合方式实现 $d$、用对偶单元实现 $\star$，自动保持 $d^2=0$ 与斯托克斯定理 |
