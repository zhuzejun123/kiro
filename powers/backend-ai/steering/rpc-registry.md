---
inclusion: always
---

# 二方 RPC 接口注册表

本文件提供二方 RPC 服务的全路径包名和接口定义，用于 Core 层 Adapter 代码生成时的依赖注入和方法调用。

---

## 一、使用说明

### 1.1 文件用途

- 记录已确认的二方 RPC 接口全路径和方法签名
- 作为 Adapter 层代码生成的参考依据
- 减少重复确认，提高代码生成效率

### 1.2 代码生成流程

生成 Core 层 Adapter 代码时，必须遵循以下流程：

```
1. 检查本文件是否已有对应 RPC 接口定义
   ├── 有 → 直接使用已定义的接口和方法
   └── 无 → 进入步骤 2

2. 向用户确认 RPC 接口信息
   - 列出所有需要调用的二方 RPC 服务
   - 询问用户是否能提供接口全路径和方法签名
   ├── 用户提供 → 记录到本文件，生成完整代码
   └── 用户无法提供 → 生成 TODO 占位代码

3. TODO 占位代码格式
   // TODO 待确认：{服务名称} RPC 接口
   // 预期功能：{功能描述}
   // 预期入参：{参数说明}
   // 预期出参：{返回值说明}
   // @Resource
   // private XxxRpcService xxxRpcService;
```

### 1.3 确认问题模板

生成 Adapter 代码前，使用以下模板向用户确认：

```markdown
## 二方 RPC 接口确认

在生成 Adapter 层代码时，发现以下二方 RPC 调用需要确认：

| 序号 | 服务名称 | 预期功能 | 状态 |
|------|----------|----------|------|
| 1 | 审批服务 | 创建审批流程、查询审批状态 | 待确认 |
| 2 | 客户服务 | 查询客户信息 | 待确认 |
| 3 | 仓库服务 | 查询仓库列表、仓库详情 | 待确认 |

请提供以下信息（如无法提供则留空，将生成 TODO 占位）：
1. RPC 接口全路径（如 `com.xxx.api.ApprovalRpcService`）
2. 方法签名（方法名、入参类型、返回类型）
3. 关键字段说明
```

---

## 二、已注册的 RPC 接口

### 2.1 审批服务（Approval）

**状态**: 待补充

```java
// TODO 待用户提供接口定义
// 包路径: 
// 接口名: 
// 方法:
//   - 创建审批: 
//   - 查询状态: 
//   - 撤回审批: 
```

### 2.2 客户服务（Customer）

**状态**: 待补充

```java
// TODO 待用户提供接口定义
// 包路径: 
// 接口名: 
// 方法:
//   - 查询客户列表: 
//   - 查询客户详情: 
```

### 2.3 仓库服务（Warehouse）

**状态**: 待补充

```java
// TODO 待用户提供接口定义
// 包路径: 
// 接口名: 
// 方法:
//   - 查询仓库列表: 
//   - 查询仓库详情: 
```

### 2.4 库存服务（Stock）

**状态**: 待补充

```java
// TODO 待用户提供接口定义
// 包路径: 
// 接口名: 
// 方法:
//   - 查询库存信息: 
```

### 2.5 运费服务（Freight）

**状态**: 待补充

```java
// TODO 待用户提供接口定义
// 包路径: 
// 接口名: 
// 方法:
//   - 计算运费: 
```

### 2.6 文档中心服务（DocumentCenter）

**状态**: 待补充

```java
// TODO 待用户提供接口定义
// 包路径: 
// 接口名: 
// 方法:
//   - 生成 PDF: 
//   - 获取 PDF 地址: 
//   - PDF 盖章: 
```

### 2.7 价格管控服务（PriceControl）

**状态**: 待补充

```java
// TODO 待用户提供接口定义
// 包路径: 
// 接口名: 
// 方法:
//   - 校验月租价: 
//   - 查询标准价: 
```

### 2.8 字典服务（Dict）

**状态**: 待补充

```java
// TODO 待用户提供接口定义
// 包路径: 
// 接口名: 
// 方法:
//   - 查询字典项: 
```

---

## 三、接口定义模板

用户提供接口信息后，按以下格式记录：

```markdown
### X.X 服务名称（英文名）

**状态**: ✅ 已确认

**包路径**: `com.xxx.api`

**接口名**: `XxxRpcService`

**Maven 依赖**:
```xml
<dependency>
    <groupId>com.xxx</groupId>
    <artifactId>xxx-api</artifactId>
    <version>${xxx.version}</version>
</dependency>
```

**方法定义**:

| 方法名 | 入参 | 出参 | 说明 |
|--------|------|------|------|
| `createXxx` | `XxxCreateReq req` | `Result<Long>` | 创建 XXX |
| `getXxxById` | `Long id` | `Result<XxxDTO>` | 查询 XXX 详情 |

**关键 DTO 字段**:

```java
// XxxDTO
public class XxxDTO {
    private Long id;           // 主键
    private String name;       // 名称
    private Integer status;    // 状态
}
```

**Adapter 使用示例**:

```java
@Component
public class XxxAdapter {
    
    @DubboReference
    private XxxRpcService xxxRpcService;
    
    public XxxBO getById(Long id) {
        Result<XxxDTO> result = xxxRpcService.getXxxById(id);
        if (result == null || !result.isSuccess()) {
            return null;
        }
        return CopyUtil.copyBean(result.getData(), XxxBO.class);
    }
}
```
```

---

## 四、TODO 代码生成规范

当用户无法提供 RPC 接口信息时，生成以下格式的 TODO 占位代码：

```java
@Component
@Slf4j
public class XxxAdapter {

    // TODO 待确认：Xxx 服务 RPC 接口
    // 预期功能：查询 Xxx 信息
    // 预期入参：Long id
    // 预期出参：XxxDTO（包含 id, name, status 等字段）
    // @DubboReference
    // private XxxRpcService xxxRpcService;

    /**
     * 查询 Xxx 信息
     * <p>
     * TODO 待对接 RPC 服务
     *
     * @param id Xxx ID
     * @return Xxx 信息
     */
    public XxxBO getById(Long id) {
        // TODO 对接 Xxx 服务 RPC
        log.info("查询 Xxx 信息，id={}", id);
        return null;
    }
}
```

**TODO 注释要素**:
1. 服务名称和预期功能
2. 预期入参类型和说明
3. 预期出参类型和关键字段
4. 被注释的 `@DubboReference` 注入代码

---

## 五、更新记录

| 日期 | 更新内容 | 操作人 |
|------|----------|--------|
| 2025-12-26 | 创建文件，定义基础结构 | Kiro |

