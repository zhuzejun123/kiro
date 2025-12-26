# 代码生成 Prompt 模板

## 项目上下文

finance-platform - Java 17 金融平台，Maven 多模块架构
基础包路径: `com.dahuangf.finance`
技术栈: Spring Boot + MyBatis-Plus + Dubbo + Nacos + Redis/Redisson + PowerJob

---

## 模块与目录规范

### 1. finance-dao (数据访问层)

路径: `com.dahuangf.finance.dao.{业务域}`

```
finance-dao/src/main/java/com/dahuangf/finance/dao/{业务域}/
├── entity/          # 实体类 (DO)，对应数据库表
├── mapper/          # Mapper 接口，继承 CrudMapper<DO>
└── dto/             # 数据库查询 DTO（可选）

finance-dao/src/main/resources/mapper/{业务域}/
└── XxxMapper.xml    # MyBatis XML 映射文件
```

#### 表结构设计流程（必须遵循）

根据需求内容进行表结构设计，需完成以下步骤：

**步骤一：需求分析与表拆解**
- 分析需求中的业务实体和关系
- 根据实际业务需求拆解为若干合理的表
- 给出拆解理由（如：主从关系、一对多、多对多等）

**步骤二：表结构确认**
- 列出每张表的字段清单（字段名、数据库类型、Java 类型、说明）
- 标注主键、外键、索引
- 等待用户确认后再生成 DO

**步骤三：生成 DO 文件**
- 确认无误后生成对应的 DO 类
- 命名必须符合实际业务场景
- 禁止使用拼音和缩写
- 后缀必须带 DO

**表命名规范**:
- 前缀: `finance_`
- 按实际业务需求命名，禁止无意义后缀（如 `_main`、`_detail`）
- 示例: `finance_invoice_apply`、`finance_settlement_record`

**输出格式示例**：

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
| amount | decimal(18,2) | BigDecimal | 金额 |
| status | tinyint | Integer | 状态 |
| apply_time | datetime | LocalDateTime | 申请时间 |

#### 表2: finance_invoice_item（发票明细表）
| 字段名 | 数据库类型 | Java类型 | 说明 |
|--------|------------|----------|------|
| id | bigint | Long | 主键 |
| invoice_id | bigint | Long | 关联发票ID |
| item_name | varchar(128) | String | 项目名称 |
| quantity | int | Integer | 数量 |
| unit_price | decimal(18,2) | BigDecimal | 单价 |

### 确认问题
请确认以上表结构是否满足需求，如需调整请说明。
```

#### Entity (DO) 规范

**继承与实现**:
- 必须继承 `com.dahuangf.common.mybatisplus.support.BaseDO`
- 必须实现 `java.io.Serializable` 接口

**注解规范（强制遵循）**:

1. Lombok 注解:
   - 必须添加 `@Getter`、`@Setter`、`@ToString(callSuper = true)`（ToString 需继承父类属性）
   - 禁止添加多余 Lombok 注解（如 `@Data`，需与项目已有 DO 类风格一致）

2. MyBatis-Plus 注解:
   - `@TableName`: 值为数据表名（如 `finance_invoice_apply`）
   - 注解位置: 在类定义上方，`@ToString` 之后

**命名规范**:
- 类名必须符合实际业务场景
- 禁止使用拼音和缩写
- 后缀必须带 DO

**字段规范**:
- 包含所有数据库字段
- 字段类型必须使用 Java 包装类型（如 `Long`、`Integer`、`BigDecimal`）
- VO、BO 字段必须与 DO 字段完全一致

```java
import com.baomidou.mybatisplus.annotation.TableName;
import com.dahuangf.common.mybatisplus.support.BaseDO;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

import java.io.Serializable;
import java.math.BigDecimal;
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
    private BigDecimal amount;

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

#### Mapper 规范

```java
import com.baomidou.dynamic.datasource.annotation.DS;
import com.dahuangf.common.mybatisplus.support.CrudMapper;

@DS("{数据源名称}")  // 根据实际业务需求传入数据源名称
public interface InvoiceApplyDao extends CrudMapper<InvoiceApplyDO> {
    // 自定义查询方法
}
```

业务域示例: aging, billing, cost, invoice, paycollection, performance, settlement, sale, receipt, reimbursement, profit, special, workflow...

---

### 2. finance-core (核心业务层)

路径: `com.dahuangf.finance.core.{业务域}`

