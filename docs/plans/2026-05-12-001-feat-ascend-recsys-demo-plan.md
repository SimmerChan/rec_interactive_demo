---
title: 昇腾推荐系统可视化演示
type: feat
status: active
date: 2026-05-12
origin: docs/ideation/2026-05-12-ascend-recsys-visualization.md
---

# 昇腾推荐系统可视化演示

## Summary

在昇腾NPU平台上实现一个端到端的推荐系统可视化演示系统。使用MovieLens数据集、HSTU推荐算法、Ascend RecSDK训练框架、原生PyTorch推理引擎，通过Gradio前端和Docker容器化部署，实现用户交互式电影推荐演示。目标在2周内完成从数据处理到可视化展示的完整流程。

---

## Problem Frame

用户需要一个能够展示昇腾平台推荐系统能力的可视化演示系统。该系统需要：
- 快速搭建并在演示当天稳定运行
- 体现昇腾NPU的推理加速能力
- 让观众直观理解推荐系统的工作原理
- 支持用户交互式体验推荐效果

**约束条件**:
- 时间紧（2周内完成）
- 演示优先于算法精度
- 简单快速的实现方案
- 需要有降级策略保证演示稳定性

---

## Requirements

- R1. 支持用户选择电影并获取推荐结果
- R2. 推荐结果包含电影名称、评分、推荐理由等基本信息
- R3. 推荐系统能够在昇腾NPU上完成模型推理
- R4. 提供Docker一键部署能力
- R5. 当模型推理不可用时，自动降级到基于规则的推荐
- R6. 整个流程在2周内可完成端到端实现

**Origin actors:** 数据科学家（演示观众）、开发工程师（系统维护者）
**Origin flows:** 数据加载 → 模型训练 → 推理服务 → API调用 → 前端展示
**Origin acceptance examples:** 用户选择电影后3秒内返回推荐结果；Docker环境下一键启动所有服务

---

## Scope Boundaries

- 不追求推荐算法的最高精度，以演示功能完整性为主
- 暂不实现真实的在线学习能力，反馈数据仅用于展示
- 暂不实现多模型对比和A/B测试功能
- 不包含用户管理系统，演示环境为单用户模式

### Deferred to Follow-Up Work

- 嵌入空间可视化（方案2）：后续迭代中加入UMAP/t-SNE可视化
- NPU量化对比（方案5）：后续版本加入CPU vs NPU性能对比展示
- 实时反馈更新：后续版本实现模拟的在线学习效果

---

## Context & Research

### Relevant Code and Patterns

- **recsys-examples (NVIDIA)**: HSTU算法和DynamicEmb实现的参考
- **Ascend RecSDK**: 昇腾推荐系统SDK，提供训练和推理的基础API
- **fbgemm-ascend**: FBGEMM量化库昇腾适配，支持INT8/FP8推理
- **HierarchicalKV-ascend**: 分层KV缓存优化（可选集成）

### External References

- MovieLens数据集: https://grouplens.org/datasets/movielens/
- Gradio文档: https://gradio.app/docs/
- FastAPI文档: https://fastapi.tiangolo.com/
- Ascend PyTorch后端: CANN (Compute Architecture for Neural Networks)

---

## Key Technical Decisions

- **数据集**: MovieLens 1M - 公开可用，有丰富的预处理参考代码
- **推荐算法**: 使用简化版HSTU或类似结构（避免完整HSTU的复杂性）
- **训练方式**: 单节点训练，不使用分布式训练以简化流程
- **推理引擎**: 原生PyTorch + Ascend NPU后端（直接加载模型推理）
- **API服务**: FastAPI（轻量、Python原生、自动文档）
- **前端**: Gradio（推荐系统专用组件、快速原型）
- **部署**: Docker + docker-compose（开箱即用）
- **降级策略**: 基于规则的热门电影推荐作为兜底

---

## Open Questions

### Resolved During Planning

- **Q: 是否需要预训练模型？** A: 是，先训练好模型，演示时只做推理，加快演示启动速度
- **Q: 如何处理新用户冷启动？** A: 使用热门电影+分类均衡的规则推荐作为冷启动方案
- **Q: 是否需要实时推理？** A: 是，但实现预计算缓存作为加速层

