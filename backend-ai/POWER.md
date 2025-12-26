---
name: backend-ai
description: "{MODULE} Platform 代码生成 Power"
keywords: ["java", "spring", "mybatis", "dubbo", "finance", "dao", "service", "controller"]
---

# 后端代码生成 Power

## 概述

专为 dahuangf 后端项目设计的代码生成 Power，支持从需求分析到代码生成的完整开发流程。

**支持的项目**：
- finance-platform（财务平台）
- sale-platform（销售平台）

**核心能力**：
- 需求预处理：将钉钉聊天、口头描述等非标需求转换为结构化需求
- 架构设计：生成用例驱动的架构需求文档
- 技术方案：生成详细的技术方案文档（含 ER 图、时序图、UML 类图）
- 代码生成：按分层架构生成 DAO → Core → Interfaces → API 代码
- API 文档：自动生成接口文档

---

## 项目变量定义（重要）

根据当前工作目录自动识别项目，或由用户指定：

| 变量 | finance-platform | sale-platform |
|------|------------------|---------------|
| `{PROJECT}` | finance-platform | sale-platform |
| `{MODULE}` | finance | sale |
| `{MODULE}-dao` | finance-dao | sale-dao |
| `{MODULE}-core` | finance-core | sale-core |
| `{MODULE}-interfaces` | finance-interfaces | sale-platform-interfaces |
| `{MODULE}-api` | finance-api | sale-api |
| `{MODULE}-common` | finance-common | sale-common |
| `{BASE_PACKAGE}` | com.dahuangf.finance | com.dahuangf.sale |

**识别规则**：
1. 优先根据当前工作目录名称识别（finance-platform 或 sale-platform）
2. 如果无法识别，询问用户当前是哪个项目
3. 用户可以在对话中明确指定：`使用 finance-platform` 或 `使用 sale-platform`

---

## 项目信息

| 项目 | {PROJECT} |
|------|-----------|
| 语言 | Java 17 |
| 架构 | Maven 多模块 |
| 基础包 | `{BASE_PACKAGE}` |
| 技术栈 | Spring Boot + MyBatis-Plus + Dubbo |

---

## 模块架构

```
{PROJECT}/
├── {MODULE}-dao          # 数据访问层（DO、Mapper）
├── {MODULE}-core         # 核心业务层（BO、Service 四层结构）
├── {MODULE}-interfaces   # Web 层（Controller、VO、RPC 实现）
├── {MODULE}-api          # RPC 接口定义（TO、Rpc）
├── {MODULE}-common       # 公共组件
└── {MODULE}-biz-service  # RPC 实现（已迁移至 interfaces）
```

| 模块 | 职责 | 规范文档 |
|------|------|----------|
| {MODULE}-dao | 数据实体、Mapper 接口 | #[[file:steering/dao.md]] |
| {MODULE}-core | 业务逻辑、Service 四层结构 | #[[file:steering/core.md]] |
| {MODULE}-interfaces | REST 接口、VO、RPC 实现 | #[[file:steering/interfaces.md]] |
| {MODULE}-api | RPC 接口定义 | #[[file:steering/api.md]] |
| {MODULE}-common | 通用工具、常量 | #[[file:steering/common.md]] |

---

## Steering 文件说明

### 自动加载（编辑 Java 文件时）

| 文件 | 触发条件 | 用途 |
|------|----------|------|
| general.md | `**/*.java` | 全局规范：命名、DO/BO/TO/VO 一致性、Controller-Service 对应、技术约束 |
| dao.md | `**/finance-dao/**/*.java` | DAO 层：DO 实体、Mapper 接口规范 |
| core.md | `**/finance-core/**/*.java` | Core 层：用例驱动分层设计、Service 四层结构、BO 规范 |
| interfaces.md | `**/finance-interfaces/**/*.java` | Interfaces 层：Controller、VO 规范、验证器注解、API 文档生成 |
| api.md | `**/finance-api/**/*.java` | API 层：RPC 接口、TO 规范 |
| common.md | `**/finance-common/**/*.java` | Common 层：公共组件规范 |

### 手动加载（需求分析/设计阶段）

| 文件 | 用途 | 使用场景 |
|------|------|----------|
| code-generation-flow.md | 完整开发流程指引 | 新功能开发全流程 |
| requirements-preprocessing.md | 非标需求处理 | 钉钉聊天、口头需求、图片分析 |
| requirements-analysis.md | 架构需求分析 | PRD → 架构文档 |
| technical-design.md | 技术方案设计 | 架构 → 技术方案（含 ER 图、UML 类图） |
| test.md | 单元测试生成 | 生成 JUnit 5 + Mockito 测试代码 |

---

## 开发流程

详细流程：#[[file:steering/code-generation-flow.md]]

```
Phase 0: 需求预处理（非标需求 → 结构化需求）
    ↓ 用户确认
Phase 1: 架构需求分析（结构化需求 → 架构需求文档）
    ↓ 用户确认
Phase 2: 技术方案设计（架构需求 → 技术方案文档）
    ↓ 用户确认
Phase 3: 代码生成
    Step 1: DAO 层 → 用户确认
    Step 2: Core 层（用例模型 → 分层拆分 → Service 方法）→ 用户确认
    Step 3: Interfaces 层 → 用户确认
    Step 4: API 层（如需）→ 用户确认
    Step 5: API 文档生成
    Step 6: 单元测试生成（重要）
```