```
finance-core/src/main/java/com/dahuangf/finance/core/{业务域}/
├── bo/              # 业务对象 (Business Object)
├── enums/           # 业务枚举
├── contracts/       # 契约/接口定义（策略接口、扩展点）
├── validate/        # 校验逻辑
├── service/         # 子模块 Service 目录
│   └── {子模块功能}/
│       └── XxxSubService.java
└── XxxService.java  # 对外整合 Service（与 service 目录同级）
```

#### 设计原则

**核心理念**: 高内聚低耦合 + 可维护/可扩展 + 简单实用，不做过度设计，明确每一个功能域的边界

#### 分层职责

**简单场景**: 仅基础增删改查（如配置项管理），可直接在外层 Service 完成，无需拆分子模块

**复杂场景**: 必须遵循以下原则：

| 层级 | 职责 | 说明 |
|------|------|------|
| 外层 XxxService | 子模块整合 + 对外统一入口 + 轻量级业务 | 包括：过滤、判断、简单业务判断、权限控制、BO 转换、DAO 数据查询、同级 Service 调用等 |
| 子模块 Service | 复杂需求场景的功能拆分 | 按业务子功能拆分，不一定只做一件事，可包含完整子业务流程 |
| BO 模块 | 仅封装业务数据 | 禁止包含 DAO/Service 依赖；禁止包含远程调用/数据库操作 |

#### Service 类规范

```java
@Service  // 无其他多余注解
public class XxxService {
    @Resource
    private XxxDao xxxDao;  // 依赖注入使用 @Resource
    
    @Resource
    private YyyService yyyService;  // 关联 Service
}
```

#### 命名规范

**DO 命名**:
- 实体类: `{业务语义}DO`，如 `InvoiceApplyDO`、`SettlementRecordDO`

**BO 命名**:
- 基础 BO: `{业务语义}BO`，如 `InvoiceApplyBO`
- 批量新增 BO: `{业务语义}BatchAddBO`，如 `InvoiceApplyBatchAddBO`
- 更新 BO: `{业务语义}UpdateBO`，如 `InvoiceApplyUpdateBO`

**方法命名**: 严格语义化，禁止模糊动词
- 分页查询: `page`
- 批量保存: `batchSave`
- 更新: `update`
- 详情查询: `getDetail`

#### 方法模板（必须严格遵循）

**1. 分页查询方法（page）**
- 入参: `PageParam pageParam, LambdaQueryWrapper<{业务语义}DO> queryWrapper`
- 出参: `IPage<{业务语义}BO>`
- 核心逻辑:
  1. 调用 Dao 的 page 方法: `dao.page(pageParam.getPage(), queryWrapper)`，获取 `IPage<DO>`
  2. 通过 `CopyUtil.copyPageInfo` 转换 DO 分页为 BO 分页
  3. DO 转 BO 时保留扩展字段填充逻辑（空实现也需保留，便于后续扩展）

```java
public IPage<InvoiceApplyBO> page(PageParam pageParam, LambdaQueryWrapper<InvoiceApplyDO> queryWrapper) {
    IPage<InvoiceApplyDO> doPage = invoiceApplyDao.page(pageParam.getPage(), queryWrapper);
    IPage<InvoiceApplyBO> boPage = CopyUtil.copyPageInfo(doPage, InvoiceApplyBO.class);
    // 扩展字段填充（预留扩展点）
    return boPage;
}
```

**2. 批量保存方法（batchSave）**
- 入参: `{业务语义}BatchAddBO bo`
- 出参: `void`
- 核心逻辑:
  1. 业务唯一性校验: 通过 `LambdaQueryWrapper` + `Dao.getOne` 查询 DO，`Objects.nonNull` 判断是否重复
  2. 重复校验失败: `AssertUtil.ifThrow` 抛业务异常，提示 "{核心业务字段}已使用，无法重复添加"
  3. 批量处理: 循环遍历 bo 中的集合参数，逐个转换为 DO，收集后调用 `Dao.saveBatch`

```java
public void batchSave(InvoiceApplyBatchAddBO bo) {
    LambdaQueryWrapper<InvoiceApplyDO> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(InvoiceApplyDO::getInvoiceNo, bo.getInvoiceNo());
    InvoiceApplyDO existDo = invoiceApplyDao.getOne(wrapper);
    AssertUtil.ifThrow(Objects.nonNull(existDo), "该发票编号已使用，无法重复添加");
    
    List<InvoiceApplyDO> doList = new ArrayList<>();
    for (Long itemId : bo.getItemIds()) {
        InvoiceApplyDO invoiceApplyDO = CopyUtil.copyBean(bo, InvoiceApplyDO.class);
        invoiceApplyDO.setItemId(itemId);
        doList.add(invoiceApplyDO);
    }
    invoiceApplyDao.saveBatch(doList);
}
```