### Deferred to Implementation

- **模型训练的超参数**: 需要根据实际硬件配置调整
- **推荐结果数量**: 需要根据UI效果测试确定（默认Top-10）
- **嵌入维度**: 根据模型大小和推理速度权衡

---

## Output Structure

```
rec_interactive_demo/
├── data/                      # 数据目录
│   ├── raw/                   # 原始数据
│   │   └── ml-1m/
│   ├── processed/              # 处理后数据
│   │   ├── train/             # 训练数据
│   │   └── test/              # 测试数据
│   └── models/                # 保存的模型
├── src/                       # 源代码
│   ├── data/                  # 数据处理
│   │   ├── __init__.py
│   │   ├── download.py        # 数据下载
│   │   ├── preprocess.py      # 数据预处理
│   │   └── movielens.py       # MovieLens数据集类
│   ├── model/                 # 模型定义
│   │   ├── __init__.py
│   │   ├── hstu.py           # HSTU模型简化版
│   │   └── embedding.py       # 嵌入层
│   ├── training/              # 训练脚本
│   │   ├── __init__.py
│   │   ├── trainer.py         # 训练器
│   │   └── train.py           # 训练入口
│   ├── inference/             # 推理服务
│   │   ├── __init__.py
│   │   ├── predictor.py       # 预测器
│   │   └── service.py         # 推理服务
│   ├── api/                  # API服务
│   │   ├── __init__.py
│   │   ├── routes.py          # API路由
│   │   └── main.py            # FastAPI入口
│   └── ui/                    # 前端UI
│       ├── __init__.py
│       └── gradio_app.py      # Gradio应用
├── tests/                     # 测试
│   ├── __init__.py
│   ├── test_data.py
│   ├── test_model.py
│   ├── test_inference.py
│   └── test_api.py
├── docker/                    # Docker配置
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── entrypoint.sh
├── requirements.txt
├── README.md
└── .gitignore
```

---

## High-Level Technical Design

> *This illustrates the intended approach and is directional guidance for review, not implementation specification.*

```
┌─────────────────────────────────────────────────────────────────┐
│                        Docker Container                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐  │
│  │  Gradio UI  │───▶│  FastAPI   │───▶│  PyTorch Inference  │  │
│  │  (Port 7860)│    │ (Port 8000)│    │  + Ascend NPU      │  │
│  └─────────────┘    └─────────────┘    └─────────────────────┘  │
│                           │                                      │
│                           ▼                                      │
│                    ┌─────────────┐                               │
│                    │  Fallback   │                               │
│                    │   Rules     │                               │
│                    └─────────────┘                               │
└─────────────────────────────────────────────────────────────────┘
```

**数据流**:
1. 用户在Gradio界面选择电影
2. 请求发送到FastAPI `/recommend` 端点
3. FastAPI调用PyTorch推理服务
4. 推理服务加载模型，输入用户历史，输出Top-K推荐
5. 如果推理失败或超时，触发降级规则
6. 返回推荐结果给Gradio展示

**模型训练流程** (提前完成，演示时只加载):
1. 下载MovieLens 1M数据
2. 预处理：用户-电影交互矩阵、特征工程
3. 定义简化版HSTU模型
4. 使用Ascend NPU进行训练
5. 保存模型到 `data/models/`

---

## Implementation Units

- U1. **项目基础结构搭建**

**Goal:** 创建项目目录结构、配置文件和依赖管理

**Requirements:** R6

**Dependencies:** None

**Files:**
- Create: `requirements.txt`
- Create: `.gitignore`
- Create: `src/__init__.py`
- Create: `src/data/__init__.py`
- Create: `src/model/__init__.py`
- Create: `src/training/__init__.py`
- Create: `src/inference/__init__.py`
- Create: `src/api/__init__.py`
- Create: `src/ui/__init__.py`
- Create: `tests/__init__.py`
- Create: `data/raw/.gitkeep`
- Create: `data/processed/train/.gitkeep`
- Create: `data/processed/test/.gitkeep`
- Create: `data/models/.gitkeep`

