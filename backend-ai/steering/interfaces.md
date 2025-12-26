---
inclusion: fileMatch
fileMatchPattern: "**/*-interfaces/**/*.java"
---

# interfaces (Web 层/主应用入口)

路径: `{BASE_PACKAGE}.interfaces`

```
{MODULE}-interfaces/src/main/java/{BASE_PACKAGE}/interfaces/
├── controller/      # REST 控制器
│   ├── {业务域}/    # 按业务域分子目录
│   │   ├── vo/
│   │   │   ├── req/     # 请求 VO
│   │   │   │   ├── XxxPageReqVO.java
│   │   │   │   ├── XxxCreateReqVO.java
│   │   │   │   └── XxxUpdateReqVO.java
│   │   │   └── resp/    # 响应 VO
│   │   │       └── XxxRespVO.java
│   │   └── XxxController.java
├── rpc/             # RPC 接口实现 (Dubbo Service 实现)
│   ├── {业务域}/
│   └── XxxRpcImpl.java
├── mq/              # 消息队列监听器
├── job/             # 定时任务 (PowerJob)
│   └── {业务域}/
├── download/        # 导出相关
│   └── {业务域}/
├── listener/        # 事件监听器
├── config/          # 配置类
├── filter/          # 过滤器
├── intercept/       # 拦截器
├── annotation/      # 自定义注解
└── exception/       # 异常处理

**变量说明**:
- `{MODULE}`: finance 或 sale（注意 sale 项目的 interfaces 模块名为 sale-platform-interfaces）
- `{BASE_PACKAGE}`: com.dahuangf.finance 或 com.dahuangf.sale（路径中 `.` 替换为 `/`）
```

---

## VO 规范

**目录结构**: `vo/req/` 存放请求 VO，`vo/resp/` 存放响应 VO

### 验证器注解规范（强制遵循）

**所有验证器注解的 message 必须使用中文，提示语要友好、简洁、有意义**

| 注解 | 正确示例 | 错误示例 |
|------|----------|----------|
| `@NotNull` | `@NotNull(message = "用户ID不能为空")` | `@NotNull(message = "id is required")` |
| `@NotBlank` | `@NotBlank(message = "用户名不能为空")` | `@NotBlank(message = "username must not be blank")` |
| `@NotEmpty` | `@NotEmpty(message = "设备清单不能为空")` | `@NotEmpty` |
| `@Size` | `@Size(max = 200, message = "备注最多200字")` | `@Size(max = 200)` |
| `@Min` | `@Min(value = 1, message = "数量至少为1")` | `@Min(1)` |
| `@Max` | `@Max(value = 30, message = "有效期最多30天")` | `@Max(30)` |
| `@Pattern` | `@Pattern(regexp = "...", message = "手机号格式不正确")` | `@Pattern(regexp = "...")` |

**禁止**：
- 英文提示
- 无意义的格式化输出（如 `{field} is invalid`）
- 省略 message 参数

### VO 字段注释规范（API Doc 生成必备）

所有 VO 字段必须添加清晰、详细的 JavaDoc 注释，用于后续生成 API 文档。

**注释要素**:
| 要素 | 说明 | 是否必填 |
|------|------|----------|
| 字段含义 | 简明扼要说明字段用途 | 必填 |
| 业务规则 | 字段的业务约束或规则说明 | 按需 |
| 取值范围 | 枚举值、数值范围等 | 按需 |
| 关联说明 | 关联其他字段或表的说明 | 按需 |
| 单位说明 | 金额（分）、时间（秒）等单位 | 按需 |

**注释格式规范**:
```java
/**
 * 字段含义（必填）
 * <p>业务规则说明（按需）</p>
 * <p>取值范围：xxx（按需）</p>
 * <p>关联说明：xxx（按需）</p>
 * <p>单位：xxx（按需）</p>
 */
private String fieldName;
```

**注释示例**:

