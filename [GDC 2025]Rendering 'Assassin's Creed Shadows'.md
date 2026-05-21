# [GDC 2025]Rendering 'Assassin's Creed Shadows'

# 《刺客信条：影》渲染技术深度解析

> 主题：本演讲由 Nicolas Lopez 主讲，系统介绍了 Envil 引擎从多分支混战到统一代码库的演进历程，并深入剖析《刺客信条：影》在虚拟几何体、混合光线追踪全局光照、天气季节系统以及跨平台可扩展性方面的核心渲染技术。

## 一、Envil 引擎演进与统一代码库战略

### 1.1 引擎起源与分支演化
- **起源**：伴随 **2007 年《Assassin's Creed 1》** 问世，实际开发始于 2004 年，专为**大型系统化开放世界**、**超远距离渲染**及**系统化玩法**打造。
- **分裂与代价**：最初为《刺客信条》专用引擎 **C-meter**，后分化出 **Blacksmiths**（《Rainbow Six》系列）与 **Silix**（《Ghost Recon》开放世界作品）。育碧内部曾同时维护大量引擎及分支，导致开发力量分散、功能重复实现、竞争力下降。
- **统一战略**：当前以 **Anvil 作为共享引擎**，采用**单一代码库（Monorepo）**，全球团队共同维护，实现跨游戏、跨品类、跨 IP 的复用。

### 1.2 GPU 驱动管线演进
自 2014 年《刺客信条：大革命》起，**GPU 驱动管线**便成为引擎基石，历经多次迭代：
- **初代 Batch Renderer（批次渲染器）**：《大革命》中基于 DX11 构建，引入 GPU 集群剔除与间接绘制调用，但 CPU 参与度高、不支持无绑定纹理，仅支持逐材质批处理。
- **新一代 GPU Instance Renderer（GPU IR）**：为 DX12 级 API 设计，实现**完全无绑定**渲染，批处理粒度细化为**逐着色器**，支持运行时动态合批，设计目标为每帧剔除数百万实例。
- **核心数据结构：Database**：一个 CPU 与 GPU 间共享的**超级结构化缓冲区**，遵循数据导向设计。通过内部着色器绑定编译器 **SIG** 自动生成访问代码，并引入**关系（Relations）** 机制在不同表之间建立链接，类似关系数据库的高效组织方式。

---

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_main_000_000000.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_001_000003.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_003_000011.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_005_000053.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_main_006_000112.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_007_000116.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_008_000120.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_010_000128.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_011_000132.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_012_000136.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_013_000140.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_014_000144.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_015_000148.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_016_000152.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_main_018_000202.jpg)


## 二、《刺客信条：影》渲染目标与技术支柱

### 2.1 世界规模与可扩展性目标
- **基础规格**：**16km × 16km 开放世界**，具备动态时间循环、系统性天气、四季变换及大规模破坏。
- **平台定位**：纯次世代作品，提供**性能、平衡、质量**三档渲染模式，对应不同目标分辨率。
- **核心目标**：将可扩展性推至新高度，质量模式优先采用光线追踪。

### 2.2 面向长距离的世界架构
世界数据按**单元格**组织，由数据驱动的**流式网格层系统**管理，距离由远及近分为：
- **远景地形**：8 公里以外，仅渲染烘焙细节的地形远景。
- **点云渲染器**：超快速大规模 Impostor 渲染，将森林等远景延伸至 **8 公里**。
- **虚拟实体网格**：经减面和聚合处理的资产，可视距离最远达 **4 公里**。
- **LOD 网格层**：短程主网格包含绝大部分资产，长程网格仅保留大型资产与重要兴趣点。
- **LOD 选择器**确保实体加载距离与网格层匹配，防止异常消失。

---

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_019_000206.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_020_000209.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_023_000221.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_024_000225.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_025_000229.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_026_000233.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_027_000237.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_028_000241.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_029_000245.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_030_000249.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_032_000257.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_033_000301.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_main_035_000322.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_036_000326.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_037_000334.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_041_000354.jpg)


## 三、微多边形几何管线

### 3.1 设计理念与核心结构
- **灵感来源**：基于 Brian Karis 的 Nanite 系统演进而来。
- **集群层次结构**：模型由**集群层次（Cluster Hierarchy）** 构成，每个集群 **128 三角形**，具备连续细节层次，根据可见性逐个流式加载卸载。
- **混合光栅化策略**：根据三角形屏幕投影尺寸动态选择——大三角形走 **Mesh Shader 硬件光栅化**，微小三角形走**软件光栅化**。几何数据内存占用约为传统 GPU 层存储的一半。