**Approach:**
- 使用标准Python项目结构
- requirements.txt包含所有依赖：torch, fastapi, gradio, uvicorn, pandas, numpy, scikit-learn
- 昇腾相关依赖通过Ascend驱动提供，不在requirements.txt中

**Patterns to follow:**
- Python包的标准 `__init__.py` 结构
- 数据目录和代码目录分离

**Test scenarios:**
- Happy path: 所有目录创建成功，Python包可导入
- Edge case: 重复创建目录不会报错

**Verification:**
- `python -c "import src; import src.data; import src.model"` 无报错
- 项目结构符合Output Structure定义

---

- U2. **数据下载和预处理**

**Goal:** 下载MovieLens数据集并预处理为模型训练格式

**Requirements:** R1, R6

**Dependencies:** U1

**Files:**
- Create: `src/data/download.py`
- Create: `src/data/preprocess.py`
- Create: `src/data/movielens.py`
- Create: `data/raw/ml-1m/` (运行时)
- Create: `data/processed/train/train.csv`
- Create: `data/processed/test/test.csv`

**Approach:**
- download.py: 使用pandas下载并解压MovieLens 1M数据集
- preprocess.py: 将原始数据转换为训练格式（用户ID、电影ID、评分、特征）
- movielens.py: Dataset类封装，支持PyTorch DataLoader

**Patterns to follow:**
- torch.utils.data.Dataset接口
- pandas处理结构化数据

**Test scenarios:**
- Happy path: 成功下载并解析MovieLens数据
- Edge case: 网络下载失败时有合理的错误提示
- Edge case: 数据格式解析错误时的容错处理

**Verification:**
- `python -c "from src.data.movielens import MovieLensDataset; ds = MovieLensDataset('./data/processed'); print(len(ds))"` 输出数据集大小

---

- U3. **推荐模型定义**

**Goal:** 定义简化的HSTU推荐模型结构

**Requirements:** R3

**Dependencies:** U2

**Files:**
- Create: `src/model/embedding.py`
- Create: `src/model/hstu.py`
- Modify: `src/model/__init__.py`

**Approach:**
- embedding.py: 电影嵌入层和用户嵌入层，支持PyTorch内置
- hstu.py: 简化版HSTU模型，包含：
  - 嵌入层（用户特征、电影特征）
  - 多头注意力层（简化版）
  - 前馈网络
  - 输出层（评分预测）

**Patterns to follow:**
- torch.nn.Module接口
- recsys-examples中的HSTU架构参考

**Test scenarios:**
- Happy path: 模型可以正常初始化和前向传播
- Edge case: batch维度不匹配时的错误处理
- Error path: 模型输入维度错误时的异常

**Verification:**
- `python -c "from src.model.hstu import SimpleHSTU; model = SimpleHSTU(6040, 3706, 64); x = model(torch.randint(0, 100, (32, 10)), torch.randint(0, 100, (32, 10))); print(x.shape)"` 输出 `(32, 1)`

---

- U4. **模型训练脚本**

**Goal:** 实现模型训练流程，支持昇腾NPU训练

**Requirements:** R3, R6

**Dependencies:** U3

**Files:**
- Create: `src/training/trainer.py`
- Create: `src/training/train.py`
- Create: `data/models/model.pt` (运行时生成)
- Create: `data/models/model_config.json` (运行时生成)

**Approach:**
- trainer.py: 训练器类，包含：
  - 优化器配置（Adam）
  - 训练循环
  - 验证循环
  - 模型保存/加载
- train.py: 训练入口脚本
  - 参数：数据路径、模型保存路径、超参数
  - 支持Ascend NPU设备（通过torch.npu设置）
  - 训练完成后保存模型和配置

**Patterns to follow:**
- torch训练循环标准模式
- 模型检查点保存

**Test scenarios:**
- Happy path: 模型可以正常训练并保存
- Edge case: 训练中断后可以恢复
- Error path: GPU/NPU内存不足时的处理

