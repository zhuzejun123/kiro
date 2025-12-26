---
inclusion: fileMatch
fileMatchPattern: "**/*-core/**/*.java"
---

# core (核心业务层)

路径: `{BASE_PACKAGE}.core.{业务域}`

```
{MODULE}-core/src/main/java/{BASE_PACKAGE}/core/{业务域}/
├── bo/                              # 业务对象 (Business Object)
├── enums/                           # 业务枚举
├── contracts/                       # 契约/接口定义（策略接口、扩展点）
├── validate/                        # 校验逻辑
├── service/                         # 子模块 Service 目录
│   └── {子模块功能}/                # 业务域下若干子模块核心业务层文件夹
│       └── {子模块名称}Service.java
├── {业务域}Service.java             # 业务域应用层 Service（对外统一入口）
├── {业务域}BaseService.java         # 业务域基础层 Service（通用能力封装）
└── {业务域}AdapterService.java      # 业务域适配层 Service（外部系统对接）
```

**变量说明**:
- `{MODULE}`: finance 或 sale
- `{BASE_PACKAGE}`: com.dahuangf.finance 或 com.dahuangf.sale（路径中 `.` 替换为 `/`）

---

## 一、设计原则

**核心理念**: 高内聚低耦合 + 可维护/可扩展 + 简单实用，不做过度设计，明确每一个功能域的边界

### 1.1 分层职责

**简单场景**: 仅基础增删改查（如配置项管理），可直接在外层 Service 完成，无需拆分子模块

**复杂场景**: 必须遵循以下原则：

| 层级 | 职责 | 说明 |
|------|------|------|
| 外层 XxxService | 子模块整合 + 对外统一入口 + 轻量级业务 | 包括：过滤、判断、简单业务判断、权限控制、BO 转换、DAO 数据查询、同级 Service 调用等 |
| 子模块 Service | 复杂需求场景的功能拆分 | 按业务子功能拆分，不一定只做一件事，可包含完整子业务流程 |
| BO 模块 | 仅封装业务数据 | 禁止包含 DAO/Service 依赖；禁止包含远程调用/数据库操作 |

### 1.2 层间交互规则

**依赖方向**（单向依赖，反向不依赖）:
```
应用层 → 核心业务层 → 外部适配层 → 基础层
   或
应用层 → 核心业务层 → 基础层
```

**异常处理链路**:
```
适配层捕获外部API异常 → 基础层转换为统一异常 → 核心层判断业务影响 → 应用层统一返回给参与者
```

**扩展触发机制**:
- 通过 "配置 + 接口" 触发扩展逻辑
- 示例：配置 "外部平台 = 百旺"，应用层自动调用百旺适配实现

---

## 二、Service 四层结构设计

### 2.1 应用层（{业务域}Service）

**承接内容**: 用例的完整流程编排

**职责范围**:
- 用例全流程串联（前置校验→核心逻辑→后置操作）
- 对外提供统一接口
- 跨服务交互（调用外部、被外部调用）

**设计要点**:
- 价值型：适配参与者需求（例如：发起开票、发起调票）
- 非功能性：接口权限管控（满足安全性）
- 通用性：统一接口规范，适配所有参与者的调用方式

### 2.2 核心业务层（service/{子模块功能}/）

**承接内容**: 核心用例（例如：开票/调票）

**职责范围**:
- 业务规则校验（例如：红冲条件、金额限制）
- 业务状态流转（例如：发票单出单状态流转）
- 业务结果处理

**设计要点**:
- 目标性：所有逻辑围绕用例的核心目标
- 价值型：聚焦核心业务规则，不掺杂技术细节
- 完整性：覆盖正常/异常业务场景（如已作废发票不可红冲）
- 扩展性：预留规则配置化入口（如单张发票金额上限可配置）

### 2.3 外部适配层（{业务域}AdapterService）

**承接内容**: 二方RPC、三方外部 API 对接用例（如果存在的话）

**职责范围**:
- API 请求封装/响应解析
- 超时/重试/降级处理/性能/错误自包含
- 多平台适配（例如：航天金税/百旺）

**设计要点**:
- 完整性：覆盖 API 调用的正常/异常场景
- 扩展性：定义通用对接接口，新增平台仅需实现接口
- 非功能性：设置 API 超时（≤3s）、重试 3 次（满足可靠性）、传输加密（满足安全性）

**二方 RPC 对接流程**（必须遵循）:

生成 Adapter 代码时，必须按以下流程处理二方 RPC 调用：

1. **检查 rpc-registry.md**: 查看 `#[[file:.kiro/powers/backend-ai/steering/rpc-registry.md]]` 是否已有对应接口定义
2. **向用户确认**: 列出所有需要调用的二方 RPC，询问用户是否能提供接口全路径和方法签名
3. **生成代码**:
   - 用户提供接口信息 → 生成完整的 `@DubboReference` 注入和调用代码
   - 用户无法提供 → 生成 TODO 占位代码，注释中说明预期功能、入参、出参