### 3.2 独特技术实现
- **全流程无网格化（Meshless）**：与 GPU 实例渲染器一致，彻底摆脱传统网格概念。
- **可编程着色**：支持在着色器图中手动插入自定义代码。
- **软件光栅化优化**：对垂直走向三角形交换 XY 坐标，改善分支一致性，提升并行效率。
- **集群与角度剔除**：在最终光栅化前可选的额外剔除步骤。

### 3.3 应用场景与性能数据
- **微多边形管线**：当前仅支持静态不透明几何体，负责城市主体建筑与静态物体。典型城市场景约 **28,000 实例**、**3,400 万三角形**，高达 90% 三角形由软件光栅化完成。
- **GPU 实例渲染器**：作为互补，处理海量带 Alpha Test 的植被。森林场景渲染约 **9,000 实例**、近 **200 万三角形**。

---

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_01_00m00s-04m47s/frame_aux_044_000415.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_02_04m30s-07m45s/frame_main_000_000430.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_02_04m30s-07m45s/frame_main_002_000537.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_02_04m30s-07m45s/frame_aux_003_000541.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_02_04m30s-07m45s/frame_main_004_000554.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_02_04m30s-07m45s/frame_aux_008_000703.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_02_04m30s-07m45s/frame_aux_012_000721.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_03_07m25s-10m36s/frame_main_000_000725.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_03_07m25s-10m36s/frame_main_001_000741.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_03_07m25s-10m36s/frame_aux_002_000745.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_03_07m25s-10m36s/frame_main_003_000801.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_03_07m25s-10m36s/frame_aux_005_000929.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_03_07m25s-10m36s/frame_main_006_000939.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_03_07m25s-10m36s/frame_main_007_000947.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_03_07m25s-10m36s/frame_main_010_001007.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_03_07m25s-10m36s/frame_aux_011_001011.jpg)


## 四、全局光照管线演进与混合方案

### 4.1 从体积 GI 到可扩展方案
- **历史迭代**：《大革命》引入体积化开放世界 GI；《起源》将其稀疏化以适应 16km×16km 世界；《影》新增季节支持并构建全新光线追踪 GI 管线。
- **光谱式方案**：从全烘焙到全光线追踪，涵盖硬件光追与计算着色器软件实现，适配多样化硬件。

### 4.2 稀疏探针与级联体积
针对大规模世界数据爆炸问题（均匀方案将达 2TB），采用：
- **密度图**：美术仅在关键区域保留高精度探针。
- **稀疏八叉树**：仅当体素包含场景表面时细分，丢弃几何体内部探针，最终仅需约 10% 探针数量。
- **时间与季节**：存储 11 个时间关键帧，光照数据分离为太阳/局部光/天空并采用 YCoCg 格式。季节上假设春/夏相同，Lumen 分量跨季节共享，色度分量每季节单独存储。
- **运行时**：将稀疏探针解压缩注入**级联 3D 体积**，实现跨级联无缝混合，最终运行时数据仅约 **9GB**。

### 4.3 混合光线追踪 GI 管线
- **分层回退策略**：优先屏幕空间追踪（低成本）→ 命中存入 Retracing G-Buffer；未命中时用硬件 DXR 世界空间追踪；始终无命中则采样 DDGI 探针缓存。
- **DDDGI 级联探针**：5 个级联约 10,000 探针，每帧增量更新约 1,000 个，提供近中远距离间接光照，同时作为后续反弹光源。
- **后处理**：对 G-Buffer 执行重光照与降噪，然后叠加 **RT-AO** 补偿探针频率损失与 BVH 缺失物体（如草）的遮蔽，最后叠加镜面高光输出最终图像。
- **性能**：漫反射总开销约 5-6 毫秒（分布于图形与异步计算队列）；RT 探针更新约 1 毫秒。

### 4.4 镜面反射
在游戏延期后最后一刻加入，沿用漫反射 GI 管线设计。采用半分辨率追踪以平衡画质与性能，对高频信号下的 BVH 质量与降噪提出更高要求。最终选用 **Sondrop 的 MSD 降噪器**，一种基于递归模糊的特殊各向异性时空滤波器，支持球谐函数直接降噪。

