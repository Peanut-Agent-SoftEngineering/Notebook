# [GDC 2025]Rendering 'Assassin's Creed Shadows'

# Envoy 引擎与《刺客信条：影》渲染技术详解

> 主题：深入剖析 Anvil 引擎的发展历程、技术架构以及《刺客信条：影》中大规模开放世界渲染、全局光照、天气系统与跨平台可伸缩性的核心实现方案。

---

## 一、Anvil 引擎的起源与统一化进程

> 主题：回顾 Anvil 引擎的诞生背景、多分支发展导致的效率稀释，以及育碧转向单一代码库与共享引擎的战略决策。

### 1.1 发展脉络与核心定位
- **诞生背景**：诞生于 **2007年** 发布的《刺客信条1》，实际开发始于 **2004年**。设计初衷是为 **大规模系统性开放世界、远距离渲染及系统性玩法** 量身打造。
- **核心定位**：被广泛应用于《刺客信条》系列，同时支撑《幽灵行动》、《荣耀战魂》、《彩虹六号：围攻》等产品线。

### 1.2 历史教训：分支泛滥导致的效率稀释
- **多引擎、多分支的困境**：育碧曾同时维护 Anvil、Snowdrop、Dunia、Voyager、UbiArt 等多个引擎，且每个引擎内部有大量并行分支（例如 Anvil 有 3 个分支，Dunia 也类似）。
- **长周期系统的代价**：渲染、物理等大型系统研发耗时数年，在代码库中存活十年之久；分支泛滥导致相同特性需在不同引擎中重复实现，几乎不存在“推倒重来”的机会。
- **竞争力下降**：分散开发力量使整体效率降低，同一功能在多个引擎上重复劳动不再具有合理性。

### 1.3 当前策略：单一代码库与共享引擎
- **统一方案**：以 Anvil 作为 **跨游戏、跨品类、跨品牌** 的共享引擎，采用 **Monorepo 模式**将所有内容集中于一个 **单一代码库**，由遍布全球的技术团队与制作团队共同协作。

---

## 二、《刺客信条：影》的项目目标与可伸缩愿景

> 主题：阐述《刺客信条：影》作为首款真正次世代 Anvil 引擎作品的渲染范围、技术指标，以及面向第五代主机的图形模式规划。

| 类型 | 细节 |
|------|------|
| **世界规模** | 16km × 16km 开放世界 |
| **动态系统** | 全动态 **时间推进、系统性天气与季节** |
| **破坏与几何** | 大规模破坏、**虚拟几何体、光线追踪** |
| **季节循环** | 完整的四季变换（冬季包含积雪等状态） |

- 定位为 **首款真正次世代（Gen 5）Anvil 引擎作品**，将可伸缩性推向新高度。
- 仅面向 **第五代主机**（PS5 / Xbox Series），提供三种图形模式：**性能模式、平衡模式、画质模式**，分别对应不同目标分辨率。光线追踪优先应用于 **画质模式**。

---

## 三、大规模环境渲染与 LOD 架构

> 主题：介绍 Anvil 引擎为支撑超远距离渲染所设计的网格流式加载、多级 LOD 策略以及从 CPU 到 GPU 的驱动管线演进。

### 3.1 世界划分与网格流式加载
- 世界被划分为 **单元格**，采用 **流式网格层** 结构，层定义完全由 **数据驱动**。
- **LOD 网格分级**（统称 LOD Degrades）：
  - **短程主网格**：容纳大多数普通资产，小型道具优先因距离消失。
  - **长程网格**：专门保留大型地标、兴趣点等关键资产，使其在更远距离可见。
- 每个实体的加载距离由 **LOD 选择器** 控制，必须与其所在的 **加载网格** 范围匹配。

### 3.2 超远距离渲染层次
在标准 LOD 网格之外引入更远距离的表现手段：
- **假实体网格**：包含减面资产与聚合资产，有效范围最远可达 **4 公里**。
- **点云渲染器**：基于批量四边形渲染的大规模 Impostor 渲染器，专门用于树木等远景物体，最远可渲染至 **8 公里**，支持与真实模型的渐变过渡。
- **远景地形**：超过 8 公里后仅保留预烘焙细节的地形远景，不再渲染任何独立模型。

---

## 四、GPU 驱动渲染管线的演进与数据库架构