**3. 更新方法（update）**
- 入参: `{业务语义}UpdateBO bo`
- 出参: `void`
- 核心逻辑:
  1. 数据校验: `Dao.listByIds(bo.getIds())` 查询 DO 列表，`CollectionUtil.isEmpty` 判空则直接返回
  2. 关联数据获取: 调用关联 Service 获取更新所需的关联数据
  3. 批量更新: 使用 `LambdaUpdateWrapper`，in 条件匹配 ID，set 更新核心业务字段

```java
public void update(InvoiceApplyUpdateBO bo) {
    List<InvoiceApplyDO> doList = invoiceApplyDao.listByIds(bo.getIds());
    if (CollectionUtil.isEmpty(doList)) {
        return;
    }
    
    // 关联数据获取（按需）
    // XxxBO relatedData = xxxService.getXxx();
    
    // 批量更新
    LambdaUpdateWrapper<InvoiceApplyDO> updateWrapper = new LambdaUpdateWrapper<>();
    updateWrapper.in(InvoiceApplyDO::getId, bo.getIds())
        .set(InvoiceApplyDO::getStatus, bo.getStatus());
    invoiceApplyDao.update(updateWrapper);
}
```

**4. 详情查询方法（getDetail）**
- 入参: `Long id`（主键）
- 出参: `{业务语义}BO`
- 核心逻辑:
  1. `Dao.getById(id)` 查询 DO
  2. `CopyUtil.copyBean` 将 DO 转换为 BO 返回（无数据则返回 null）

```java
public InvoiceApplyBO getDetail(Long id) {
    InvoiceApplyDO invoiceApplyDO = invoiceApplyDao.getById(id);
    return CopyUtil.copyBean(invoiceApplyDO, InvoiceApplyBO.class);
}
```

#### 技术栈 & 合规约束（强制遵循）

**数据转换**: 仅使用项目统一工具类 `CopyUtil`
- `CopyUtil.copyBean` - 单对象转换
- `CopyUtil.copyList` - 列表转换
- `CopyUtil.copyPageInfo` - 分页转换

**异常处理**:
- 业务规则校验必须使用 `AssertUtil.ifThrow` 抛异常
- 禁止 if + return 吞异常
- 统一使用 `com.dahuangf.common.core.exception.BizException`

```java
AssertUtil.ifThrow(Objects.nonNull(existDo), "该业务字段已使用，无法重复添加");
throw new BizException("订单不存在");
```

**MyBatis-Plus**:
- 仅使用 `LambdaQueryWrapper` / `LambdaUpdateWrapper`
- 禁止手写 SQL
- 优先使用内置方法: `getOne` / `listByIds` / `page` / `saveBatch` / `update`

**空值处理**:
| 场景 | 方式 |
|------|------|
| 集合判空 | `CollectionUtil.isEmpty`（hutool） |
| 对象判空 | `Objects.nonNull` / `Objects.isNull` |
| 空值安全获取 | `Optional.ofNullable()` |

**阿里手册合规**:
- 方法粒度: 单一职责，每个方法仅做一件事，行数 ≤ 80 行
- 命名规范: 无魔法值、方法名语义化（动词+名词）、类/BO/DO 命名贴合业务
- 依赖注入: 优先 `@Resource`，禁止字段注入以外的方式

#### 典型业务场景示例

**场景一：子业务拆分整合（发票业务）**

```
finance-core/src/main/java/com/dahuangf/finance/core/invoice/
├── bo/
│   └── InvoiceBO.java
├── enums/
│   └── InvoiceTypeEnum.java
├── service/                          # 子模块目录
│   ├── issue/
│   │   └── InvoiceIssueService.java
│   ├── adjust/
│   │   └── InvoiceAdjustService.java
│   ├── redflush/
│   │   └── InvoiceRedFlushService.java
│   └── third/
│       └── BaiwangCloudService.java
└── InvoiceService.java               # 对外整合 Service（与 service 目录同级）
```

**场景二：策略多选一（在线支付）**

```
finance-core/src/main/java/com/dahuangf/finance/core/onlinepay/
├── bo/
│   └── PaymentBO.java
├── enums/
│   └── PayChannelEnum.java
├── contracts/
│   └── PaymentStrategy.java
├── service/                          # 子模块目录
│   ├── PaymentFactory.java
│   └── channel/
│       ├── AlipayService.java
│       ├── WechatPayService.java
│       └── UnionPayService.java
└── OnlinePayService.java             # 对外整合 Service（与 service 目录同级）
```

#### BO 规范

BO 字段必须与 DO 字段完全一致：

