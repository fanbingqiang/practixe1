# 任务计划：Java 后端实战教学项目（practice-order）

## 目标
用一个业务极简（Hello World 级别）但工程化完整的 Spring Boot 3.5.11 + JDK 21 项目，让用户逐阶段积累企业级实战经验：Maven/脚手架 → 接口 → 测试 → Debug 抓 bug → 数据库 → 接口文档 → 第三方库 → 支付宝沙箱 → xxl-job → 缓存幂等 → 多模块安全 → Docker 部署 → CI/CD 监控。
**全程按企业迭代流程走**：需求单 → 设计/接口文档 → 分支开发 → 自测 → Code Review → 提测 → Bug 流转 → 上线 → 复盘。
**不推 GitHub**，用本地 bare 仓库（工作区内 `_remote/`）模拟团队协作把 Git 教透。

**协作方式（用户已确认）**：代码全部由我写，用户负责跟学、提问、验收。

## 当前阶段
阶段 3：连接数据库（Docker MySQL + Flyway + MyBatis-Plus）

## 已完成

### 阶段 0：选型与脚手架 ✅
- [x] 环境探测、技术选型（Spring Boot 3.5.11 + JDK 21）
- [x] 设计思路文档 + 接口文档
- [x] Maven 骨架、启动类、配置、`.gitignore`、`.gitattributes`
- [x] 故障复盘 BUG-001（Maven 仓库路径 + Mockito agent）
- **状态：** complete

### 阶段 1：Hello 接口 + 工程地基 ✅
- [x] `Result<T>` 统一响应体 + `ErrorCode` 错误码枚举（分区段）
- [x] `BusinessException` + `GlobalExceptionHandler`（6 类异常分支）
- [x] `TraceIdFilter` + `TraceIdUtil`（链路追踪，支持上游透传）
- [x] `JacksonConfig`（java.time 统一 `yyyy-MM-dd HH:mm:ss`）
- [x] `HelloController` + `HelloVO`（含 `@Validated` 参数校验）
- [x] 测试 10 个全绿（切片 + 契约 + 集成）
- [x] 真实 HTTP 接口测试 5 个用例通过
- [x] 修复真实 bug：`isSuccess()` 泄漏成 JSON 字段 → `@JsonIgnore` + 契约测试
- **状态：** complete

### 基础工程：Git ✅
- [x] 本地 bare 仓库 `_remote/practice-order.git` 作为"公司远程仓库"
- [x] 3 次规范提交（Conventional Commits）并推送
- [x] `.gitattributes` 统一行尾符
- **状态：** complete

## 待办

### 阶段 2：Debug 实战（价格计算，PRD-002）✅
- [x] 需求单 PRD-002 + 先写测试断言正确行为
- [x] 真实红灯：`Tests run: 5, Failures: 3, Errors: 2`
- [x] 三类故障定位：崩溃型 / 静默错误型 / 校验缺失型
- [x] 修复并留下回归测试，转为绿灯 `Tests run: 22, Failures: 0`
- [x] 真实 HTTP 验证 5 个用例
- [x] 文档：`docs/04-IDEA开发环境配置.md`、`docs/05-Debug实战记录-PRD-002.md`
- **状态：** complete

### 阶段 G：Git 实战（团队协作演练）✅
- [x] 分支模型搭建：main + develop，并推送
- [x] 分支本质实证：三个分支文件都是 41 字节的同一提交 ID
- [x] 模拟同事入职：clone 出 `_dev2/practice-order`（身份 LiSi）
- [x] feature 分支开发 PRD-003（Hello 自定义前缀）+ `git add -p` 逐块审查
- [x] **冲突现场**：同方法同行冲突，读懂 `<<<<<<< ======= >>>>>>>` 并合成解决
- [x] **语义冲突**：文本冲突解决后 3 个测试红，验证"Git 检测不到语义冲突"
- [x] `merge --no-ff`（实验证明本可快进，强制保留分支痕迹）
- [x] PR 描述生成（`git request-pull`）+ Code Review 3 条意见 + 整改
- [x] 后悔药：reset --soft/--mixed/--hard、revert、reflog 复活、stash -u
- [x] 发布 `v0.2.0`（`git tag -a` 带注释标签）
- [x] hotfix HOTFIX-001：从 main 切分支修 traceId 日志注入，合回 main + develop，`v0.2.1`
- [x] 文档：`docs/07-Git实战演练记录.md`
- **状态：** complete