> 主题：详述 Anvil 引擎如何将渲染任务从 CPU 迁移至 GPU，历经 Batch Renderer 到 GPU Instance Renderer 两个标志性阶段，并引入 Database 数据结构实现 CPU/GPU 高效数据共享。

### 4.1 初代 GPU 驱动管线：Batch Renderer（DX11 时代）
- **代表游戏**：《刺客信条：大革命》（2014）
- **核心机制**：依赖 **Multi-Draw Indirect** 在 GPU 端完成集群网格的剔除与绘制，实现实例、集群、三角形三级可见性判断。
- **主要局限**：管线未完全移至 GPU，CPU 仍承压；不支持无绑定纹理，批处理仅能 **按材质** 进行；计算着色器兼容性差。

### 4.2 次代全 GPU 管线：GPU Instance Renderer（DX12 时代）
- **代表游戏**：《刺客信条：英灵殿》
- **核心革新**：
  - **完全无绑定**：消除纹理绑定限制，批处理转为 **按着色器** 进行，实现跨材质实例的合批。
  - **运行时批处理**：在 GPU 端实时完成实例数据的合并与提交。
  - **全流程 GPU 化**：将视锥剔除、LOD 选择、实例收集等步骤全部下沉至 GPU，彻底消除 CPU 瓶颈。

### 4.3 关键数据架构：Database
为实现 CPU 与 GPU 对完整场景描述的高效共享，设计了名为 **Database** 的特化数据结构。
- **本质**：遵循数据导向设计的容器，可作为超级结构化缓冲区，提供比裸缓冲区更高级的抽象。
- **引用机制**：**引用行**等价于指针，可直接跨 CPU/GPU 解引用；**关系** 支持在不同表之间建立链接，表达对象间的从属或关联逻辑。
- **生成支持**：引擎使用 **SIG 语言扩展**自动生成着色器绑定及 C++/HLSL 两端完全一致的访问接口，开发者只需使用类 SQL 的表声明语法即可定义数据结构。

---

## 五、场景数据管理与实例剔除管线

> 主题：讲解 CPU 与 GPU 间的数据同步策略，以及从 CPU 粗筛到 GPU 多级精筛的完整实例剔除渲染管线。

### 5.1 CPU/GPU 数据同步模式
- **复制模式**：将整个 CPU 数据拷贝至 GPU 的 Byte Address Buffer。
- **脏行/脏页更新模式**：维护行级或页级脏掩码，仅复制变更数据以节省带宽。
- **仅 CPU 暂存脏行模式**：面向超大表，CPU 端仅保留脏行，刷出后释放，避免内存饱和。

### 5.2 多级实例剔除管线
- **CPU 粗筛**：对以 **核心树叶节点（实例组）** 组织的空间进行粗略可见性判断，快速排除完全不可见的实例组。
- **GPU 帧级剔除**：对通过 CPU 剔除的实例，使用所有渲染通道的视锥体进行多视锥体同时测试，获得每实例的通道可见性掩码，并执行 LOD 选择。
- **GPU 通道级剔除**：对每个渲染通道独立执行视锥体剔除、遮蔽剔除等，阴影通道特化执行抗视锥体剔除。同时为通过测试的实例计算并填充描述符信息。
- **运行时可选精细剔除**：在最终提交绘制前，支持开启 **集群剔除** 和 **角度剔除** 作为补充优化。

---

## 六、微多边形几何管线

> 主题：介绍基于类似 Nanite 思想打造的微多边形管线架构，包括其核心原理、与 GPU 实例渲染器的场景分工及独特优化点。

- **核心架构**：网格由簇层级结构构成，具备连续细节层次；根据多边形在屏幕上的尺寸，混合采用硬件光栅化或软件光栅化。相同几何体的内存占用约为传统管线的一半。
- **光栅化决策**：依据三角形大小，选择 **Mesh Shader** 或 **软件光栅化**。
- **独特优化 —— 坐标交换**：软件光栅化本质为扫描线算法，当遇到垂直方向细长三角形时，交换 X 与 Y 坐标后再光栅化，以提高分支一致性，改善 SIMD 工作负载均衡。
- **场景分工**：
  - **微多边形管线**：仅支持静态不透明几何体（如城市建筑）。城市场景示例：约 28,000 个实例、3400 万三角形，90% 由软件光栅化完成。
  - **GPU 实例渲染器**：处理大量 Alpha Test 植被。森林场景示例：管理约 9,000 个实例、约 200 万三角形。

