---
inclusion: fileMatch
fileMatchPattern: "**/*-api/**/*.java"
---

# api (对外 RPC 接口定义)

## 目录结构

```
{MODULE}-api/src/main/java/{BASE_PACKAGE}/api/
├── enums/           # 对外枚举
├── model/           # TO 对象
└── service/         # RPC 接口定义
```

**变量说明**:
- `{MODULE}`: finance 或 sale
- `{BASE_PACKAGE}`: com.dahuangf.finance 或 com.dahuangf.sale（路径中 `.` 替换为 `/`）

## 命名规范

| 类型 | 规则 | 示例 |
|------|------|------|
| RPC 接口 | 业务名 + Rpc | AllocationNoticeRpc |
| TO | 业务名 + TO | AllocationNoticeCreateTO |
| 枚举 | 大驼峰 + Enum | AllocationTypeEnum |

## RPC 接口规范

- 接口放 `service/` 目录
- 返回值统一用 `RpcResult` 包装
- 入参使用 TO 对象

```java
import com.dahuangf.common.rpc.core.result.RpcResult;

public interface AllocationNoticeRpc {

    /**
     * 创建调配通知单
     */
    RpcResult<String> createAllocationNotice(AllocationNoticeCreateTO param);
}
```

## TO 规范

- TO 放 `model/` 目录
- 必须实现 `Serializable`
- 必须定义 `serialVersionUID`

```java
@Data
public class AllocationNoticeCreateTO implements Serializable {
    private static final long serialVersionUID = 1L;
    
    /** 调配类型 */
    private Integer allocationType;
}
```

## RPC 实现

实现类放 `{MODULE}-interfaces` 模块对应业务域目录下：

```
{MODULE}-interfaces/src/main/java/{BASE_PACKAGE}/interfaces/rpc/
└── {业务域}/
    └── XxxRpcImpl.java
```

- 类名：RPC 接口名 + Impl（如 `AllocationNoticeRpcImpl`）
- 必须加注解：`@DubboService(protocol = {"dubbo"}, timeout = 3000, retries = 3)` 和 `@Slf4j`

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.dubbo.config.annotation.DubboService;

@Slf4j
@DubboService(protocol = {"dubbo"}, timeout = 3000, retries = 3)
public class AllocationNoticeRpcImpl implements AllocationNoticeRpc {

    @Resource
    private AllocationNoticeBizService bizService;

    @Override
    public RpcResult<String> createAllocationNotice(AllocationNoticeCreateTO param) {
        return RpcResult.success(bizService.create(param));
    }
}
```
