---
inclusion: fileMatch
fileMatchPattern: "**/*-common/**/*.java"
---

# common (公共模块)

路径: `{BASE_PACKAGE}.common`

**定位**: 存放与业务完全无关的通用组件

```
{MODULE}-common/src/main/java/{BASE_PACKAGE}/common/
├── constants/       # 常量类
├── enums/           # 通用枚举（与业务无关）
├── utils/           # 工具类
└── thread/          # 线程池初始化
```

**变量说明**:
- `{MODULE}`: finance 或 sale
- `{BASE_PACKAGE}`: com.dahuangf.finance 或 com.dahuangf.sale（路径中 `.` 替换为 `/`）

## 适用范围

| 类型 | 说明 | 示例 |
|------|------|------|
| 工具类 | 通用工具方法 | DateUtil、StringUtil |
| 线程池 | 线程池配置和初始化 | ThreadPoolConfig |
| 通用枚举 | 与业务无关的枚举 | YesNoEnum、StatusEnum |
| 常量 | 全局常量定义 | CommonConstants |

## 禁止放入

- 业务相关的枚举（放 dao 或 api 模块）
- 业务相关的工具类（放 core 模块）
- BO/VO/TO/DO 等数据对象
