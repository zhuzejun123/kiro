---
inclusion: fileMatch
fileMatchPattern: "**/*-dao/**/*.java"
---

# dao (数据访问层)

路径: `{BASE_PACKAGE}.dao.{业务域}`

```
{MODULE}-dao/src/main/java/{BASE_PACKAGE}/dao/{业务域}/
├── entity/          # 实体类 (DO)，对应数据库表
├── enums/           # 枚举类（可选）
└── {业务语义}Dao.java  # Dao 接口，继承 CrudMapper<DO>
```

**重要约束**: 禁止创建 XML 映射文件，所有查询必须通过 Dao 内置方法 + Wrapper 实现。

**变量说明**:
- `{MODULE}`: finance 或 sale
- `{BASE_PACKAGE}`: com.dahuangf.finance 或 com.dahuangf.sale（路径中 `.` 替换为 `/`）

---

## 表结构设计流程（必须遵循）

根据需求内容进行表结构设计，需完成以下步骤：

### 步骤一：需求分析与表拆解
- 分析需求中的业务实体和关系
- 根据实际业务需求拆解为若干合理的表
- 给出拆解理由（如：主从关系、一对多、多对多等）

### 步骤二：表结构确认
- 列出每张表的字段清单（字段名、数据库类型、Java 类型、说明）
- 标注主键、外键、索引
- 等待用户确认后再生成 DO

### 步骤三：生成 DO 文件
- 确认无误后生成对应的 DO 类
- 命名必须符合实际业务场景
- 禁止使用拼音和缩写
- 后缀必须带 DO

### 表命名规范
- 前缀: `{MODULE}_`（finance 项目用 `finance_`，sale 项目用 `sale_`）
- 按实际业务需求命名，禁止无意义后缀（如 `_main`、`_detail`）
- 示例: `finance_invoice_apply`、`sale_order_record`

---

## 字段命名规范（强制遵循）

### 时间 / 日期类

| 场景 | 命名规则 | 示例 |
|------|----------|------|
| 具体时间（含时分秒） | `xxx_time` | `issue_time`（开票时间）、`pay_time`（支付时间）、`apply_time`（申请时间） |
| 仅日期（年月日） | `xxx_date` | `invoice_date`（开票日期）、`due_date`（到期日期）、`settle_date`（结算日期） |

### 数量类

| 场景 | 命名规则 | 示例 |
|------|----------|------|
| 数量 | `xxx_num` | `goods_num`（商品数量）、`invoice_num`（发票份数）、`order_num`（订单数量） |

### 布尔类

| 场景 | 命名规则 | 示例 |
|------|----------|------|
| 是否xxx | `is_xxx` | `is_general_taxpayer`（是否一般纳税人）、`is_valid`（是否有效）、`is_deleted`（是否删除） |

### 关联字段

| 场景 | 命名规则 | 示例 |
|------|----------|------|
| 关联其他表 | `关联表名_id` | `order_id`（关联订单表）、`user_id`（关联用户表）、`invoice_id`（关联发票表） |

### 金额 / 成本 / 价格类

| 场景 | 命名规则 | 示例 |
|------|----------|------|
| 成本类 | `xxx_cost` | `goods_cost`（商品成本）、`service_cost`（服务成本）、`total_cost`（总成本） |
| 金额类（营收、收入、实收、退款等） | `xxx_amount` | `total_amount`（总金额）、`refund_amount`（退款金额）、`actual_amount`（实收金额） |
| 价格类（定价、单价） | `xxx_price` | `unit_price`（单价）、`sale_price`（销售价）、`original_price`（原价） |

### 地址 / 联系方式类

| 场景 | 命名规则 | 示例 |
|------|----------|------|
| 手机号 | `mobile` | `mobile`（避免用 phone，易混淆固定电话） |
| 邮箱 | `email` | `email` |
| 详细地址 | `address` | `address` |
| 省份 | `province` | `province` |
| 城市 | `city` | `city` |

### 税务相关（发票场景）