```java
/**
 * 发票类型
 * <p>业务规则：不同发票类型对应不同的税率计算规则</p>
 * <p>取值范围：1-增值税普通发票，2-增值税专用发票，3-电子发票</p>
 */
@NotNull(message = "发票类型不能为空")
private Integer invoiceType;

/**
 * 开票金额
 * <p>业务规则：单张发票金额不得超过10万元</p>
 * <p>单位：分（如 100 表示 1.00 元）</p>
 */
@NotNull(message = "开票金额不能为空")
@Price
private Long amount;

/**
 * 购方纳税人识别号
 * <p>业务规则：企业开票必填，个人开票可为空</p>
 * <p>格式要求：15位、17位、18位或20位数字/字母组合</p>
 */
private String buyerTaxNo;

/**
 * 发票状态
 * <p>取值范围：0-待开票，1-开票中，2-开票成功，3-开票失败，4-已作废，5-已红冲</p>
 * <p>状态流转：0→1→2/3，2→4/5</p>
 */
private Integer status;

/**
 * 关联订单ID
 * <p>关联说明：关联 finance_order 表的主键 id</p>
 * <p>业务规则：一个订单可对应多张发票（拆分开票场景）</p>
 */
private Long orderId;

/**
 * 开票申请时间
 * <p>业务规则：系统自动记录，不可修改</p>
 */
private LocalDateTime applyTime;

/**
 * 发票备注
 * <p>业务规则：最多200字符，特殊字符会被过滤</p>
 */
private String remark;
```

**字段类型注释要点**:

| 字段类型 | 注释重点 |
|----------|----------|
| 枚举/状态字段 | 必须列出所有取值及含义 |
| 金额字段 | 必须说明单位（分/元） |
| 时间字段 | 说明时间格式或业务含义 |
| ID关联字段 | 说明关联的表和字段 |
| 字符串字段 | 说明长度限制、格式要求 |
| 布尔字段 | 说明 true/false 的业务含义 |

---

## 1. 分页查询 VO（XxxPageReqVO）

**必须包含所有业务字段**（用于查询条件），以及以下基础字段用于时间范围查询：

| Java 字段 | 数据库字段 | 类型 | 说明 |
|-----------|------------|------|------|
| createdAtStart | created_at | LocalDate | 创建时间-开始 |
| createdAtEnd | created_at | LocalDate | 创建时间-结束 |
| createdBy | created_by | Long | 创建人 |
| updatedAtStart | updated_at | LocalDate | 更新时间-开始 |
| updatedAtEnd | updated_at | LocalDate | 更新时间-结束 |
| updatedBy | updated_by | Long | 更新人 |

```java
import com.dahuangf.common.mybatisplus.support.PageParam;
import com.dahuangf.common.mybatisplus.support.IQuery;
import com.dahuangf.common.mybatisplus.annotation.Condition;
import com.dahuangf.common.mybatisplus.enums.Operator;
import lombok.Data;
import lombok.EqualsAndHashCode;

import java.time.LocalDate;

@EqualsAndHashCode(callSuper = false)
@Data
public class UserPageReqVO extends PageParam implements IQuery {

    /**
     * 用户名
     */
    @Condition(operator = Operator.eq)
    private String username;

    /**
     * 手机号
     */
    @Condition(operator = Operator.eq)
    private String phone;

    /**
     * 状态
     */
    @Condition(operator = Operator.in)
    private Long status;

    /**
     * 创建时间-开始
     */
    @Condition(operator = Operator.ge, column = "created_at")
    private LocalDate createdAtStart;

    /**
     * 创建时间-结束
     */
    @Condition(operator = Operator.le, column = "created_at")
    private LocalDate createdAtEnd;

    /**
     * 创建人
     */
    @Condition(operator = Operator.in)
    private Long createdBy;

    /**
     * 更新时间-开始
     */
    @Condition(operator = Operator.ge, column = "updated_at")
    private LocalDate updatedAtStart;

    /**
     * 更新时间-结束
     */
    @Condition(operator = Operator.le, column = "updated_at")
    private LocalDate updatedAtEnd;

    /**
     * 更新人
     */
    @Condition(operator = Operator.in)
    private Long updatedBy;
}
```