| 阶段 | 输入 | 输出 | 规范文档 |
|------|------|------|----------|
| Phase 0 | 非标需求（聊天/图片/口头） | 结构化需求 | #[[file:steering/requirements-preprocessing.md]] |
| Phase 1 | 结构化需求/PRD | 架构需求文档 | #[[file:steering/requirements-analysis.md]] |
| Phase 2 | 架构需求 | 技术方案文档 | #[[file:steering/technical-design.md]] |
| Phase 3 | 技术方案 | 代码 + API 文档 + 单元测试 | 各层规范文档 + #[[file:steering/test.md]] |

---

## 核心设计理念

### DO/BO/TO/VO 字段一致性（强制）

排除继承父类属性后，DO、BO、TO、VO 的业务字段必须完全一致：

| 对象类型 | 允许的个性化字段 |
|----------|------------------|
| DO | 无（与数据库表字段一一对应） |
| BO | 无（与 DO 字段完全一致） |
| TO | 无（与 BO 字段完全一致） |
| PageReqVO | 时间范围查询字段（createdAtStart/End 等） |
| CreateReqVO | 无（排除 id、审计字段） |
| UpdateReqVO | id（继承 CreateReqVO） |
| RespVO | 审计字段（id、createdAt、createdBy 等） |

### Controller 与 Service 方法对应（强制）

| Controller 方法 | Service 方法 | Service 入参 | Service 出参 |
|-----------------|--------------|--------------|--------------|
| `page` | `page` | `PageParam`, `LambdaQueryWrapper<XxxDO>` | `IPage<XxxBO>` |
| `detail` | `detail` | `Long id` | `XxxBO` |
| `add` | `add` | `XxxCreateReqVO` | `int` |
| `update` | `update` | `XxxUpdateReqVO` | `int` |
| `delete` | `delete` | `Long id` | `int` |

### Core 层用例驱动分层设计

```
{MODULE}-core/src/main/java/{BASE_PACKAGE}/core/{业务域}/
├── bo/                              # 业务对象
├── enums/                           # 业务枚举
├── contracts/                       # 策略接口、扩展点
├── validate/                        # 校验逻辑
├── service/                         # 子模块 Service
│   └── {子模块}Service.java
├── {业务域}Service.java             # 应用层（对外统一入口）
├── {业务域}BaseService.java         # 基础层（通用能力）
└── {业务域}AdapterService.java      # 适配层（外部系统对接）
```

| 层级 | 职责 | 承接内容 |
|------|------|----------|
| 应用层 | 流程编排、对外统一入口 | 用例完整流程 |
| 核心业务层 | 业务规则、状态流转 | 核心用例 |
| 适配层 | API 封装、多平台适配 | 外部对接用例 |
| 基础层 | 校验、工具、规则 | 通用能力 |

**Core 层代码生成前必须完成三次确认**：
1. 用例模型确认（参与者、用例清单、用例关系）
2. 分层拆分确认（各层职责、承接用例）
3. Service 方法确认（目录结构、方法清单、调用关系）

---

## 全局约束（强制遵循）

参考：#[[file:steering/general.md]]

1. **架构文档边界**: 接口入参出参不放架构需求文档，放技术方案文档
2. **图表格式**: 流程图使用 `sequenceDiagram`，状态机使用 `stateDiagram-v2`
3. **代码分层**: Manager 层不写代码，外部调用放 Core 层 Adapter
4. **文档存储**: 文档必须写入 `docs/{功能模块}/` 目录
5. **验证器注解**: message 必须使用中文
6. **VO 字段注释**: 必须添加详细 JavaDoc（用于 API Doc 生成）
7. **API 文档**: Controller 代码生成后必须同步生成 API 文档到 `api/` 目录

---

## 技术栈约束

| 类别 | 规范 |
|------|------|
| 数据转换 | 仅使用 `CopyUtil`（copyBean/copyList/copyPageInfo） |
| 异常处理 | `AssertUtil.ifThrow` 或 `BizException` |
| MyBatis-Plus | 仅使用 `LambdaQueryWrapper`/`LambdaUpdateWrapper`，禁止手写 SQL |
| 空值处理 | `CollectionUtil.isEmpty`、`Objects.nonNull/isNull`、`Optional.ofNullable` |
| 依赖注入 | 使用 `@Resource` |

---

## 快速开始

### 方式一：非标需求（钉钉聊天、口头描述、图片）

直接粘贴需求内容或上传图片，我会：
1. 识别输入类型，提取关键信息（支持图片分析）
2. 整理为结构化需求，等待确认
3. 生成架构需求文档 → 技术方案文档 → 代码 → API 文档

### 方式二：标准 PRD

提供 PRD 文档，跳过 Phase 0，直接从架构需求分析开始。

### 方式三：直接生成代码

如果已有技术方案，可以直接说明需要生成哪一层的代码。

---

**开始**: 请提供你的需求（钉钉聊天、PRD、图片、或直接描述）。如果我无法自动识别当前项目，请告诉我是 finance-platform 还是 sale-platform。