### 阶段 0.5：接手项目方法论 + 实战演练 ✅
- [x] 产出方法论文档：`docs/03-拿到一个项目我怎么做.md`（六阶段 SOP + 一页纸清单 + 常见 7 错）
- [x] 用 `practice-order` 当"刚接手的项目"实跑 SOP 第 1~4 阶段
- [x] 事实采集（技术栈/接口/Bean/热点文件/测试地图）
- [x] 让它跑起来：编译 → 27 测试全绿 → 启动 → 打真实请求
- [x] 产出 `practice-order/docs/08-项目地图.md`（四条调用链 + 改进项 + 待确认清单）
- [x] 新发现并修复 2 个真实缺陷：NEW-001（actuator/info 静默失效）、NEW-002（YAML 重复 key）
- **状态：** complete

### 阶段 3：连接数据库
- [ ] Docker 起 MySQL（需先启动 Docker Desktop）
- [ ] Flyway 建表 / MyBatis-Plus CRUD / HikariCP
- [ ] JPA 对比文档
- [ ] `@MybatisTest` + 集成测试
- **状态：** pending

### 阶段 4：接口测试与文档
- [ ] Knife4j/OpenAPI 在线文档、Apifox 用例、`.http` 脚本
- **状态：** pending

### 阶段 5：抓 Bug 专章
- [ ] ①NPE ②SQL 字段不匹配 ③事务自调用失效 ④JSON 序列化 ⑤时区差 8 小时
- **状态：** pending

### 阶段 6：第三方库（Hutool / MapStruct）
- **状态：** pending

### 阶段 7：支付宝沙箱（下单 + 回调验签 + 查单）
- **状态：** pending

### 阶段 8：xxl-job 分布式调度
- **状态：** pending

### 阶段 9：Redis 缓存 + 幂等 + 分布式锁
- **状态：** pending

### 阶段 10：多模块 + AOP + JWT + 限流
- **状态：** pending

### 阶段 11：Docker 部署
- **状态：** pending

### 阶段 12：CI/CD + 监控 + 压测
- **状态：** pending

### 阶段 13：线上故障演练
- [ ] 连接池耗尽 / 慢 SQL / 死锁 / OOM / 依赖不可用 + 复盘报告
- **状态：** pending

## 已做决策
| 决策 | 理由 |
|------|------|
| Spring Boot 3.5.11 + JDK 21 | 3.x 生态成熟；4.x 刚发布，第三方适配不稳 |
| 数据层 MyBatis-Plus 为主 + JPA 对比 | 国内主流；对比教学保证举一反三 |
| 不推 GitHub，本地 bare 仓库模拟 | 用户明确要求 |
| Maven 本地仓库放工作区 `.m2repo` | 全局配置指向不可写路径（详见复盘 BUG-001） |
| Mockito 用 `mock-maker-subclass` | 受限环境禁止 JVM agent attach |
| 项目文档放仓库内 `practice-order/docs/` | 设计文档/接口文档属项目资产，应与代码一起版本管理 |
| 教学元材料放工作区根 `docs/` | 教学总纲等不属于项目代码 |

## 遇到的错误
| 错误 | 尝试次数 | 解决方案 |
|------|---------|---------|
| Maven `AccessDeniedException` | 1 | `.mvn/maven.config` 指定工作区内仓库 |
| Mockito agent attach 失败 | 1 | `mock-maker-subclass` |
| `MethodArgumentTypeMismatchException` 包名写错 | 1 | 改为 `org.springframework.web.method.annotation` |
| `@WebMvcTest` 不加载 `@Configuration` | 1 | 测试类 `@Import(JacksonConfig.class)` |
| 嵌套测试 Controller 404 | 1 | 改顶层 public 类 |
| AssertJ JSON 断言链式编译失败 | 1 | 改用 `hasJsonPath*` |
| `isSuccess()` 泄漏成 JSON 字段 | 1 | `@JsonIgnore` + 契约测试 |
| 验证脚本正则误匹配 `"message":"success"` | 1 | 改为解析 JSON 后精确断言 |

## 备注
- 每阶段结束更新状态：pending → in_progress → complete
- 真实命令 + 真实输出记录到 `progress.md`
- 用户偏好：代码我全写；Docker 优先；安装放 D 盘；多停下来问
