# claude-java

面向 Java 项目的 Claude Code 共享配置仓库。本仓库集中维护 `.claude/` 目录下的 commands、skills 与 settings，便于在多个 Java 项目间复用同一套 AI 协作规范、审查流程与自动化脚本。

## 使用方式

将本仓库 `.claude/` 目录下需要的文件拷贝到目标项目根目录的 `.claude/` 下，Claude Code 会自动加载其中的 commands 与 skills。

## 目录结构

```
.claude/
├── settings.json              # 项目级权限配置
├── settings.local.json        # 本地个人配置（不纳入团队共享）
├── commands/                  # 自定义 slash commands
│   ├── commit.md
│   ├── docker-build.md
│   ├── java-constitution.md
│   ├── package.md
│   └── review-java.md
└── skills/                    # 自定义 skills
    ├── sdd-init/              # SDD 项目初始化骨架
    └── tqh-monitor/           # 通勤号车票监控服务
```

## Commands 清单

在 Claude Code 会话中以 `/<name>` 形式调用。

| 命令 | 功能描述 |
|------|---------|
| `/commit` | 分析 `git diff --staged`，生成符合 Conventional Commits 规范的单行提交信息，展示给用户确认后执行 `git commit`。 |
| `/docker-build` | 调用 `make docker-build IMAGE_TAG=<tag>` 构建 Docker 镜像，支持通过参数指定 Tag（默认 `latest`），成功后展示镜像信息，失败时自动分析错误日志给出修复建议。 |
| `/java-constitution` | 为 Java 项目初始化 `constitution.md`（项目宪法）与 `CLAUDE.md`（AI 协作指南），建立"简单性 / 测试先行 / 明确性"三原则及 Conventional Commits、TDD、异常处理等规范；已存在的文件不会被覆盖。 |
| `/package` | 调用 `make package` 执行项目打包，失败时自动分析日志并给出修复建议。 |
| `/review-java` | 作为高级 Java 工程师对指定代码执行严格的 Code Review，严格依据项目宪法三原则输出"必须修复 / 建议改进 / 值得肯定 / 合规检查"四段式清单。 |

## Skills 清单

Skills 由 Claude 依据场景自动触发，也可显式引用。

| Skill | 功能描述 |
|-------|---------|
| `sdd-init` | 初始化 Spec-Driven Development 项目骨架：创建 `./specs/` 与 `./.claude/{skills,agents,commands,hooks}/` 目录，写入默认 `settings.json` 权限配置与 `.gitignore`，在 `constitution.md` / `CLAUDE.md` 缺失时从模板填充。 |
| `tqh-monitor` | 通勤号巴士车票监控与自动下单系统的操作入口：支持启动/停止监控服务、监控本周/下周/指定日期车票、监听购票日志，并提供 `check-service.sh`、`watch-ticket-log.sh` 辅助脚本。 |

## 项目宪法三原则

由 `/java-constitution` 命令写入的项目宪法是本仓库所有 Java 代码协作的最高准则：

1. **简单性** — 遵循 SOLID，拒绝不必要的抽象与依赖。
2. **测试先行** — 所有功能/修复，从失败的测试开始（TDD Red-Green-Refactor）。
3. **明确性** — 代码首先服务于人类的理解：命名清晰、无魔法数字、圈复杂度 ≤ 10、嵌套 ≤ 3 层。
