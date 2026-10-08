# 会话日志

## 会话 1 — 2026-10-07（规划）
- 确认工作区为空，未创建任何代码；扫描本机环境。
- 产出 `docs/00-教学总纲.md`。
- 测试结果：无（项目不存在）。

## 会话 2 — 2026-10-07（阶段 0 完成）
- 决策：项目名 `practice-order`:8080、MyBatis-Plus 为主、不推 GitHub 改本地 bare 仓库模拟、核心诉求"模拟企业实战"。
- 产出：`docs/01-企业实战模拟手册.md`、`docs/02-Git实战教学.md` + 项目骨架。
- 测试：`Tests run: 1, Failures: 0`，连续 2 次绿；应用启动 2.494s，健康检查 UP。
- 解决：Maven 仓库路径错误、Mockito agent attach 失败（复盘见 `practice-order/docs/03-故障复盘-BUG-001.md`）。

## 会话 3 — 2026-10-07（阶段 1 完成 + 项目搭建完成）
**产出代码**
| 文件 | 作用 |
|------|------|
| `common/result/Result.java` | 统一响应体，静态工厂 + 私有构造 |
| `common/result/ErrorCode.java` | 错误码枚举，按 1xxx/2xxx/3xxx/5xxx 分区 |
| `common/exception/BusinessException.java` | 业务异常，继承 RuntimeException（事务回滚友好） |
| `common/exception/GlobalExceptionHandler.java` | 全局异常处理，6 类异常分支 |
| `common/util/TraceIdUtil.java` + `common/filter/TraceIdFilter.java` | 链路追踪，支持上游透传 |
| `config/JacksonConfig.java` | 统一 java.time 序列化格式 |
| `controller/HelloController.java` + `vo/HelloVO.java` | 第一个业务接口 |
| `testsupport/ThrowingTestController.java` | 测试专用异常源（不进生产包） |

**测试结果（真实）**
```
Tests run: 10, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS  |  mvn 退出码: 0
```
分项：GlobalExceptionHandlerTest 2 / ResultSerializationTest 2 / HelloControllerTest 4 / PracticeOrderApplicationTests 2

**真实接口测试（curl 实测，非测试代码）**
| 用例 | 结果 |
|------|------|
| `GET /api/hello` | `code=0`，`greeting=Hello, world`，`serverTime=2026-10-07 14:46:31` ✅ |
| `GET /api/hello?name=张三` | `greeting=Hello, 张三`（中文编码正常）✅ |
| `name` 超长 21 字符 | `code=1001`，提示文案与文档一致 ✅ |
| `GET /api/not-exist` | HTTP 404 + `code=2001` ✅ |
| 传 `X-Trace-Id` 头 | 响应 traceId 原样透传 ✅ |

**开发中发现并修复的真实 Bug**
- 现象：响应体多出 `"success":true`，不在接口文档约定内
- 根因：Lombok `@Data` 把 `isSuccess()` 识别为 JSON 属性
- 修复：`@JsonIgnore` + 新增 `ResultSerializationTest` 契约测试锁死字段
- 复验：JSON 解析后顶层字段恰好为 `code, message, data, traceId, timestamp` 5 个 ✅
- 附带教训：第一次验证脚本用正则 `"success"` 误匹配到 `"message":"success"`，改为解析 JSON 后精确断言

**Git 进展**
```
91acc50 (HEAD -> main, origin/main) feat: 完成阶段1 统一响应体、全局异常处理与 Hello 接口
9e23a46 chore: 添加 .gitattributes 统一行尾符为 LF
ce6788d chore: 初始化 practice-order 项目骨架
```
- 模拟远程仓库：`C:\Users\ZhuanZ\Desktop\项目实战\_remote\practice-order.git`（bare）
- 已知限制：`git push` 在受限沙箱下必须提权（sh.exe 需要命名管道）

**遇到的错误（累计）**
| 错误 | 尝试次数 | 解决方案 |
|------|---------|---------|
| Maven `AccessDeniedException` | 1 | `.mvn/maven.config` 指定工作区内仓库 |
| Mockito agent attach 失败 | 1 | `mock-maker-subclass` |
| `MethodArgumentTypeMismatchException` 包名写错 | 1 | 改为 `org.springframework.web.method.annotation` |
| `@WebMvcTest` 不加载 `@Configuration` | 1 | 测试类加 `@Import(JacksonConfig.class)` |
| 嵌套测试 Controller 404 | 1 | 改为顶层 public 类 `ThrowingTestController` |
| AssertJ JSON 断言链式调用编译失败 | 1 | 改用可链式的 `hasJsonPath*` 系列 |
| git push 管道失败 | 1 | 提权执行（danger-full-access） |