**确认问题示例**:
```
## 二方 RPC 接口确认

在生成 Adapter 层代码时，发现以下二方 RPC 调用需要确认：

| 序号 | 服务名称 | 预期功能 | 状态 |
|------|----------|----------|------|
| 1 | 审批服务 | 创建审批流程、查询审批状态 | 待确认 |
| 2 | 客户服务 | 查询客户信息 | 待确认 |

请提供以下信息（如无法提供则留空，将生成 TODO 占位）：
1. RPC 接口全路径（如 `com.xxx.api.ApprovalRpcService`）
2. 方法签名（方法名、入参类型、返回类型）
```

### 2.4 基础层（{业务域}BaseService）

**承接内容**: 所有用例的通用能力（前提：业务需求中有较多的通用性）

**职责范围**:
- 通用校验（例如：税号、金额、商品编码）
- 通用规则（例如：发票号生成/验真）
- 通用工具（例如：加密、日志、异常转换）

**设计要点**:
- 通用性：所有规则/工具无业务属性，可复用至所有用例
- 非功能性：封装数据加密能力（满足合规性）、日志标准化（满足可审计性）
- 扩展性：预留规则配置接口（如税号校验规则可配置）

**注意**: 若简单需求没有必要抽一层基础层，可将通用能力移动至核心业务层

---

## 三、Service 编码规范

### 3.1 类定义规范

```java
@Service  // 无其他多余注解
public class XxxService {
    @Resource
    private XxxDao xxxDao;  // 依赖注入使用 @Resource
    
    @Resource
    private YyyService yyyService;  // 关联 Service
}
```

### 3.2 事务注解规范（@Transactional）

对于业务应用层（{业务域}Service）、子模块核心业务层（service/{子模块}/）中涉及数据变更的业务场景（新增、删除、修改），需要根据以下规则决定是否添加事务注解：

**必须添加 @Transactional 的场景**:
- 操作多个表的数据变更（前提条件）
- 方法内部没有调用二方 RPC 或三方 API

**可以不加 @Transactional 的场景**:
- 方法内部调用了二方 RPC（如 Dubbo 服务）或三方 API（如外部 HTTP 接口）
- 原因：外部调用可能导致事务长时间无法释放，影响数据库连接池和系统性能

**判断流程**:
```
是否涉及多表数据变更？
├── 否 → 无需事务
└── 是 → 方法内部是否调用二方RPC/三方API？
    ├── 是 → 不加事务（避免长事务）
    └── 否 → 必须加 @Transactional
```

**代码示例**:

```java
// 场景1：多表操作 + 无外部调用 → 必须加事务
@Transactional(rollbackFor = Exception.class)
public void applyInvoice(InvoiceApplyBO bo) {
    // 保存发票申请主表
    invoiceApplyDao.save(applyDO);
    // 保存发票明细表
    invoiceDetailDao.saveBatch(detailDOList);
}

// 场景2：多表操作 + 有外部调用 → 不加事务
public void applyInvoiceWithExternal(InvoiceApplyBO bo) {
    // 保存发票申请主表
    invoiceApplyDao.save(applyDO);
    // 调用外部发票平台（二方RPC/三方API）
    invoiceAdapterService.callExternalPlatform(bo);
    // 更新发票状态
    invoiceApplyDao.updateById(applyDO);
}

// 场景3：单表操作 → 无需事务
public int update(InvoiceApplyUpdateReqVO reqVO) {
    InvoiceApplyDO invoiceApplyDO = CopyUtil.copyBean(reqVO, InvoiceApplyDO.class);
    return invoiceApplyDao.updateById(invoiceApplyDO) ? 1 : 0;
}
```

**注意事项**:
1. 使用 `@Transactional(rollbackFor = Exception.class)` 确保所有异常都能触发回滚
2. 对于有外部调用的场景，如需保证数据一致性，考虑使用最终一致性方案（如消息队列、定时任务补偿）
3. 避免在事务方法中进行耗时操作（如大量循环、复杂计算）

### 3.3 注释规范

所有 Service 类和方法必须添加清晰的 JavaDoc 注释，注释应简洁明了，避免冗余。

#### 3.3.1 类注释规范

```java
/**
 * {业务域}应用层服务
 * <p>职责：流程编排、对外统一入口、跨服务交互</p>
 * <p>承接用例：{列出主要用例}</p>
 *
 * @author {作者}
 * @since {日期}
 */
@Service
public class InvoiceService {
}
```

**各层类注释模板**:

