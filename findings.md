# 调研发现

> 外部内容（网页/搜索结果）只写入本文件，不写入 task_plan.md。

## 1. 本机环境探测（2026-10-07 实测）

| 组件 | 版本 | 路径/说明 |
|------|------|-----------|
| JDK | 21.0.11 LTS | `JAVA_HOME=D:\Java\jdk-21.0.11\` |
| Maven | 3.9.11 | `D:\apache-maven-3.9.11` |
| Git | 2.52.0.windows.1 | — |
| Docker | 29.4.1（Client） | 守护进程**未运行**，需先启动 Docker Desktop |
| Docker Compose | v5.1.3 | 可用 |
| MySQL | 8.0.43 | 本机已装，按决策改用 Docker 版 |
| IDEA | 2025.1 / DataGrip 2025.2.4 / Apifox | 已装 |
| JMeter | 5.6.3 | `D:\apache-jmeter-5.6.3` |
| Nginx | 1.29.4 | `D:\nginx-1.29.4` |
| Redis / Grafana | 已存在 | `D:\Redis`、`D:\grafana-grafana-445cc24` |
| 现成示例项目 | `D:\spring-boot-demo` | 脚手架参考 |

**结论：环境齐全，无需下载新工具。**

## 2. Maven 环境的关键事实（踩坑后实测）

| 事实 | 数据 | 影响 |
|------|------|------|
| 全局 `conf/settings.xml` 的 localRepository | `D:\apache-maven-3.9.11\解压目录`（第 56 行，遗留乱配置） | 默认构建直接失败 |
| 该路径是否可写 | ❌ 不可写（工作区外） | 必须换仓库位置 |
| `~/.m2/repository` | 1133.2 MB / 16033 文件，缓存丰富但**不可写** | 只能读，不能作为仓库 |
| `~/.m2/settings.xml` | 配置了阿里云镜像 `https://maven.aliyun.com/repository/public` | 下载速度可接受 |
| 网络 | 阿里云镜像**可达**（Invoke-WebRequest 报 TLS 错误是 .NET 侧问题，与 Maven 无关） | 正常下载依赖 |

**采用的方案**：`practice-order/.mvn/maven.config` 写一行
`-Dmaven.repo.local=C:\Users\ZhuanZ\Desktop\项目实战\.m2repo`
→ 本地仓库落在工作区内，可写、免提权、可移植，且 `mvn` 命令无需额外参数。

## 3. Mockito 在受限环境的坑（重要）

- 现象：`@SpringBootTest` 一律报 `Could not initialize plugin: interface org.mockito.plugins.MockMaker`
- 完整根因：`MockitoInitializationException: Could not initialize inline Byte Buddy mock maker. It appears as if your JDK does not supply a working agent attachment mechanism.`
- 触发链路：`@SpringBootTest` → `ResetMocksTestExecutionListener` → `MockReset.get()` → `Mockito.<clinit>` → 无条件初始化 Mockito，**即使一个 mock 都没写**
- 修复：`src/test/resources/mockito-extensions/org.mockito.plugins.MockMaker` 内容 `mock-maker-subclass`
- 代价：不能 mock final 类与静态方法（本项目用不到）
- 备注：修复后仍会打印一行 "Mockito is currently self-attaching to enable the inline-mock-maker" 警告，但**测试已稳定通过**（连续 2 次绿）。若后续复现失败，改用 `-javaagent` 方式或升级 Mockito。

## 4. 已确认可用的依赖版本（本机缓存 + 下载验证）

| 依赖 | 选定版本 | 说明 |
|------|---------|------|
| spring-boot-starter-parent | 3.5.11 | 实测编译、测试、启动全部通过（Spring 6.2.16 / Tomcat 10.1.52） |
| java.version | 21 | 与 JDK 21.0.11 匹配 |
| mybatis-plus（阶段 3） | 3.5.11（boot3 starter） | 本机有缓存 |
| mysql-connector-j（阶段 3） | 8.x | 本机有缓存 |
| knife4j（阶段 4） | 4.x | 本机有缓存 |
| hutool-all（阶段 6） | 5.8.26 | 本机有缓存 |
| alipay-sdk-java（阶段 7） | 待定 | **无缓存，需联网下载** |
| xxl-job-core（阶段 8） | 待定 | **无缓存，需联网下载** |
| flyway-core / redisson（阶段 3/9） | 待定 | **无缓存，需联网下载** |

## 5. 关键约束（来自用户/工作规范）
- 所有新安装内容放 D 盘；中间件优先只跑 Docker。（注：当前沙箱下 D 盘不可写，需要提权或改放工作区，遇到时向用户确认）
- 敏感配置必须进 `.gitignore`。
- 写代码前先写设计文档 + 接口文档。
- 前端 UI：蓝白简约、按钮不用蓝色、正文黑色、少用 AI 表情。
- 每完成"工具属性"的阶段要主动告知用户。