**Verification:**
- 训练脚本运行完成后，`data/models/` 目录下存在 `model.pt` 和 `model_config.json`

---

- U5. **推理服务**

**Goal:** 实现PyTorch模型推理服务，支持昇腾NPU后端

**Requirements:** R3, R5

**Dependencies:** U4

**Files:**
- Create: `src/inference/predictor.py`
- Create: `src/inference/service.py`
- Modify: `src/inference/__init__.py`

**Approach:**
- predictor.py: 预测器类
  - 加载训练好的模型
  - 支持批量推理
  - 支持Top-K推荐
  - 集成降级规则（当模型不可用时）
- service.py: 推理服务包装
  - 模型预热
  - 请求队列（可选）
  - 性能指标收集

**Patterns to follow:**
- 轻量级推理服务封装
- 降级策略模式

**Test scenarios:**
- Happy path: 模型可以正常加载并推理
- Edge case: 模型文件不存在时的处理
- Edge case: 推理超时时的降级
- Error path: 模型推理失败的错误处理

**Verification:**
- `python -c "from src.inference.predictor import Recommender; r = Recommender('./data/models'); print(r.recommend([1,2,3], k=5))"` 返回推荐结果

---

- U6. **FastAPI推荐服务**

**Goal:** 实现RESTful API服务，暴露推荐接口

**Requirements:** R2, R5

**Dependencies:** U5

**Files:**
- Create: `src/api/routes.py`
- Create: `src/api/main.py`
- Modify: `src/api/__init__.py`

**Approach:**
- routes.py: API路由定义
  - `GET /health`: 健康检查
  - `POST /recommend`: 推荐接口
    - 输入: `{"user_id": int, "movie_ids": [int], "k": int}`
    - 输出: `{"recommendations": [{"movie_id": int, "score": float, "title": str}], "source": "model|fallback"}`
  - `GET /movies/{movie_id}`: 电影信息查询
- main.py: FastAPI应用入口
  - CORS配置
  - 依赖注入推理服务
  - 启动命令: `uvicorn src.api.main:app --host 0.0.0.0 --port 8000`

**Patterns to follow:**
- FastAPI最佳实践
- Pydantic数据验证

**Test scenarios:**
- Happy path: API正常启动并响应请求
- Edge case: 无效输入参数的处理
- Edge case: 推理超时降级到规则推荐
- Error path: 推理服务不可用时的错误返回

**Verification:**
- `curl http://localhost:8000/health` 返回 `{"status": "ok"}`
- `curl -X POST http://localhost:8000/recommend -H "Content-Type: application/json" -d '{"user_id": 1, "movie_ids": [1,2,3], "k": 5}'` 返回推荐结果

---

- U7. **Gradio前端界面**

**Goal:** 实现交互式Web界面，展示推荐效果

**Requirements:** R1, R2

**Dependencies:** U6

**Files:**
- Create: `src/ui/gradio_app.py`
- Modify: `src/ui/__init__.py`

**Approach:**
- gradio_app.py: Gradio应用定义
  - 电影选择组件（Dropdown或类似）
  - 推荐结果展示组件（JSON或自定义卡片）
  - 用户历史展示
  - 反馈按钮（喜欢/不喜欢 - 仅展示用）
  - 启动命令: `python -m src.ui.gradio_app`
- 界面布局:
  - 左侧：用户电影选择和历史
  - 右侧：推荐结果展示
  - 底部：性能指标（可选）

**Patterns to follow:**
- Gradio Blocks API自定义布局
- 重用API服务客户端

**Test scenarios:**
- Happy path: Gradio界面正常启动
- Edge case: API连接失败时的提示
- Edge case: 推荐结果为空时的展示

**Verification:**
- Gradio应用在 http://localhost:7860 正常访问
- 选择电影后可以获取推荐结果

---

- U8. **Docker容器化部署**

**Goal:** 实现一键部署，支持Docker环境下的完整演示

**Requirements:** R4, R6

**Dependencies:** U7

**Files:**
- Create: `docker/Dockerfile`
- Create: `docker/docker-compose.yml`
- Create: `docker/entrypoint.sh`
- Modify: `requirements.txt` (添加gunicorn等)