| 层级 | 类注释模板 |
|------|------------|
| 应用层 | `{业务域}应用层服务 - 流程编排、对外统一入口` |
| 基础层 | `{业务域}基础层服务 - 通用校验、工具方法、规则封装` |
| 适配层 | `{业务域}适配层服务 - 外部系统对接、API封装` |
| 核心业务层 | `{子模块}核心业务服务 - {具体业务职责}` |

#### 3.3.2 方法注释规范

**注释要素**:
| 要素 | 说明 | 是否必填 |
|------|------|----------|
| 方法职责 | 一句话说明方法做什么 | 必填 |
| 业务规则 | 关键业务规则说明 | 按需 |
| @param | 参数说明 | 必填 |
| @return | 返回值说明 | 必填 |
| @throws | 异常说明 | 按需 |

**方法注释模板**:

```java
/**
 * 发起开票申请
 * <p>业务规则：单张发票金额不超过10万，同一订单24小时内不可重复开票</p>
 *
 * @param bo 开票申请信息
 * @return 开票申请结果，包含发票申请ID
 * @throws BizException 当税号校验失败或金额超限时抛出
 */
public InvoiceApplyResultBO applyInvoice(InvoiceApplyBO bo) {
}
```

**各类方法注释示例**:

```java
// 查询类方法
/**
 * 分页查询发票列表
 *
 * @param pageParam 分页参数
 * @param queryWrapper 查询条件
 * @return 发票分页数据
 */
public IPage<InvoiceBO> page(PageParam pageParam, LambdaQueryWrapper<InvoiceDO> queryWrapper) {
}

// 保存类方法
/**
 * 批量保存发票申请
 * <p>业务规则：发票编号不可重复</p>
 *
 * @param bo 发票申请信息
 * @throws BizException 当发票编号已存在时抛出
 */
public void batchSave(InvoiceBO bo) {
}

// 更新类方法
/**
 * 更新发票状态
 * <p>业务规则：已作废/已红冲的发票不可更新</p>
 *
 * @param bo 发票更新信息，必须包含id
 */
public void update(InvoiceBO bo) {
}

// 校验类方法（基础层）
/**
 * 校验纳税人识别号格式
 *
 * @param taxNo 纳税人识别号
 * @return true-格式正确，false-格式错误
 */
public boolean validateTaxNo(String taxNo) {
}

// 外部调用方法（适配层）
/**
 * 调用百旺云平台开具发票
 * <p>超时设置：3秒，失败重试：3次</p>
 *
 * @param request 开票请求参数
 * @return 平台返回的开票结果
 * @throws BizException 当平台调用失败或超时时抛出
 */
public BaiwangInvoiceResponse issueInvoice(BaiwangInvoiceRequest request) {
}
```

**注释原则**:
1. 简洁明了：一句话说清方法职责，避免废话
2. 突出重点：只写关键业务规则，不写显而易见的内容
3. 参数完整：每个参数都要有说明
4. 异常明确：可能抛出的业务异常要说明触发条件

### 3.4 单表操作约束（必须严格遵循）

**核心规则**: 所有 CRUD 数据库操作必须是单表操作，禁止关联查询和子查询。

**禁止的操作**:
- ❌ JOIN 关联查询（LEFT JOIN、INNER JOIN、RIGHT JOIN）
- ❌ 子查询（WHERE id IN (SELECT ...)）
- ❌ 多表联合更新/删除
- ❌ 在 Wrapper 中使用 `inSql`、`apply` 等方法拼接 SQL
- ❌ 使用 `like`、`likeLeft`、`likeRight` 进行模糊查询
- ❌ 任何形式的 SQL 拼接（字符串拼接、动态 SQL）

**正确的做法**:
- ✅ 将多表查询拆解为多步单表操作
- ✅ 先查询主表，再根据关联 ID 查询从表
- ✅ 在 Java 代码中进行数据组装

**错误示例**（禁止）:
```java
// ❌ 禁止：子查询
wrapper.inSql(OrderDO::getUserId, "SELECT id FROM user WHERE status = 1");

// ❌ 禁止：拼接 SQL
wrapper.apply("date_format(create_time, '%Y-%m-%d') = {0}", dateStr);

// ❌ 禁止：模糊查询
wrapper.like(OrderDO::getOrderNo, keyword);
wrapper.likeRight(OrderDO::getCustomerName, keyword);

// ❌ 禁止：在 Service 中期望 Dao 执行关联查询
List<OrderDetailVO> list = orderDao.selectOrderWithItems(orderId);
```

