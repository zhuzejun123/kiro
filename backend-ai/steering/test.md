---
inclusion: always
---

# 单元测试生成规范

## 概述

为 {MODULE} Platform 项目生成高质量的单元测试代码，确保业务逻辑的正确性和代码的可维护性。

---

## 技术栈

| 组件 | 用途 |
|------|------|
| JUnit 4 | 测试框架（`@Test`、`@RunWith`） |
| Spring Test | Spring 集成测试（`@SpringBootTest`、`SpringRunner`） |
| AssertJ | 断言库（可选，增强断言可读性） |

---

## BaseTest 基类

**路径**: `{MODULE}-interfaces/src/test/java/{BASE_PACKAGE}/test/BaseTest.java`

**生成规则**: 
- 生成测试代码前，先检查 BaseTest.java 是否存在
- 如果不存在，先创建 BaseTest.java
- 如果已存在，跳过创建

```java
package {BASE_PACKAGE}.test;

import {BASE_PACKAGE}.interfaces.Application;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * 测试基类 - 所有测试类必须继承
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class BaseTest {

}
```

**变量说明**:
- `{MODULE}`: finance 或 sale（注意 sale 项目的 interfaces 模块名为 sale-platform-interfaces）
- `{BASE_PACKAGE}`: com.dahuangf.finance 或 com.dahuangf.sale（路径中 `.` 替换为 `/`）

**继承规范**：
```java
// 正确 ✅
public class InvoiceServiceTest extends BaseTest {
    
    @Test
    public void test_add_success() {
        // 测试代码
    }
}

// 错误 ❌ - 不继承 BaseTest
public class InvoiceServiceTest {
}

// 错误 ❌ - 测试方法没有 @Test 注解
public class InvoiceServiceTest extends BaseTest {
    
    public void test_add_success() {  // 缺少 @Test
        // 测试代码
    }
}
```

---

## 测试目录结构

**测试模块**: `{MODULE}-interfaces`
**统一测试包路径**: `{BASE_PACKAGE}.test`

```
{MODULE}-interfaces/src/test/java/{BASE_PACKAGE}/test/
├── BaseTest.java                # 测试基类（所有测试类必须继承）
└── {业务域}/                    # 按业务域分目录
    ├── {类名}Test.java          # 测试类，继承 BaseTest
    └── fixture/                 # 测试数据工厂（可选）
        └── {业务域}Fixture.java
```

**示例**:
```
{MODULE}-interfaces/src/test/java/{BASE_PACKAGE}/test/
├── BaseTest.java
└── invoice/
    ├── InvoiceServiceTest.java
    ├── InvoiceBaseServiceTest.java
    ├── InvoiceIssueServiceTest.java
    └── fixture/
        └── InvoiceFixture.java
```

---

## 测试类命名规范

| 被测类 | 测试类 | 测试类包路径 |
|--------|--------|--------------|
| XxxService | XxxServiceTest | `{BASE_PACKAGE}.test.{业务域}` |
| XxxBaseService | XxxBaseServiceTest | `{BASE_PACKAGE}.test.{业务域}` |
| XxxAdapterService | XxxAdapterServiceTest | `{BASE_PACKAGE}.test.{业务域}` |

**强制规范**：
1. 所有测试类必须继承 `BaseTest`
2. 所有测试方法必须添加 `@Test` 注解
3. 依赖注入统一使用 `@Resource` 注解

---

## 测试方法命名规范

**格式**: `test_{方法名}_{场景}_{预期结果}`

| 场景 | 命名示例 |
|------|----------|
| 正常流程 | `test_add_success` |
| 参数校验失败 | `test_add_paramInvalid_throwException` |
| 业务规则校验失败 | `test_add_duplicateInvoiceNo_throwBizException` |
| 边界条件 | `test_add_amountExceedLimit_throwBizException` |
| 空值处理 | `test_detail_idNotExist_returnNull` |

---

## 测试类结构模板

### Service 层测试模板