**字段注解规则**:
| 字段类型 | 语义 | 注解 |
|----------|------|------|
| String | - | `@Condition(operator = Operator.eq)`（禁止使用 like） |
| Long | ID 类（门店id、用户id、区域id 等） | `@Condition(operator = Operator.in)` |
| Long | 金额/费用/成本类 | `@Condition(operator = Operator.eq)` |
| LocalDate | 范围开始 | `@Condition(operator = Operator.ge, column = "{数据库字段名_下划线}")` |
| LocalDate | 范围结束 | `@Condition(operator = Operator.le, column = "{数据库字段名_下划线}")` |

> **注意**: String 类型字段必须使用 `Operator.eq`，不允许使用 `Operator.like`

**基础字段映射**（驼峰转下划线）:
| Java 字段 | 数据库字段 |
|-----------|------------|
| createdAt | created_at |
| createdBy | created_by |
| updatedAt | updated_at |
| updatedBy | updated_by |

**注意**: column 值为数据库字段名（下划线格式），如 `created_at`、`updated_at`

---

## 2. 新增 VO（XxxCreateReqVO）

```java
import lombok.Data;

@Data
public class UserCreateReqVO {

    /**
     * 用户名
     */
    private String username;

    /**
     * 手机号
     */
    private String phone;

    /**
     * 昵称
     */
    private String nickname;

    /**
     * 地址
     */
    private String address;
}
```

---

## 3. 更新 VO（XxxUpdateReqVO）

```java
import com.dahuangf.common.mybatisplus.support.UpdateVO;
import lombok.Data;
import lombok.EqualsAndHashCode;

import javax.validation.constraints.NotNull;

@EqualsAndHashCode(callSuper = false)
@Data
public class UserUpdateReqVO extends UserCreateReqVO implements UpdateVO {

    /**
     * ID
     */
    @NotNull(message = "id不能为空")
    private Long id;
}
```

---

## 4. 响应 VO（XxxRespVO）

需要返回数据库全字段的 VO 类，必须包含以下基础字段：

| Java 字段 | 数据库字段 | 类型 | 说明 |
|-----------|------------|------|------|
| id | id | Long | 主键 |
| createdAt | created_at | LocalDateTime | 创建时间 |
| createdBy | created_by | Long | 创建人 |
| updatedAt | updated_at | LocalDateTime | 更新时间 |
| updatedBy | updated_by | Long | 更新人 |

**金融字段注解规则**:

对于 `Long` 类型的金融相关字段（如金额、成本、价格等），必须添加 `@Price` 注解用于格式化展示：

| 字段语义 | 类型 | 注解 |
|----------|------|------|
| 金额（amount） | Long | `@Price` |
| 成本（cost） | Long | `@Price` |
| 价格（price） | Long | `@Price` |
| 费用（fee） | Long | `@Price` |
| 金钱（money） | Long | `@Price` |
| 总额（total） | Long | `@Price` |
| 单价（unitPrice） | Long | `@Price` |

```java
import com.dahuangf.common.annotation.Price;
import lombok.Data;

import java.time.LocalDateTime;

@Data
public class UserRespVO {

    /**
     * 主键
     */
    private Long id;

    /**
     * 用户名
     */
    private String username;

    /**
     * 手机号
     */
    private String phone;

    /**
     * 昵称
     */
    private String nickname;

    /**
     * 地址
     */
    private String address;

    /**
     * 创建时间
     */
    private LocalDateTime createdAt;

    /**
     * 创建人
     */
    private Long createdBy;

    /**
     * 更新时间
     */
    private LocalDateTime updatedAt;

    /**
     * 更新人
     */
    private Long updatedBy;
}
```

**金融字段示例**（如涉及金额相关字段）:

```java
import com.dahuangf.common.annotation.Price;
import lombok.Data;

import java.time.LocalDateTime;

@Data
public class InvoiceRespVO {

    /**
     * 主键
     */
    private Long id;

    /**
     * 发票编号
     */
    private String invoiceNo;

    /**
     * 金额
     */
    @Price
    private Long amount;

    /**
     * 税额
     */
    @Price
    private Long taxAmount;

    /**
     * 总金额
     */
    @Price
    private Long totalAmount;

    /**
     * 创建时间
     */
    private LocalDateTime createdAt;

    /**
     * 创建人
     */
    private Long createdBy;

    /**
     * 更新时间
     */
    private LocalDateTime updatedAt;

    /**
     * 更新人
     */
    private Long updatedBy;
}
```

---

## Controller 规范

### 基础规范

1. **路径前缀**: `/{业务模块名}`（如 `/user`、`/invoice`、`/fundAccount`）
2. **统一返回体**: `BaseResult`（支持 IPage 分页、基础数据类型返回）
3. **数据转换**: 使用 `CopyUtil` 完成 VO/BO 的对象/列表/分页拷贝
4. **参数校验**: 入参必须添加 `@Validated` / `@NotNull`，保证参数合法性
5. **类注解**: `@RestController`、`@Validated`
6. **依赖注入**: 使用 `@Resource`

### 必含核心接口（CRUD 全覆盖）

| 接口 | 方法 | 路径 | 入参 | 出参 |
|------|------|------|------|------|
| 分页查询 | GET | /page | XxxPageReqVO | BaseResult<IPage<XxxRespVO>> |
| 单条详情 | GET | /detail | @NotNull Long id | BaseResult<XxxRespVO> |
| 批量新增 | POST | /add | @Validated XxxCreateReqVO | BaseResult<Integer> |
| 单条修改 | POST | /update | @Validated XxxUpdateReqVO | BaseResult<Integer> |
| 单条删除 | POST | /delete | @NotNull Long id | BaseResult<Integer> |

### 日志注解

新增接口添加 `@LogRecord` 日志注解：
```java
@LogRecord("新增【XX业务】配置: {TO_STRING{#param}}")
```

### Controller 示例

```java
package {BASE_PACKAGE}.interfaces.controller.user;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.dahuangf.common.core.result.BaseResult;
import com.dahuangf.common.log.annotation.LogRecord;
import com.dahuangf.common.utils.CopyUtil;
import {BASE_PACKAGE}.core.user.UserService;
import {BASE_PACKAGE}.core.user.bo.UserBO;
import {BASE_PACKAGE}.dao.user.entity.UserDO;
import {BASE_PACKAGE}.interfaces.controller.user.vo.req.UserCreateReqVO;
import {BASE_PACKAGE}.interfaces.controller.user.vo.req.UserPageReqVO;
import {BASE_PACKAGE}.interfaces.controller.user.vo.req.UserUpdateReqVO;
import {BASE_PACKAGE}.interfaces.controller.user.vo.resp.UserRespVO;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import javax.validation.constraints.NotNull;

@RestController
@RequestMapping("/user")
@Validated
public class UserController {

    @Resource
    private UserService userService;

    /**
     * 分页查询
     */
    @GetMapping("/page")
    public BaseResult<IPage<UserRespVO>> page(UserPageReqVO reqVO) {
        // PageReqVO 继承 PageParam，调用 toLambdaQueryWrapper() 生成查询条件
        LambdaQueryWrapper<UserDO> queryWrapper = reqVO.toLambdaQueryWrapper();
        IPage<UserBO> boPage = userService.page(reqVO, queryWrapper);
        IPage<UserRespVO> voPage = CopyUtil.copyPageInfo(boPage, UserRespVO.class);
        return BaseResult.success(voPage);
    }

    /**
     * 单条详情
     */
    @GetMapping("/detail")
    public BaseResult<UserRespVO> detail(@NotNull(message = "id不能为空") Long id) {
        UserBO bo = userService.detail(id);
        UserRespVO vo = CopyUtil.copyBean(bo, UserRespVO.class);
        return BaseResult.success(vo);
    }

    /**
     * 新增
     */
    @PostMapping("/add")
    @LogRecord("新增【用户】配置: {TO_STRING{#reqVO}}")
    public BaseResult<Integer> add(@RequestBody @Validated UserCreateReqVO reqVO) {
        int count = userService.add(reqVO);
        return BaseResult.success(count);
    }

    /**
     * 修改
     */
    @PostMapping("/update")
    public BaseResult<Integer> update(@RequestBody @Validated UserUpdateReqVO reqVO) {
        int count = userService.update(reqVO);
        return BaseResult.success(count);
    }

    /**
     * 删除
     */
    @PostMapping("/delete")
    public BaseResult<Integer> delete(@RequestBody @NotNull(message = "id不能为空") Long id) {
        int count = userService.delete(id);
        return BaseResult.success(count);
    }
}
```

