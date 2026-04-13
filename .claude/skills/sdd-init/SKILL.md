---
name: sdd-init
description: 初始化一个 SDD（Spec-Driven Development）项目骨架。当用户说"初始化 SDD 项目"、"init SDD project"、"初始化 claude code 项目"、"初始化 spec 驱动开发项目"、"创建 .claude 目录结构"、"scaffold SDD"、"setup SDD project" 时使用此 skill。它会创建 ./specs/、./.claude/{skills,agents,commands,hooks}/ 目录，写入 .claude/settings.json 权限配置、.gitignore，并在 constitution.md 和 CLAUDE.md 缺失时从模板填充。
---

# SDD 项目初始化 Skill

## 目标
在当前项目根目录（`./`）初始化一个支持 Spec-Driven Development 的 Claude Code 项目骨架。

## 前置交互（必做）
在执行任何文件写入前，向用户确认以下信息：

1. **项目名**：用于 `.gitignore` 中忽略构建产物的条目（示例中的 `/my-project-name` 需要替换）。
2. **是否已有同名目录/文件**：提醒用户，本 skill 不会覆盖已存在的 `constitution.md`、`CLAUDE.md`、`.gitignore`、`.claude/settings.json`；如需重新生成，请用户先手动备份或删除。

## 执行步骤

### Step 1：创建目录结构
在项目根目录下创建以下目录（存在则跳过）：

- `./specs/`
- `./.claude/skills/`
- `./.claude/agents/`
- `./.claude/commands/`
- `./.claude/hooks/`

使用 `mkdir -p` 一次性创建。

### Step 2：写入 `.gitignore`
如果 `./.gitignore` 不存在，创建并写入以下内容。如果已存在，**仅追加缺失的条目**（逐行比对，避免重复）：

```
.claude/settings.local.json
.vscode/
.idea/
*.DS_Store
/<project-name>
```

> `<project-name>` 使用 Step 0 中用户提供的项目名替换。

### Step 3：写入 `.claude/settings.json`
如果 `./.claude/settings.json` 不存在，写入以下内容。如果已存在，**跳过并提示用户**：现有 settings.json 未被覆盖，如需使用默认权限配置请手动合并。

```json
{
  "permissions": {
    "defaultMode": "plan",
    "allow": [
      "Read(*.java)",
      "Read(pom.xml)",
      "Read(Makefile)",
      "Read(README.md)",
      "Read(constitution.md)",
      "Read(specs/**)",
      "Grep",
      "Glob",
      "LS"
    ],
    "ask": [
      "Write",
      "Edit",
      "MultiEdit",
      "Bash(git:add:*)",
      "Bash(git:commit:*)"
    ],
    "deny": [
      "Read(./.env*)",
      "Read(*.pem)",
      "Read(*.key)",
      "Bash(rm:*)",
      "Bash(git:push:*)",
      "WebFetch"
    ]
  }
}
```

### Step 4：初始化 `constitution.md` 和 `CLAUDE.md`
对每个文件：

- 如果文件**不存在**或**内容为空**（`wc -c` 为 0 或仅空白字符），从模板拷贝：
  - `~/.claude/skills/sdd-init/templates/constitution.md` → `./constitution.md`
  - `~/.claude/skills/sdd-init/templates/CLAUDE.md` → `./CLAUDE.md`
- 如果文件**已有非空内容**，跳过并告知用户：该文件已有内容，如需参照模板重建可查看 `~/.claude/skills/sdd-init/templates/` 下对应文件。

### Step 5：汇总报告
执行结束后，以清单形式向用户报告：

```
已创建：
  - 目录：specs/, .claude/skills/, .claude/agents/, .claude/commands/, .claude/hooks/
  - 文件：.gitignore、.claude/settings.json、constitution.md、CLAUDE.md

已跳过（文件已存在且非空）：
  - <列出被跳过的文件>

下一步建议：
  - 编辑 constitution.md 调整技术栈与项目约束
  - 在 ./specs/ 目录下创建第一份规格文件
```

## 实现要点

- **不覆盖已有文件**：所有写入前先检查文件是否存在；对 `.gitignore` 采用追加模式（逐条去重）。
- **模板路径**：模板文件位于 `~/.claude/skills/sdd-init/templates/`，使用 `cp` 拷贝而非生成内容，避免内容漂移。
- **批量执行**：目录创建和 `.gitignore` 追加可用单次 Bash 调用完成；文件存在性判断应在写入前执行。
- **路径规范**：所有目标路径以 `./` 开头，表示当前项目根目录（执行 skill 时的 cwd）。
