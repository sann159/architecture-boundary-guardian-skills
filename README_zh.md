# Architecture Boundary Guardian Skill

**Architecture Boundary Guardian Skill** 是一份面向 AI Coding 的架构边界守护型 Skill，用于帮助 AI 编程工具在创建、扩展、重构项目时，提前识别高风险技术债。

它的目标不是让项目一开始就变复杂，而是在快速开发时保留必要边界，避免项目后期进入难以维护、难以测试、难以回滚的状态。

一句话概括：

```text
早期实现可以简单，但不能没有边界。
```

---

## 解决什么问题

在使用 AI Coding 工具快速开发项目时，经常会出现这类问题：

```text
功能能跑，但代码越来越难改
一个文件不断膨胀，所有逻辑都堆在一起
外部 API 超时导致核心流程失败
报告只能从内存对象读取，服务重启后打不开
缓存没有清理机制，用户看不到最新数据
配置来源分散，前端显示和后端实际生效不一致
同一业务字段在 UI、报告、导出、API 中各维护一套
前端状态混乱，AI 每次改动都容易误伤无关功能
```

这个 Skill 的作用是让 AI Coding 在动手写代码前，先检查项目边界，识别可能引入长期技术债的风险，并给出最小安全实现方案。

---

## 适用场景

适合在以下场景使用：

```text
新项目初始化
长期项目迭代
新增核心模块
新增异步任务
新增最近记录 / 历史记录
新增报告 / 导出 / 下载能力
接入第三方 API / 付费 API / AI API
设计缓存策略
设计配置系统
前端模块化拆分
后端模块化重构
规则库 / 插件化能力建设
项目进入下一阶段迭代规划
```

尤其适合这类项目：

```text
会持续迭代
会不断接入外部服务
会生成报告或导出文件
有任务状态、历史记录、最近分析
前端逐渐从单文件变复杂
使用 AI Agent 持续协助开发
```

---

## 核心原则

### 1. 编码前先识别边界

AI Coding 不能只问：

```text
这个功能能不能实现？
```

还必须问：

```text
这个改动触碰了哪些边界？
这个边界以后会不会继续增长？
这个改动会不会制造未来难以修复的结构性问题？
```

重点检查：

```text
模块边界
任务边界
数据边界
报告 / 导出 / 展示输出边界
外部服务边界
配置边界
缓存边界
前端状态边界
测试边界
```

---

### 2. 核心能力优先，增强能力可选

核心能力必须独立稳定，不能被可选增强能力拖垮。

例如：

```text
核心能力：
本地分析、基础计算、核心报告生成、基础数据处理

增强能力：
AI 总结、第三方 API 查询、付费接口、CDN fallback、远程规则源
```

默认规则：

```text
外部服务失败不得导致核心任务失败。
```

外部服务失败时，优先降级为：

```text
partial
skipped
enhancement_failed
external_failed
```

---

### 3. 长期结果不能只依赖内存

如果系统支持：

```text
任务历史
最近记录
报告查看
报告下载
结果回看
任务删除
```

就必须回答：

```text
任务状态在哪里保存？
结果在哪里保存？
服务重启后还能不能查看？
报告能不能从磁盘、数据库或对象存储回读？
```

不能只依赖：

```text
task.report
内存 map
运行时对象
```

---

### 4. 一个数据契约，多种输出格式

不同项目可能需要不同输出：

```text
Web 页面
API 响应
JSON
Markdown
HTML
PDF
Word
Excel / CSV
邮件正文
审计报告
客户定制模板
第三方系统回传格式
```

正确做法是：

```text
先定义统一结构化数据契约
再根据不同受众和格式渲染输出
```

不要让不同输出格式各自维护一套业务字段。

---

### 5. 缓存是高风险优化

新增缓存前必须明确：

```text
为什么缓存？
缓存什么？
缓存多久？
谁能清理？
如何强制刷新？
如何禁用？
是否影响最新数据？
是否进入报告或导出？
```

默认规则：