**下一步候选**
- 阶段 2：Debug 入门（埋 NPE，教断点/异常断点/Evaluate）
- 阶段 G：Git 实战（分支/冲突/PR/回滚）
- 阶段 3：接数据库（Docker MySQL + MyBatis-Plus）

---

## 会话 4 — 2026-10-07（阶段 2 完成：Debug 实战）

**用户决策**：后续阶段顺序由我安排（Debug → Git → 数据库）；Maven 仓库保持现状；IDEA 自行按文档配置。

**做了什么**
1. 产出 `practice-order/docs/04-IDEA开发环境配置.md`（导入、Maven 仓库、编码、Lombok、跑测试、启动、断点快捷键、常见报错速查）
2. 按需求单 PRD-002 实现"订单总价计算"，**先写测试断言正确行为 → 跑出红灯 → 定位 → 修复 → 转绿**
3. 三个真实缺陷全部修复并留下回归测试
4. 产出 `practice-order/docs/05-Debug实战记录-PRD-002.md`（三类故障的定位方法论）

**红灯现场（修复前，真实输出）**
```
Tests run: 5, Failures: 3, Errors: 2
java.lang.NullPointerException: Cannot read field "intCompact" because "subtrahend" is null
    at java.math.BigDecimal.subtract(BigDecimal.java:1530)
    at ...PriceService.calculatePrice(PriceService.java:36)
expected: 53.97  but was: 59.07
Expecting BusinessException but was: NullPointerException
```

**三个缺陷**

| 编号 | 类型 | 现象 | 根因 | 修复 | 回归测试 |
|------|------|------|------|------|---------|
| BUG-101 | 崩溃型 | 不传折扣率直接 NPE | `subtract(null)`，未处理可选参数 | 用 `NO_DISCOUNT = 1` 兜底，把"不传"翻译成业务语义 | `calculatePrice_withoutDiscount` |
| BUG-102 | 静默错误型 | 九折算出 59.07 | 把折扣率当成减免金额，用了减法 | 改为 `multiply(rate)`，变量改名 `discountRate` | `calculatePrice_withDiscountRate` |
| BUG-103 | 缺失型 | 数量 0/负数不报错 | 需求约束未实现，且无拦截点 | Service 层校验（非 Controller，覆盖定时任务等入口） | 4 条异常用例 |

**绿灯结果（修复后，真实输出）**
```
Tests run: 22, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS  |  mvn 退出码: 0
```
新增测试：`PriceServiceTest` 8 个（L1 单元）+ `DebugControllerTest` 4 个（L2 切片）

**真实 HTTP 验证（5 个用例全对）**
| 用例 | 实测结果 |
|------|---------|
| `?unitPrice=19.99&quantity=3` | `data=59.97` ✅ |
| `...&discountRate=0.9` | `data=53.97` ✅ |
| `...&quantity=0` | `code=1001`，`数量必须大于 0` ✅ |
| `?unitPrice=abc` | `code=1002` 参数类型错误 ✅ |
| `...&discountRate=1.5` | `code=1001`，`折扣率必须大于 0 且不超过 1` ✅ |

**Git**
```
9750897 (HEAD -> main, origin/main) feat(price): 新增订单总价计算能力（PRD-002）
91acc50 feat: 完成阶段1 统一响应体、全局异常处理与 Hello 接口
9e23a46 chore: 添加 .gitattributes 统一行尾符为 LF
ce6788d chore: 初始化 practice-order 项目骨架
```
- 遵守"测试红了不许提交"：带 bug 的版本从未进入版本历史
- 沙箱策略放开后，`git push` 已无需提权

**学到的可复用知识**
- JDK 15+ 的 **Helpful NPE** 直接点名哪个变量为 null（`because "subtrahend" is null`）
- 故障分三类：崩溃型（异常断点）、静默错误型（单步对照期望值）、缺失型（需求逐条转测试）
- `@WebMvcTest` 不加载 `@Service`，需 `@Import` 或 `@MockBean`
- 金额比较必须用 `isEqualByComparingTo`，不能用 `isEqualTo`