---

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_04_10m16s-13m25s/frame_main_000_001016.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_04_10m16s-13m25s/frame_aux_002_001103.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_04_10m16s-13m25s/frame_main_005_001247.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_05_13m07s-15m59s/frame_main_000_001307.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_06_15m41s-18m43s/frame_main_000_001541.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_06_15m41s-18m43s/frame_main_003_001842.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_07_18m25s-21m52s/frame_main_000_001825.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_07_18m25s-21m52s/frame_main_001_001841.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_07_18m25s-21m52s/frame_main_003_001914.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_08_21m35s-24m32s/frame_main_000_002135.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_08_21m35s-24m32s/frame_main_003_002228.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_08_21m35s-24m32s/frame_main_005_002345.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_08_21m35s-24m32s/frame_aux_007_002422.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_08_21m35s-24m32s/frame_main_008_002430.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_09_24m14s-27m05s/frame_main_000_002414.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_09_24m14s-27m05s/frame_aux_001_002418.jpg)


## 五、光线追踪加速结构与材质优化

### 5.1 BVH 构建策略
- 根据几何与材质复杂度动态决定放入 BVH 的 LOD 级别，支持资产级/材质级覆盖。
- 使用全局 LOD 偏移统一控制简化程度，对微多边形基于 LOD 误差度量输出近似层级。
- 典型城市场景约 2,000 个 BLAS（服务于 30,000 实例），BVH 内存占用约 320MB。

### 5.2 植被 Alpha Test 优化
- **困境**：Any Hit 着色器精确但开销极大，限制命中次数则导致过度遮挡。
- **解决方案**：根据材质平均不透明度缩放三角形，将半透区域转化为缩小后的偏不透明实体，从而用 **Closest Hit** 高效追踪，追踪速度比 Any Hit 方案快 30%，漫反射 GI 结果与参考图非常接近。
- 对镜面反射追踪同样有效，且配合 Closest Hit 在屏幕空间中后续求解可获得像素级精确近距离结果。

### 5.3 统一材质与内联光追
- 采用 **Uber Shader** 方法处理所有光追材质分支，避免复杂的 JSON 阴影重映射。
- 所有材质数据集中存储在**材质表**中，通过命中几何体 ID 索引。季节效果通过材质版本切换，叶片颜色使用 LUT 控制。
- 支持软件光线追踪（三级 BVH + 空间分区按需局部更新），主要面向 GTX 1070 级别低端平台。

---

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_09_24m14s-27m05s/frame_main_003_002430.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_09_24m14s-27m05s/frame_aux_005_002450.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_09_24m14s-27m05s/frame_main_007_002602.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_10_26m50s-29m59s/frame_main_000_002650.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_10_26m50s-29m59s/frame_aux_003_002848.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_10_26m50s-29m59s/frame_main_004_002915.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_10_26m50s-29m59s/frame_aux_005_002919.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_10_26m50s-29m59s/frame_main_007_002949.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_11_29m42s-32m40s/frame_main_000_002942.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_11_29m42s-32m40s/frame_main_001_002950.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_11_29m42s-32m40s/frame_main_002_003006.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_11_29m42s-32m40s/frame_main_003_003026.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_11_29m42s-32m40s/frame_aux_005_003055.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_11_29m42s-32m40s/frame_aux_009_003236.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_12_32m25s-35m33s/frame_main_000_003225.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_12_32m25s-35m33s/frame_aux_001_003234.jpg)


## 六、天气与季节系统

### 6.1 Atmos 大气模拟框架
- 实时模拟并传播水汽、温度、湿度等大气因子，利用玩家周围低分辨率体素解算风场与碰撞。
- 物理量驱动下游系统：体积云生成消散、风与雨表现。部分量按海拔高度模拟。

### 6.2 Amiens 数据驱动图表系统
- 从旧有曲线驱动系统升级为完全数据驱动、技术美术可自定义的图表系统。
- 消费引擎与 Atmos 输入，驱动整个天气影响逻辑。

### 6.3 雨水与积雪渲染
- **雨水**：基于 Sebastian Lagarde 博客方案，通过暗化基础色和降低粗糙度表现湿润效果。湿润遮罩存储于 G-Buffer 的单精度浮点通道，生命周期为干燥→湿润→积水。
- **积雪**：分为数据驱动印章系统的深雪，以及基于遮罩的动态积雪。材质修改包括向雪 Albedo 插值、粗糙度调整、透射率降低、微可见度遮罩剔除，并依据表面法线方向决定积雪阈值（垂直面不积雪）。雪后积雪缓慢衰减并转化为湿润度与水洼，最终融化变干。

### 6.4 室内遮挡
复用全局光照的**室内体积**，光栅化生成室内深度图以计算遮挡因子，防止天气效果穿透室内。雨向深度图用于遮挡雨粒子与涟漪，使粒子可透过窗户进入室内。

