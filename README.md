# vibe-bootstrap for Windows + Codex

今天让 Codex 改一段代码，明天换个会话还能接着做。vibe-bootstrap 会在项目里留下开发规则、运行和测试命令、当前进度。先把一个真实功能跑通，再往上加。

## 1. 安装

准备好 Codex、Git 和 Python 3.11+。下载 [vibe-bootstrap-win-codex.skill](./vibe-bootstrap-win-codex.skill)。在下载目录打开 PowerShell，粘贴以下命令。文件可以直接解包，保留 `.skill` 后缀即可。

```powershell
$package = (Resolve-Path -LiteralPath '.\vibe-bootstrap-win-codex.skill' -ErrorAction Stop).Path
$skillsDir = Join-Path $env:USERPROFILE '.agents\skills'
$skillDir = Join-Path $skillsDir 'vibe-bootstrap'

if (Test-Path -LiteralPath $skillDir) {
    throw '已安装 vibe-bootstrap。升级前请先备份并移开旧目录。'
}

python -X utf8 -m zipfile -e "$package" "$skillsDir"
if ($LASTEXITCODE -ne 0) { throw '解包失败。' }
Get-Item -LiteralPath (Join-Path $skillDir 'SKILL.md') -ErrorAction Stop
```

看到 `SKILL.md` 就解包成功。新会话找不到技能时，重启 Codex。团队也可以把整个 `vibe-bootstrap` 文件夹放进项目的 `.agents/skills/`，供该仓库使用。 [Codex Skill 安装位置](https://learn.chatgpt.com/docs/build-skills)

## 2. 在 Codex 里开始

在目标项目文件夹打开 PowerShell。它还不是 Git 仓库时，在这里运行 `git init -b main`。用 Codex 打开同一文件夹，新开会话，发送下面这段话，把例子换成你想做的项目。

```text
$vibe-bootstrap 请用 Windows + Codex 预设初始化这个项目。
我想做一个本地记账工具。第一版只要能记一笔账，退出后还能查到。先和我确认怎么拆分，再开始写。
```

Codex 会先和你确认第一版范围及模块设计。生成后，`AGENTS.md` 写项目规则，`project-tasks.json` 写运行和测试命令，`docs/STATUS.md` 写做到哪里。生成的是模板。让 Codex 填好实际命令和依赖，再启动应用、试一次核心功能、跑测试与构建。

### 已经有项目

在项目里发送这句，让 Codex 先了解现状。

```text
$vibe-bootstrap 接管这个 Windows 项目。先读现有代码、运行方式和测试，列出建议新增和需要合并的文件，先别动业务代码。
```

脚手架跳过已有同名文件。核对改动清单后再合并已有的 `AGENTS.md`、任务清单、GitHub Actions 自动检查和钩子设置。安全规则和 CI 的修改由你确认并通过授权入口应用。升级 S/M/L 时一并核对任务清单与自动检查的等级。S 适合个人工具，M 适合多模块应用，L 适合多包、多服务或并行开发。

## 3. 运行和接着做

在项目根目录打开 PowerShell。应用持续运行时，用另一个终端执行检查。

```powershell
python -X utf8 scripts/project.py dev
```

```powershell
python -X utf8 scripts/project.py init-verify
python -X utf8 scripts/project.py check
python -X utf8 scripts/project.py build
```

`init-verify` 检查配置和模板有没有填完。实际功能、测试、构建和下面的钩子验收都通过后，由你移除 `docs/INIT_PROGRESS.md`，后续会话按 `docs/STATUS.md` 继续。

要暂停时，对 Codex 说“收工，记下最后跑过什么和下一步”。下次说“继续”，它就有进度和 Git 记录可查。

需要亲自看脚手架会生成哪些文件时，在项目根目录运行下面的预览。

```powershell
$scaffold = Join-Path $env:USERPROFILE '.agents\skills\vibe-bootstrap\scripts\scaffold.py'
python -X utf8 "$scaffold" --codex --level S --stack python --dry-run
```

也可选 `--stack node`。S 默认原生构建，M/L 默认容器构建，可用 `--deployment native` 或 `--deployment container` 指定。容器项目需要 Docker，构建后运行 `python -X utf8 scripts/project.py smoke` 试启动并检查容器。M/L 的密钥检查需要 Gitleaks。

## 需要你确认的操作

如果 Codex 拦下删除、重写 Git 历史、发布或权限变更，请先看它报告的目标、影响和恢复办法。项目的操作钩子（Hook）会直接拒绝命中的调用。聊天里回“同意”后，仍由你通过授权的入口执行。

启用前核对 `.codex/hooks.json`。在 Codex CLI 用 `/hooks`，桌面端用设置里的 Hooks 核对并信任。运行 `python -X utf8 scripts/project.py guard` 检查配置，再在可丢弃的仓库里试一次拒绝，确认测试文件没变。需要强制人审的文件、主分支和生产环境，应由系统或远端平台限制 AI 的直接权限，再让人逐次批准。出现预先配置的人工确认弹窗时，核对目标与参数，只批准本次。Codex 的 `auto_review` 由模型审核。 [Codex Hooks](https://learn.chatgpt.com/docs/hooks) · [Codex 审批与安全](https://learn.chatgpt.com/docs/agent-approvals-security)

更多细节见解包后的 `references/12-windows-codex.md`。里面有接管、构建、审批和换会话的验收步骤。
