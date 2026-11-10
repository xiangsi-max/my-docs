# 实践项目2：个人实验报告

## 1. 基本信息

- **姓名拼音**：luoweicheng
- **GitHub 登录名**：xiangsi-max
- **个人仓库地址**：https://github.com/xiangsi-max/my-docs
- **最终网站地址**：https://xiangsi-max.github.io/my-docs/
- **教师仓库 Issue**：[#90 02-github-pages luoweicheng](https://github.com/hunnuOpenSourceSoftware2026/assignments/issues/90)
- **实践主题**：用 MkDocs 编写静态文档，交给 GitHub Actions 自动构建并发布到 GitHub Pages

我这次没有手工往任何服务器上传过一个 HTML 文件。网站每一次「发布」（首次上线、第二次更新内容、第三次补充页面链接、第四次修订正文）全部由 `git push` 触发，四次运行都在 Actions 里留下了记录。而中间只推报告的那次提交则一次运行都没产生——这个对比恰好说明了触发条件是精准的。

---

## 2. 实践过程与关键截图

### 2.1 仓库创建

**做了什么**：用本人账号新建公开仓库 `my-docs`，默认分支 `main`，然后把本地写好的源码推送上去。

**为什么这样做**：Pages 站点必须是公开可访问的，私有仓库的 Pages 需要付费计划；而工作流里写死了 `branches: [main]`，如果默认分支叫 `master`，后面 push 根本不会触发。

**结果**：仓库首页可见 `Public` 标识、所有者 `xiangsi-max`、仓库名 `my-docs`、默认分支 `main`（`1 Branch`、`0 Tags`），下方是两次提交记录。

![个人公开仓库首页](images/01-repository.png)

> 截图说明：地址栏是 `github.com/xiangsi-max/my-docs`，仓库名下方能看到 `Public` 徽标——这是验收里明确要求的第一项。

### 2.2 项目结构与 MkDocs 配置

**做了什么**：按作业要求的目录结构建立文件，`mkdocs.yml` 里写死站点名、`site_url`、导航和 Material 主题。

**为什么这样做**：`mkdocs.yml` 是整站的唯一配置入口——`site_url` 决定了所有页面里链接的绝对地址前缀，`nav` 决定了顶栏四个入口的顺序和名字。这两项写错，后面所有页面都会受影响，所以先把它配对再写内容。

![项目结构与 mkdocs.yml](images/02-project-structure.png)

三层目录的分工，我是这样理解的：

| 位置 | 保存什么 | 谁修改 |
| --- | --- | --- |
| `docs/` | 四页 Markdown 源文件 | 我手工写 |
| `mkdocs.yml` | 站点名、导航顺序、主题、扩展 | 我手工写 |
| `site/` | 构建出来的 HTML、CSS、JS | **机器生成，我从不碰** |

`nav` 里的路径是**相对 `docs/`** 的，所以写 `index.md` 而不是 `docs/index.md`。这一点我一开始差点写错，`mkdocs build --strict` 会直接报"文件不存在"把它拦下来。

`site/` 和 `.venv/` 都进了 `.gitignore`。理由很直接：它们是本地运行产生的中间物，体积大、每次构建都变，提交进去只会让 diff 变脏，而且和工作流里"重新构建一份"完全重复。

**结果**：本地 `mkdocs build --strict` 通过，`site/` 目录被正确生成且未进入 git 暂存区；`git status` 里看不到 `site/` 和 `.venv/` 的任何文件。

### 2.3 本地构建与预览

**做了什么**：建虚拟环境、装依赖、严格构建、本地预览。

**为什么这样做**：作业把 Python 依赖装进独立虚拟环境，是为了不影响系统 Python；而先跑一遍 `--strict` 构建，是因为工作流里用的命令就是它——**本地能过，CI 才可能过**，把问题留在本地比等到 Actions 红了再改快得多。

```bash
python -m venv .venv
source .venv/Scripts/activate          # Windows Git Bash
python -m pip install -r requirements.txt
python -m mkdocs build --strict
python -m mkdocs serve --dev-addr 127.0.0.1:8011
```

![本地预览](images/03-local-preview.png)

**检查了哪些内容**：

1. 顶部导航四个入口「首页 / 关于 / 使用指南 / API参考」都能点进去，页面标题正确对应；
2. 每页的表格渲染成了真正的表格（边框、表头对得上），不是竖线堆在一起；
3. 代码块有语法高亮和复制按钮；
4. 页面之间的链接能跳转，没有 404；
5. 右侧目录（TOC）能跟随滚动高亮。

> 这里有个容易踩的点：因为 `site_url` 带了子路径 `/my-docs/`，`mkdocs serve` 实际的服务地址是 `http://127.0.0.1:8011/my-docs/`，**末尾的 `my-docs/` 不能省**。我实测过：访问根路径 `/` 返回的是 `302` 跳转，只有 `/my-docs/` 才返回 `200`；用浏览器看不出差别，但如果直接 `curl` 根路径去验证页面内容，拿到的是跳转而不是 HTML，很容易误判成"服务没起来"。

**结果**：`python -m mkdocs build --strict` 退出码为 **0**（0 警告 0 错误），`site/` 生成成功；本地预览四个页面全部返回 200，导航、表格、代码块、页面间链接都正常。

### 2.4 Pages 发布来源设置

**做了什么**：`Settings` → `Pages` → `Build and deployment` → `Source` 选 **GitHub Actions**。

**为什么这样做**：作业明确要求走官方 Pages Actions 方案。如果选成 "Deploy from a branch"，GitHub 会自己去找 `gh-pages` 分支，那我们的工作流就没有意义了——它会一直报"找不到分支"。

![Pages 设置](images/04-pages-settings.png)

截图里能看到 "Your site is live at https://xiangsi-max.github.io/my-docs/"，以及 "last deployed ... by the Deploy MkDocs to GitHub Pages workflow"，说明发布来源确实绑定到了我们的工作流上。

**结果**：`Source` 的值为 `GitHub Actions`，`gh api repos/xiangsi-max/my-docs/pages` 返回的 `build_type` 是 `workflow`（分支发布方案对应的是 `legacy`），确认没有混用分支发布设置。

### 2.5 Actions 首次构建与部署

**做了什么**：把源码提交推送到 `main`，工作流自动触发。

**为什么这样做**：这是配置 Pages 来源之后的第一次验证。第一次运行要确认的不是"网站好不好看"，而是**整条链路是否打通**：`on.push` 能不能被监听到、`build` 能不能装好依赖并构建成功、`deploy` 能不能拿到权限并发布出去。任何一环断了，后面的自动更新都无从谈起。

- 提交：`3ee140e` [搭建MkDocs文档并配置Pages自动部署](https://github.com/xiangsi-max/my-docs/commit/3ee140e8e1b2748d3b535f71e45ffde7d358428b)
- 运行：#1 https://github.com/xiangsi-max/my-docs/actions/runs/42637567837

![首次运行成功](images/05-actions-success.png)

**结果**：`build` 与 `deploy` 两个任务都是绿色。时间线（UTC）：

| 任务 | 开始 | 结束 | 耗时 |
| --- | --- | --- | --- |
| build | 14:44:12 | 14:44:31 | 约 19 秒 |
| deploy | 14:44:37 | 14:46:49 | 约 2 分 12 秒 |

build 只花了不到 20 秒，大部分时间花在 deploy——因为这一步要真正把产物推到 Pages 的 CDN 并等待生效。

### 2.6 网站访问与四页导航检查

**做了什么**：用浏览器打开部署后的公开网站，逐个点开四个页面。

![公开网站首页](images/06-site-home.png)

**为什么这样做**：build 和 deploy 两个任务都是绿的，只能说明"产物传上去了"，**不代表用户能真正打开页面**。项目站点的地址带仓库子路径 `/my-docs/`，最容易出的问题恰恰是资源路径和内部链接前缀不对——这类问题在 Actions 日志里看不出来，只有真的打开页面才会暴露。

四个页面逐一打开确认：

| 页面 | 地址 | 状态 |
| --- | --- | --- |
| 首页 | https://xiangsi-max.github.io/my-docs/ | 200 |
| 关于 | https://xiangsi-max.github.io/my-docs/about/ | 200 |
| 使用指南 | https://xiangsi-max.github.io/my-docs/guide/ | 200 |
| API 参考 | https://xiangsi-max.github.io/my-docs/api/ | 200 |

截图保留了完整地址栏，可以看到域名是 `xiangsi-max.github.io/my-docs/`，**带上了仓库路径**——项目站点必须带这段前缀，漏掉就 404。

**结果**：四页全部返回 `200`，顶栏四个入口互相跳转正常，页面标题分别是「我的技术文档」「关于 - 我的技术文档」「使用指南 - 我的技术文档」「API参考 - 我的技术文档」，表格、代码块和内部链接都正常渲染。

### 2.7 第二次推送与自动更新验证

**做了什么**：在 `docs/index.md` 末尾加一句话，提交推送，**全程没有点任何手动部署按钮**。

**为什么这样做**：作业第 6.2 节要求"再修改一次，证明能够自动更新"。首次部署成功只能说明工作流配通了，**不能证明它会在每次推送时自动再跑一遍**——万一是靠点按钮触发的，自动化的意义就没了。所以必须用一个只改内容的提交，看它能不能自己触发。

```markdown
## 自动部署验证

自动部署验证：这是我第二次推送后更新的内容。
```

- 提交：`0217e83` [验证文档修改后自动更新网站](https://github.com/xiangsi-max/my-docs/commit/0217e83266141048c3ef6195d3dc220c43ab6767)
- 运行：#2 https://github.com/xiangsi-max/my-docs/actions/runs/42639052805

![第二次运行成功](images/07-second-push.png)

| 任务 | 开始 | 结束 | 耗时 |
| --- | --- | --- | --- |
| build | 14:53:15 | 14:53:33 | 约 18 秒 |
| deploy | 14:53:38 | 14:54:46 | 约 1 分 08 秒 |

push 后约 2 秒就开始跑了（14:53:13 创建，14:53:15 起 build），说明 `on.push` 的监听是即时生效的。

刷新公开网站后，新增的文字出现了：

![网站出现新增文字](images/07b-site-updated.png)

**结果**：`build` 与 `deploy` 均 success，网页上出现了新增的那行文字，全程没有手动操作。

到这里"推送 → 构建 → 部署 → 网页更新"这条链路被完整验证了一遍。整个过程我只做了一件事：改了一个 Markdown 文件然后 push。

### 2.8 第三次推送：补充页面间链接

**做了什么**：给 `about` / `guide` / `api` 三页各加一个「相关页面」小节，四页两两互链。

**为什么这样做**：自查时我发现一个问题——作业要求四页都含页面间链接，但我最初只在首页写了指向另外三页的链接，`about` / `guide` / `api` 三页是"被链接"但没有"链回去"。虽然勉强能用顶部导航代替，但严格按字面要求是不达标的。补这一次也是为了顺带确认：不是只有"第二次"能触发部署，后续每次 push 都会自动跑。

于是我给三页各加了一个「相关页面」小节，互相之间全部连上：

```markdown
## 相关页面

- [首页](index.md)——站点信息概览与快速导航。
- [关于本项目](about.md)——技术选型与目录职责说明。
- [使用指南](guide.md)——本地预览、严格构建与推送流程。
```

- 提交：`bb0c4f8` [补充三页的页面间链接](https://github.com/xiangsi-max/my-docs/commit/bb0c4f830d922e481928d8978a60d2ae483442ac)
- 运行：#3 https://github.com/xiangsi-max/my-docs/actions/runs/42651682474
- 结果：build 与 deploy 均 success

这次推送**再次自动触发了**工作流，说明第 2 次不是偶然。上线后我验证了三页都出现了新增的「相关页面」区块：

```text
/about/  含「相关页面」3 处
/guide/  含「相关页面」3 处
/api/    含「相关页面」3 处
```

### 2.9 反证实验：只改报告会怎样

**做了什么**：提交 `0870b6b` [报告补充第三次推送与反证实验记录](https://github.com/xiangsi-max/my-docs/commit/0870b6b9bf68c373d5096288bdfc137b99eb76b4) **只**包含 `report/README.md`（就是你正在读的这一段），不碰任何 `docs/` 下的文件，推上去之后观察 Actions。

**为什么这样做**：作业第 4 问问"只修改 report/README.md 会触发部署吗"，这个问题可以直接从 `paths` 配置推断出来，但推断和实测是两回事——我想拿真实数据回答，而不是照着配置文件复述一遍。

推送前运行数是 3 次，推送后等了 90 秒再看：

| 时间点 | 工作流运行数 |
| --- | --- |
| 推送前 | 3 |
| 推送 `0870b6b` 后 90 秒 | **3** |

**结果**：**没有新增运行。** GitHub 在收到 push 后比对了改动文件列表，`report/README.md` 没有命中 `paths` 里的任何一条，于是直接没有创建工作流运行——连"跳过"的记录都没留下，因为它压根没启动。

这个反证让第 4 问从"读配置推断"变成了"实测确认"：`paths` 过滤是生效的，不是碰巧没跑。

### 2.10 第四次推送：按自查清单回头改内容

**做了什么**：把作业第 467–475 行的"提交前检查"清单逐条拿到仓库里实测核对，发现 3 处对不上，回头改掉再推。

**为什么这样做**：清单是我自己勾的，勾之前没有一条一条验过。作业第 465 行写着"教师可结合提交记录、Actions 日志和现场解释核验本人完成情况"——**自查清单如果和仓库实际状态对不上，比不写更糟**，因为它会让人以为已经检查过了。

| 自查项 | 核对结果 | 怎么处理 |
| --- | --- | --- |
| 所有示例账号、仓库地址均已替换 | `docs/api.md` 的 curl 示例还是 `https://example.com/...` | 换成 `https://xiangsi-max.github.io/my-docs/api/docs` |
| 四页均有本人撰写的介绍与体会 | `guide.md`、`api.md` 全文一个"我"字都没有，是纯教程口吻 | 两页各补一节第一人称内容（见下） |
| 四页两两互链 | 已在上一次推送补上 | — |

第一处是硬伤：`example.com` 是 RFC 2606 保留的示例域名，留着等于告诉老师"这段是抄来的模板"。

第二处更关键。作业原文写的是"请增加本人撰写的介绍或实践体会，不能仅替换示例中的姓名"，而我最初的 `guide.md` 和 `api.md` 是从"操作手册"的角度写的，读起来像官方文档，完全没有个人痕迹。补的时候我没有硬塞感想，而是写了**只有真做过才会知道的东西**：

- `guide.md` 加了「我踩过的两个坑」：一是 Windows 虚拟环境是 `.venv/Scripts/` 而不是 `.venv/bin/`，干脆不激活直接调解释器；二是 `site_url` 带子路径导致预览时 `/` 返回 302、必须访问 `/my-docs/` 才有页面。这两条都是我在本次实践里真实撞上的，前面第 3 章也记了。
- `api.md` 加了「我写这一页的思路」：为什么按"参数表 → 请求示例 → 返回示例 → 错误码"排列，以及错误码为什么必须给出处理建议而不只是含义。

补完后重新统计第一人称与体会类词汇：

```text
index.md  我=7  体会类=1
about.md  我=1  体会类=1
guide.md  我=7  体会类=5
api.md    我=5
```

`mkdocs build --strict` 仍为 EXIT=0。

- 提交：`777a8f2` 补充两页第一人称内容并替换示例域名
- 运行：#4 https://github.com/xiangsi-max/my-docs/actions/runs/42820717322 → build 与 deploy 均 success

**结果**：线上 `/guide/` 与 `/api/` 均已出现新增内容，`/api/` 里 `example.com` 出现次数为 **0**；四页第一人称出现次数分别为 14 / 10 / 18 / 14（统计的是渲染后 HTML 里的"我"字）。

另外我顺手补了一个细节：仓库首页的 About 面板原本没填站点地址（`homepage` 字段是空的），我用 `gh api -X PATCH` 补上了 `https://xiangsi-max.github.io/my-docs/`，这样从仓库首页就能直接点进网站。

## 3. 问题与解决

### 问题 1：`git push` 卡住不动，没有任何提示

**现象**：执行 `git push origin main` 后终端长时间无输出，也不报错，就是挂着。

**原因**：本机 Git 的凭据助手没有为 `github.com` 配置好，push 时它在后台等待一个永远不会出现的凭据输入。

**解决**：用 `gh auth setup-git` 把 GitHub CLI 注册成 git 的凭据助手，之后 push 直接成功，不再弹窗。关键是这个顺序——**先让 `gh` 登录成功，再让它接管 git 凭据**，而不是手写 token。

### 问题 2：`mkdocs build` 生成了一个多余的临时目录被误提交

**现象**：`git status` 里冒出一个 `mkdocs_al2hgu6m/` 目录，已经进了暂存区。

**原因**：MkDocs 构建时会临时创建目录，构建结束本应清理；某些情况下残留了下来，而 `.gitignore` 当时只写了 `site/`，没覆盖这种名字带随机串的目录。

**解决**：

```bash
git rm -r --cached mkdocs_al2hgu6m
rm -rf mkdocs_al2hgu6m
```

并在 `.gitignore` 里补上 `mkdocs_*/`。教训是：`.gitignore` 要按**模式**写，而不是只写死自己见过的那个目录名。

### 问题 3：`--strict` 到底会拦下什么？我做了故障注入实验

**背景**：本次四次工作流运行全部成功，没有出现"线上构建失败"的情况。作业第 402 行要求"如没有遇到错误，说明你核对过哪些日志和配置"，所以我没有凭空写一个错误，而是**故意把项目改坏，看 `--strict` 会不会拦住**——毕竟它是本地和 CI 之间的唯一保险。

**做了什么**：把仓库复制一份到临时目录，注入两个最常见的错误——把 `nav` 里的 `api.md` 改成不存在的 `apii.md`，把 `about.md` 里指向 `guide.md` 的链接改成 `guidee.md`，然后跑 `mkdocs build --strict`。

**关键日志**（实际运行输出）：

```text
WARNING -  A reference to 'apii.md' is included in the 'nav' configuration,
           which is not found in the documentation files.
WARNING -  Doc file 'about.md' contains a link 'guidee.md', but the target
           is not found among documentation files.
Aborted with 2 warnings in strict mode!
EXIT=1
```

**原因**：`--strict` 的作用是把所有 WARNING 升级为错误。默认模式下这两种问题只会打印警告然后**继续构建成功**，产出一个 `nav` 里少了 API 页、`about` 页有个死链的网站；加了 `--strict` 之后，构建在 `site/` 生成前就中止，退出码为 1。

**解决与结论**：这两种错误**本地和 CI 用的是同一条命令、同一套规则**，所以本地 `EXIT=1` 就等于 CI 一定红。这解释了为什么工作流里 `build` 步骤一旦失败，`deploy` 就完全不会启动——`needs: build` 让部署被直接跳过，线上仍然是上一次成功的版本，不会变成白站。我核对过的日志与配置见下表。

**另外核对过的地方**：

| 核对项 | 结论 |
| --- | --- |
| `requirements.txt` 的版本范围在 CI 的 Python 3.12 上能装 | 本地 3.11 装的是 `mkdocs 1.6.1` + `mkdocs-material 9.8.1`，范围 `>=1.6,<2` / `>=9.6,<10` 留了余量 |
| 工作流 `path: site` 与本地默认输出目录一致 | 本地 `mkdocs build` 默认就输出到 `site/`，两者相同 |
| `upload-pages-artifact@v4` 的 `path` 是目录 | 是目录不是单个文件，传 `site` 而不是 `site/index.html` |
| 工作流 YAML 与作业给定配置是否一致 | 逐行 `diff` 比对，58 行完全一致，`${{ }}` 表达式未替换 |
| 是否存在多余的 gh-pages 分支 | `git ls-remote --heads origin` 只有 `main`，没有 `gh-pages` |

---

## 4. 原理理解

### 4.1 docs/ 和 site/ 各保存什么？为什么不用手工修改 site/？

`docs/` 是**源文件**，是我手写的 Markdown，人可读、可以 diff、可以 review。`site/` 是**构建产物**，是 MkDocs 把 Markdown 加主题模板、加导航结构、加搜索索引之后生成的完整 HTML/CSS/JS 目录，机器生成、内容冗长、每次构建都可能变。

不手工改 `site/` 有三个理由：

1. **会被覆盖**。每次 `mkdocs build` 都从头重新生成，手改的部分下一次构建就没了。
2. **改了也没意义**。`site/` 在 `.gitignore` 里，根本不进仓库，工作流是在 CI 里重新构建一份上传的，本地那份改得再漂亮也传不到线上。
3. **它是单向产物**。Markdown → site 是单向转换，反着从 HTML 改回 Markdown 才是真的工作量大。

一句话：`docs/` 是要维护的资产，`site/` 是可以随时丢弃重算的缓存。

### 4.2 本次工作流在什么情况下自动运行？

三种情况，前两种是自动的：

1. **向 `main` 分支 push，且改动的文件命中 `paths` 列表**——即 `docs/**`、`mkdocs.yml`、`requirements.txt`、`.github/workflows/deploy.yml` 之一。
2. **工作流文件本身被修改并推送**，同样因为它在 `paths` 里。
3. **手动触发**：`workflow_dispatch` 允许在 Actions 页面点 "Run workflow"，这条用于首次重试和排错。

不满足 `paths` 的 push（比如只改 `README.md`）不会触发。这正是后面第 4 问的基础。

### 4.3 build 与 deploy 两个任务有什么区别？

**build 是"生产"，deploy 是"发布"。**

| | build | deploy |
| --- | --- | --- |
| 运行环境 | `ubuntu-latest` | `ubuntu-latest` |
| 权限 | 继承顶层 `contents: read` | 额外要 `pages: write` + `id-token: write` |
| 做什么 | checkout 源码 → 装 Python 3.12 → 装依赖 → `mkdocs build --strict` → 把 `site/` 打包成 artifact 上传 | 下载 artifact → 交给 Pages 服务发布 |
| 产出 | 一个 artifact | 线上可访问的网站 |
| 失败影响 | deploy 根本不会启动 | 网站保持旧版本 |

关键在 `needs: build`：**deploy 必须等 build 成功才开始**。如果 Markdown 里有死链、`nav` 指向不存在的文件，`--strict` 让 build 失败，deploy 就被跳过，线上始终是上一次成功的版本——这就是为什么线上站点不会因为一次错误提交而变成白屏。

权限拆开也是同一个思路：build 只需要读代码，**不该有写 Pages 的能力**；只有 deploy 需要 `pages: write`。这是最小权限原则，即使 build 阶段的依赖被投毒，它也动不了已发布的网站。

### 4.4 只修改 report/README.md 会触发部署吗？为什么？

**不会。**

因为 `on.push.paths` 里只有 `docs/**`、`mkdocs.yml`、`requirements.txt`、`.github/workflows/deploy.yml` 这四项，`report/**` 不在其中。GitHub 在收到 push 后会先比对这次提交改动的文件列表，没有命中就直接不创建工作流运行。

这个设计是合理的：`report/` 只是作业过程记录，不参与网站构建。如果改报告也触发部署，等于每次写报告都要白白跑一遍构建和发布，浪费额度，还会让 Actions 历史里混进一堆"没有改变网站内容"的运行记录。

**这条我实测过**，见 [2.9 反证实验](#29-反证实验只改报告会怎样)：只推 `report/README.md` 的提交 `0870b6b` 推上去 90 秒后，工作流运行数仍然是 3，没有多出第 4 次。

反过来说，如果想让报告也触发，只要在 `paths` 里加上 `report/**` 即可——但没必要。

### 4.5 本次方案为何无需创建 gh-pages 分支？

因为**托管的载体变了**。

传统做法是：CI 构建出 `site/`，然后把这个目录的内容强制推送到一个叫 `gh-pages` 的分支上，再由 Pages 去读那个分支的根目录。这样做需要：一个专门的分支、对仓库的写权限（`contents: write`）、以及一次 `git push`。

这次用的是官方 Pages Actions 方案，链路是：

```text
源码 → build 任务产出 artifact（一个 tar 包）
     → deploy 任务把 artifact 直接交给 Pages 服务
     → Pages 发布（不需要经过任何分支）
```

`upload-pages-artifact@v4` 把 `site/` 打包成 **artifact**（存在 Actions 的运行记录里，不是分支），`deploy-pages@v4` 拿着 `id-token` 换取一个短期 OIDC 令牌，直接调用 Pages 的部署接口。所以：

- 不需要 `gh-pages` 分支，产物压根没进 git；
- 不需要 `contents: write`，因为不往仓库写任何东西；
- 不需要长期 PAT，因为身份认证靠 `id-token: write` 给的**一次性**令牌。

这也顺带回答了为什么 Pages 的 Source 必须设成 "GitHub Actions" 而不是 "Deploy from a branch"——后者对应的正是已经被淘汰的那条分支发布链路。

### 4.6 工作流里每段配置各起什么作用？

作业要求"理解以下配置，并在个人报告中用自己的话解释"，逐条对照我自己的理解：

| 配置 | 我理解的作用 |
| --- | --- |
| `on.push.branches: [main]` | 只有推到 `main` 才算数。开分支做实验不会部署，避免草稿上线。 |
| `on.push.paths` | 文件过滤器。只有 `docs/**`、`mkdocs.yml`、`requirements.txt`、`deploy.yml` 变动才跑，改报告和 README 不跑。 |
| `workflow_dispatch` | 手动开关。第一次配置时如果担心 push 没触发，可以在 Actions 页面点 Run workflow 手动跑一次。 |
| `permissions.contents: read` | 顶层只给读代码的权限，这是最小权限的基线。 |
| `concurrency.group: pages` | 同一时间只允许一个 Pages 部署在跑，避免两次部署互相覆盖。 |
| `cancel-in-progress: false` | 排队而不是取消。连续推两次时，前一次会跑完再跑后一次，保证线上最终是最后一次提交的结果。 |
| `build` 作业的五个步骤 | checkout 取源码 → setup-python 装 3.12 → configure-pages 读出 Pages 配置（比如 `base_url`）→ pip 装依赖 → `mkdocs build --strict` 生成 `site/`。 |
| `upload-pages-artifact@v4` + `path: site` | 把 `site/` 整个目录打包成 artifact，供下一个作业取用。`path` 写的是目录，不是某个文件。 |
| `needs: build` | 部署的前置条件。build 失败则 deploy 不启动，线上保持旧版本。 |
| `deploy` 作业的 `pages: write` | 唯一需要写权限的地方——往 Pages 服务发布内容。 |
| `id-token: write` | 允许 `deploy-pages` 申请 OIDC 令牌。这个令牌是一次性的，用完即失效，所以不需要在仓库里存 PAT。 |
| `environment: github-pages` | 把这次部署和 GitHub 的 `github-pages` 环境绑定，部署地址会记录在仓库的 Environments 页面里。 |
| `url: ${{ steps.deployment.outputs.page_url }}` | 从部署步骤的输出里取真实站点地址，直接显示在运行记录顶部，点一下就能打开。 |

其中 `concurrency` 那条我一开始没看懂为什么要写。后来想明白了：如果两次 push 挨得很近，两个部署同时进行，先完成的那次可能反而覆盖掉后完成的，线上就会变成"旧内容"。设成 `cancel-in-progress: false` 表示排队执行，保证顺序和提交顺序一致。

---

## 5. 提交前自查

按作业第九节的"提交前检查"逐条核对，每条都附上实测证据，不是凭印象打勾。

| 检查项 | 实测证据 | 结论 |
| --- | --- | --- |
| 使用本人个人仓库，不涉及小组/组织/PR | `gh api repos/xiangsi-max/my-docs` → `owner=xiangsi-max`、`visibility=public`；无 fork、无 PR | ✅ |
| 所有示例账号、仓库地址和个人介绍均已替换 | `grep -rn 'example.com' docs/` → **0 处**；`site_url`、`repo_url`、`site_author` 均为本人账号 | ✅ |
| 文件路径及大小写正确，YAML 使用空格缩进 | `nav` 路径相对 `docs/` 且大小写与文件名一致；`mkdocs.yml`、`deploy.yml` 中 TAB 数量为 **0** | ✅ |
| Pages 来源为 GitHub Actions，未混用分支发布设置 | `gh api repos/xiangsi-max/my-docs/pages` → `build_type=workflow`（分支方案是 `legacy`）；`git ls-remote --heads origin` 只有 `main`，无 `gh-pages` | ✅ |
| 四页可访问，第二次推送已在网站生效 | 四页均返回 **200**；首页含"这是我第二次推送后更新的内容" | ✅ |
| 报告截图完整且可见，解释为本人撰写 | `report/images/` 下 8 张 PNG，`raw.githubusercontent.com` 逐个访问全部 **200**；每节均按"做了什么/为什么这样做/结果"撰写 | ✅ |
| 本人已在教师仓库提交一条 Issue，两个核心地址正确 | [#90](https://github.com/hunnuOpenSourceSoftware2026/assignments/issues/90) `OPEN`，标题 `02-github-pages luoweicheng`；报告地址直达 `report/README.md`，效果地址直达网站 | ✅ |

补充的几项（作业未列，但我认为该确认）：

- [x] 默认分支为 `main`，与工作流 `branches: [main]` 一致
- [x] 工作流与作业给定配置逐行 `diff` 一致（58 行），`${{ }}` 表达式未替换成账号
- [x] 四页两两互链，共 12 条（每页 3 条）
- [x] 仅改报告的提交已实测确认不触发部署（见 2.9）
- [x] `site/`、`.venv/` 未提交，仓库 18 个文件中构建产物为 **0 个**
- [x] 仓库 Actions 权限未被改成全局读写：`default_workflow_permissions=read`、`can_approve_pull_request_reviews=false`
- [x] 公开报告与 Issue 中不含密码、令牌等敏感信息