**下一步**：阶段 G（Git 实战：分支、冲突、merge、PR、回滚）

---

## 会话 5 — 2026-10-07（阶段 G 完成：Git 实战）

**演练环境**（不上 GitHub，用本地裸仓库 + 两个工作副本模拟团队）
| 角色 | 路径 | 身份 |
|------|------|------|
| 公司远程仓库 | `_remote\practice-order.git`（bare） | — |
| 你的电脑 | `practice-order` | FanBingQiang |
| 同事李四 | `_dev2\practice-order`（clone 得来） | LiSi |

**完成内容（全部实测，非纸上谈兵）**

| # | 演练项 | 关键结果 |
|---|--------|---------|
| 1 | 分支本质实验 | 三个分支文件**都是 41 字节且内容完全相同**——分支只是指向提交的指针 |
| 2 | 模拟入职 clone | clone 后有 `remotes/origin/develop` 但本地只有 main；`switch develop` 自动建跟踪 |
| 3 | feature 分支开发 PRD-003 | Hello 接口支持自定义 `prefix`，7 个测试 |
| 4 | `git add -p` 逐块审查 | 2 个 hunk 逐块 `y` 暂存 |
| 5 | **冲突现场** | `CONFLICT (content)` in HelloController.java，读懂 `<<<<<<< ======= >>>>>>>` |
| 6 | **语义冲突** | 文本冲突解决后 3 个测试红：`expected Hello but was Hi` — **Git 检测不到，只有测试能** |
| 7 | `merge --no-ff` | 实验证明本可快进，用 `--no-ff` 强制保留分支痕迹 |
| 8 | PR + Code Review | `git request-pull` 生成 PR 描述；评审出 3 条意见（1 条阻塞：接口文档漂移） |
| 9 | 后悔药全套 | reset --soft/mixed/hard、revert、reflog 复活、stash -u |
| 10 | 发布 v0.2.0 | `git tag -a` + 推送标签 |
| 11 | hotfix HOTFIX-001 | 从 main 切分支修 traceId 日志注入，合回 main + **合回 develop**，tag v0.2.1 |

**红 → 绿 记录**
```
复现缺陷：Tests run: 9, Failures: 2  （CRLF 注入、超长 traceId 未被拦截）
修复完成：Tests run: 27, Failures: 0, Errors: 0   BUILD SUCCESS
```

**过程中自己踩的两个坑（都是真实教训）**
1. `git stash` 第一次失败：`No local changes to save` —— **默认不处理未跟踪文件，必须加 `-u`**
2. 我把评审整改**直接提交到了 develop**，正确做法是提交在 feature 分支走 PR。这正是企业要给 develop 加分支保护、禁止直接 push 的原因

**最终仓库状态**
```
c66a19d (develop, origin/develop)  docs(git): 新增 Git 实战演练完整记录
0f691aa (tag: v0.2.1, main, origin/main)  release: v0.2.1 安全补丁
d60371f  fix(security): 修复 traceId 日志注入漏洞（HOTFIX-001）
1a14b09 (tag: v0.2.0)  release: v0.2.0
bc45f56  docs(api): 同步 Hello 接口文档（PR-001 评审整改）
51acfb7  merge: 合并 feature/hello-prefix 到 develop（PR-001 评审通过）
ec32b1d  merge: 合并 develop 主干（解决 HelloController 冲突）
96a64fe  feat(hello): 默认问候语由 Hello 改为 Hi（同事 LiSi）
eb565d8  feat(hello): 支持自定义问候前缀（PRD-003）
9750897  feat(price): 新增订单总价计算能力（PRD-002）
```
- 标签：`v0.2.0`（功能发布）、`v0.2.1`（安全补丁）
- 分支：`main` / `develop`，均已与远程同步，工作区干净
- 测试：27 个全绿

**交付文档**：`practice-order/docs/07-Git实战演练记录.md`（含完整命令、真实输出、10 条经验、速查表）

**下一步**：阶段 3 —— 接数据库（Docker MySQL + Flyway + MyBatis-Plus）

---

## 会话 6 — 2026-10-07（接手项目方法论 + 实战演练）

**用户诉求**：带我从 0 学，把"拿到一个项目后会干什么"的思路写清楚，要模拟实战。