---

## 七、全局光照（GI）技术的演进与可伸缩管线

> 主题：梳理从《大革命》到《影》的 GI 技术迭代路径，重点讲解为解决季节变化而开发的全新混合光线追踪 GI 管线。

### 7.1 三部曲的 GI 发展路线
- **《大革命》**：首创体积化开放世界 GI，使用 GPU 光线束进行离线烘焙，划分为均匀 Mipmap 辐射度体积，不支持动态时间变化。
- **《起源》**：将体积 GI 稀疏化，以适配 16km×16km 的超大开放世界。
- **《影》**：为支持季节变化引入动态 GI，开发全新光线追踪 GI 管线。

### 7.2 可伸缩的 GI 管线设计
- **方案跨度**：从完全烘焙的漫反射+镜面反射，到全实时光线追踪的漫反射+镜面反射一次性计算。
- **执行方式**：可选择硬件光线追踪或 Compute Shader 的软件光线追踪。
- **特殊需求**：游戏中的“隐匿处”是完全动态的自定义建造空间，必须依赖光追产生 GI。

---

## 八、稀疏化探针分布与数据压缩方案

> 主题：详解为应对大规模开放世界数据膨胀而设计的密度图驱动稀疏探针放置策略，以及针对时间变化与季节变化的创新数据压缩技术。

### 8.1 关键解决方案：稀疏化与自适应分布
- **核心理念**：GI 的精度无需全局均匀。通过 **密度图** 标记需要高精度 GI 的区域（如城市关键区域），采用 **稀疏八叉树** 仅对包含模型表面的体素进行递归细分，并丢弃位于几何体内部的探针。
- **存储效果**：最终探针数量仅为传统均匀分布的约 **10%**。以《影》为例，数据量从潜在的 2 TB 压缩至约 9 GB。

### 8.2 时间变化与季节变化的近似处理
- **时间变化**：共 11 个关键帧（1 帧局部光源 + 10 帧太阳位置），光源独立存储，采用 YCoCg 颜色空间编码，亮度通道以方向性数据形式存储，色度通道支持时间插值。
- **季节变化近似**：
  - **核心假设**：春/夏视为同一季节（间接光照几乎相同）；亮度全季节共享，仅色度分季节存储。
  - **工程实践**：剧烈的几何变化可能产生亮-色不匹配，通过标记并剔除问题几何体来解决，实践中问题极少且大幅减少了多季节数据体积。

### 8.3 运行时级联体积与 LOD 混合
运行时对稀疏数据解压缩并注入多层级、均匀分布的 **级联 3D 体积** 中，执行跨级联混合与就地 GI 块 LOD 混合，消除视觉跳变，获得连续细节层次。

---

## 九、混合光线追踪全局光照流程

> 主题：剖析《影》中混合使用屏幕空间射线与世界空间射线的逐像素追踪管线，以及与探针缓存结合的多次弹射 GI 完整流程。

### 9.1 统一光线追踪抽象层
- **Fusion 硬件抽象层**：抽象平台特定 API（DXR/Vulkan RT），同时抽象硬件/软件光线追踪，统一内联与非内联光线追踪的编程模型。
- **回调接口**：提供统一追踪循环，内联模式和非内联模式均可调用相同的回调函数，可在两个管线间灵活切换，极大方便验证假设与测试新功能。

### 9.2 混合追踪流水线
1.  **屏幕空间追踪（加速层）**：首先在屏幕空间进行射线步进，命中结果写入 Hit Buffer；未命中则回退到世界空间。
2.  **世界空间追踪（回退层）**：延续屏幕空间射线的 T 值，使用硬件加速 BVH 进行追踪。
3.  **命中重光照与缺省处理**：对所有命中进行重光照并去噪/滤波；若像素完全没有命中，则采样 **探针缓存** 提供第一次弹射的辐射度。
4.  **多次弹射**：第一次弹射完成后，通过采样 DDGI 级联探针获取之后的所有间接弹射。
5.  **后处理**：降噪后，可选叠加 **RT-AO** 及镜面反射项生成最终图像。

### 9.3 为何额外添加 RT-AO
全局光照理论上已包含 AO，但实际仍需追加 RT-AO 以补偿因探针插值模糊、四分之一分辨率追踪丢失高频遮蔽、光追世界与光栅化世界不一致（如草等小型物体不参与光追）等问题，增强大型物体接地感。额外叠加微妙的屏幕空间 AO 弥补厘米级以下细节。