```java
package {BASE_PACKAGE}.test.invoice;

import {BASE_PACKAGE}.core.invoice.InvoiceService;
import {BASE_PACKAGE}.core.invoice.bo.InvoiceBO;
import {BASE_PACKAGE}.dao.invoice.InvoiceDao;
import {BASE_PACKAGE}.interfaces.controller.invoice.vo.req.InvoiceCreateReqVO;
import {BASE_PACKAGE}.test.BaseTest;
import org.junit.Test;

import javax.annotation.Resource;

import static org.junit.Assert.*;

/**
 * InvoiceService 单元测试
 */
public class InvoiceServiceTest extends BaseTest {

    @Resource
    private InvoiceService invoiceService;

    @Resource
    private InvoiceDao invoiceDao;

    // ==================== 新增测试 ====================

    @Test
    public void test_add_success() {
        // Given
        InvoiceCreateReqVO reqVO = buildCreateReqVO();

        // When
        int result = invoiceService.add(reqVO);

        // Then
        assertEquals(1, result);
    }

    @Test
    public void test_add_duplicateInvoiceNo_throwBizException() {
        // Given
        InvoiceCreateReqVO reqVO = buildCreateReqVO();
        // 先插入一条数据
        invoiceService.add(reqVO);

        // When & Then
        try {
            invoiceService.add(reqVO);
            fail("应该抛出业务异常");
        } catch (Exception e) {
            assertTrue(e.getMessage().contains("已存在"));
        }
    }

    // ==================== 查询测试 ====================

    @Test
    public void test_detail_success() {
        // Given
        InvoiceCreateReqVO reqVO = buildCreateReqVO();
        invoiceService.add(reqVO);
        Long id = 1L;

        // When
        InvoiceBO result = invoiceService.detail(id);

        // Then
        assertNotNull(result);
    }

    @Test
    public void test_detail_idNotExist_returnNull() {
        // Given
        Long id = 999L;

        // When
        InvoiceBO result = invoiceService.detail(id);

        // Then
        assertNull(result);
    }

    // ==================== 测试数据构建方法 ====================

    private InvoiceCreateReqVO buildCreateReqVO() {
        InvoiceCreateReqVO reqVO = new InvoiceCreateReqVO();
        reqVO.setInvoiceNo("INV202401010001");
        reqVO.setAmount(10000L);
        reqVO.setInvoiceType(1);
        return reqVO;
    }
}
```

---

## 测试覆盖要求

### 必须覆盖的场景

| 场景类型 | 说明 | 优先级 |
|----------|------|--------|
| 正常流程 | 主流程正常执行 | P0 |
| 参数校验 | 必填参数为空、格式错误 | P0 |
| 业务规则 | 唯一性校验、状态校验、权限校验 | P0 |
| 边界条件 | 金额上限、数量限制、时间范围 | P1 |
| 异常处理 | 外部调用失败、数据不存在 | P1 |
| 空值处理 | 查询结果为空、可选参数为空 | P2 |

### 各层测试重点

| 层级 | 测试重点 | 注入对象 |
|------|----------|----------|
| Service 应用层 | 流程编排、调用顺序 | @Resource 被测 Service |
| Service 核心业务层 | 业务规则、状态流转 | @Resource 被测 Service、Dao |
| Service 基础层 | 校验逻辑、工具方法 | @Resource 被测 Service |
| Service 适配层 | 外部调用封装、异常处理 | @Resource 被测 Service |

---

## 测试数据管理

### 方式一：测试方法内构建（简单场景）

```java
private InvoiceCreateReqVO buildCreateReqVO() {
    InvoiceCreateReqVO reqVO = new InvoiceCreateReqVO();
    reqVO.setInvoiceNo("INV202401010001");
    reqVO.setAmount(10000L);
    return reqVO;
}
```

### 方式二：Fixture 工厂类（复杂场景）

```java
/**
 * 发票测试数据工厂
 */
public class InvoiceFixture {

    /**
     * 构建默认的创建请求
     */
    public static InvoiceCreateReqVO createReqVO() {
        InvoiceCreateReqVO reqVO = new InvoiceCreateReqVO();
        reqVO.setInvoiceNo("INV202401010001");
        reqVO.setAmount(10000L);
        reqVO.setInvoiceType(1);
        reqVO.setBuyerName("测试公司");
        reqVO.setBuyerTaxNo("91110000MA00XXXXX");
        return reqVO;
    }

    /**
     * 构建指定金额的创建请求
     */
    public static InvoiceCreateReqVO createReqVO(Long amount) {
        InvoiceCreateReqVO reqVO = createReqVO();
        reqVO.setAmount(amount);
        return reqVO;
    }

    /**
     * 构建默认的 DO 对象
     */
    public static InvoiceDO invoiceDO() {
        InvoiceDO invoiceDO = new InvoiceDO();
        invoiceDO.setId(1L);
        invoiceDO.setInvoiceNo("INV202401010001");
        invoiceDO.setAmount(10000L);
        invoiceDO.setInvoiceType(1);
        invoiceDO.setStatus(0);
        return invoiceDO;
    }
}
```

---

## 断言规范

### JUnit 4 断言（推荐）

```java
import static org.junit.Assert.*;

// 基础断言
assertEquals(expected, actual);
assertNotNull(result);
assertNull(result);
assertTrue(condition);
assertFalse(condition);

// 异常断言
@Test(expected = BizException.class)
public void test_add_throwException() {
    service.add(invalidReqVO);
}

// 或使用 try-catch 方式验证异常消息
@Test
public void test_add_throwException() {
    try {
        service.add(invalidReqVO);
        fail("应该抛出业务异常");
    } catch (BizException e) {
        assertEquals("发票编号已存在", e.getMessage());
    }
}

// 集合断言
assertEquals(3, list.size());
assertTrue(list.isEmpty());
```

