# Backend AI Power

专为 dahuangf 后端项目设计的代码生成 Power，支持从需求分析到代码生成的完整开发流程。

## 支持的项目

- finance-platform（财务平台）
- sale-platform（销售平台）

## 核心能力

- 需求预处理：将钉钉聊天、口头描述等非标需求转换为结构化需求
- 架构设计：生成用例驱动的架构需求文档
- 技术方案：生成详细的技术方案文档（含 ER 图、时序图、UML 类图）
- 代码生成：按分层架构生成 DAO → Core → Interfaces → API 代码
- API 文档：自动生成接口文档

## 安装

将此文件夹复制到 `.kiro/powers/` 目录下即可。

## 使用方式

### 方式一：非标需求
直接粘贴钉钉聊天记录、口头描述或上传图片，Power 会自动处理。

### 方式二：标准 PRD
提供 PRD 文档，直接从架构需求分析开始。

### 方式三：直接生成代码
如果已有技术方案，可以直接说明需要生成哪一层的代码。

## Steering 文件说明

| 文件 | 加载模式 | 用途 |
|------|----------|------|
| general.md | fileMatch (`**/*.java`) | 全局规范 |
| dao.md | fileMatch (`**/*-dao/**/*.java`) | DAO 层规范 |
| core.md | fileMatch (`**/*-core/**/*.java`) | Core 层规范 |
| interfaces.md | fileMatch (`**/*-interfaces/**/*.java`) | Interfaces 层规范 |
| api.md | fileMatch (`**/*-api/**/*.java`) | API 层规范 |
| common.md | fileMatch (`**/*-common/**/*.java`) | Common 层规范 |
| code-generation-flow.md | manual | 完整开发流程 |
| requirements-preprocessing.md | manual | 需求预处理 |
| requirements-analysis.md | manual | 架构需求分析 |
| technical-design.md | manual | 技术方案设计 |
| test.md | manual | 单元测试生成 |

## 技术栈

- Java 17
- Spring Boot + MyBatis-Plus + Dubbo
- Maven 多模块架构
