# Claude Code Skills 完整学习指南

> 这份指南整理了 Claude Code 中 Skills 系统的核心概念、使用方法、最佳实践和实战案例。

---

## 目录

1. [Skills 是什么？](#1-skills-是什么)
2. [Skills 的核心概念](#2-skills-的核心概念)
3. [Skills 文件结构](#3-skills-文件结构)
4. [创建你的第一个 Skill](#4-创建你的第一个-skill)
5. [Skills 的高级特性](#5-skills-的高级特性)
6. [Skills vs 其他功能](#6-skills-vs-其他功能)
7. [实战案例](#7-实战案例)
8. [官方 Skills 资源](#8-官方-skills-资源)
9. [常见问题与最佳实践](#9-常见问题与最佳实践)

---

## 1. Skills 是什么？

### 简单理解

**Skills 就是给 Claude 的"专业知识包"**。

想象一下：
- 你雇了一个万能助手（Claude）
- 但某些专业任务需要特定知识（比如处理 PDF、创建 Excel 表格、做代码审查）
- Skills 就是你给助手的"培训手册"

### 技术定义

Skills 是 Claude Code 的**按需上下文扩展机制**：
- **不是**单独的进程或外部工具
- **不是**子代理（Subagents）
- **是**在需要时动态加载到 Claude 上下文中的指令集

### 为什么需要 Skills？

**问题**：如果把所有专业知识都放在 Claude 的主提示词中：
- 上下文会爆炸（太长）
- Token 成本飙升
- Claude 容易混淆

**解决方案**：Skills 实现了"懒加载"：
- 只在需要时加载相关指令
- 保持主提示词简洁
- 按需扩展 Claude 的能力

---

## 2. Skills 的核心概念

### 2.1 工作原理

```
用户请求 
    ↓
Claude 检查可用的 Skills 描述
    ↓
匹配到相关 Skill
    ↓
调用 Skill 工具（加载 SKILL.md）
    ↓
扩展上下文，执行任务
```

### 2.2 Skills 存储位置

Skills 有三个级别，优先级从高到低：

| 位置 | 路径 | 作用域 | 优先级 |
|------|------|--------|--------|
| **企业级** | 企业配置的位置 | 整个组织 | 最高 |
| **个人级** | `~/.claude/skills/` | 所有项目 | 中 |
| **项目级** | `.claude/skills/` | 当前项目 | 最低 |

**规则**：同名 Skill 时，高优先级覆盖低优先级。

### 2.3 Skills 的类型

#### 纯提示词型 Skill
只包含指令，不需要执行脚本。

**示例**：代码审查 Skill
```yaml
---
name: code-review
description: Review code for security, performance, and best practices
---

You are an expert code reviewer. Focus on:
1. Security vulnerabilities
2. Performance issues
3. Code maintainability
4. Best practices adherence
```

#### 脚本型 Skill
包含可执行脚本（Python、Shell 等）。

**示例**：PDF 处理 Skill
```yaml
---
name: pdf
description: Extract and analyze text from PDF documents
---

Use the extract_text.py script to process PDFs:

```bash
python3 extract_text.py <input_file>
```

After extraction, summarize key points.
```

---

## 3. Skills 文件结构

### 3.1 基本结构

每个 Skill 是一个文件夹，包含：

```
my-skill/
├── SKILL.md          # 必需：Skill 定义文件
├── script.py         # 可选：执行脚本
├── config.json       # 可选：配置文件
└── README.md         # 可选：文档
```

### 3.2 SKILL.md 格式

SKILL.md 分为两部分：

#### Part 1：YAML Frontmatter（元数据）

```yaml
---
name: skill-name              # 必需：Skill 名称（也是 /slash-command）
description: 简短描述          # 必需：帮助 Claude 判断何时使用
context: fork                 # 可选：执行上下文（main/fork）
agent: Explore                # 可选：使用哪个子代理
allowed-tools: [Read, Write]  # 可选：允许使用的工具
model: sonnet                 # 可选：使用的模型
color: blue                   # 可选：UI 显示颜色
---
```

#### Part 2：Markdown 内容（指令）

```markdown
# Skill 的详细指令

## 何时使用
当用户请求 X、Y、Z 时激活此 Skill。

## 工作流程
1. 第一步
2. 第二步
3. 第三步

## 注意事项
- 注意点 1
- 注意点 2
```

### 3.3 Frontmatter 字段详解

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `name` | 必需 | Skill 名称，用作 `/` 命令 | `pdf`, `code-review` |
| `description` | 必需 | 描述何时使用（关键！） | `"Use when extracting PDF text"` |
| `context` | 可选 | `main` 或 `fork` | `fork`（独立上下文） |
| `agent` | 可选 | 子代理名称 | `Explore`, `Debug` |
| `allowed-tools` | 可选 | 工具白名单 | `[Read, Write, Bash]` |
| `model` | 可选 | 模型选择 | `sonnet`, `opus`, `haiku` |
| `color` | 可选 | UI 颜色 | `blue`, `green`, `red` |

---

## 4. 创建你的第一个 Skill

### 示例：代码解释 Skill

**目标**：让 Claude 用类比和 ASCII 图解释代码。

#### 步骤 1：创建目录

```bash
mkdir -p ~/.claude/skills/explain-code
cd ~/.claude/skills/explain-code
```

#### 步骤 2：创建 SKILL.md

```bash
cat > SKILL.md << 'EOF'
---
name: explain-code
description: Use when explaining how code works, teaching about a codebase, or when the user asks "how does this work?"
---

# Code Explanation Skill

When explaining code, always include:

1. **Start with an analogy**: Compare the code to something from everyday life
2. **Draw a diagram**: Use ASCII art to show the flow, structure, or relationships
3. **Walk through the code**: Explain step-by-step what happens
4. **Highlight a gotcha**: What's a common mistake or misconception?

Keep explanations conversational. For complex concepts, use multiple analogies.

## Example Format

**Analogy**: Think of this function like a restaurant kitchen...

**Flow Diagram**:
```
Input → Process → Output
  ↓       ↓        ↓
 Args   Logic   Result
```

**Step-by-Step**:
1. First, we...
2. Then, we...
3. Finally, we...

**Common Gotcha**: Many people think X, but actually Y because Z.
EOF
```

#### 步骤 3：测试 Skill

**方法 A：自动调用**
```bash
claude

# 然后问：
> 解释一下这段代码是怎么工作的？
```

**方法 B：手动调用**
```bash
claude

> /explain-code
```

Claude 应该会按照 Skill 的指令，用类比和图表来解释代码。

---

## 5. Skills 的高级特性

### 5.1 动态上下文注入（!命令语法）

在 Skill 中执行 Shell 命令，并将输出注入到提示词中。

**示例：PR 总结 Skill**

```yaml
---
name: pr-summary
description: Summarize changes in a pull request
allowed-tools: [Bash(gh *)]
---

## Pull request context

- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
Summarize this pull request focusing on:
1. Main changes
2. Potential impacts
3. Review priorities
```

**工作原理**：
1. `!`gh pr diff`` 命令会在 Skill 加载**之前**执行
2. 命令输出替换占位符
3. Claude 看到的是实际的 PR 数据，而不是命令

### 5.2 子代理集成

将 Skill 与专门的子代理结合。

**示例：深度代码探索 Skill**

```yaml
---
name: deep-explore
description: Deep dive into complex codebases
context: fork        # 使用独立上下文
agent: Explore       # 使用 Explore 子代理
allowed-tools: [Read, Grep, Glob, Bash]
---

# Deep Exploration Protocol

You are a specialized code archaeologist. Your mission:

1. **Map the territory**: Build a mental model of the codebase
2. **Find patterns**: Identify architectural patterns
3. **Trace dependencies**: Follow the data flow
4. **Document insights**: Create a comprehensive report

Use your tools liberally to explore.
```

**优势**：
- 独立上下文，不污染主对话
- 专用子代理有针对性的行为
- 完成后返回简洁总结

### 5.3 Extended Thinking（扩展思考）

让 Claude 在执行 Skill 时进行更深入的推理。

**启用方法**：在 Skill 内容中包含 `ultrathink` 关键词。

```yaml
---
name: architecture-review
description: Review system architecture
---

# Architecture Review (ultrathink enabled)

Perform a comprehensive architecture analysis...
```

**注意**：Extended Thinking 会消耗更多 Token（按思考 Token 计费）。

### 5.4 工具权限控制

限制 Skill 可以使用的工具，增强安全性。

**示例：只读分析 Skill**

```yaml
---
name: security-audit
description: Audit code for security issues
allowed-tools: [Read, Grep, Glob]  # 不允许修改文件
---

Perform security analysis without modifying any files.
```

**常用工具组合**：

| 场景 | 工具组合 |
|------|----------|
| 只读分析 | `[Read, Grep, Glob]` |
| 创建/修改代码 | `[Read, Write, Edit, Bash]` |
| 研究 + 实现 | `[Read, Write, Edit, WebFetch, WebSearch]` |
| Git 操作 | `[Read, Write, Bash(git *)]` |

### 5.5 脚本打包

将复杂逻辑封装在脚本中，Skill 只负责调用。

**示例：可视化工具 Skill**

**目录结构**：
```
visualize-codebase/
├── SKILL.md
└── generate-map.py
```

**SKILL.md**：
```yaml
---
name: visualize-codebase
description: Generate visual codebase maps
allowed-tools: [Read, Bash]
---

# Codebase Visualization

To visualize the codebase structure:

1. Run the bundled script:
   ```bash
   python3 generate-map.py .
   ```

2. Open the generated `codebase-map.html` in a browser

3. Summarize key insights from the visualization
```

**generate-map.py**：
```python
#!/usr/bin/env python3
import os
import sys

def generate_map(root_dir):
    # 生成 HTML 可视化
    html = "<html>...</html>"
    with open("codebase-map.html", "w") as f:
        f.write(html)

if __name__ == "__main__":
    generate_map(sys.argv[1])
```

---

## 6. Skills vs 其他功能

Claude Code 有多种扩展机制，何时使用 Skills？

### 决策树

```
是否需要显式控制何时执行？
├─ 是 → 使用 Slash Command
│   示例：/deploy, /test, /security-review
│   你控制调用时机
│
└─ 否 → 专业知识是否应自动应用？
    ├─ 是 → 使用 Skill
    │   示例：安全模式、领域规则、代码标准
    │   Claude 根据上下文自动激活
    │
    └─ 否 → 工作是否需要隔离上下文？
        ├─ 是 → 使用 Subagent
        │   示例：深度探索、并行分析
        │   防止上下文膨胀
        │
        └─ 否 → 行为是否必须保证执行？
            ├─ 是 → 使用 Hook（确定性）
            │   示例：每次编辑后格式化、记录所有 bash 命令、阻止访问 .env
            │   Claude 无法跳过或忘记
            │
            └─ 否 → 直接提示
                不是所有事情都需要抽象
```

### 对比表

| 特性 | Skill | Slash Command | Subagent | Hook |
|------|-------|---------------|----------|------|
| **触发方式** | 自动（描述匹配） | 手动（用户调用） | 自动/手动 | 事件驱动 |
| **上下文** | 主会话 | 主会话 | 独立（fork） | 主会话 |
| **用例** | 专业知识 | 工作流快捷键 | 深度任务 | 自动化保证 |
| **可跳过** | 可以 | 可以 | 可以 | **不可以** |
| **示例** | PDF 处理 | `/deploy` | 代码探索器 | 自动格式化 |

---

## 7. 实战案例

### 案例 1：测试生成 Skill

**需求**：自动为代码生成全面的测试。

```yaml
---
name: test-generator
description: Use when user asks to write tests, add test coverage, or create test cases
allowed-tools: [Read, Write, Grep, Glob, Bash]
---

# Test Generator Skill

## When to Activate
Use this Skill when user requests:
- "Write tests for..."
- "Add test coverage"
- "Generate test cases"
- "Create unit/integration tests"

## Test Generation Process

### 1. Analyze Target Code
- Read the file/function to test
- Identify inputs, outputs, side effects
- Find dependencies and mocks needed
- Check existing test patterns (Grep for test files)

### 2. Determine Test Type
- **Unit Tests**: Individual functions, pure logic
- **Integration Tests**: API endpoints, database operations
- **Component Tests**: React/Vue components (if frontend)

### 3. Generate Comprehensive Tests
Cover all scenarios:
- ✅ Happy path (expected usage)
- ❌ Error cases (invalid inputs, failures)
- 🔀 Edge cases (empty, null, boundary values)
- 🔁 Side effects (database changes, API calls)

### 4. Follow Project Patterns
- Check CLAUDE.md for testing conventions
- Match existing test file structure
- Use the same testing framework
- Follow naming conventions

## Example Test Structure

```javascript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', async () => {
      // Happy path
    });
    
    it('should reject invalid email', async () => {
      // Error case
    });
    
    it('should handle empty input', async () => {
      // Edge case
    });
  });
});
```

## Quality Checklist
- [ ] All public methods tested
- [ ] Edge cases covered
- [ ] Error handling verified
- [ ] Mocks used appropriately
- [ ] Tests are independent
- [ ] Clear test descriptions
```

### 案例 2：API 文档生成 Skill

**需求**：从代码注释生成 API 文档。

```yaml
---
name: api-docs
description: Generate API documentation from code comments. Use when asked to document APIs, create API docs, update endpoint documentation, or generate OpenAPI specs.
allowed-tools: [Read, Write, Grep, Bash]
---

# API Documentation Generator

## Process

### 1. Discover API Endpoints
```bash
# Find all route definitions
grep -r "@app.route\|@router\|@api" --include="*.py"
grep -r "app.get\|app.post\|app.put\|app.delete" --include="*.js"
```

### 2. Extract Documentation
For each endpoint, extract:
- HTTP method (GET, POST, PUT, DELETE)
- Path (e.g., `/api/users/:id`)
- Request parameters
- Request body schema
- Response schema
- Status codes
- Example requests/responses

### 3. Generate Documentation
Create a structured document with:

```markdown
# API Documentation

## Endpoints

### GET /api/users
**Description**: Retrieve list of users

**Query Parameters**:
- `page` (integer, optional): Page number (default: 1)
- `limit` (integer, optional): Items per page (default: 20)

**Response** (200 OK):
```json
{
  "users": [...],
  "total": 100,
  "page": 1
}
```

**Error Responses**:
- 400: Invalid parameters
- 401: Unauthorized
```

### 4. Generate OpenAPI Spec (if requested)
```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
paths:
  /api/users:
    get:
      summary: Retrieve users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
```
```

### 案例 3：System Skill Pattern（系统技能模式）

这是一个强大的模式：**CLI + SKILL.md + SQLite**。

**核心思想**：
1. 创建一个专用 CLI 工具（如 Pomodoro 计时器）
2. 用 SQLite 存储数据
3. 用 SKILL.md 教 Claude 如何使用你的系统

**示例：Pomodoro Skill**

**目录结构**：
```
pomodoro/
├── SKILL.md
├── pomodoro (可执行二进制)
└── pomodoro.db (SQLite 数据库)
```

**SKILL.md**：
```yaml
---
name: pomodoro
description: Manage focused work sessions with Pomodoro technique
allowed-tools: [Bash(./pomodoro *)]
---

# Pomodoro Timer Skill

## System Overview
This is a Pomodoro timer that tracks your focus sessions.

## Quick Decision Tree
User task → What kind of request?
├─ Start focused work → Check status first, then start session
├─ Check current timer → Use status command
├─ Review productivity → Use stats command (day/week/month/year)
├─ View past sessions → Use history command
└─ Stop early → Use stop command

## Commands

### Start a session
```bash
./pomodoro start --task "Refactor ABC-123"
```
This starts a 25-minute focus session.

### Check status
```bash
./pomodoro status
```

### View statistics
```bash
./pomodoro stats --period week
```

### Stop session early
```bash
./pomodoro stop
```

## Workflow Example

User: "Hey let's pomodoro, I need to refactor ABC-123"

Your response:
1. Run `./pomodoro start --task "Refactor ABC-123"`
2. Acknowledge the session started
3. Wait 25 minutes (use sleep)
4. Alert when session is complete
```

**使用效果**：

```bash
claude

> 启动一个 Pomodoro，我要写测试

Claude: 好的，为你启动一个专注会话来写测试。
[执行] ./pomodoro start --task "写测试"
✅ Pomodoro 会话已开始，25 分钟专注时间。
⏰ 我会在 25 分钟后提醒你。

[25 分钟后]
🎉 Pomodoro 完成！你刚刚完成了 25 分钟的专注工作。
需要休息一下吗？（建议 5 分钟休息）
```

---

## 8. 官方 Skills 资源

### 8.1 Anthropic 官方 Skills

Claude Code 内置了一些官方 Skills，位于：
- `/mnt/skills/public/` (只读)

**主要官方 Skills**：

| Skill 名称 | 用途 | 触发条件 |
|-----------|------|---------|
| `docx` | Word 文档处理 | 创建/编辑 .docx 文件 |
| `pdf` | PDF 处理 | 读取/提取 PDF 内容 |
| `pptx` | PowerPoint 处理 | 创建/编辑演示文稿 |
| `xlsx` | Excel 处理 | 创建/编辑电子表格 |
| `frontend-design` | 前端设计 | 创建高质量 UI 组件 |
| `product-self-knowledge` | 产品知识 | 回答 Anthropic 产品问题 |

**查看官方 Skills**：
```bash
claude

> /context

# 或者直接查看文件
ls /mnt/skills/public/
```

### 8.2 社区 Skills 资源

**1. Awesome Claude Code**
- GitHub: https://github.com/hesreallyhim/awesome-claude-code
- 内容：社区精选的 Skills、Hooks、插件集合

**2. Claude Code Ultimate Guide**
- 作者：Florian BRUNIAUX
- 内容：从入门到高级的完整指南，包含生产级模板

**3. Claude Code Skill Factory**
- GitHub: https://github.com/alirezarezvani/claude-code-skill-factory
- 内容：交互式 Skill 生成工具，69 个专业预设模板

**4. Blake Crosley's Technical Reference**
- 网址：https://blakecrosley.com/en/guides/claude-code
- 内容：25,000+ 字的技术参考文档

### 8.3 推荐的社区 Skills

**DevOps Skills**（by akin-ozer）
- 用途：DevOps 工程师工具集
- 包含：IaC 代码生成、验证器、Shell 脚本

**Book Factory**（by Robert Guss）
- 用途：非虚构图书创作流程
- 包含：完整的出版基础设施复制

**Britfix**（by Talieisin）
- 用途：美式英语 → 英式英语转换
- 特点：智能处理代码文件（只转换注释）

---

## 9. 常见问题与最佳实践

### 9.1 常见问题

#### Q1: Skill 没有被自动调用？

**可能原因**：
1. `description` 字段不够清晰
2. 用户请求与描述不匹配
3. Skill 数量太多，超出上下文预算

**解决方案**：
```yaml
# ❌ 不好的描述
description: "PDF stuff"

# ✅ 好的描述
description: "Extract and analyze text from PDF documents. Use when users ask to process PDFs, read PDFs, or extract text from PDF files."
```

#### Q2: Skill 超出上下文预算？

**症状**：运行 `/context` 时看到警告。

**解决方案**：
1. 精简 Skill 描述
2. 删除不常用的 Skills
3. 使用项目级 Skills（而非个人级）

**上下文预算**：
- 默认：上下文窗口的 2%
- 最小：16,000 字符
- 查看：`/context`

#### Q3: 如何调试 Skill？

**方法 1：详细日志**
```bash
export DEBUG=*
claude --verbose
```

**方法 2：手动调用测试**
```bash
claude

> /my-skill  # 直接调用 Skill
```

**方法 3：查看 Skill 是否加载**
```bash
claude

> /context  # 查看可用 Skills 列表
```

### 9.2 最佳实践

#### ✅ DO（应该做）

1. **写清晰的描述**
   ```yaml
   description: "Reviews code for security. Use when asked to: review security, audit code, find vulnerabilities, check for exploits, analyze risks."
   ```

2. **提供具体示例**
   ```markdown
   ## Example Usage
   User: "Review this for security issues"
   Your response: [分析代码，列出漏洞]
   ```

3. **使用工具限制**
   ```yaml
   # 只读分析，不修改代码
   allowed-tools: [Read, Grep, Glob]
   ```

4. **分离关注点**
   - 不要创建"万能 Skill"
   - 每个 Skill 专注一个任务

5. **版本控制**
   ```bash
   git init ~/.claude/skills/my-skill
   git add .
   git commit -m "Initial skill version"
   ```

#### ❌ DON'T（不应该做）

1. **模糊的描述**
   ```yaml
   # ❌ 太模糊
   description: "Helps with coding"
   
   # ✅ 具体
   description: "Generate comprehensive unit tests for JavaScript/TypeScript code"
   ```

2. **过长的 Skill**
   - 如果 SKILL.md 超过 500 行，考虑拆分
   - 使用脚本封装复杂逻辑

3. **硬编码路径**
   ```yaml
   # ❌ 不要这样
   Run: /Users/alice/scripts/tool.py
   
   # ✅ 使用相对路径
   Run: python3 tool.py
   ```

4. **忽略错误处理**
   ```markdown
   ## Error Handling
   If the script fails:
   1. Check if dependencies are installed
   2. Verify file permissions
   3. Fall back to manual approach
   ```

5. **过度依赖 Skills**
   - 简单任务直接提示就好
   - 不需要为每个小任务创建 Skill

### 9.3 性能优化

#### 优化 Skill 描述长度

Skills 描述会被加载到 Claude 的上下文中。

**策略**：
- 保持描述简洁（1-2 句话）
- 详细指令放在 SKILL.md 内容部分

```yaml
# ❌ 描述太长
description: "This skill helps you generate comprehensive unit tests for your codebase. It analyzes your code structure, identifies test scenarios, creates test files with proper naming conventions, uses the correct testing framework, and ensures good coverage of happy paths, edge cases, and error conditions."

# ✅ 简洁有效
description: "Generate comprehensive unit tests. Use when asked to write tests, add test coverage, or create test cases."
```

#### 使用脚本减少 Token 消耗

**模式**：将重复逻辑移到脚本中。

```yaml
# ❌ 在 SKILL.md 中写大量指令
---
name: api-analyzer
---
1. 读取所有 .js 文件
2. 查找 app.get, app.post 等
3. 提取路径和参数
4. 生成文档
5. 创建 OpenAPI spec
...（50 行详细步骤）

# ✅ 使用脚本
---
name: api-analyzer
---
Run the analyzer script:
```bash
python3 analyze_api.py
```

Then summarize the findings.
```

### 9.4 安全考虑

#### 限制文件访问

```yaml
---
name: log-analyzer
allowed-tools: [Read(logs/*), Grep(logs/*)]
---
# 只能读取 logs/ 目录
```

#### 禁用危险命令

```yaml
---
name: safe-analyzer
allowed-tools: [Read, Grep]  # 不包含 Bash
---
# 无法执行任意命令
```

#### 沙箱模式

```bash
# 启用沙箱模式运行 Claude Code
claude --sandbox

# 在 SKILL.md 中明确说明
## Security
This skill runs in sandbox mode for safety.
```

---

## 10. 总结与下一步

### 核心要点

1. **Skills 是按需上下文扩展**：不是独立进程，而是动态加载的指令集
2. **描述决定触发**：写好 `description` 是关键
3. **三种存储位置**：企业 > 个人 > 项目
4. **工具限制增强安全**：使用 `allowed-tools` 控制权限
5. **脚本封装复杂逻辑**：保持 SKILL.md 简洁

### 学习路径

```
第 1 周：基础
├─ 创建第一个简单 Skill（纯提示词）
├─ 理解 YAML frontmatter
└─ 测试自动触发和手动调用

第 2 周：进阶
├─ 创建脚本型 Skill
├─ 使用动态上下文注入（!命令）
└─ 配置工具权限

第 3 周：实战
├─ 实现一个完整的工作流 Skill
├─ 集成子代理
└─ 优化性能和安全

第 4 周：贡献
├─ 开源你的 Skill
├─ 参与社区讨论
└─ 学习高级模式（System Skill Pattern）
```

### 推荐资源

**官方文档**：
- https://code.claude.com/docs/en/skills

**社区资源**：
- Awesome Claude Code: https://github.com/hesreallyhim/awesome-claude-code
- Blake's Guide: https://blakecrosley.com/en/guides/claude-code
- Skill Factory: https://github.com/alirezarezvani/claude-code-skill-factory

**实践建议**：
1. 从简单的 Skill 开始（如代码格式化）
2. 逐步增加复杂度（添加脚本、工具限制）
3. 阅读官方 Skills 源码学习最佳实践
4. 参与社区分享你的 Skills

---

## 附录：快速参考

### Skill 模板

```yaml
---
name: skill-name
description: Clear description of when to use this skill
context: main  # or fork
agent: AgentName  # optional
allowed-tools: [Read, Write, Edit, Bash]
model: sonnet  # or opus, haiku
color: blue  # UI color
---

# Skill Title

## When to Activate
Describe the exact scenarios when this skill should be used.

## Process
1. Step 1
2. Step 2
3. Step 3

## Examples
Show concrete examples of input/output.

## Error Handling
Describe how to handle common failures.
```

### 常用命令

```bash
# 查看可用 Skills
claude
> /context

# 手动调用 Skill
> /skill-name

# 列出 Skills 目录
ls ~/.claude/skills/

# 创建新 Skill
mkdir -p ~/.claude/skills/my-skill
cd ~/.claude/skills/my-skill
touch SKILL.md

# 测试 Skill
claude --verbose
> /my-skill
```

### 故障排除清单

- [ ] SKILL.md 文件存在吗？
- [ ] YAML frontmatter 格式正确吗？
- [ ] `description` 字段清晰吗？
- [ ] Skill 在 `/context` 中可见吗？
- [ ] 是否超出上下文预算？（查看 `/context` 警告）
- [ ] `allowed-tools` 配置正确吗？
- [ ] 脚本有执行权限吗？（`chmod +x script.py`）

---

**最后更新**：2026-02-13  
**文档版本**：v1.0  
**作者**：Claude (整理自官方文档和社区资源)

欢迎提供反馈和改进建议！ 🚀
