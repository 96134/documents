# Everything Claude Code (ECC) 工作流速查手册

> 适用场景：个人项目开发加速
> 更新时间：2026-05-07

---

## 0. 环境准备

### 前置条件

- 已安装 Claude Code CLI
- 已安装 ECC 插件：`claude plugin install affaan-m/everything-claude-code`

### 每次新项目的启动步骤

```bash
cd your-project-folder    # 必须先进入项目目录
claude                    # 启动 Claude Code
```

进 Claude Code 后，第一步永远是：

```
/agent-sort
```

它会扫描你的项目，把 ECC 组件分为：

- **DAILY**（每次自动加载）— 你项目真正需要的技能/规则
- **LIBRARY**（按需搜索调用）— 不相关的语言/框架规则不会浪费上下文

> **重要**：不要在非项目目录（如 `C:\Users\admin`）下直接跑 `/agent-sort`，它扫不到代码就没法分类。

---

## 1. 完整开发流水线（四阶段）

### 阶段 1：需求 → 设计

| 你要做什么       | 技能/命令                 | 说明                    |
| ----------- | --------------------- | --------------------- |
| 把模糊需求变精确规格  | `/product-capability` | 生成"工程合同"：接口定义、边界条件、约束 |
| 分析现有代码、设计方案 | `/plan`               | 自动探查代码库，输出分步实现方案      |
| 技术选型/架构决策   | `architect` 代理        | 评估技术方案、模块拆分           |
| 设计 REST API | `/api-design`         | 路由命名、分页、错误格式、版本控制     |
| 画架构图/流程图    | 直接描述，Claude 生成        | 用 Mermaid 或 ASCII art |

### 阶段 2：编码实现

| 你要做什么        | 技能/命令                    | 说明                           |
| ------------ | ------------------------ | ---------------------------- |
| **写新功能（核心）** | `/tdd-workflow`          | 先写测试 → 红灯 → 绿灯 → 重构，覆盖率≥80%  |
| 遵循代码规范       | `/coding-standards`      | KISS/DRY/YAGNI/早返回/不可变性      |
| 后端设计模式       | `/backend-patterns`      | 仓库模式、服务层、中间件、缓存、JWT          |
| 前端设计模式       | `/frontend-patterns`     | 组合组件、自定义 Hook、性能优化、错误边界      |
| 查库最新文档       | `/docs-lookup`           | 通过 Context7 获取最新 API 文档，不用猜测 |
| 写数据库迁移       | `/database-migrations`   | 自动生成和执行迁移脚本                  |
| 接入第三方 API    | `/api-connector-builder` | 生成 API 客户端代码                 |

**TDD 工作流内部循环：**

```
1. 描述需求 → Claude 先写测试
2. 运行测试 → 红灯（测试失败，因为功能还没写）
3. Claude 写实现 → 运行测试 → 绿灯
4. Claude 重构 → 再次运行测试 → 确认还是绿的
5. 完成
```

### 阶段 3：质量检查

| 你要做什么           | 技能/命令                | 说明                            |
| --------------- | -------------------- | ----------------------------- |
| **提交前全面检查（核心）** | `/verification-loop` | 构建+类型检查+lint+测试+安全扫描+diff审查   |
| 安全审查            | `/security-review`   | OWASP Top 10：注入、XSS、CSRF、密钥泄露 |
| 自动代码审查          | `code-reviewer` 代理   | 每次修改代码后自动触发                   |
| 清理死代码           | `/refactor-clean`    | 自动检测未用代码、重复代码，安全删除            |
| E2E 测试          | `/e2e-testing`       | Playwright 自动化浏览器测试           |
| 测试覆盖率检查         | `/test-coverage`     | 确保≥80%覆盖率                     |

### 阶段 4：提交 & 部署

| 你要做什么         | 技能/命令                                      | 说明                        |
| ------------- | ------------------------------------------ | ------------------------- |
| 从想法到 PR（一键三连） | `/prp-plan` → `/prp-implement` → `/prp-pr` | 规划→实现→创建PR                |
| 只创建 PR        | `/prp-pr`                                  | 自动分析变更，生成标题和描述            |
| 自动提交          | `/prp-commit`                              | 生成规范的 commit message      |
| Review PR     | `/review-pr`                               | 审查指定 PR 的代码质量             |
| 进程管理          | `/pm2`                                     | PM2 启动/停止/监控 Node 进程      |
| CI/CD 配置      | `/deployment-patterns`                     | 生成 GitHub Actions 等 CI 配置 |
| 多服务并行开发       | `/dmux-workflows`                          | 多代理并行：研究+实现、测试+修复         |

---

## 2. 必配钩子（settings.json）

