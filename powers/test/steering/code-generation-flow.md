---
inclusion: manual
---

# 代码生成流程

## 执行顺序（必须严格遵循）

```
Step 1: DAO 层（数据访问层）
    ↓ 用户确认
Step 2: Core 层（核心业务层）
    ↓ 用户确认
Step 3: Interfaces 层（Web 层/Controller）
    ↓ 用户确认
Step 4: API 层（对外接口定义，如需）
```

---

## Step 1: DAO 层

参考规范：#[[file:dao.md]]

**执行内容**:
1. 需求分析与表结构设计
2. 等待用户确认表结构
3. 生成 DO、Mapper、XML

**完成后询问**: "DAO 层代码已生成，是否确认继续生成 Core 层？"

---

## Step 2: Core 层

参考规范：#[[file:core.md]]

**执行内容**:
1. Service 分层设计
2. 等待用户确认分层设计
3. 生成 BO、Enum、Service

**完成后询问**: "Core 层代码已生成，是否确认继续生成 Interfaces 层？"

---

## Step 3: Interfaces 层

参考规范：#[[file:interfaces.md]]

**执行内容**:
1. Controller 接口设计
2. 等待用户确认接口设计
3. 生成 VO、Controller

**完成后询问**: "Interfaces 层代码已生成，是否需要生成 API 层？"

---

## Step 4: API 层（可选）

参考规范：#[[file:api.md]]

**执行条件**: 仅当需要对外提供 RPC 能力时执行

**执行内容**:
1. RPC 接口设计
2. 等待用户确认
3. 生成 TO、RPC 接口

**完成后**: "所有代码生成完毕！"
