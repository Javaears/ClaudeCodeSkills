# Claude Code: Skill vs Prompt 完全对比指南

> 一份帮助你彻底理解 Claude Code 中 Skill 和 Prompt 区别的实用文档

---

## 目录

1. [核心区别一句话总结](#核心区别一句话总结)
2. [什么是 Prompt？](#什么是-prompt)
3. [什么是 Skill？](#什么是-skill)
4. [详细对比](#详细对比)
5. [技术原理对比](#技术原理对比)
6. [实战场景对比](#实战场景对比)
7. [何时使用 Prompt vs Skill](#何时使用-prompt-vs-skill)
8. [类比理解](#类比理解)
9. [快速决策树](#快速决策树)
10. [常见误区](#常见误区)
11. [最佳实践](#最佳实践)

---

## 核心区别一句话总结

**Prompt（提示词）** = 你每次"口头"告诉 Claude 的话（临时、一次性）

**Skill（技能）** = 预先写好的"专业培训手册"，Claude 需要时自动翻阅（永久、可复用）

---

## 什么是 Prompt？

### 定义

**Prompt** 是你在对话中直接输入给 Claude 的文字指令。

### 示例

```bash
claude

> 你是一个代码审查专家。请审查这段代码的安全问题、性能问题和最佳实践。
> 重点关注：
> 1) SQL 注入漏洞
> 2) XSS 跨站脚本
> 3) 错误处理
> 4) 代码可维护性
```

### 特点

| 特点 | 说明 |
|------|------|
| **输入方式** | 直接在对话框输入 |
| **生命周期** | 仅存在于当前对话 |
| **复用方式** | 手动复制粘贴 |
| **上下文占用** | 全程占用对话上下文 |
| **触发方式** | 手动输入 |

### 优点

✅ 灵活自由，想说什么就说什么  
✅ 适合临时、一次性的需求  
✅ 不需要额外配置  
✅ 适合快速实验  

### 缺点

❌ 每次都要重新输入（或复制粘贴）  
❌ 占用对话上下文空间  
❌ 难以在团队间共享  
❌ 复杂指令容易出错（漏掉细节）  
❌ 多个专业角色指令容易混乱  

---

## 什么是 Skill？

### 定义

**Skill** 是预先定义好的、存储在文件系统中的可复用"专业知识包"，Claude 会在需要时自动加载到上下文中。

### 示例

**文件位置**：`~/.claude/skills/code-review/SKILL.md`

```yaml
---
name: code-review
description: Review code for security, performance, and best practices. Use when user asks to review, audit, or analyze code quality.
allowed-tools: [Read, Grep, Glob]
---

# 代码审查专家

你是一个经验丰富的代码审查专家。审查代码时，请关注以下方面：

## 安全性
- SQL 注入漏洞
- XSS 跨站脚本
- CSRF 攻击
- 认证授权问题
- 敏感信息泄露

## 性能
- 算法复杂度
- 数据库查询优化
- 缓存策略
- 内存泄漏

## 最佳实践
- 代码可读性
- 错误处理
- 日志记录
- 测试覆盖率

## 输出格式
使用以下格式输出审查结果：

### 🔴 严重问题
[列出需要立即修复的问题]

### 🟡 建议改进
[列出可以优化的地方]

### ✅ 良好实践
[列出做得好的地方]
```

### 使用方式

**自动触发**（推荐）：
```bash
claude

> 帮我审查这段代码
# Claude 自动检测到需要 code-review Skill，加载并使用
```

**手动调用**：
```bash
claude

> /code-review
# 强制加载 code-review Skill
```

### 特点

| 特点 | 说明 |
|------|------|
| **存储方式** | 文件系统（SKILL.md） |
| **生命周期** | 永久存储，跨会话使用 |
| **复用方式** | 自动触发或手动调用 |
| **上下文占用** | 按需加载（用时才占用） |
| **触发方式** | 自动（基于 description 匹配） |

### 优点

✅ 写一次，永久使用  
✅ 自动触发，无需手动输入  
✅ 按需加载，节省上下文  
✅ 易于团队共享（文件形式）  
✅ 可以包含脚本和工具  
✅ 支持版本控制（Git）  
✅ 可以设置权限和安全限制  

### 缺点

❌ 需要提前创建文件  
❌ 需要学习 YAML 格式  
❌ 描述（description）写不好可能无法触发  

---

## 详细对比

### 对比表格

| 维度 | Prompt | Skill |
|------|--------|-------|
| **本质** | 临时对话指令 | 可复用知识包 |
| **存储位置** | 对话历史 | 文件系统 |
| **生命周期** | 当前对话 | 永久存储 |
| **输入方式** | 手动输入/粘贴 | 自动加载 |
| **触发方式** | 每次手动 | 自动检测 |
| **上下文占用** | 全程占用 | 按需加载 |
| **复用性** | 低（需要复制粘贴） | 高（自动复用） |
| **团队共享** | 困难（需要文档传递） | 容易（文件共享） |
| **版本控制** | 不支持 | 支持（Git） |
| **权限控制** | 无 | 支持（allowed-tools） |
| **适用场景** | 临时、简单、实验性 | 专业、复杂、重复性 |
| **学习成本** | 无 | 中等（需要学习格式） |
| **维护成本** | 高（分散在各处） | 低（集中管理） |

### 使用频率对比

#### Prompt 使用模式（重复劳动）

```
第 1 次对话：
你：[复制粘贴 500 字的代码审查指令]
Claude：好的，我来审查...

第 2 次对话：
你：[再次复制粘贴 500 字的代码审查指令]  ← 重复
Claude：好的，我来审查...

第 10 次对话：
你：[还是复制粘贴...]  ← 继续重复
Claude：好的...

总输入量：500 字 × 10 次 = 5000 字
```

#### Skill 使用模式（一次配置）

```
配置阶段（一次性）：
创建 code-review Skill（500 字）

使用阶段（无限次）：
第 1 次：你：审查代码
        Claude：[自动加载 Skill] 好的...

第 2 次：你：review this
        Claude：[自动加载 Skill] 好的...

第 10 次：你：帮我看看这段代码
         Claude：[自动加载 Skill] 好的...

总输入量：你每次只需说 3-5 个字
```

---

## 技术原理对比

### Prompt 的工作原理

```
┌─────────────────────────────────────────────────┐
│     Claude 的上下文窗口（200K tokens）           │
├─────────────────────────────────────────────────┤
│                                                 │
│  [系统提示词] (10K tokens)                       │
│  ↓                                              │
│  [你的 Prompt - 代码审查专家] (2K tokens)        │  ← 每次对话都要带着
│  ↓                                              │
│  [你的 Prompt - 性能优化专家] (2K tokens)        │  ← 占用大量空间
│  ↓                                              │
│  [你的 Prompt - 安全审计专家] (2K tokens)        │  ← 即使不需要也在
│  ↓                                              │
│  [对话历史] (50K tokens)                        │
│  ↓                                              │
│  [可用空间] (134K tokens)                       │  ← 剩余空间减少
│                                                 │
└─────────────────────────────────────────────────┘

问题：所有 Prompt 全程占用上下文，浪费空间
```

### Skill 的工作原理

```
┌─────────────────────────────────────────────────┐
│     Claude 的上下文窗口（200K tokens）           │
├─────────────────────────────────────────────────┤
│                                                 │
│  [系统提示词] (10K tokens)                       │
│  ↓                                              │
│  [Skill 索引 - 只存描述] (200 tokens)            │  ← 非常小
│    - code-review: "审查代码时使用..."           │
│    - performance: "优化性能时使用..."           │
│    - security: "安全审计时使用..."              │
│  ↓                                              │
│  ─────────── 需要时才加载 ───────────           │
│  [code-review Skill 完整内容] (2K tokens)       │  ← 按需加载
│  ↓                                              │
│  [对话历史] (50K tokens)                        │
│  ↓                                              │
│  [可用空间] (≈138K tokens)                      │  ← 更多剩余空间
│                                                 │
└─────────────────────────────────────────────────┘

优势：
1. 索引非常小（每个 Skill 描述只有 20-50 tokens）
2. 完整内容只在需要时加载
3. 用完可以释放空间
4. 总上下文占用更少
```

### 成本对比计算

假设你有 **10 个专业领域**的指令，每个 **500 字**（≈750 tokens）：

#### Prompt 方式成本

```
所有 Prompt 同时加载到上下文：
10 个 Prompt × 750 tokens = 7,500 tokens

即使你只需要 1 个，其他 9 个也在占用空间
```

#### Skill 方式成本

```
Skill 索引（所有 10 个 Skill 的描述）：
10 个描述 × 30 tokens = 300 tokens

实际使用时（只加载 1 个）：
300 tokens (索引) + 750 tokens (实际 Skill) = 1,050 tokens

节省：(7,500 - 1,050) / 7,500 = 86% 的上下文空间！
```

---

## 实战场景对比

### 场景 1：代码审查

#### 使用 Prompt

**第 1 次审查**：
```
你：你是一个代码审查专家。请审查这段代码的安全问题、性能问题和最佳实践。
    重点关注：
    1) SQL注入
    2) XSS漏洞
    3) 性能瓶颈
    4) 错误处理
    
    [贴上代码]

Claude：[按照你的指令进行审查]
```

**第 2 次审查（新对话）**：
```
你：你是一个代码审查专家...（又要重复一遍）  ← 😫 痛苦
    [贴上代码]

Claude：[审查]
```

**第 10 次审查**：
```
你：你是一个...（继续重复）  ← 😭 崩溃
```

#### 使用 Skill

**配置阶段（只需一次）**：
```bash
# 创建 Skill
mkdir -p ~/.claude/skills/code-review
cat > ~/.claude/skills/code-review/SKILL.md << 'EOF'
---
name: code-review
description: Review code for security, performance, and best practices
---
你是一个代码审查专家...（详细指令）
EOF
```

**使用阶段（无限次）**：
```
第 1 次：
你：审查这段代码
    [贴上代码]
Claude：[自动加载 Skill，进行审查]  ← ✅ 自动

第 2 次：
你：review this code
    [贴上代码]
Claude：[自动加载 Skill，审查]  ← ✅ 自动

第 10 次：
你：帮我看看这段代码
    [贴上代码]
Claude：[自动加载 Skill，审查]  ← ✅ 依然自动
```

**结果**：
- Prompt 方式：重复输入 10 次 × 200 字 = 2000 字
- Skill 方式：配置 1 次 + 每次输入 5 字 = 配置 200 字 + 使用 50 字

---

### 场景 2：多个专业角色

#### 使用 Prompt（混乱）

```
你：你是一个全栈专家，同时具备以下能力：
    1) 代码审查专家 - 关注安全、性能、最佳实践
    2) 数据库优化专家 - 关注查询优化、索引设计
    3) 前端架构师 - 关注组件设计、状态管理
    4) DevOps 工程师 - 关注部署、CI/CD、监控
    5) 测试专家 - 关注测试覆盖率、测试策略
    
    现在请用代码审查专家的身份审查这段代码...

Claude：好的（但上下文已经很混乱了）
```

**问题**：
- 所有角色的指令堆在一起
- Claude 容易混淆不同角色的要求
- 占用大量上下文
- 不灵活（无法单独使用某个角色）

#### 使用 Skill（清晰）

创建独立的 Skills：

```yaml
# ~/.claude/skills/code-review/SKILL.md
---
name: code-review
description: Code quality review
---
专注于：代码质量、安全、性能

# ~/.claude/skills/db-optimize/SKILL.md
---
name: db-optimize
description: Database optimization
---
专注于：SQL 优化、索引设计

# ~/.claude/skills/frontend-arch/SKILL.md
---
name: frontend-arch
description: Frontend architecture
---
专注于：组件设计、状态管理

# ~/.claude/skills/devops/SKILL.md
---
name: devops
description: DevOps and deployment
---
专注于：CI/CD、部署、监控

# ~/.claude/skills/test-expert/SKILL.md
---
name: test-expert
description: Testing strategy and coverage
---
专注于：测试设计、覆盖率
```

**使用时**：
```
你：审查这段代码的安全问题
Claude：[只加载 code-review Skill]  ← 精准、不浪费

你：优化这个 SQL 查询
Claude：[只加载 db-optimize Skill]  ← 按需加载

你：设计这个 React 组件的状态管理
Claude：[只加载 frontend-arch Skill]  ← 专注单一角色
```

**优势**：
- 每个角色独立、清晰
- 按需加载，不浪费上下文
- 易于维护和更新
- 团队可以共享特定 Skill

---

### 场景 3：团队协作

#### 使用 Prompt（困难）

**问题**：
```
团队成员 A：我写了一个很好的代码审查 Prompt！
团队成员 B：太好了，发给我看看
团队成员 A：[在 Slack 发了一大段文字]
团队成员 B：[复制到 Notion 文档]
团队成员 C：在哪里找到这个 Prompt？
团队成员 D：这个 Prompt 是最新版本吗？
团队成员 E：我改进了这个 Prompt，怎么同步？
```

**痛点**：
- 难以共享（需要复制粘贴）
- 难以更新（不知道谁有最新版）
- 难以版本控制（没有历史记录）
- 难以发现（散落在各个文档中）

#### 使用 Skill（简单）

**方案**：
```bash
# 1. 创建团队 Skills 仓库
mkdir company-claude-skills
cd company-claude-skills
git init

# 2. 创建 Skills
mkdir -p code-review security-audit testing

# 3. 提交到 Git
git add .
git commit -m "Add team skills"
git push origin main

# 4. 团队成员使用
git clone https://github.com/company/claude-skills.git ~/.claude/skills/company

# 5. 更新 Skills
cd ~/.claude/skills/company
git pull  # 一键更新所有 Skills
```

**优势**：
- ✅ 统一存储（Git 仓库）
- ✅ 版本控制（Git history）
- ✅ 易于发现（README.md 文档）
- ✅ 一键更新（git pull）
- ✅ 权限管理（GitHub/GitLab 权限）
- ✅ 代码审查（Pull Request）

---

### 场景 4：复杂工作流

#### 使用 Prompt（有限）

**限制**：
- 只能写文字指令
- 无法执行脚本
- 无法使用工具
- 无法动态获取数据

**示例**：
```
你：分析这个项目的测试覆盖率
Claude：好的，我来查看代码...
你：等等，你需要先运行 `npm test -- --coverage`
Claude：好的，但我无法直接执行...
你：[手动运行，复制结果]
Claude：[分析复制的结果]
```

#### 使用 Skill（强大）

**能力**：
- ✅ 执行脚本
- ✅ 使用工具（Bash, Grep, 等）
- ✅ 动态获取数据
- ✅ 多步骤自动化

**示例**：
```yaml
---
name: test-coverage
description: Analyze test coverage
allowed-tools: [Bash, Read, Write]
---

# 测试覆盖率分析

## 工作流程

1. 运行测试并生成覆盖率报告
```bash
npm test -- --coverage
```

2. 读取覆盖率数据
```bash
cat coverage/coverage-summary.json
```

3. 分析结果
- 总覆盖率
- 未覆盖的文件
- 关键路径的覆盖情况

4. 生成报告并提出改进建议
```

**使用时**：
```
你：分析测试覆盖率
Claude：[自动执行所有步骤]
        运行测试...
        读取数据...
        分析结果...
        
        测试覆盖率报告：
        - 总覆盖率：78%
        - 未覆盖文件：auth.js, utils.js
        - 建议：增加对认证逻辑的测试
```

---

## 何时使用 Prompt vs Skill

### 使用 Prompt 的场景

#### ✅ 适合用 Prompt

**1. 临时的、一次性的指令**
```
你：用俏皮的语气回答我接下来的问题
```

**2. 针对当前对话的特殊要求**
```
你：接下来用中文回答，但代码用英文注释
```

**3. 简单的角色扮演**
```
你：假设你是苏格拉底，用提问的方式引导我思考
```

**4. 快速实验新想法**
```
你：帮我用三种不同的风格重写这段文案，看看哪个更好
```

**5. 上下文相关的修改**
```
你：把上面的回答简化成 3 句话
```

**6. 简短的格式要求**
```
你：用表格形式展示
```

### 使用 Skill 的场景

#### ✅ 适合用 Skill

**1. 反复使用的工作流**
```yaml
name: code-review
description: 代码审查（每周都要用多次）
```

**2. 特定领域的专业知识**
```yaml
name: security-audit
description: 安全审计（需要深入的专业检查清单）
```

**3. 团队共享的标准**
```yaml
name: company-style-guide
description: 公司代码规范（所有开发者都要遵守）
```

**4. 复杂的多步骤流程**
```yaml
name: api-docs-generator
description: API 文档生成（包含脚本和工具调用）
```

**5. 需要执行脚本的任务**
```yaml
name: test-runner
description: 运行测试并分析结果
allowed-tools: [Bash, Read, Write]
```

**6. 跨项目使用的通用能力**
```yaml
name: git-commit-helper
description: 生成规范的 Git commit 消息（所有项目都用）
```

**7. 需要权限控制的操作**
```yaml
name: prod-deploy
description: 生产环境部署（只允许特定工具）
allowed-tools: [Bash(kubectl *), Bash(helm *)]
```

---

## 类比理解

### 类比 1：医生看病

#### Prompt = 口头描述病情
```
你每次去看医生都要重新说：
"医生，我今年 30 岁，有高血压病史，对青霉素过敏，
 曾经做过阑尾切除手术，父母有糖尿病家族史..."
```

**问题**：
- 每次都要重复
- 容易遗漏重要信息
- 占用看病时间

#### Skill = 电子病历
```
医生调出你的电子病历就知道了：
- 基本信息
- 病史
- 过敏史
- 家族史
- 既往检查结果
```

**优势**：
- 一次录入，永久使用
- 完整准确
- 节省时间
- 医生之间可以共享

---

### 类比 2：餐厅点餐

#### Prompt = 每次详细描述
```
你每次去餐厅都要说：
"我要一份意大利面，但是不要蒜，多加点罗勒，
 酱汁要奶油的不要番茄的，面要煮 7 分钟不要太软，
 最后撒上帕玛森芝士..."
```

**问题**：
- 每次都要详细说明
- 容易说错或遗漏
- 浪费时间

#### Skill = 常客偏好记录
```
服务员看到你就知道：
"老样子？"（系统里有你的偏好设置）
```

**优势**：
- 记录一次，每次生效
- 准确无误
- 高效快捷

---

### 类比 3：软件开发

#### Prompt = 内联代码
```javascript
// 每次都要写重复的逻辑
function processUserA() {
  validate();
  sanitize();
  save();
  sendEmail();
}

function processUserB() {
  validate();  // 重复
  sanitize();  // 重复
  save();      // 重复
  sendEmail(); // 重复
}
```

#### Skill = 可复用函数
```javascript
// 定义一次，到处使用
function processUser(user) {
  validate(user);
  sanitize(user);
  save(user);
  sendEmail(user);
}

// 使用
processUser(userA);
processUser(userB);
processUser(userC);
```

---

## 快速决策树

```
┌─────────────────────────────────────┐
│ 这个指令需要反复使用吗？             │
└──────────────┬──────────────────────┘
               │
       ┌───────┴───────┐
       │               │
      是              否
       │               │
       ▼               ▼
┌─────────────┐  ┌──────────────┐
│ 使用 Skill   │  │ 使用 Prompt  │
└──────┬──────┘  └──────────────┘
       │
       ▼
┌─────────────────────────────────────┐
│ 是否跨多个项目使用？                 │
└──────────────┬──────────────────────┘
               │
       ┌───────┴───────┐
       │               │
      是              否
       │               │
       ▼               ▼
┌─────────────┐  ┌──────────────┐
│ 个人级 Skill │  │ 项目级 Skill │
│ ~/.claude/  │  │ .claude/     │
│   skills/   │  │   skills/    │
└─────────────┘  └──────────────┘
```

### 详细决策流程

```
1. 这个指令会使用多次吗？
   ├─ 否 → 用 Prompt（一次性任务）
   └─ 是 → 继续

2. 指令是否超过 100 字？
   ├─ 否 → 用 Prompt（够简单）
   └─ 是 → 继续

3. 是否需要执行脚本或使用工具？
   ├─ 是 → 用 Skill（Prompt 做不到）
   └─ 否 → 继续

4. 是否需要团队共享？
   ├─ 是 → 用 Skill（便于共享）
   └─ 否 → 继续

5. 是否有专业领域知识？
   ├─ 是 → 用 Skill（专业性强）
   └─ 否 → 继续

6. 是否需要版本控制？
   ├─ 是 → 用 Skill（Git 管理）
   └─ 否 → 用 Prompt（够灵活）
```

---

## 常见误区

### 误区 1："Skill 比 Prompt 更高级"

**错误理解**：
Skill 是高级功能，Prompt 是低级功能。

**正确理解**：
Skill 和 Prompt 是**互补**的，不是替代关系。

- Prompt：灵活、即时、适合临时需求
- Skill：可复用、专业、适合重复任务

**类比**：
就像编程中的内联代码 vs 函数：
- 内联代码：简单、直接
- 函数：复用、抽象

都有各自的使用场景。

---

### 误区 2："所有 Prompt 都应该做成 Skill"

**错误做法**：
```yaml
# 没必要的 Skill
---
name: say-hello
description: Say hello
---
Say hello to the user.
```

**正确做法**：
这种简单的任务直接用 Prompt：
```
你：用热情的语气打个招呼
```

**判断标准**：
如果创建 Skill 的成本 > 重复输入 Prompt 的成本，就不值得做成 Skill。

---

### 误区 3："Skill 会自动优化 Claude 的回答"

**错误理解**：
创建 Skill 后，Claude 的回答质量会自动提升。

**正确理解**：
Skill 只是**存储和自动加载指令的机制**，回答质量取决于：
1. Skill 内容的质量（指令写得好不好）
2. Claude 模型本身的能力

**类比**：
医生的电子病历系统不会让医生变得更专业，
但会让专业医生的工作更高效。

---

### 误区 4："Skill 的 description 可以随便写"

**错误示例**：
```yaml
description: "PDF stuff"
```

**问题**：
Claude 无法判断何时使用这个 Skill。

**正确示例**：
```yaml
description: "Extract and analyze text from PDF documents. Use when users ask to process PDFs, read PDFs, extract text from PDF files, or analyze PDF content."
```

**关键**：
description 是触发 Skill 的**唯一依据**，必须：
- 清晰描述使用场景
- 包含关键词（PDF, extract, analyze）
- 说明何时应该触发

---

## 最佳实践

### Prompt 最佳实践

#### 1. 保持简洁
```
✅ 好：用表格展示结果
❌ 差：请你把结果整理成一个表格的形式，这样会更清晰...
```

#### 2. 清晰具体
```
✅ 好：用 Python 写一个快速排序算法
❌ 差：写个排序
```

#### 3. 分步骤说明
```
✅ 好：
1. 先分析代码结构
2. 然后找出性能瓶颈
3. 最后给出优化建议

❌ 差：分析并优化这段代码
```

#### 4. 提供示例
```
✅ 好：
用这种格式输出：
Issue: [问题描述]
Impact: [影响]
Solution: [解决方案]

❌ 差：列出问题和解决方案
```

---

### Skill 最佳实践

#### 1. 写清晰的 description

```yaml
# ✅ 好
description: "Review code for security vulnerabilities. Use when asked to: review security, audit code, find vulnerabilities, check for exploits, analyze security risks."

# ❌ 差
description: "Code review"
```

**技巧**：
- 描述功能（做什么）
- 列举触发词（review security, audit, find vulnerabilities）
- 保持在 1-2 句话

#### 2. 结构化 Skill 内容

```markdown
# ✅ 好的结构
---
name: test-generator
description: Generate comprehensive tests
---

# 测试生成专家

## 何时使用
当用户请求生成测试、添加测试覆盖率时

## 工作流程
1. 分析目标代码
2. 识别测试场景
3. 生成测试代码
4. 验证覆盖率

## 测试类型
- 单元测试
- 集成测试
- E2E 测试

## 输出格式
[示例代码]

## 注意事项
- 使用项目的测试框架
- 遵循命名规范
```

```markdown
# ❌ 差的结构
---
name: test-generator
description: Generate tests
---

生成测试，关注覆盖率、边界情况、错误处理...
（没有结构，一段话堆在一起）
```

#### 3. 提供具体示例

```markdown
## 输出示例

### 单元测试
```javascript
describe('calculateTotal', () => {
  it('should calculate total with tax', () => {
    expect(calculateTotal(100, 0.1)).toBe(110);
  });
  
  it('should handle zero tax', () => {
    expect(calculateTotal(100, 0)).toBe(100);
  });
});
```

这样 Claude 知道期望的输出格式。
```

#### 4. 使用工具限制

```yaml
# 只读分析，不修改代码
---
name: security-audit
allowed-tools: [Read, Grep, Glob]
---

# 部署操作，只允许特定命令
---
name: deploy
allowed-tools: [Bash(kubectl *), Bash(helm *)]
---
```

#### 5. 分离关注点

```
# ✅ 好：每个 Skill 专注一件事
code-review/        # 代码审查
security-audit/     # 安全审计
performance-opt/    # 性能优化
test-generator/     # 测试生成

# ❌ 差：万能 Skill
super-code-helper/  # 做所有事情
```

#### 6. 版本控制

```bash
# 为 Skills 创建 Git 仓库
cd ~/.claude/skills
git init
git add .
git commit -m "Initial skills"

# 更新 Skill
# 编辑 SKILL.md
git commit -am "Update code-review skill"

# 查看历史
git log
```

#### 7. 文档化

```markdown
# 在 Skill 目录创建 README.md
code-review/
├── SKILL.md          # Skill 定义
├── README.md         # 使用说明
└── examples/         # 示例
    └── example.md
```

**README.md 示例**：
```markdown
# Code Review Skill

## 功能
自动审查代码的安全性、性能和最佳实践。

## 使用方法
直接说：
- "审查这段代码"
- "review this code"
- "帮我看看代码质量"

或手动调用：
```
/code-review
```

## 检查清单
- [x] SQL 注入
- [x] XSS 漏洞
- [x] 性能瓶颈
- [x] 错误处理
- [x] 代码可读性

## 更新日志
- 2026-02-13: 添加 OWASP Top 10 检查
- 2026-02-01: 初始版本
```

---

## 总结

### 核心要点

1. **Prompt 是临时对话，Skill 是可复用知识包**
   - Prompt：当场说
   - Skill：预先配置

2. **两者互补，不是替代**
   - 简单临时任务 → Prompt
   - 重复专业任务 → Skill

3. **Skill 的核心优势**
   - 写一次，用无数次
   - 自动触发，无需手动
   - 节省上下文空间
   - 便于团队共享

4. **何时使用 Skill**
   - 重复使用（>3 次）
   - 复杂指令（>100 字）
   - 需要脚本/工具
   - 团队协作
   - 专业领域知识

5. **description 是关键**
   - 清晰描述功能
   - 包含触发关键词
   - 1-2 句话说清楚

### 快速参考卡片

```
┌────────────────────────────────────────────────┐
│           Prompt vs Skill 速查表                │
├────────────────────────────────────────────────┤
│                                                │
│  临时任务     → Prompt                          │
│  重复任务     → Skill                           │
│  简单指令     → Prompt                          │
│  复杂流程     → Skill                           │
│  个人实验     → Prompt                          │
│  团队标准     → Skill                           │
│  纯文字指令   → Prompt                          │
│  需要脚本     → Skill                           │
│  一次性       → Prompt                          │
│  可复用       → Skill                           │
│                                                │
└────────────────────────────────────────────────┘
```

### 下一步行动

**如果你现在主要用 Prompt**：
1. 找出你最常重复的 3-5 个指令
2. 把它们做成 Skills
3. 体验自动触发的便利

**如果你想深入 Skills**：
1. 阅读《Claude Code Skills 完整学习指南》
2. 查看官方 Skills 源码学习
3. 参与社区分享你的 Skills

**推荐资源**：
- 官方文档: https://code.claude.com/docs/skills
- 社区集合: https://github.com/hesreallyhim/awesome-claude-code
- 学习指南: https://blakecrosley.com/en/guides/claude-code

---

**文档版本**：v1.0  
**最后更新**：2026-02-13  
**作者**：Claude

希望这份文档能帮你彻底理解 Prompt 和 Skill 的区别！🚀