---

## 十、着色、材质与阴影优化

> 主题：涵盖 Uber Shader 着色器管理、材质表的季节处理，以及针对透明度追踪的三角形缩放优化策略。

### 10.1 着色器与材质管理
- 采用 **Uber Shader** 处理光追材质，配合严格的着色器种类管理（由 LODMaster 控制）减少绘制调用。
- 所有材质存储在 **材质表** 中，通过命中几何体 ID 访问。季节处理通过 **材质版本** 实现，叶子颜色依赖 **查找表** 进行映射。

### 10.2 透明度问题与三角形缩放优化
- **问题**：植被 BVH 中直接使用带透明度纹理追踪阴影时，Any-Hit 遍历开销极高；限制命中次数又会产生过重遮蔽。
- **解决方案**：将叶片三角形 **按其平均不透明度进行缩放**（不透明部分保留，全透明部分收缩甚至剔除）。追踪效率比 Any-Hit 提升约 30%，漫反射 GI 结果与参考非常接近。该策略对镜面反射追踪同样能提供高质量近似。

---

## 十一、天气渲染与动态材质系统

> 主题：介绍由 Atmos 大气仿真系统驱动的全过程动态天气表现，以及湿润、积雪等动态材质效果的渲染技术细节。

### 11.1 数据驱动的大气模拟
- 通过 **Atmos 模拟系统** 在玩家周围运行时低分辨率体素网格中模拟并传播水汽、温度、湿度等大气因子。
- 计算出的数据驱动 **体积云系统**（控制云的生成与消散）以及风场和降雨。所有逻辑由技术美术驱动的 **Amiens 图形系统** 可视化编排。

### 11.2 动态湿润与渲染
- 根据 **湿润度** 动态调整材质：暗化漫反射、降低粗糙度。湿润遮罩仅为一个单浮点数，存储在 G-Buffer 中。

### 11.3 动态积雪系统
- **核心原理**：与动态湿润系统一致，引入冷区与暖区概念，由静态/动态积雪遮罩控制。
- **材质修改流程**：根据雪量将 Albedo 向雪 Albedo 渐变、调整粗糙度、降低透射与次表面散射、利用微可见度遮罩，并基于法线方向进行阈值处理（确保雪不堆积在垂直表面）。
- **完整生命周期**：降雪结束后，积雪缓慢衰减，转化为湿度与水坑，最终融化并变干。

---

## 十二、工程化实践与未来展望

> 主题：汇总平台管理器、资源调试、性能遥测等工程化支撑系统，并总结项目里程碑意义与未来技术演进方向。

### 12.1 平台管理器与可扩展性
为系统化管理不同图形模式带来的巨大差异，开发 **平台管理器**。通过数据驱动配置按平台及上下文指定性能设置，支持实时编辑。提供 Profile 设置与启动设置两大类，并支持通过 Air Context（如拍照模式）和 Modifiers（如洞穴场景禁用开放世界系统）进行精细微调。

### 12.2 资源追踪与性能监控
- **两级内部工具**：高层视图追踪资源生命周期与帧内存峰值；低层工具深入检查分配器、调试内存浪费与别名模式。
- **广泛植入遥测**：所有性能计数器存入数据库，用于追踪性能回归，如动态分辨率因子分布、不同构建版本间的 GPU 性能变化。

### 12.3 项目总结与未来方向
- **里程碑意义**：首款搭载 Monorepo 与共享引擎并交付的游戏，史上规模最大、Anvil 引擎可扩展性最强的刺客信条作品。
- **未来改进**：
  - 更广泛地使用微多边形，提升几何细节。
  - 简化 GI 管线以减轻开发者负担。
  - 推进 **帧模块化** 与 **定制化**，探索 **数据驱动的帧图调度** 以应对多样化需求，但团队对此持谨慎态度。
  - 重新思考如何应对动态系统组合带来的 QA 复杂度爆炸。

### 12.4 跨平台适配要点
- **Apple 平台**：从 M1 到 M4 性能跨度巨大，低端平台在不破坏核心体验的前提下对非关键特性妥协。
- **多模式调度**：同时支撑 30fps 与 60fps 双帧率目标是巨大挑战，不同模式渲染帧内容差异大，被迫走向定制化调度方案。


---

## 关键截图

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