```java
import lombok.Data;

@Data
public class InvoiceApplyBO {
    private Long id;
    private String invoiceNo;
    private BigDecimal amount;
    private Integer status;
    private LocalDateTime applyTime;
    // 禁止包含 DAO/Service 依赖
    // 禁止包含远程调用/数据库操作
}

@Data
public class InvoiceApplyBatchAddBO {
    private String invoiceNo;
    private List<Long> itemIds;
}

@Data
public class InvoiceApplyUpdateBO {
    private List<Long> ids;
    private Integer status;
}
```

#### 输出要求

1. 仅输出完整的 `{业务语义}Service.java` 代码，无任何解释、注释、```java 代码块标记
2. 流式生成: 按「包名 → 导入类 → 类注解 → 依赖注入 → 方法定义」顺序逐行输出
3. 导入类: 必须使用全限定名（如 `com.dahuangf.common.utils.AssertUtil`、`cn.hutool.core.collection.CollectionUtil`）
4. 代码合规: 无语法错误、可直接编译运行，严格遵循上述所有约束和阿里巴巴 Java 开发手册
5. 扩展性: 保留空的扩展逻辑块（如 DO 转 BO 的字段填充），便于后续业务扩展

业务域示例: aging, billing, commission, contract, cost, invoice, nc, onlinepay, paycollection, performance, profit, receipt, reimbursement, revenue, settlement, special, workflow...

---

### 3. finance-api (对外接口定义)

路径: `com.dahuangf.finance.api`

```
finance-api/src/main/java/com/dahuangf/finance/api/
├── enums/           # 对外枚举
├── model/           # DTO/TO 对象
│   ├── {业务域}/    # 按业务域分子目录
│   └── XxxTO.java   # 通用 TO 可直接放 model 根目录
├── result/          # 通用返回结果
└── service/         # RPC 接口定义 (Dubbo)
    ├── {业务域}/    # 按业务域分子目录
    └── XxxRpc.java  # RPC 接口可直接放 service 根目录
```

---

### 4. finance-interfaces (Web 层/主应用入口)

路径: `com.dahuangf.finance.interfaces`

```
finance-interfaces/src/main/java/com/dahuangf/finance/interfaces/
├── controller/      # REST 控制器
│   ├── {业务域}/    # 按业务域分子目录
│   ├── vo/          # Controller 专用 VO
│   └── XxxController.java
├── vo/              # 视图对象
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
```

---

### 5. finance-manager (依赖管理层)

仅用于 pom.xml 依赖管理和资源加载，不包含业务代码。

---

### 6. finance-common (公共模块)

路径: `com.dahuangf.finance.common`

```
finance-common/src/main/java/com/dahuangf/finance/common/
├── constants/       # 常量类
├── enums/           # 通用枚举
├── utils/           # 工具类
└── thread/          # 线程相关
```

---

### 7. finance-task (定时任务独立应用)

路径: `com.dahuangf.finance.task`

独立部署的定时任务应用，上下文路径: `/finance-task`

---

## 全局命名规范汇总

| 类型 | 规则 | 示例 |
|------|------|------|
| 数据库表 | finance_ + 业务语义 | finance_invoice_apply |
| DO | 业务语义 + DO | InvoiceApplyDO |
| Dao | 业务语义 + Dao | InvoiceApplyDao |
| 基础 BO | 业务语义 + BO | InvoiceApplyBO |
| 批量新增 BO | 业务语义 + BatchAddBO | InvoiceApplyBatchAddBO |
| 更新 BO | 业务语义 + UpdateBO | InvoiceApplyUpdateBO |
| 枚举 | 大驼峰 + Enum | InvoiceTypeEnum |
| 契约接口 | 大驼峰 + Strategy/Handler/Service | PaymentStrategy |
| 对外 Service | 业务域 + Service | InvoiceService |
| 子模块 Service | 子功能 + Service | InvoiceIssueService |
| Controller | 业务名 + Controller | InvoiceController |
| RPC 接口 | 业务名 + Rpc | FinanceInvoiceRpc |
| RPC 实现 | RPC接口 + Impl | FinanceInvoiceRpcImpl |
| TO/DTO | 业务名 + TO/DTO | FinanceInvoiceTO |
| VO | 业务名 + VO | InvoiceVO |
| 工厂类 | 业务名 + Factory | PaymentFactory |

---

## 代码生成任务

【在此描述你的需求】

示例:
- 新增一个 XXX 功能，包含增删改查接口
- 为 XXX 表生成 Entity、Mapper、Service、Controller
- 实现 XXX 业务逻辑