在你的项目 `.claude/settings.json` 中加入以下配置：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "node .claude/hooks/suggest-compact.js"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "node .claude/hooks/pre-bash-dispatcher.js"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node .claude/hooks/session-end.js"
          }
        ]
      }
    ]
  }
}
```

**三个钩子各自的作用：**

| 钩子                    | 触发时机          | 作用                                  |
| --------------------- | ------------- | ----------------------------------- |
| `suggest-compact`     | 每次编辑/写入文件后    | 上下文太长时提醒你 `/compact`                |
| `pre-bash-dispatcher` | 每次执行 Bash 命令前 | 拦截危险命令（`rm -rf`、`git push --force`） |
| `session-end`         | 会话结束时         | 自动保存会话状态，下次可恢复                      |

---

## 3. 快速决策表（做什么 → 敲什么）

| 场景        | 命令/技能                                |
| --------- | ------------------------------------ |
| 新功能开发     | `/tdd-workflow` 然后描述需求               |
| 修复 Bug    | 直接描述 Bug，自动调用 `build-error-resolver` |
| 理解现有代码    | `/code-tour` 或 `code-explorer` 代理    |
| 审查我刚写的代码  | `code-reviewer` 代理                   |
| 提交前质量把关   | `/verification-loop`                 |
| 做深度研究     | `/deep-research`                     |
| 写技术文章     | `/article-writing`                   |
| 做演示文稿     | `/frontend-slides`                   |
| 接入外部 API  | `/api-connector-builder`             |
| 清理无用代码    | `/prune` 或 `/refactor-clean`         |
| 设计数据库     | `/database-migrations`               |
| 安全扫描      | `/security-review`                   |
| E2E 测试    | `/e2e-testing`                       |
| 查看所有可用技能  | `/skill-health`                      |
| 配置 ECC 设置 | `/update-config`                     |
| 上下文太长时    | `/compact` 或 `/strategic-compact`    |
| 保存当前进度    | `/save-session`                      |
| 恢复上次进度    | `/resume-session`                    |
| 查看会话列表    | `/sessions`                          |

---

## 4. 典型工作流示例

### 示例：给项目加"用户邮箱登录"功能

```
步骤 1: /product-capability
        输入: "用户可用邮箱+密码登录，支持JWT，有注册和登录两个接口"
        产出: 工程约束文档（接口定义、数据流、边界条件）

步骤 2: /plan
        输入: "实现 email+password 登录注册功能"
        产出: 分步实现方案（涉及哪些文件、数据流、实现顺序）

步骤 3: /tdd-workflow
        输入: "按plan方案，从auth模块的单元测试开始"
        → Claude 写测试 → 红灯 → Claude 写实现 → 绿灯 → 重构

步骤 4: /verification-loop
        自动跑: 构建 → 类型检查 → lint → 测试 → 安全扫描 → diff审查

步骤 5: /prp-pr
        自动分析变更，创建 Pull Request
```

### 示例：修复一个 Bug

```
步骤 1: 描述 Bug（附上错误信息/截图）
步骤 2: Claude 自动诊断，调用 build-error-resolver 修复
步骤 3: 跑测试确认修复
步骤 4: /verification-loop（快速检查）
步骤 5: /prp-commit（提交修复）
```

---

## 5. 知识积累系统

| 场景       | 命令                     | 说明                      |
| -------- | ---------------------- | ----------------------- |
| 学到新模式/经验 | `/learn`               | 记录到 `instincts/`，下次自动引用 |
| 查看已学知识   | `/instinct-status`     | 列出所有已记录的本能              |
| 导入外部经验   | `/instinct-import`     | 从 git 历史或其他项目导入模式       |
| 导出经验     | `/instinct-export`     | 把经验分享给团队                |
| 持续学习     | `/continuous-learning` | 从项目历史自动提取模式             |

---

## 6. 常见问题

### Q: 技能太多记不住怎么办？

A: 看上面"快速决策表"。核心只有 4 个：`/tdd-workflow`（写代码）、`/verification-loop`（检查）、`/prp-pr`（提PR）、`/plan`（规划）。

### Q: 上下文窗口不够用？

A: 跑 `/compact` 手动压缩，或在里程碑节点（写完一个模块、跑完测试后）压缩。ECC 的 `suggest-compact` 钩子也会自动提醒你。

### Q: 我用的语言 ECC 有对应规则吗？

A: 跑 `/agent-sort` 自动检测。ECC 内置 Go、Kotlin、PHP、Python、Swift、TypeScript/JavaScript、Rust、Java、C++、C#、Dart/Flutter 等语言规则。

### Q: 怎么知道哪些技能是当前项目真正在用的？

A: `/skill-health` 会列出所有技能的状态。`/agent-sort` 会分类 DAILY vs LIBRARY。

### Q: 可以只装部分功能吗？

A: 可以。`/agent-sort` 本身就是选择性安装。你还可以在 settings.json 中禁用特定钩子或 MCP 服务器。

---

## 7. 快捷键记忆口诀

```
需求不清 → /product-capability 或 /plan
开始写码 → /tdd-workflow
写完了   → /verification-loop
提交代码 → /prp-commit 或 /prp-pr
新知识   → /learn
不知道用啥 → 看本文第 3 节"快速决策表"
```

---

> 更多信息：[ECC 官网](https://ecc.tools) | [GitHub](https://github.com/affaan-m/everything-claude-code)
