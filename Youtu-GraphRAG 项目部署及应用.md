## Youtu-GraphRAG 项目部署及应用

### Ⅰ 成果

以`人教版小学数学四年级上册`课本（PDF格式）为语料：

#### 一、Chunks 文档

[Chunks-数学四年级上册.txt](https://github.com/Xueyouing/youtu-graphrag/blob/main/Chunks-数学四年级上册.txt)

#### 二、Graph 知识图谱

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122125915187.png" alt="image-20260122125915187" style="zoom:50%;" />

#### 三、交互问答

> **小升初应用题设计**
>
> “学校操场旁边有一块形状是平行四边形的空地，底是 250 米，高是 80 米。
>
> 1. 这块空地的面积是多少平方米？
> 2. 合多少公顷？
> 3. 果要在空地边上画一条射线来标志边界，射线和直线有什么区别？”

##### 3.1 回答效果

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122030625599.png" alt="image-20260122030625599" style="zoom:33%;" />

**分析：**回答正确无误。

##### 3.2 推理过程

**终端关键推理过程如下：**

```
# 检索定位：此前，模型对于知识图谱（终端呈现出来的是完整分隔开的书籍）进行了完整检索定位，由于过长未放置。
Previous Thoughts: Initial: 1. 这块空地的面积是20000平方米。
2. 合2公顷。
3. 射线有一个端点，可以向一端无限延伸；直线没有端点，可以向两端无限延伸。
# 判断部分
Instructions:   
1. If enough info answer with: So the answer is: <answer>
2. Else propose new query with: The new query is: <query>
Your reasoning:
# 逻辑推导并回答：判断出，有充足知识点支持完整回答。（但是不确定模型是否进行了多次检索定位才找到，过程太长终端没有显示完全【后续可看日志】）
[2026-01-22 05:11:55] INFO     enhanced_kt_retriever:1741 - Answer: 根据知识上下文，特别是关于平行四边形面积计算和几何概念的内容，我可以 回答这个问题。

1. 平行四边形面积 = 底 × 高 = 250米 × 80米 = 20000平方米。
2. 1公顷 = 10000平方米，所以20000平方米 = 2公顷。
3. 射线有一个端点，可以向一端无限延伸；直线没有端点，可以向两端无限延伸。在空地边上画射线标志边界时，射线从端点开始沿一个方向延伸，适合表示从某点出发的边界方向。

So the answer is:
1. 这块空地的面积是20000平方米。
2. 合2公顷。
3. 射线有一个端点，可以向一端无限延伸；直线没有端点，可以向两端无限延伸。在空地边上画射线标志边界时，射线从端点开始沿一个方向延伸，适合表示从某点出发的边界方向。
[2026-01-22 05:11:55] INFO     faiss_filter:596 - Saved embedding cache with 242 entries to retriever/faiss_cache_new/义务教育教科书__数 学四年级上册/node_embedding_cache.pt (size: 440014 bytes)   # 转化成向量并保存，下次更快
```

**分析（代码中存在对应注释）：**模型拿到问题后，大致思维链：首先对于每个拆分后的子问题，在生成的知识图谱进行**检索**，**定位**到问题中的关键知识点；**整合判断**是否有充足信息满足回答问题的需求：若充足（即本题情况），进行**逻辑推导并回答**，若不充足，尝试新的查询，再次定位检索推导回答。

##### 3.3 详细检索过程

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122030902317.png" alt="image-20260122030902317" style="zoom:33%;" />

**分析：**模型速度比较快。思维链：首先，将问题分成三个子问题（并不是因为该题目有三个小题，**经多个题目测试**，满足复杂逻辑问题推导需求，如下图，可以层层拆解）；随后对每个子问题检索元组（Triples）及关联的文本块（Chunks）；整合判断，推导并组成题目回答。

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122135401888.png" alt="image-20260122135401888" style="zoom: 50%;" />

**该题依赖的知识图谱如下：**

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122030931748.png" alt="image-20260122030931748" style="zoom:33%;" />

### Ⅱ 环境

据官方文档，采用 Docker 及 Conda **两种方式**进行部署：

1. Windows系统下，基于WSL安装的**Docker**，执行dockerfile。【本地】

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122142112562.png" alt="image-20260122142112562" style="zoom: 50%;" />

2. Github Codespaces（基于Linux系统）下，创建**Conda**环境，执行start_env.sh。【云服务器】

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122142436576.png" alt="image-20260122142436576" style="zoom:50%;" />

**通用配置**

大模型API：LLM_MODEL=deepseek-ai/DeepSeek-V3.2          LLM_BASE_URL=https://api.siliconflow.cn/v1

### Ⅲ 实验

经多次实验（包括基础参数配置），上传 **JSON、Word、PDF（含文本层）、Markdown、Txt** 格式文档均能正常工作（生成Chunks文档、Graph、问答）。

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122145514603.png" alt="image-20260122145514603" style="zoom:50%;" />



### Ⅳ 问题

#### 一、发现

上传PDF发现无法识别，报错。

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122011344986.png" alt="image-20260122011344986" style="zoom: 67%;" />

#### 二、探究

<u>明确</u>：主要问题就是PDF识别问题。

<u>思考</u>：视频发布时提到仅支持JSON格式，返回GitHub项目查找PDF识别相关信息。

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122151705291.png" alt="image-20260122151705291" style="zoom:50%;" />

回看代码相关部分：**文档解析工具类 (`DocumentParser`)**，主要用于从 **PDF**、**DOCX/DOC** 和 **RTF** 文件中提取纯文本。

识别是使用**多级降级策略**：

**Priority 1: MinerU (`magic-pdf`)**——高质量 PDF 解析，代码通过 `PymuDocDataset` 加载 PDF 字节流进行处理。**【纯图 / 复杂 / 公式 等等都行——视觉驱动（YOLO+OCR）】**

**Priority 2: PyMuPDF (`fitz`)**——回退到 `fitz`，非常快且轻量级的 PDF 文本提取库。**【只能读文本层】**

**Priority 3: pypdf**——最后的保底方案，使用纯 Python 的 `pypdf` 库进行提取。**【简单的纯文字 PDF】**

<u>探明</u>：可能是考虑到模型部署的体量和复杂度问题，用于PDF文件识别最关键的MinerU技术（工具链），其中的**核心部件YOLO依赖库ultralytics未在部署时安装**，所以自动降级仅能识别文本层的PDF。

#### 三、解决

方法：安装，可能需要安装相关的一系列包。

已经成功安装了 `ultralytics` 库，但由于网络原因，其中的权重文件无法下载。

<img src="C:\Users\辛上有木\AppData\Roaming\Typora\typora-user-images\image-20260122152552425.png" alt="image-20260122152552425" style="zoom:50%;" />

服务器为云服务器，可手动下载安装（时间原因暂不进行）。

### Ⅴ 附录

**项目：**https://github.com/Xueyouing/youtu-graphrag

**b站讲解**：https://www.bilibili.com/video/BV1KmnpzGEGp/

**AI使用：**Gemini；Claude；豆包——**本文全文手工撰写**。