**正确示例**:
```java
// ✅ 正确：拆解为多步单表操作
public OrderDetailBO getOrderDetail(Long orderId) {
    // 第一步：查询订单主表
    OrderDO orderDO = orderDao.getById(orderId);
    if (orderDO == null) {
        return null;
    }
    
    // 第二步：根据订单ID查询订单明细表
    LambdaQueryWrapper<OrderItemDO> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(OrderItemDO::getOrderId, orderId);
    List<OrderItemDO> itemList = orderItemDao.list(wrapper);
    
    // 第三步：在 Java 代码中组装数据
    OrderDetailBO detailBO = CopyUtil.copyBean(orderDO, OrderDetailBO.class);
    detailBO.setItems(CopyUtil.copyList(itemList, OrderItemBO.class));
    return detailBO;
}

// ✅ 正确：批量查询场景
public List<OrderBO> listOrdersWithUser(List<Long> orderIds) {
    // 第一步：批量查询订单
    List<OrderDO> orderList = orderDao.listByIds(orderIds);
    
    // 第二步：提取用户ID，批量查询用户
    List<Long> userIds = orderList.stream()
            .map(OrderDO::getUserId)
            .distinct()
            .collect(Collectors.toList());
    List<UserDO> userList = userDao.listByIds(userIds);
    Map<Long, UserDO> userMap = userList.stream()
            .collect(Collectors.toMap(UserDO::getId, Function.identity()));
    
    // 第三步：在 Java 代码中组装数据
    return orderList.stream().map(order -> {
        OrderBO bo = CopyUtil.copyBean(order, OrderBO.class);
        UserDO user = userMap.get(order.getUserId());
        if (user != null) {
            bo.setUserName(user.getName());
        }
        return bo;
    }).collect(Collectors.toList());
}
```

**设计原则**:
1. 每次 Dao 调用只操作一张表
2. 关联数据通过多次单表查询 + Java 内存组装实现
3. 批量场景使用 `listByIds` 或 `wrapper.in()` 减少查询次数
4. 复杂聚合统计考虑使用缓存或异步计算，而非复杂 SQL

---

## 四、默认 CRUD 方法模板（必须严格遵循）

Service 默认提供与 Controller 一一对应的 CRUD 方法，方法名与 Controller 保持一致。

**重要约束**：
- 方法名必须与 Controller 方法名**完全一致**，不允许自定义命名（如 `pageList`、`getDetail`、`findById` 等）
- 入参和出参类型必须严格按照下表定义，不允许自行扩展参数
- 如需扩展功能，应新增独立方法，而非修改默认 CRUD 方法签名

### 4.1 Service 与 Controller 方法对应关系（特别特别重要）

| Controller 方法 | Service 方法 | Service 入参 | Service 出参 |
|-----------------|--------------|--------------|--------------|
| `page` | `page` | `PageParam`, `LambdaQueryWrapper<XxxDO>` | `IPage<XxxBO>` |
| `detail` | `detail` | `Long id` | `XxxBO` |
| `add` | `add` | `XxxCreateReqVO` | `int` |
| `update` | `update` | `XxxUpdateReqVO` | `int` |
| `delete` | `delete` | `Long id` | `int` |

**禁止的命名方式**（错误示例）:
- ❌ `pageList` → 应使用 `page`
- ❌ `getDetail` / `getById` / `findById` → 应使用 `detail`
- ❌ `create` / `save` / `insert` → 应使用 `add`
- ❌ `modify` / `edit` → 应使用 `update`
- ❌ `remove` / `del` → 应使用 `delete`

### 4.2 分页查询方法（page）

- 入参: `PageParam pageParam, LambdaQueryWrapper<{业务语义}DO> queryWrapper`
- 出参: `IPage<{业务语义}BO>`
- 核心逻辑:
  1. Controller 层调用 VO 的 `toLambdaQueryWrapper()` 生成查询条件后传入
  2. 调用 Dao 的 page 方法: `dao.page(pageParam.getPage(), queryWrapper)`
  3. 通过 `CopyUtil.copyPageInfo` 转换 DO 分页为 BO 分页

```java
/**
 * 分页查询发票列表
 *
 * @param pageParam 分页参数
 * @param queryWrapper 查询条件（由 Controller 层调用 VO.toLambdaQueryWrapper() 生成）
 * @return 发票分页数据
 */
public IPage<InvoiceApplyBO> page(PageParam pageParam, LambdaQueryWrapper<InvoiceApplyDO> queryWrapper) {
    IPage<InvoiceApplyDO> doPage = invoiceApplyDao.page(pageParam.getPage(), queryWrapper);
    IPage<InvoiceApplyBO> boPage = CopyUtil.copyPageInfo(doPage, InvoiceApplyBO.class);
    // 扩展字段填充（预留扩展点）
    return boPage;
}
```

**关键说明**: `toLambdaQueryWrapper()` 在 Controller 层调用，Service 层直接接收 `LambdaQueryWrapper` 参数。

### 4.3 新增方法（add）

- 入参: `{业务语义}CreateReqVO reqVO`（Controller 直接传入 VO）
- 出参: `int`（新增记录数）
- 核心逻辑:
  1. VO 转 BO: `CopyUtil.copyBean(reqVO, XxxBO.class)`
  2. 业务唯一性校验: 通过 `LambdaQueryWrapper` + `Dao.getOne` 查询
  3. BO 转 DO 后调用 `Dao.save`