**产出 1：方法论文档** `docs/03-拿到一个项目我怎么做.md`
六阶段 SOP：搞清是什么 → 让它跑起来 → 画地图 → 建安全网 → 小步改动 → 交付复盘。
含「一页纸 SOP 清单」「新人常犯的 7 个错」「四区域/三线心智模型」。

**产出 2：实战演练**——把 `practice-order` 当作"刚接手的新项目"，真实执行 SOP 第 1~4 阶段

第 1 阶段（事实采集，全部用命令取证，不靠猜）
- 技术栈：Spring Boot 3.5.11 / JDK 21 / **无数据库、无缓存、无 MQ**（零外部依赖）
- 12 个生产类、6 个测试类 + 1 个测试支撑类
- 4 个接口：`/api/hello`、`/api/debug/price`、`/actuator/health`、`/actuator/info`
- 热点文件排行：`HelloControllerTest`(4 次) > 接口文档(3) = `HelloController`(3)
- 无 TODO/FIXME；**无 README**（列为高优改进项）

> **采集教训**：第一次用 `Select-String -Path "src\**\*.java"` 搜接口，返回**空结果**——
> PowerShell 的 `**` 不递归。改为 `Get-ChildItem -Recurse | Select-String` 才拿到完整清单。
> **空结果 ≠ 没有**，工具用错导致的漏查是接手项目时最危险的坑。

第 2 阶段（让它跑起来，真实输出）
```
编译 mvn clean compile       → 退出码 0
测试 mvn test                → Tests run: 27, Failures: 0   BUILD SUCCESS
启动 mvn spring-boot:run     → 3 秒就绪，/actuator/health = UP
```

第 3 阶段（画地图）→ 产出 `practice-order/docs/08-项目地图.md`（325 行）
- **四条完整调用链**：正常请求 / 业务异常 / 404 / 参数绑定失败
- 接口清单、Bean 清单、测试地图、热点文件与风险区
- 5 起历史事故索引 + 6 条改进项（含"无 README""debug 接口暴露"两条高优）
- 5 条待确认问题清单

第 4 阶段（真实验证调用链）
| 链路 | 实测结果 |
|------|---------|
| A 正常 `/api/hello?name=张三` | `code=0`，`greeting=Hi, 张三` ✅ |
| B 业务异常 `quantity=0` | `code=1001`，`数量必须大于 0` ✅ |
| C 404 `/api/not-exist` | HTTP 404 + `code=2001` ✅ |
| D 类型错误 `unitPrice=abc` | `code=1002` ✅ |
| 安全：500 位超长 traceId | 被拒并重新生成 16 位 ✅ |
| 安全：合法 traceId | 原样透传，不误伤 ✅ |
| 安全边界：64 位 / 65 位 | 放行 / 拒绝，边界精确 ✅ |

**过程中新发现并修复的 2 个真实缺陷**

| 编号 | 问题 | 根因 | 修复 |
|------|------|------|------|
| NEW-001 | `/actuator/info` 返回 `{}`，`info.app.*` 完全没生效 | Spring Boot 2.6 起 `management.info.env.enabled` 默认 false，**不报错不警告，静默不生效** | 开启该开关，实测返回 app.name/version/encoding |
| NEW-002 | 修复 NEW-001 时应用直接启动失败 | 我另起了一个 `management:` 顶层块，**YAML 不允许重复 key** | 合并进已有的 `management` 块 |

> **NEW-002 的自我复盘**：我改完配置没跑测试就急着启动，白踩一次。
> 错误发生在**配置解析阶段**（`ConfigDataEnvironmentPostProcessor`），
> 而 `@SpringBootTest` 会加载同一份 `application.yml`——
> **按 SOP「改一处 → 立刻跑测试」，3 秒内就会被拦住。**
> 这条铁律被自己违反了一次，反而成了最有力的证明。

**Git 操作**（全程走分支规范）
```
eb41927 (develop, origin/develop)  merge: 合并 /actuator/info 修复（NEW-001/NEW-002）
878214a  merge: 合并接手项目地图文档（docs/onboarding-map）
20a240d  fix(config): 修复 /actuator/info 返回空对象的问题
b13667b  docs: 新增接手项目地图（按 onboarding SOP 采集）
```
- 两个改动各走独立分支，`--no-ff` 合并后删除分支
- 测试 27 个全绿，工作区干净，已推送

**下一步**：阶段 3 —— 接数据库（Docker MySQL + Flyway + MyBatis-Plus）