**Approach:**
- Dockerfile: 基于Python 3.10镜像
  - 安装系统依赖
  - 复制代码
  - 安装Python依赖
  - 设置Ascend环境变量
  - 暴露端口 7860 (Gradio) 和 8000 (API)
- docker-compose.yml:
  - 定义服务：api (FastAPI) 和 ui (Gradio)
  - 端口映射
  - 依赖关系（ui依赖api）
  - Ascend设备访问
- entrypoint.sh:
  - 检查Ascend驱动
  - 启动API服务
  - 启动Gradio服务

**Patterns to follow:**
- Docker最佳实践
- docker-compose多服务编排

**Test scenarios:**
- Happy path: `docker-compose up` 成功启动所有服务
- Edge case: Ascend驱动不可用时的处理
- Edge case: 端口冲突时的错误提示

**Verification:**
- `docker-compose up -d` 成功构建并启动
- 访问 http://localhost:7860 可以看到Gradio界面
- API健康检查通过

---

- U9. **集成测试**

**Goal:** 验证端到端流程的正确性

**Requirements:** R1, R2, R3, R4, R5

**Dependencies:** U8

**Files:**
- Create: `tests/test_data.py`
- Create: `tests/test_model.py`
- Create: `tests/test_inference.py`
- Create: `tests/test_api.py`

**Approach:**
- test_data.py: 数据处理测试
  - MovieLens数据下载和解析
  - 数据预处理正确性
- test_model.py: 模型测试
  - 模型前向传播
  - 梯度计算
- test_inference.py: 推理测试
  - 模型加载
  - 推荐结果生成
  - 降级规则触发
- test_api.py: API测试
  - 健康检查
  - 推荐接口
  - 错误处理

**Patterns to follow:**
- pytest测试框架
- fixture复用

**Test scenarios:**
- Happy path: 所有测试通过
- Edge case: 部分模块独立运行的能力
- Error path: 异常输入的处理

**Verification:**
- `pytest tests/ -v` 所有测试通过

---

## System-Wide Impact

- **Interaction graph**:
  - Gradio UI → FastAPI → PyTorch Inference → Ascend NPU
  - FastAPI → Fallback Rules (when inference fails)
- **Error propagation**:
  - 推理错误 → FastAPI返回503 → Gradio显示降级结果
  - API超时 → 自动降级到规则推荐
- **State lifecycle risks**:
  - 模型只需加载一次，保持在内存中
  - 无持久化状态，演示环境无状态
- **Integration coverage**:
  - 需要端到端测试验证各组件协作

---

## Risks & Dependencies

| Risk | Mitigation |
|------|------------|
| Ascend NPU驱动安装复杂性 | 预先在Docker镜像中集成Ascend环境 |
| 模型训练时间不可控 | 限制训练epoch数量，设置早停 |
| 推理延迟影响用户体验 | 预计算缓存 + 降级规则 |
| MovieLens数据下载失败 | 提供备选数据源或本地缓存 |
| Docker环境与本地环境差异 | 提供本地开发模式 |

---

## Documentation / Operational Notes

- **README.md**: 项目说明、快速启动指南、Docker部署说明
- **环境要求**: Ascend CANN驱动、Python 3.10+、Docker 20.10+
- **启动顺序**: Docker启动 → API就绪 → Gradio就绪
- **监控指标**: API响应时间、推理延迟、推荐结果数量

---

## Sources & References

- **Origin document:** [docs/ideation/2026-05-12-ascend-recsys-visualization.md](../ideation/2026-05-12-ascend-recsys-visualization.md)
- MovieLens数据集: https://grouplens.org/datasets/movielens/
- NVIDIA recsys-examples: https://github.com/NVIDIA/recsys-examples
- Ascend RecSDK: https://gitcode.com/Ascend/RecSDK
- fbgemm-ascend: https://gitcode.com/Ascend/fbgemm-ascend
- Gradio: https://gradio.app/
- FastAPI: https://fastapi.tiangolo.com/