```text
没有清理入口的缓存不上线。
没有失效策略的缓存不上线。
影响准确性的缓存默认关闭。
```

---

### 6. 配置必须有统一解析边界

项目可以有多个配置来源：

```text
环境变量
配置文件
数据库配置
前端设置
用户偏好
默认值
```

但业务代码不应该到处直接读取配置。

推荐做法：

```text
业务模块只读取已经解析后的有效配置。
```

---

## 如何使用

本仓库只提供核心 Skill：

```text
SKILL.md
```

使用方式取决于你的 AI Coding 工具。不同工具的 Skill、Rules、Memory、Project Instructions、System Prompt 接入方式不同，你可以根据自己的 Agent 环境进行适配。

通用使用方式：

```text
1. 将 SKILL.md 放入你的 AI Coding 工具支持的 skill / rule / instruction 目录。
2. 或者将 SKILL.md 内容合并进项目级 AI 指令文件。
3. 在让 AI Coding 执行复杂开发任务前，要求它先读取并应用该 Skill。
4. 每次新增核心能力、外部服务、报告、缓存、配置、任务状态时，都让 AI Coding 先做边界扫描。
```

可以直接在任务 Prompt 中加入：

```text
请先阅读并应用 architecture-boundary-guardian-skills/SKILL.md。
在编码前，先输出：
1. 本次改动涉及哪些边界；
2. 可能引入哪些高风险技术债；
3. 最小安全实现方案；
4. 本轮明确不做什么；
5. 需要增加哪些测试或验收项。
然后再开始修改代码。
```

---

## 推荐执行流程

使用该 Skill 时，建议 AI Coding 按以下流程工作：

```text
1. 复述用户目标
2. 识别受影响边界
3. 标记高风险技术债
4. 给出最小安全实现方案
5. 明确本轮不做的内容
6. 定义文件改动范围
7. 编码实现
8. 增加测试或验收检查
9. 总结风险是否被消除
```

---

## 推荐输出格式

当你要求 AI Coding 制定实现方案时，可以让它按这个结构输出：

```markdown
# Architecture Boundary Guardian Assessment

## 1. User Goal

简要描述本次需求。

## 2. Affected Boundaries

- Module boundary:
- Task boundary:
- Data boundary:
- Report / export / presentation output boundary:
- External service boundary:
- Configuration boundary:
- Cache boundary:
- Frontend state boundary:
- Test boundary:

## 3. High-Risk Technical Debt

| Risk | Present? | Explanation | Handling Strategy |
|---|---|---|---|

## 4. Smallest Safe Implementation

说明如何用最小改动完成目标，并避免高风险技术债。

## 5. Explicitly Out of Scope

列出本轮不混入的高风险大改。

## 6. File Change Scope

| File | Purpose |
|---|---|

## 7. Tests and Verification

- Functional verification:
- Architecture boundary verification:
- Regression verification:

## 8. Future Evolution

说明哪些长期优化应该放到后续迭代，而不是混入本轮。
```

---

## 不适合用它做什么

这个 Skill 不是：

```text
具体框架模板
项目脚手架
代码生成器
前端 UI 规范
后端分层强制规范
某个 Agent 的专用适配文件
```

它不会强制项目采用固定目录、固定类名、固定框架或固定工程结构。

它只关注一件事：

```text
在 AI Coding 快速开发时，守住那些一旦失控就很难补救的架构边界。
```

---


## Reference

本项目的组织思路和简洁规则写作风格参考了：

```text
multica-ai/andrej-karpathy-skills
```

参考内容主要包括：

```text
项目级 AI Coding 规则的组织方式
简洁、行动导向的规则写法
将可复用开发准则沉淀为 Skill 的方式
```

本项目用途不同，聚焦于：

```text
架构边界守护
高风险技术债预防
长期项目可维护性
```

本项目不是上游项目的官方扩展，也不代表其维护者背书。

---

## License

本项目使用 MIT License。

如果你复用、修改或分发本项目，请保留许可证声明。
