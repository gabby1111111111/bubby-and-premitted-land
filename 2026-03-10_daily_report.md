---
date: 2026-03-11
author: Gabby
project: Aegis-Isle & Universe-Manager Ecosystem
---

# 📝 2026-03-11 工作日报：从核心 Bug 修复到成功解耦开源

> **今日核心成就 (TL;DR)**：
> 今天完成了从「主生态 RAG 检索异常排查、超时处理」，到「架构重构层面的高阶解耦操作」，最终实现了 **Universe-Manager 独立看版的成型、全自动验收测试与 GitHub 开源上传**。这一整套连招直接为即将到来的 20k 级 AI 应用岗面试准备了极佳的“秀肌肉”筹码。

---

## 📅 1. 疑难 Bug 攻坚：RAG 检索引擎多路召回异常修复
- **背景/症状**：前端界面提示 `API 请求 timeout`，并且明明在此前限制了“仅搜索初次见面宇宙”，系统却在盲目检索全部 78 个多平行宇宙，进而陷入全量暴力扫描导致过载。
- **排障与排查**：
  - 定位到 `api/routers/universe_manager.py` 和 `frontend/universe_manager.py`。
  - 抓出“过滤失效”的元凶：前端传递的目标宇宙 ID 带有时间戳后缀，而后端代码将其与未带时间戳的 FAISS `index` 物理文件名直接对比，导致匹配条件始终为 `False`，进而在后续抛弃了缩小范围的拦截网。
- **修复方案**：
  - 重构了后端的文件过滤条件，使其利用 `any()` 进行准确的子串模糊匹配。
  - 同步放宽了前端的 `httpx` HTTP 客户端的最大等待超时时间 (Timeout)，保证查询长尾文本时界面不会过早崩溃。

## 🏗️ 2. 项目架构改造：剥离并封装独立 `Universe-Manager`
- **思考动机**：随着底层 `Aegis-Isle`（主微服务）内部的记忆流、事件总线、甚至主动开口 Agent 逻辑的日益复杂化，原先为了测试数据质量而在内部写的路由探针，开始和主干代码“互相打架”。
- **实施解耦**：
  - **模块化抽离**：在 `E:\universe_manager` 新建了完全干净的工作区。将主库中负责 FAISS 向量读取的 `core_rag.py` 纯净读取类提取出来。
  - **前后端微服务建立**：单独书写了一个极轻量版的 `api_app.py`（FastAPI 微路由）和 `dashboard.py`（Streamlit 界面）。
  - **环境隔离配置**：配置 `.env.example` 和 `requirements.txt`。让系统只需通过 `AEGIS_DATA_PATH` 指针去跨盘符偷读数据，完全不依赖、不干涉主干工程的业务流转。

## 🚀 3. CI/CD 雏形与开源沉淀 (Open Source Submission)
- **文档的降维打击**：利用中英文双语编写了极具逼格的 `README.md`，并在开篇清晰指出：“该产品是在主导构建了 `Aegis-Isle` 复杂生态后的自然延伸与架构解耦。”从而向面试官表明全栈控制力和模块化架构经验。
- **全自动盲测验证**：利用自动化测试跑通了独立版。涵盖“混合语义重排序 (Hybrid Re-ranking)”防崩溃机制，以及复杂的 HTML 脏数据（剧场体、动作签）探针测试，输出最高级别置信数据。捕获了完整录像并以独立的 `TESTING_REPORT.md` 生成了技术验收报告。
- **发布**：将此剥离系统纯净、无任何真实数据的状态，全套防污染 (`.gitignore`) 地推送到远端个人 GitHub (`gabby1111111111/Universe-Manager`)。

## 📋 明日/后续待办 (P0 接线任务预告)
明天将转回主力场，打通大盘 `ROADMAP.md` 中最关键的两条视神经**数据回流**脉络：
- [ ] **D → B 连网线**：在《Love & Code》的 Streamlit 面试系统后端增加 HTTP POST，将“Gabby 正做了一道题”的动作推给 `Aegis :8001/v1/diary/event` 记录中枢。
- [ ] **E → B 视神经**：完善 Chrome 浏览器 `ST-Companion-Link` 扩展，监听我日常阅读的知乎/文章网址，实时传导给 `Aegis` 成为长时日记的 FAISS 切片拼图。