```java
/**
 * 新增发票申请
 *
 * @param reqVO 新增请求参数
 * @return 新增记录数
 */
public int add(InvoiceApplyCreateReqVO reqVO) {
    // VO → BO
    InvoiceApplyBO bo = CopyUtil.copyBean(reqVO, InvoiceApplyBO.class);
    
    // 业务唯一性校验
    LambdaQueryWrapper<InvoiceApplyDO> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(InvoiceApplyDO::getInvoiceNo, bo.getInvoiceNo());
    InvoiceApplyDO existDo = invoiceApplyDao.getOne(wrapper);
    AssertUtil.ifThrow(Objects.nonNull(existDo), "该发票编号已存在");
    
    // BO → DO，保存
    InvoiceApplyDO invoiceApplyDO = CopyUtil.copyBean(bo, InvoiceApplyDO.class);
    return invoiceApplyDao.save(invoiceApplyDO) ? 1 : 0;
}
```

### 4.4 更新方法（update）

- 入参: `{业务语义}UpdateReqVO reqVO`（Controller 直接传入 VO）
- 出参: `int`（更新记录数）
- 核心逻辑:
  1. VO 转 BO: `CopyUtil.copyBean(reqVO, XxxBO.class)`
  2. 数据存在性校验: `Dao.getById(bo.getId())`
  3. BO 转 DO 后调用 `Dao.updateById`

```java
/**
 * 更新发票申请
 *
 * @param reqVO 更新请求参数
 * @return 更新记录数
 */
public int update(InvoiceApplyUpdateReqVO reqVO) {
    // VO → BO
    InvoiceApplyBO bo = CopyUtil.copyBean(reqVO, InvoiceApplyBO.class);
    
    // 数据存在性校验
    InvoiceApplyDO existDo = invoiceApplyDao.getById(bo.getId());
    if (Objects.isNull(existDo)) {
        return 0;
    }
    
    // BO → DO，更新
    InvoiceApplyDO invoiceApplyDO = CopyUtil.copyBean(bo, InvoiceApplyDO.class);
    return invoiceApplyDao.updateById(invoiceApplyDO) ? 1 : 0;
}
```

### 4.5 详情查询方法（detail）

- 入参: `Long id`（主键）
- 出参: `{业务语义}BO`
- 核心逻辑:
  1. `Dao.getById(id)` 查询 DO
  2. `CopyUtil.copyBean` 将 DO 转换为 BO 返回

```java
/**
 * 查询发票详情
 *
 * @param id 发票ID
 * @return 发票详情
 */
public InvoiceApplyBO detail(Long id) {
    InvoiceApplyDO invoiceApplyDO = invoiceApplyDao.getById(id);
    return CopyUtil.copyBean(invoiceApplyDO, InvoiceApplyBO.class);
}
```

### 4.6 删除方法（delete）

- 入参: `Long id`（主键）
- 出参: `int`（删除记录数）
- 核心逻辑:
  1. 调用 `Dao.removeById(id)` 删除

```java
/**
 * 删除发票申请
 *
 * @param id 发票ID
 * @return 删除记录数
 */
public int delete(Long id) {
    return invoiceApplyDao.removeById(id) ? 1 : 0;
}
```

---

## 五、BO 规范

- BO 字段必须与 DO 字段完全一致
- 只需要一个 `{业务语义}BO`，不需要 BatchAddBO、UpdateBO 等
- Controller 层的 VO（CreateReqVO、UpdateReqVO）通过 `CopyUtil` 转换为 BO 传入 Service

```java
import lombok.Data;

import java.time.LocalDateTime;

@Data
public class InvoiceApplyBO {
    private Long id;
    private String invoiceNo;
    private Long amount;
    private Integer status;
    private LocalDateTime applyTime;
    // 禁止包含 DAO/Service 依赖
    // 禁止包含远程调用/数据库操作
}
```

### 5.1 禁止内部类（重要约束）

**Core 层所有类（Service、Adapter、Calculator 等）禁止创建内部类**：
- 所有数据传输对象必须定义为独立的 BO 类，放在 `bo/` 目录下
- 包括但不限于：请求参数对象、响应结果对象、中间数据对象
- 即使是 Adapter 层的请求/响应对象，也必须定义为独立的 BO 类

**错误示例**（禁止）:
```java
@Component
public class ApprovalAdapter {
    // ❌ 禁止：内部类定义
    @Data
    public static class ApprovalRequest {
        private String businessType;
        private Long businessId;
    }
    
    // ❌ 禁止：内部类定义
    @Data
    public static class ApprovalStatus {
        private String approvalNo;
        private Integer status;
    }
}
```

