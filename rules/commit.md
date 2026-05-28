# Commit 与分支规范

## Commit 格式

```
<type>(<scope>): <简短描述>
```

| type | 含义 |
|------|------|
| `feat` | 新功能 |
| `fix` | 缺陷修复 |
| `refactor` | 重构（功能不变） |
| `perf` | 性能优化 |
| `docs` | 文档更新 |
| `style` | 代码格式（不影响功能） |
| `test` | 测试 |
| `chore` | 构建/依赖配置 |
| `revert` | 撤回提交 |

- 简短描述不超过 50 个字符，结尾不加句号
- scope 字段使用英文小写字母（例如 `auth`、`api`、`prescription`）
- 每次提交只包含一项变更

## 分支命名

| 分支 | 职责 | 操作权限 |
|------|------|----------|
| `main` | 生产环境 | 项目负责人 |
| `develop` | 日常开发集成 | 全体项目成员 |
| `feature/xxx` | 新功能 | AI 创建，项目成员合并 |
| `bugfix/xxx` | 缺陷修复 | AI 创建，项目成员合并 |
| `release/x.x.x` | 发布前测试 | 项目负责人 |
| `hotfix/xxx` | 紧急修复 main 分支 | 项目负责人 |

feature 分支必须从 develop 分支创建。AI 禁止合并到 develop 分支，禁止操作 main 分支。

## 文档模板

**Plan 模板**：由 `writing-plans` skill 统一定义，是唯一权威模板。
路径约定：`docs/superpowers/plans/<YYYY-MM-DD-slug>.md`。
本文件不再自带 Plan 模板——避免双宪法冲突。

**Spec 模板**（`docs/superpowers/specs/<YYYY-MM-DD-slug>.md`）：

```markdown
# <设计标题>

> 日期: YYYY-MM-DD

## 背景
## 目标与边界
## 架构
## 详细设计
## 风险与验证
```