---

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_12_32m25s-35m33s/frame_aux_004_003524.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_13_35m15s-38m25s/frame_main_000_003515.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_13_35m15s-38m25s/frame_main_002_003546.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_13_35m15s-38m25s/frame_main_004_003713.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_14_38m04s-41m25s/frame_main_000_003804.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_14_38m04s-41m25s/frame_main_001_003824.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_14_38m04s-41m25s/frame_main_002_003830.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_14_38m04s-41m25s/frame_aux_003_003834.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_14_38m04s-41m25s/frame_aux_004_003919.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_14_38m04s-41m25s/frame_aux_005_003931.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_14_38m04s-41m25s/frame_main_006_004018.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_14_38m04s-41m25s/frame_main_008_004103.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_15_41m05s-44m06s/frame_main_000_004105.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_15_41m05s-44m06s/frame_main_003_004236.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_15_41m05s-44m06s/frame_main_004_004244.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_16_43m46s-46m53s/frame_main_000_004346.jpg)


## 七、跨平台可扩展性与性能管理

### 7.1 平台管理器
为应对多套 GI 系统与不同模式下帧结构差异，开发了**平台管理器**作为引擎一等公民：
- **数据驱动性能设置**：针对平台与情景定制，UI 自动生成且支持实时编辑。
- **配置文件**：对应不同渲染模式，切换需维持世界状态。
- **情景触发**：由玩法逻辑触发（菜单、照片模式等），允许为特定场景精细调整功能（如照片模式提升毛发质量）。
- **修饰器**：数据驱动触发，可笔刷绘制或 3D 体积触发，用于解决局部性能问题或满足特殊场景需求。

### 7.2 资源追踪与遥测
- 内部工具提供资源生命周期追踪与内存峰值定位（高/低层视图）。
- 内置大量性能遥测，包括动态分辨率因子分布和 GPU 回归对比，数据存入数据库持续追踪性能回归。

### 7.3 macOS/iOS 平台适配
- 硬件跨度覆盖 M1 到 M4，低端机型为主要挑战，核心策略是对非关键特性进行妥协式降级。
- 依赖平台管理器灵活调配不同平台的特性组合。

---

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_16_43m46s-46m53s/frame_main_002_004415.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_16_43m46s-46m53s/frame_main_007_004516.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_17_46m36s-49m46s/frame_main_000_004636.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_17_46m36s-49m46s/frame_main_001_004719.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_17_46m36s-49m46s/frame_aux_002_004731.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_17_46m36s-49m46s/frame_main_005_004804.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_17_46m36s-49m46s/frame_main_007_004836.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_17_46m36s-49m46s/frame_aux_011_004923.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_18_49m26s-52m33s/frame_main_000_004926.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_18_49m26s-52m33s/frame_aux_005_004949.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_18_49m26s-52m33s/frame_aux_008_005041.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_18_49m26s-52m33s/frame_aux_009_005104.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_18_49m26s-52m33s/frame_main_010_005112.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_18_49m26s-52m33s/frame_main_014_005205.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_18_49m26s-52m33s/frame_main_016_005221.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_19_52m17s-55m22s/frame_main_000_005217.jpg)


## 八、总结与未来展望

- **项目里程碑**：首款搭载微多边形与统一引擎架构的刺客信条作品，系列最大规模与最强可扩展性。
- **未来方向**：更广泛使用微多边形覆盖更多管线；让光线追踪更普及以简化美术管线；推动帧模块化与数据驱动帧图调度以适应多游戏上下文；持续迭代天气季节系统的 QA 复杂度问题。
- **行业反思**：上采样器碎片化严重增加兼容性负担；不同时间/季节/天气组合导致测试工作量爆炸式增长，需重新思考复杂系统的管理与呈现方式。

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_19_52m17s-55m22s/frame_aux_001_005221.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_19_52m17s-55m22s/frame_aux_005_005240.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_19_52m17s-55m22s/frame_aux_006_005244.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_19_52m17s-55m22s/frame_main_007_005302.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_19_52m17s-55m22s/frame_aux_009_005324.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_20_55m04s-58m28s/frame_main_000_005504.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_20_55m04s-58m28s/frame_aux_001_005508.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_20_55m04s-58m28s/frame_main_004_005531.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_20_55m04s-58m28s/frame_main_005_005541.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_20_55m04s-58m28s/frame_main_006_005557.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_20_55m04s-58m28s/frame_aux_007_005601.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_20_55m04s-58m28s/frame_main_015_005749.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_21_57m44s-61m35s/frame_main_000_005744.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_21_57m44s-61m35s/frame_main_001_005750.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_22_61m19s-61m56s/frame_main_000_010119.jpg)

![截图](Rendering_%27Assassin%27s_Creed_Shadows%27_frames/chunk_22_61m19s-61m56s/frame_main_001_010154.jpg)