**正确示例**:
```java
// bo/ApprovalRequestBO.java
@Data
public class ApprovalRequestBO {
    private String businessType;
    private Long businessId;
}

// bo/ApprovalStatusBO.java
@Data
public class ApprovalStatusBO {
    private String approvalNo;
    private Integer status;
}

// adapter/ApprovalAdapter.java
@Component
public class ApprovalAdapter {
    public String createApproval(ApprovalRequestBO request) { ... }
    public ApprovalStatusBO getApprovalStatus(String approvalNo) { ... }
}
```

---

## 六、命名规范

**DO 命名**:
- 实体类: `{业务语义}DO`，如 `InvoiceApplyDO`、`SettlementRecordDO`

**BO 命名**:
- `{业务语义}BO`，如 `InvoiceApplyBO`、`SettlementRecordBO`
- 字段与 DO 完全一致，Controller 层 VO 通过 `CopyUtil` 转换为 BO 传入 Service

**方法命名**: 严格语义化，禁止模糊动词
- 分页查询: `page`
- 批量保存: `batchSave`
- 更新: `update`
- 详情查询: `getDetail`

---

## 七、用例驱动的 Service 分层设计方法（必须遵循）

本方法将用例分析与 Service 模块层次设计深度结合，确保每一层的职责都能精准承接用例需求。

### 7.1 第一步：明确系统边界，识别参与者（Actor）

参与者是与系统发生交互的外部实体（人、其他系统、设备、定时任务），是用例分析的起点。

**参与者分类**:
| 类型 | 说明 | 示例 |
|------|------|------|
| 主要参与者 | 发起核心业务操作、直接达成业务目标的实体 | 财务人员、运营人员、用户 |
| 次要参与者 | 辅助主要参与者完成目标的实体 | 支付系统、发票平台、物流系统 |
| 被动参与者 | 被系统行为影响的实体 | 短信平台、消息通知系统 |

**关键规则**:
- 排除内部元素：系统内部模块（如 "订单模块"）、数据（如 "订单数据"）不是参与者
- 列出所有与系统有交互的外部实体，标注类型

**输出格式**:
```
## 参与者识别

| 参与者 | 类型 | 交互说明 |
|--------|------|----------|
| 财务人员 | 主要 | 发起开票、调票操作 |
| 百旺云平台 | 次要 | 提供发票开具能力 |
| 短信平台 | 被动 | 接收开票成功通知 |
```

### 7.2 第二步：为每个参与者提炼核心用例（Use Case）

用例是 "参与者与系统的一系列交互动作，最终达成一个具体的业务目标"。

**用例命名规则**: 动宾结构（如 "用户下外卖订单"、"骑手接单"）

**筛选标准**:
| 标准 | 说明 | 正例 | 反例 |
|------|------|------|------|
| 价值性 | 对业务有实际意义 | 用户完成外卖下单支付 | 用户滚动订单列表 |
| 完整性 | 能独立完成一个目标 | 用户查看订单详情 | 用户点击支付按钮 |
| 粒度适中 | 不过细也不过大 | 下单、支付、接单 | 外卖系统全流程 |

**输出格式**:
```
## 用例清单

| 参与者 | 用例名称 | 业务目标 |
|--------|----------|----------|
| 财务人员 | 发起开票申请 | 完成发票开具 |
| 财务人员 | 发起红冲申请 | 完成发票红冲 |
| 定时任务 | 同步发票状态 | 更新发票最新状态 |
```

### 7.3 第三步：细化用例规约（消除歧义的核心）

用例规约是用例的详细描述，确保开发、测试、产品等干系人对需求理解一致。

**用例规约要素**:
| 要素 | 说明 |
|------|------|
| 用例名称 | 唯一标识（如 "用户下外卖订单"） |
| 参与者 | 关联的参与者（如 "用户、支付系统"） |
| 前置条件 | 执行用例前必须满足的状态 |
| 后置条件 | 用例成功执行后系统的状态 |
| 基本流程 | 正常场景的步骤（编号 + 参与者动作 + 系统响应） |
| 扩展流程 | 异常/可选场景及处理逻辑 |
| 非功能约束 | 关联的非功能性需求（如 "支付响应时间≤3秒"） |

**输出格式示例**:
```
## 用例规约：财务人员发起开票申请

### 参与者
财务人员、百旺云平台、定时任务

### 前置条件
1. 用户已登录
2. 开票信息已完善（税号、抬头等）
3. 开票金额在限额范围内

### 后置条件
1. 发票状态为"开票中"
2. 生成发票申请记录
3. 调用外部平台发起开票

### 基本流程
1. 财务人员填写开票信息；系统校验信息完整性
2. 财务人员点击"提交开票"；系统校验开票规则
3. 系统调用百旺云平台发起开票
4. 系统更新发票状态为"开票中"，返回申请结果
5. 定时任务定时轮询状态为开票失败，重试开票

### 扩展流程
- 扩展1：税号校验失败 → 系统提示"税号格式错误"，返回填写页
- 扩展2：开票金额超限 → 系统提示"单张发票金额超过上限"
- 扩展3：外部平台调用失败 → 系统记录失败原因，状态置为"开票失败"
```

