# Git 实战教学（不推 GitHub，但把 Git 教透）

> 你的要求："不提交，但 Git 确实需要教学，例如拉取项目、怎么 merge、怎么提 PR、分支是什么。"
> 所以：**我们不上传你的代码到 GitHub**，而是在本地建一个"公司远程仓库"（bare 仓库），完整模拟团队协作。
> PR（Pull Request）在真实公司用 GitHub/GitLab，我用本地 bare 仓库 + 模拟评审流程来教，流程完全一致。

---

## 一、必须先建立的心智模型

### 1. 四个区域
```
工作区(Working Dir) → 暂存区(Stage/Index) → 本地仓库(Local Repo) → 远程仓库(Remote)
   git add ──────────────┘   git commit ────┘        git push ──┘
```
- **工作区**：你能看到的文件
- **暂存区**：`git add` 后进入，"这次要提交的内容"
- **本地仓库**：`git commit` 后进入，有完整历史
- **远程仓库**：`git push` 后别人才能看到

### 2. 分支到底是什么
分支不是"文件夹副本"，**分支只是一个指向某次提交的可移动指针**。
- `HEAD` 是"你现在在哪儿"的指针
- 新建分支 = 造一个新指针，几乎零成本
- 提交 = 指针往前移动一格

所以"开分支开发"在企业里才这么廉价：`git switch -c feature/order-create`。

### 3. 合并的两种本质
| 类型 | 什么时候发生 | 历史长什么样 |
|------|-------------|-------------|
| 快进 Fast-forward | 主干没动过，直接把指针挪过去 | 一条直线，**丢失"曾经有分支"的信息** |
| 三方合并 Three-way | 主干和分支都各自提交过 | 产生一个 merge commit（两个爹） |

企业里通常要求 `--no-ff`（禁用快进），**为了保留分支痕迹，方便追溯"这批改动是一个需求"**。

---

## 二、教学内容清单（12 个实战场景，逐个动手）

| # | 场景 | 要学的命令 | 为什么要学 |
|---|------|-----------|-----------|
| 1 | 第一次拉取项目 | `git clone`、`git remote -v` | 入职第一天必做 |
| 2 | 已有本地代码关联远程 | `git remote add origin`、`git branch -u` | 本地建的项目怎么接远程 |
| 3 | 看状态与历史 | `git status`、`git log --oneline --graph` | 随时知道"我在哪" |
| 4 | 拉最新代码 | `git fetch` vs `git pull` | **fetch 更安全**，pull = fetch + merge |
| 5 | 建分支干活 | `git switch -c feature/xxx` | 企业规范：绝不在 main 上直接写 |
| 6 | 提交 | `git add -p`、`git commit`（Conventional Commits） | 提交粒度与信息规范 |
| 7 | **合并** | `git merge --no-ff`、`git merge --abort` | 主干合分支 |
| 8 | **变基** | `git rebase`、`git rebase -i` 压缩提交 | 保持历史干净，什么时候该用/不该用 |
| 9 | **解决冲突** | 冲突标记、`git checkout --ours/--theirs`、`git mergetool` | 一定会遇到，我会人为制造冲突 |
| 10 | **提 PR / Code Review** | 推分支 → 发起合并请求 → 评审 → Squash 合并 → 删分支 | 企业协作核心流程 |
| 11 | 后悔药 | `git restore`、`git reset --soft/--mixed/--hard`、`git revert`、`git reflog`、`git stash` | 救命技能，比想象中常用 |
| 12 | 发布与打标 | `git tag -a v1.0.0`、`git hotfix` 分支 | 版本发布流程 |

---

## 三、分支模型（我们项目采用的简化版 Git Flow）

```
main        ●──────────●──────────────●──────●   ← 永远可部署/已上线（受保护）
             \        / \            / \    /
develop       ●──●──●   ●──●──●──●──●   ●──●      ← 集成分支，功能都合到这里
               \    /     \      /
feature/xxx     ●──●       ●────●                  ← 每个需求一个分支，从 develop 切
                           
hotfix/xxx                          ●──●           ← 线上紧急修复，从 main 切，修完合回 main + develop
```

**规则**
- 分支命名：`feature/需求号-简述`、`bugfix/BUG-003-简述`、`hotfix/REL-002-简述`
- `main` 只接受来自 `develop` 或 `hotfix` 的合并
- 合并方式：feature → develop 用 squash 或 `--no-ff`；develop → main 打 tag
- 每个分支活不过 3 天（长期分支必冲突）

---

## 四、提交信息规范（Conventional Commits）

```
<type>(<scope>): <subject>

type: feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert
例：
feat(order): 新增创建订单接口
fix(order): 修复订单金额计算精度丢失问题
test(order): 补充下单接口的边界值测试
docs(api): 更新接口文档
```

好处：能自动生成 CHANGELOG、能按类型筛选、review 时一眼看懂改了什么。

---

## 五、我们会实际演练的"事故"（我会故意制造）

1. **冲突现场**：我们俩改同一个文件同一行 → 教你读懂 `<<<<<<<` 标记并正确解决
2. **提交错分支**：在 develop 上写了代码 → 用 `git stash` + `switch` 搬到正确分支
3. **提交了密码**：教你 `.gitignore` 预防 + `git filter-repo` 补救思路（并说明"已经推出去就等于泄露"）
4. **提交历史太乱**：10 个 "fix" 提交 → `git rebase -i` 压成 1 个
5. **误删分支/误 reset**：用 `git reflog` 复活
6. **revert vs reset**：已经推到远程的错误提交，为什么必须用 `revert`

---

## 六、我们的"远程仓库"怎么搭（阶段 G1 执行）

不上 GitHub，改用本地 bare 仓库：
```
D:\git-remote\practice-order.git     ← 扮演 GitHub 上的公司仓库（bare 仓库）
C:\Users\ZhuanZ\Desktop\项目实战\practice-order   ← 你的工作副本（clone 出来）
```
用两个本地目录模拟"两个开发者的电脑"，就能真实演练：你 push、我 pull、产生冲突、开 PR、合并。

**命令预览（阶段 G1 会带你实际敲）**
```powershell
# 1. 建"远程仓库"
git init --bare D:\git-remote\practice-order.git

# 2. 克隆（模拟入职第一天）
git clone D:\git-remote\practice-order.git practice-order

# 3. 日常循环
git switch -c feature/order-create
git add -p
git commit -m "feat(order): 新增创建订单接口"
git push -u origin feature/order-create

# 4. 提 PR（本地版：请求合并到 develop）
git switch develop
git merge --no-ff feature/order-create
```