| 场景 | 命名规则 | 示例 |
|------|----------|------|
| 税号 | `xxx_tax_no` | `buyer_tax_no`（购方税号）、`seller_tax_no`（销方税号） |
| 商品编码 | `goods_code` | `goods_code` |
| 发票类型 | `invoice_type` | `invoice_type`（0=普票，1=专票） |

---

## 禁用 / 慎用的命名方式

### 禁止使用

| 禁止 | 正确 | 说明 |
|------|------|------|
| `fapiao_hao` | `invoice_no` | 禁用拼音 |
| `inv_amt` | `invoice_amount` | 禁用无意义缩写（仅 `amt`/`no`/`id` 等行业通用缩写可用） |
| `user_name1`、`user_name2` | `real_name`、`nick_name` | 禁用数字后缀 |

### 慎用

| 场景 | 规则 |
|------|------|
| 英文复数 | 表名可用复数（如 `invoices`），字段名用单数（如 `invoice_no` 而非 `invoices_no`） |

### 允许的通用缩写

| 缩写 | 全称 | 说明 |
|------|------|------|
| `id` | identifier | 主键/标识符 |
| `no` | number | 编号 |
| `amt` | amount | 金额（慎用，推荐全称） |
| `qty` | quantity | 数量（慎用，推荐用 `xxx_num`） |

---

## DDL 审计字段规范（强制遵循）

### 必须包含的审计字段

所有表的 DDL 语句必须包含以下 5 个审计字段，放在业务字段之后、PRIMARY KEY 之前：

```sql
`created_at` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
`created_by` bigint(20) DEFAULT '0' COMMENT '创建者',
`updated_at` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
`updated_by` bigint(20) DEFAULT '0' COMMENT '更新者',
`del_flag` tinyint(4) NOT NULL DEFAULT '0' COMMENT '0未删除，1已删除',
```

### DO 类中不需要定义审计字段

**重要**: DO 类继承 `BaseDO`，父类已定义这 5 个审计字段，子类无需重复定义。

### DDL 完整示例