### 7.4 第四步：梳理用例关系，消除冗余与冲突

**用例关系类型**:
| 关系 | 说明 | 示例 |
|------|------|------|
| 包含关系 | 基础用例被其他用例复用 | "用户登录" 被 "下单"、"评价" 包含 |
| 扩展关系 | 主用例的可选分支 | "下单" 扩展 "使用优惠券抵扣" |
| 泛化关系 | 父用例的通用行为，子用例特化 | "支付" 泛化为 "支付宝支付"、"微信支付" |

**检查要点**:
- 检查是否有重复用例（如 "用户支付" 和 "订单支付" 合并为一个）
- 确认用例依赖是否合理（如 "骑手接单" 必须依赖 "用户下单且支付完成"）

### 7.5 第五步：验证用例的完整性与一致性

**验证维度**:
| 维度 | 验证方法 |
|------|----------|
| 业务覆盖 | 对照原始需求文档，用「需求跟踪矩阵（RTM）」关联需求点与用例，确保 100% 覆盖 |
| 场景覆盖 | 每个用例的正常流程 + 异常流程是否完整 |
| 状态一致 | 用例之间的系统状态衔接合理 |
| 参与者覆盖 | 无孤立参与者（每个参与者至少对应 1 个用例） |

---

## 八、用例分析成果与 Service 分层映射（6 个核心关注点）

### 8.1 用例边界 → Service 职责边界（完整性 + 通用性）

**核心逻辑**: 一个用例组（同类业务目标）对应一层/一个核心 Service 的核心职责

**示例（发票需求）**:
- 开票用例、调票用例 → 核心业务层
- 外部 API 对接用例 → 外部适配层
- 所有用例都需要的 "税号校验、金额格式化" → 基础层

### 8.2 用例流程 → Service 层间交互链路（目标性 + 完整性）

**核心逻辑**: 用例的 "前置条件→基本流程→后置条件→扩展流程" 对应 Service 层的调用链路

**示例（开票用例）**:
```
用例流程：校验用户信息 → 校验开票规则 → 调用外部API → 更新发票状态 → 通知用户

Service链路：
应用层 → 基础层（校验用户信息）
      → 核心业务层（校验开票规则）
      → 外部适配层（调用API）
      → 核心业务层（更新状态）
      → 应用层（通知用户）
```

### 8.3 用例通用流程 → Service 通用提炼（通用性 + 价值型）

**核心逻辑**: 所有用例重复出现的流程/规则，统一提炼到基础层

**示例**:
- 开票、调票用例都需要 "发票号码验真" → 放到基础层
- 所有外部交互用例都需要 "API 签名/验签" → 签名规则放到基础层，适配层仅负责调用

### 8.4 用例扩展场景 → Service 扩展点预留（扩展性）

**核心逻辑**: 用例的扩展流程对应 Service 层的扩展设计，预留接口/配置化规则

**示例**:
- "对接新的外部发票平台" → 适配层定义通用接口，新增平台仅需实现接口
- "部分红冲" → 核心业务层预留 "红冲类型" 扩展字段

### 8.5 用例非功能约束 → Service 质量属性落地（非功能性）

**核心逻辑**: 用例中的非功能约束拆解到对应 Service 层

**示例**:
- "API 调用超时≤3s" → 适配层设置超时时间 + 重试策略
- "发票数据不可篡改" → 基础层封装加密/签名能力

### 8.6 用例参与者 → Service 对外接口设计（价值型 + 易用性）

**核心逻辑**: 用例的参与者对应应用层的对外接口

**示例**:
- 参与者 "财务" 需要批量开票 → 应用层提供 "批量开票接口"
- 参与者 "外部系统" 需要查询发票 → 应用层提供标准化 API 接口

---

## 九、分层设计确认流程（必须遵循）

### 9.1 用例模型确认

完成用例分析后，需输出以下内容并等待用户确认：

```
## 用例模型确认

### 1. 参与者清单
| 参与者 | 类型 | 交互说明 |
|--------|------|----------|

### 2. 用例清单
| 用例编号 | 用例名称 | 参与者 | 业务目标 |
|----------|----------|--------|----------|

### 3. 用例关系图
（描述用例间的包含、扩展、泛化关系）

### 4. 拆分理由
（说明为什么这样识别参与者和用例）

### 确认问题
请确认以上用例模型是否完整，如需调整请说明。
```

### 9.2 分层拆分确认

完成 Service 分层设计后，需输出以下内容并等待用户确认：

