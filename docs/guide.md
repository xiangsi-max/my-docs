# 使用指南

本页说明一次完整的「改文档 → 上线」流程。所有命令在仓库根目录执行。

## 操作流程

1. 修改 `docs/` 目录中的 Markdown 文件。
2. 在本地预览并检查页面及链接。
3. 提交修改并推送到 `main` 分支。
4. 在 Actions 页面查看构建与部署状态。
5. 打开网站检查内容是否更新。

## 步骤一：本地预览

先建虚拟环境并安装依赖（只需做一次）：

```bash
python -m venv .venv
source .venv/Scripts/activate     # Windows Git Bash
python -m pip install -r requirements.txt
```

??? note "Windows PowerShell 的激活命令不同"
    在 PowerShell 中应执行 `.\.venv\Scripts\Activate.ps1`。如果被执行策略拦住，
    可以直接用虚拟环境里的解释器，不必激活：`.\.venv\Scripts\python.exe -m mkdocs serve`。

启动预览服务器：

```bash
python -m mkdocs serve
```

浏览器打开 `http://127.0.0.1:8000/`。这一版是热重载的——保存 Markdown 时页面会自动刷新，
适合边写边看。检查完按 `Ctrl+C` 停止。

## 步骤二：构建检查

预览只是「看起来对」，构建才能确认「真的没问题」：

```bash
python -m mkdocs build --strict
```

`--strict` 把警告升级为错误。常见的失败原因就三类：链接指向不存在的文件、`nav` 里写了不存在的页面、
Markdown 里的图片路径拼错。**本地先过一遍 `--strict`，可以避免把失败推上去才发现。**

## 步骤三：提交并推送

```bash
git status
git add docs mkdocs.yml requirements.txt .github/workflows/deploy.yml
git diff --staged
git commit -m "更新使用指南"
git push origin main
```

`git diff --staged` 这一步不要省，它能帮你发现误提交的 `.venv/` 或 `site/`。

## 常用命令对照

| 命令 | 作用 | 什么时候用 |
| --- | --- | --- |
| `python -m mkdocs serve` | 本地预览，热重载 | 写文档的过程中 |
| `python -m mkdocs build` | 生成 `site/` | 想看看产物长什么样 |
| `python -m mkdocs build --strict` | 严格构建，警告即失败 | 提交前必做 |
| `git diff --staged` | 检查即将提交的内容 | 每次 commit 前 |
| `git log --oneline -5` | 看最近几次提交 | 确认提交成功 |

## 触发规则

工作流只在下面这些路径发生变化时才运行：

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'docs/**'
      - 'mkdocs.yml'
      - 'requirements.txt'
      - '.github/workflows/deploy.yml'
```

也就是说：改 `report/README.md`、改根目录 `README.md` 都**不会**触发构建——
因为它们不影响网站内容，跑了也是浪费。需要时可以在 Actions 页面用 `Run workflow` 手动运行（对应 `workflow_dispatch`）。

!!! tip "排错的顺序"
    部署失败时不要急着改配置，先按这个顺序看：
    **① 哪一步红了 → ② 展开那一步的日志找第一行报错 → ③ 本地复现同样的命令 → ④ 改完再推。**
    绝大多数失败都是依赖版本、路径大小写或 `--strict` 下的死链，属于本地就能复现的问题。

## 我踩过的两个坑

第一次自己搭这套东西时，有两处卡住的时间比写文档本身还长，写在这里省得别人重踩。

**第一个坑是激活虚拟环境。** 我原来照着网上教程敲 `source .venv/bin/activate`，
在 Windows 的 Git Bash 里根本找不到那个路径——Windows 的虚拟环境目录是 `.venv/Scripts/` 而不是 `.venv/bin/`，
激活脚本也分成 `activate`（bash）和 `Activate.ps1`（PowerShell）两套。后来我干脆不激活了，
直接用 `./.venv/Scripts/python.exe -m mkdocs serve` 调用虚拟环境里的解释器，效果完全一样，还少一步。
**结论：激活只是为了省点打字，不激活也不会出错。**

**第二个坑是预览地址多了个尾巴。** 因为 `mkdocs.yml` 里 `site_url` 是
`https://xiangsi-max.github.io/my-docs/`，预览服务器会把整个站点挂在 `/my-docs/` 这个子路径下。
我第一次打开 `http://127.0.0.1:8000/` 看到的是 302 跳转而不是页面，还以为服务没起来。
实测下来 `/` 返回 302、`/my-docs/` 才返回 200。**本地预览时末尾的 `my-docs/` 不能省。**

## 我的一点体会

这套流程最值钱的地方不是省了上传，而是**把「发布」变成了提交的一部分**——
我改完文档 push，剩下的全自动，不用再记得「还要去传一次文件」。
但自动化也放大了错误的代价：以前传错一个文件只是那一页坏，现在一次构建失败会让整站停在旧版本。
所以我养成了推送前必跑 `build --strict` 的习惯：本地死链警告和线上构建用的是同一套规则，
本地过不了，推上去一样过不了，与其等 CI 花两分钟报错，不如本地两秒钟就发现。

## 相关页面

- [首页](index.md)——站点信息概览与快速导航。
- [关于本项目](about.md)——技术选型与目录职责说明。
- [API 参考](api.md)——用 Markdown 写接口文档的示例。