```sql
CREATE TABLE `finance_invoice_apply` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    `invoice_no` varchar(64) NOT NULL DEFAULT '' COMMENT '发票编号',
    `amount` bigint(20) NOT NULL DEFAULT '0' COMMENT '金额（单位：分）',
    `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态：0-待审核，1-审核中，2-已通过，3-已拒绝',
    `apply_time` datetime DEFAULT NULL COMMENT '申请时间',
    `created_at` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `created_by` bigint(20) DEFAULT '0' COMMENT '创建者',
    `updated_at` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `updated_by` bigint(20) DEFAULT '0' COMMENT '更新者',
    `del_flag` tinyint(4) NOT NULL DEFAULT '0' COMMENT '0未删除，1已删除',
    PRIMARY KEY (`id`),
    KEY `idx_invoice_no` (`invoice_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='发票申请表';
```

### DO 类示例（不包含审计字段）

```java
@Getter
@Setter
@ToString(callSuper = true)
@TableName("finance_invoice_apply")
public class InvoiceApplyDO extends BaseDO implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 发票编号
     */
    private String invoiceNo;

    /**
     * 金额（单位：分）
     */
    private Long amount;

    /**
     * 状态：0-待审核，1-审核中，2-已通过，3-已拒绝
     */
    private Integer status;

    /**
     * 申请时间
     */
    private LocalDateTime applyTime;

    // 注意：created_at、created_by、updated_at、updated_by、del_flag 
    // 由 BaseDO 父类提供，子类无需定义
}
```

---

## 索引设计规范（强制遵循）

### 设计原则

1. **基于业务场景设计**: 根据需求中的查询条件、交互流程设计索引
2. **优先组合索引**: 多个字段经常一起查询时，优先创建组合索引
3. **遵循最左前缀原则**: 组合索引字段顺序按查询频率和区分度排列
4. **避免过度索引**: 每张表索引数量建议不超过 5 个

### 索引命名规范

| 索引类型 | 命名规则 | 示例 |
|----------|----------|------|
| 普通索引 | `idx_字段名` | `idx_invoice_no` |
| 组合索引 | `idx_字段1_字段2` | `idx_status_created_at` |
| 唯一索引 | `uk_字段名` | `uk_invoice_no` |

### 索引设计流程

#### 步骤一：分析查询场景

根据需求文档，列出所有查询场景：

```markdown
### 查询场景分析

| 场景 | 查询条件 | 排序字段 | 频率 |
|------|----------|----------|------|
| 列表分页查询 | status, created_at 范围 | created_at DESC | 高 |
| 按编号查询 | invoice_no | - | 高 |
| 按用户查询 | user_id, status | created_at DESC | 中 |
| 按时间范围统计 | created_at 范围, status | - | 低 |
```

#### 步骤二：设计索引方案

根据查询场景，设计组合索引（优先）和单列索引：

```markdown
### 索引设计方案

| 索引名 | 索引类型 | 字段 | 覆盖场景 | 说明 |
|--------|----------|------|----------|------|
| `uk_invoice_no` | 唯一索引 | `invoice_no` | 按编号查询 | 业务唯一键 |
| `idx_status_created_at` | 组合索引 | `status, created_at` | 列表分页、时间范围统计 | 状态+时间组合查询 |
| `idx_user_id_status` | 组合索引 | `user_id, status` | 按用户查询 | 用户维度查询 |
```

#### 步骤三：生成 DDL

```sql
CREATE TABLE `finance_invoice_apply` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    `invoice_no` varchar(64) NOT NULL DEFAULT '' COMMENT '发票编号',
    `user_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '用户ID',
    `amount` bigint(20) NOT NULL DEFAULT '0' COMMENT '金额（单位：分）',
    `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态',
    `created_at` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `created_by` bigint(20) DEFAULT '0' COMMENT '创建者',
    `updated_at` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `updated_by` bigint(20) DEFAULT '0' COMMENT '更新者',
    `del_flag` tinyint(4) NOT NULL DEFAULT '0' COMMENT '0未删除，1已删除',
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk_invoice_no` (`invoice_no`),
    KEY `idx_status_created_at` (`status`, `created_at`),
    KEY `idx_user_id_status` (`user_id`, `status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='发票申请表';
```

### 组合索引设计要点

| 要点 | 说明 | 示例 |
|------|------|------|
| 区分度高的字段在前 | 区分度 = 不同值数量 / 总行数 | `user_id` 比 `status` 区分度高 |
| 等值查询字段在前 | `=` 条件字段放在范围查询字段前 | `status =` 在 `created_at >` 前 |
| 排序字段放最后 | ORDER BY 字段放组合索引最后 | `idx_status_created_at` 支持 `ORDER BY created_at` |
| 覆盖多个场景 | 一个组合索引尽量覆盖多个查询场景 | `idx_status_created_at` 覆盖分页和统计 |

### 常见索引场景

| 业务场景 | 推荐索引 |
|----------|----------|
| 分页列表（状态+时间筛选） | `idx_status_created_at` |
| 用户维度查询 | `idx_user_id_status` |
| 订单关联查询 | `idx_order_id` |
| 业务编号查询 | `uk_xxx_no`（唯一索引） |
| 时间范围统计 | `idx_created_at` 或组合索引 |

### 索引设计确认

生成 DDL 前，需要输出索引设计方案并等待用户确认：

```markdown
### 索引设计确认

根据需求中的查询场景，设计以下索引：

| 索引名 | 索引类型 | 字段 | 覆盖场景 |
|--------|----------|------|----------|
| ... | ... | ... | ... |

请确认以上索引设计是否满足业务查询需求，如需调整请说明。
```

### 输出格式示例

```
## 表结构设计

### 拆解理由
根据需求分析，该业务涉及以下实体关系：
1. XXX表：存储核心业务数据
2. YYY表：与XXX表一对多关系，存储关联数据

### 表结构清单

#### 表1: finance_invoice_apply（发票申请表）
| 字段名 | 数据库类型 | Java类型 | 说明 |
|--------|------------|----------|------|
| id | bigint | Long | 主键 |
| invoice_no | varchar(64) | String | 发票编号 |
| amount | bigint | Long | 金额 |
| status | tinyint | Integer | 状态 |
| apply_time | datetime | LocalDateTime | 申请时间 |

#### 表2: finance_invoice_item（发票明细表）
| 字段名 | 数据库类型 | Java类型 | 说明 |
|--------|------------|----------|------|
| id | bigint | Long | 主键 |
| invoice_id | bigint | Long | 关联发票ID |
| item_name | varchar(128) | String | 项目名称 |
| quantity | int | Integer | 数量 |
| unit_price | bigint | Long | 单价 |

### 确认问题
请确认以上表结构是否满足需求，如需调整请说明。
```

---

## Entity (DO) 规范

### 继承与实现
- 必须继承 `com.dahuangf.common.mybatisplus.support.BaseDO`
- 必须实现 `java.io.Serializable` 接口

### 注解规范（强制遵循）

1. Lombok 注解:
   - 必须添加 `@Getter`、`@Setter`、`@ToString(callSuper = true)`（ToString 需继承父类属性）
   - 禁止添加多余 Lombok 注解（如 `@Data`，需与项目已有 DO 类风格一致）

2. MyBatis-Plus 注解:
   - `@TableName`: 值为数据表名（如 `finance_invoice_apply`）
   - 注解位置: 在类定义上方，`@ToString` 之后

### 命名规范
- 类名必须符合实际业务场景
- 禁止使用拼音和缩写
- 后缀必须带 DO

### 字段规范
- 包含所有数据库字段
- 字段类型必须使用 Java 包装类型（如 `Long`、`Integer`、`String`）
- 金额相关字段统一使用 `Long` 类型（单位：分）
- VO、BO 字段必须与 DO 字段完全一致

### DO 示例

```java
import com.baomidou.mybatisplus.annotation.TableName;
import com.dahuangf.common.mybatisplus.support.BaseDO;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

import java.io.Serializable;
import java.time.LocalDateTime;

@Getter
@Setter
@ToString(callSuper = true)
@TableName("finance_invoice_apply")
public class InvoiceApplyDO extends BaseDO implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 发票编号
     */
    private String invoiceNo;

    /**
     * 金额
     */
    private Long amount;

    /**
     * 状态
     */
    private Integer status;

    /**
     * 申请时间
     */
    private LocalDateTime applyTime;
}
```

---

## Dao 规范

### 基本规范

```java
import com.baomidou.dynamic.datasource.annotation.DS;
import com.dahuangf.common.mybatisplus.support.CrudMapper;

@DS("{数据源名称}")  // 根据实际业务需求传入数据源名称
public interface InvoiceApplyDao extends CrudMapper<InvoiceApplyDO> {
    // 禁止自定义 SQL 方法，所有查询通过 CrudMapper 内置方法实现
}
```

### 禁止 XML SQL（重要约束）

**核心规则**: 禁止在 XML 文件中编写 SQL，所有数据库操作必须通过 Dao 内置方法实现。

**禁止的做法**:
- ❌ 创建 `XxxDao.xml` 或 `XxxMapper.xml` 文件
- ❌ 在 Dao 接口中定义需要 XML 实现的方法
- ❌ 使用 `@Select`、`@Update`、`@Insert`、`@Delete` 注解编写原生 SQL

**正确的做法**:
- ✅ 使用 `LambdaQueryWrapper` 构建查询条件
- ✅ 使用 `LambdaUpdateWrapper` 构建更新条件
- ✅ 调用 CrudMapper 内置方法（`getOne`、`list`、`page`、`save`、`updateById`、`update`、`removeById` 等）

### 查询方式对照表

| 场景 | 禁止的方式（XML/注解） | 正确的方式（Wrapper） |
|------|------------------------|----------------------|
| 单条件查询 | `@Select("SELECT * FROM t WHERE status = #{status}")` | `dao.list(new LambdaQueryWrapper<DO>().eq(DO::getStatus, status))` |
| 多条件查询 | XML 中写复杂 SQL | `wrapper.eq(...).like(...).between(...)` |
| 条件更新 | `@Update("UPDATE t SET status = #{status} WHERE id = #{id}")` | `dao.update(new LambdaUpdateWrapper<DO>().eq(DO::getId, id).set(DO::getStatus, status))` |
| 分页查询 | XML 中写分页 SQL | `dao.page(page, wrapper)` |
| 批量查询 | `@Select("SELECT * FROM t WHERE id IN ...")` | `dao.listByIds(idList)` 或 `wrapper.in(DO::getId, idList)` |

### LambdaQueryWrapper 使用示例

```java
// 单条件查询
LambdaQueryWrapper<InvoiceApplyDO> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(InvoiceApplyDO::getInvoiceNo, invoiceNo);
InvoiceApplyDO result = invoiceApplyDao.getOne(wrapper);

// 多条件查询
LambdaQueryWrapper<InvoiceApplyDO> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(InvoiceApplyDO::getStatus, status)
       .like(StringUtils.isNotBlank(keyword), InvoiceApplyDO::getInvoiceNo, keyword)
       .between(InvoiceApplyDO::getCreatedAt, startTime, endTime)
       .orderByDesc(InvoiceApplyDO::getCreatedAt);
List<InvoiceApplyDO> list = invoiceApplyDao.list(wrapper);

// 分页查询
IPage<InvoiceApplyDO> page = invoiceApplyDao.page(pageParam.getPage(), wrapper);

// 查询指定字段
wrapper.select(InvoiceApplyDO::getId, InvoiceApplyDO::getInvoiceNo, InvoiceApplyDO::getStatus);
```

### LambdaUpdateWrapper 使用示例

```java
// 条件更新单个字段
LambdaUpdateWrapper<InvoiceApplyDO> wrapper = new LambdaUpdateWrapper<>();
wrapper.eq(InvoiceApplyDO::getId, id)
       .set(InvoiceApplyDO::getStatus, newStatus);
invoiceApplyDao.update(wrapper);

// 条件更新多个字段
LambdaUpdateWrapper<InvoiceApplyDO> wrapper = new LambdaUpdateWrapper<>();
wrapper.eq(InvoiceApplyDO::getId, id)
       .set(InvoiceApplyDO::getStatus, newStatus)
       .set(InvoiceApplyDO::getApprovalTime, LocalDateTime.now())
       .set(InvoiceApplyDO::getApproverName, approverName);
invoiceApplyDao.update(wrapper);

// 批量条件更新
LambdaUpdateWrapper<InvoiceApplyDO> wrapper = new LambdaUpdateWrapper<>();
wrapper.in(InvoiceApplyDO::getId, idList)
       .set(InvoiceApplyDO::getStatus, newStatus);
invoiceApplyDao.update(wrapper);
```

### CrudMapper 常用内置方法

| 方法 | 说明 | 示例 |
|------|------|------|
| `getById(id)` | 根据 ID 查询 | `dao.getById(1L)` |
| `getOne(wrapper)` | 条件查询单条 | `dao.getOne(wrapper)` |
| `list(wrapper)` | 条件查询列表 | `dao.list(wrapper)` |
| `listByIds(idList)` | 根据 ID 批量查询 | `dao.listByIds(Arrays.asList(1L, 2L))` |
| `page(page, wrapper)` | 分页查询 | `dao.page(page, wrapper)` |
| `count(wrapper)` | 条件统计数量 | `dao.count(wrapper)` |
| `save(entity)` | 新增 | `dao.save(entity)` |
| `saveBatch(list)` | 批量新增 | `dao.saveBatch(list)` |
| `updateById(entity)` | 根据 ID 更新 | `dao.updateById(entity)` |
| `update(wrapper)` | 条件更新 | `dao.update(wrapper)` |
| `removeById(id)` | 根据 ID 删除 | `dao.removeById(1L)` |
| `remove(wrapper)` | 条件删除 | `dao.remove(wrapper)` |

---

## 业务域示例

aging, billing, cost, invoice, paycollection, performance, settlement, sale, receipt, reimbursement, profit, special, workflow...