```
## 分层拆分确认

### 1. 分层结构
| 层级 | Service 名称 | 承接用例 | 核心职责 |
|------|--------------|----------|----------|
| 应用层 | {业务域}Service | 全部用例 | 流程编排、为其他业务域提供能力 |
| 核心业务层 | {子模块}Service | 核心用例 | 业务规则、状态流转 |
| 适配层 | {业务域}AdapterService | 系统外部第三方对接用例、调用其他业务域高层次服务 | API封装、多平台适配 |
| 基础层 | {业务域}BaseService | 通用能力 | 校验、工具、规则 |

### 2. 拆分理由
（说明为什么这样分层，每层承接哪些用例要素）

### 3. 层间依赖关系
（描述各层之间的调用关系）

### 确认问题
请确认以上分层设计是否合理，如需调整请说明。
```

### 9.3 Service 树结构与方法确认

生成代码前，需输出以下内容并等待用户确认：

```
## Service 树结构与方法确认

### 1. 目录结构
{MODULE}-core/src/main/java/{BASE_PACKAGE}/core/{业务域}/
├── bo/
│   └── {业务}BO.java
├── enums/
│   └── {业务}Enum.java
├── contracts/
│   └── {策略接口}.java（如有）
├── validate/
│   └── {校验类}.java（如有）
├── service/
│   └── {子模块}/
│       └── {子模块}Service.java
├── {业务域}Service.java
├── {业务域}BaseService.java
└── {业务域}AdapterService.java

### 2. 各 Service 方法清单

#### {业务域}Service（应用层）
| 方法名 | 入参 | 出参 | 职责说明 | 调用链路 |
|--------|------|------|----------|----------|

#### {业务域}BaseService（基础层）
| 方法名 | 入参 | 出参 | 职责说明 |
|--------|------|------|----------|

#### {业务域}AdapterService（适配层）
| 方法名 | 入参 | 出参 | 职责说明 |
|--------|------|------|----------|

#### {子模块}Service（核心业务层）
| 方法名 | 入参 | 出参 | 职责说明 |
|--------|------|------|----------|

### 3. 调用关系图
（描述各 Service 之间的调用关系）

### 4. 拆分理由
（说明方法划分的依据，与用例流程的对应关系）

### 确认问题
请确认以上 Service 结构和方法设计是否满足需求，如需调整请说明。
```

---

## 十、典型业务场景示例

### 10.1 场景一：发票业务（含外部对接）

```
{MODULE}-core/src/main/java/{BASE_PACKAGE}/core/invoice/
├── bo/
│   └── InvoiceBO.java
├── enums/
│   └── InvoiceStatusEnum.java
│   └── InvoiceTypeEnum.java
├── contracts/
│   └── InvoicePlatformStrategy.java      # 发票平台策略接口
├── validate/
│   └── InvoiceValidator.java
├── service/
│   ├── issue/
│   │   └── InvoiceIssueService.java      # 开票核心业务
│   ├── redflush/
│   │   └── InvoiceRedFlushService.java   # 红冲核心业务
│   └── query/
│       └── InvoiceQueryService.java      # 查询核心业务
├── InvoiceService.java                   # 应用层：流程编排、提供接口给其他业务域
├── InvoiceBaseService.java               # 基础层：税号校验、金额格式化、Utils基本工具、单号生成器
└── InvoiceAdapterService.java            # 适配层：百旺/航天金税对接、合同业务域的合同信息查询、用户业务域的用户基本信息查询
```

### 10.2 场景二：在线支付（策略多选一）

```
{MODULE}-core/src/main/java/{BASE_PACKAGE}/core/onlinepay/
├── bo/
│   └── PaymentBO.java
├── enums/
│   └── PayChannelEnum.java
├── contracts/
│   └── PaymentStrategy.java              # 支付策略接口
├── service/
│   ├── PaymentFactory.java               # 策略工厂
│   └── channel/
│       ├── AlipayService.java            # 支付宝实现
│       ├── WechatPayService.java         # 微信支付实现
│       └── UnionPayService.java          # 银联实现
├── OnlinePayService.java                 # 应用层
├── OnlinePayBaseService.java             # 基础层：签名、加密
└── OnlinePayAdapterService.java          # 适配层：各支付平台API对接
```

### 10.3 场景三：简单配置管理（无需完整分层）

```
{MODULE}-core/src/main/java/{BASE_PACKAGE}/core/config/
├── bo/
│   └── ConfigBO.java
├── enums/
│   └── ConfigTypeEnum.java
└── ConfigService.java                    # 简单场景仅需应用层，包含基础CRUD
```

---

## 十一、业务域示例

aging, billing, commission, contract, cost, invoice, nc, onlinepay, paycollection, performance, profit, receipt, reimbursement, revenue, settlement, special, workflow...