### AssertJ 断言（可选，更流畅）

```java
import static org.assertj.core.api.Assertions.*;

// 基础断言
assertThat(result).isNotNull();
assertThat(result.getId()).isEqualTo(1L);

// 集合断言
assertThat(list).hasSize(3);
assertThat(list).isEmpty();
assertThat(list).contains(item1, item2);

// 异常断言
assertThatThrownBy(() -> service.add(reqVO))
    .isInstanceOf(BizException.class)
    .hasMessage("发票编号已存在");
```

---

## 依赖注入规范

### 使用 @Resource 注入（强制）

```java
public class InvoiceServiceTest extends BaseTest {

    @Resource
    private InvoiceService invoiceService;

    @Resource
    private InvoiceDao invoiceDao;

    @Resource
    private InvoiceBaseService invoiceBaseService;
}
```

### 注入注意事项

| 规则 | 说明 |
|------|------|
| 使用 @Resource | 由 Spring 容器注入真实 Bean（禁止使用 @Autowired） |
| 测试真实逻辑 | 集成测试，验证完整业务流程 |
| 数据准备 | 测试前准备必要的数据库数据 |
| 数据清理 | 测试后清理测试数据（可选） |

---

## 测试用例设计模板

### 生成测试前，先列出测试用例清单

```markdown
## {类名} 测试用例清单

### {方法名} 方法

| 用例编号 | 场景描述 | 输入 | 预期结果 | 优先级 |
|----------|----------|------|----------|--------|
| TC-001 | 正常新增 | 有效参数 | 返回 1 | P0 |
| TC-002 | 发票编号重复 | 已存在的编号 | 抛出 BizException | P0 |
| TC-003 | 金额超过上限 | amount=100000001 | 抛出 BizException | P1 |
| TC-004 | 必填参数为空 | invoiceNo=null | 抛出异常 | P0 |
```

---

## 测试生成流程

### Step 1: 检查 BaseTest 是否存在

**路径**: `finance-interfaces/src/test/java/com/dahuangf/finance/test/BaseTest.java`

- 如果存在：跳过创建，继续下一步
- 如果不存在：先创建 BaseTest.java

```java
package com.dahuangf.finance.test;

import com.dahuangf.finance.interfaces.Application;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * 测试基类 - 所有测试类必须继承
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class BaseTest {

}
```

### Step 2: 创建业务域文件夹

根据被测类的业务域，在 `com.dahuangf.finance.test` 下创建对应文件夹：
- `invoice/` - 发票业务域
- `settlement/` - 结算业务域
- `payment/` - 支付业务域
- ...

### Step 3: 分析被测类

1. 识别被测类的所有公共方法
2. 分析每个方法的业务逻辑
3. 识别依赖的外部组件

### Step 4: 设计测试用例

1. 列出每个方法的测试用例清单
2. 覆盖正常流程、异常流程、边界条件
3. 确定优先级（P0 必须覆盖）

### Step 5: 生成测试代码

1. 创建测试类，继承 BaseTest
2. 使用 @Resource 注入被测对象和依赖
3. 按用例清单实现测试方法（每个方法必须有 @Test 注解）
4. 添加测试数据构建方法

### Step 6: 验证测试

1. 确保测试可以独立运行
2. 确保测试之间无依赖
3. 确保测试结果稳定可重复

---

## 测试代码质量要求

| 要求 | 说明 |
|------|------|
| 独立性 | 每个测试方法独立运行，不依赖其他测试 |
| 可重复 | 多次运行结果一致 |
| 快速 | 单个测试执行时间 < 100ms |
| 可读性 | 测试方法名清晰表达测试意图 |
| 单一职责 | 每个测试方法只验证一个场景 |

---

## 不需要测试的内容

| 类型 | 说明 |
|------|------|
| Getter/Setter | 简单的属性访问方法 |
| 纯数据类 | DO、BO、VO、TO 等 |
| 配置类 | Spring 配置类 |
| 常量类 | 只包含常量定义的类 |
| 第三方库 | 不测试第三方库的功能 |

---

## 确认清单

生成测试代码前，需要确认：

- [ ] 被测类的所有公共方法已识别
- [ ] 测试用例清单已列出
- [ ] P0 场景已全部覆盖
- [ ] 测试数据构建方法已准备

**完成后询问**: "单元测试代码已生成，请确认：
1. 测试用例是否覆盖关键场景？
2. 是否需要补充其他测试场景？"
