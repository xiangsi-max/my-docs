# 关于本项目

## 项目目标

实现从 Markdown 文档到公开网站的自动化发布，做到「推送即上线」。

具体要解决三个问题：

1. **写得方便**——用 Markdown 写，不碰 HTML 和 CSS。
2. **改得放心**——每次改动都进版本库，能看到谁在什么时候改了哪一句。
3. **发得省事**——推送到 `main` 之后，构建、上传、发布全部自动完成。

## 技术组成

| 工具 | 作用 | 在本项目中的具体位置 |
| --- | --- | --- |
| Markdown | 编写文档内容 | `docs/*.md` 四个源文件 |
| MkDocs Material | 将文档构建成静态网站 | `mkdocs.yml` 配置主题与导航 |
| GitHub Actions | 自动执行安装、构建和部署 | `.github/workflows/deploy.yml` |
| GitHub Pages | 托管并提供网站访问地址 | 仓库 Settings → Pages |

## 四个环节怎么串起来

```text
docs/*.md  ──mkdocs build──▶  site/  ──upload-pages-artifact──▶  artifact
                                                                    │
                                                          deploy-pages
                                                                    ▼
                                            https://xiangsi-max.github.io/my-docs/
```

1. **源文件**：`docs/` 下的 Markdown，是人写的、要进 Git 的那一份。
2. **构建产物**：`site/`，是 `mkdocs build` 生成的 HTML/CSS/JS，机器生成，写进 `.gitignore`。
3. **工作流**：`.github/workflows/deploy.yml`，描述「什么情况下、按什么顺序执行哪些步骤」。
4. **托管站点**：GitHub Pages 把构建产物变成可访问的网址。

> 关键区分：**`docs/` 是源，`site/` 是结果。** 改了 `site/` 会被下一次构建覆盖，所以永远不要手改它。

## 目录职责

```text
my-docs/
├── .github/workflows/deploy.yml   # 工作流定义
├── docs/                          # 网站正文（会被构建）
│   ├── index.md
│   ├── about.md
│   ├── guide.md
│   └── api.md
├── report/                        # 实践报告与截图（不参与构建）
│   ├── README.md
│   └── images/
├── mkdocs.yml                     # 站点配置
├── requirements.txt               # 依赖版本范围
└── README.md                      # 项目说明入口
```

注意 `report/` 不在 `docs/` 里，也没有出现在 `mkdocs.yml` 的 `nav` 中，所以它**不会**出现在网站上，
只作为仓库里的作业材料。这也是为什么只改报告不会触发部署工作流（`on.push.paths` 没有包含 `report/**`）。

## 技术选型说明

| 选择 | 替代方案 | 为什么选它 |
| --- | --- | --- |
| MkDocs Material | Docusaurus / VuePress | 纯 Python 依赖，无需 Node 工具链，CI 构建 30 秒内完成 |
| GitHub 官方 Pages Actions | 构建后推送到 `gh-pages` 分支 | 不需要额外分支、不需要写访问令牌，权限最小化 |
| `--strict` 构建 | 默认构建 | 有死链或导航缺页时直接失败，问题在 CI 就暴露 |

## 我的一句话体会

配置这套东西花的时间，大约等于手工发布三四次文档的时间；但它换来的是之后每一次都不用手工发布，
而且线上内容和仓库源码永远对得上。**自动化真正省下的不是操作时间，是不用再记得「上次到底传没传」。**