**关键说明**: `PageReqVO` 继承 `PageParam` 并实现 `IQuery` 接口，Controller 的 `page` 方法只接收一个 VO 参数，调用 Service 时传入 `reqVO`（作为 PageParam）和 `queryWrapper` 两个参数。


---

## API 文档生成规范（必须遵循）

### 生成时机

所有 Controller 代码生成完成后，必须同步生成 API 接口文档。

### 路径拼接规则

完整接口路径 = **工程前缀** + **@RequestMapping** + **方法注解路径**

| 组成部分 | 来源 | 示例 |
|----------|------|------|
| 工程前缀 | 固定值 | `/finance-platform` |
| 类路径 | `@RequestMapping("/user")` | `/user` |
| 方法路径 | `@GetMapping("/page")` / `@PostMapping("/add")` | `/page`、`/add` |
| 完整路径 | 拼接结果 | `/{PROJECT}/user/page` |

### 输出位置

根目录下 `api/` 文件夹，按业务域分文件：

```
api/
├── user-api.md           # 用户模块接口
├── invoice-api.md        # 发票模块接口
├── settlement-api.md     # 结算模块接口
└── ...
```

### 文档格式

每个接口包含：接口路径、请求方式、接口说明、请求参数、响应参数

```markdown
## 用户管理接口

### 分页查询用户列表

- **接口路径**: `/{PROJECT}/user/page`
- **请求方式**: GET
- **接口说明**: 分页查询用户列表

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| username | String | 否 | 用户名 |
| phone | String | 否 | 手机号 |
| status | Long | 否 | 状态 |
| pageNum | Integer | 否 | 页码，默认1 |
| pageSize | Integer | 否 | 每页条数，默认10 |

**响应参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| id | Long | 主键 |
| username | String | 用户名 |
| phone | String | 手机号 |
| nickname | String | 昵称 |
| createdAt | LocalDateTime | 创建时间 |

---

### 查询用户详情

- **接口路径**: `/{PROJECT}/user/detail`
- **请求方式**: GET
- **接口说明**: 根据ID查询用户详情

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | Long | 是 | 用户ID |

**响应参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| id | Long | 主键 |
| username | String | 用户名 |
| ... | ... | ... |

---

### 新增用户

- **接口路径**: `/{PROJECT}/user/add`
- **请求方式**: POST
- **接口说明**: 新增用户

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| username | String | 是 | 用户名 |
| phone | String | 是 | 手机号 |
| nickname | String | 否 | 昵称 |

**响应参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| data | Integer | 新增记录数 |

---

### 修改用户

- **接口路径**: `/{PROJECT}/user/update`
- **请求方式**: POST
- **接口说明**: 修改用户信息

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | Long | 是 | 用户ID |
| username | String | 否 | 用户名 |
| phone | String | 否 | 手机号 |

**响应参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| data | Integer | 更新记录数 |

---

### 删除用户

- **接口路径**: `/{PROJECT}/user/delete`
- **请求方式**: POST
- **接口说明**: 删除用户

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| id | Long | 是 | 用户ID |

**响应参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| data | Integer | 删除记录数 |
```

### 注释来源

| 文档内容 | 来源 |
|----------|------|
| 接口说明 | Controller 方法的 JavaDoc 注释 |
| 参数说明 | VO 字段的 JavaDoc 注释 |
| 必填标识 | `@NotNull`、`@NotBlank` 等校验注解 |
