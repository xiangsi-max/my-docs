# my-docs

个人技术文档网站。使用 Markdown + MkDocs Material 编写，通过 GitHub Actions 自动构建并发布到 GitHub Pages。

## 站点地址

**https://xiangsi-max.github.io/my-docs/**

## 项目简介

这是一个用于练习「静态文档自动化发布」的四页文档站：

| 页面 | 源文件 | 内容 |
| --- | --- | --- |
| 首页 | `docs/index.md` | 站点简介、信息表、快速导航 |
| 关于 | `docs/about.md` | 项目目标、技术组成、目录职责、选型说明 |
| 使用指南 | `docs/guide.md` | 本地预览、严格构建、提交推送、触发规则 |
| API参考 | `docs/api.md` | 接口说明、参数表格、请求/返回示例、错误码 |

每次向 `main` 推送 `docs/**`、`mkdocs.yml`、`requirements.txt` 或工作流文件时，
`.github/workflows/deploy.yml` 会自动执行「安装依赖 → 严格构建 → 上传产物 → 部署到 Pages」。

## 本地预览

```bash
python -m venv .venv
source .venv/Scripts/activate        # Windows Git Bash
# PowerShell 使用：.\.venv\Scripts\Activate.ps1
# macOS / Linux 使用：source .venv/bin/activate

python -m pip install -r requirements.txt
python -m mkdocs serve
```

浏览器打开 <http://127.0.0.1:8000/>。提交前建议先跑一次严格构建：

```bash
python -m mkdocs build --strict
```

## 目录结构

```text
my-docs/
├── .github/workflows/deploy.yml   # Pages 部署工作流
├── docs/                          # 网站正文（四页 Markdown）
├── report/                        # 实践报告与截图
│   ├── README.md
│   └── images/
├── mkdocs.yml                     # 站点配置
├── requirements.txt               # 依赖版本范围
└── README.md
```

`site/` 是构建产物，已在 `.gitignore` 中忽略，不提交到仓库。

## 实践报告

完整的实践过程、关键步骤截图、问题记录与原理理解见：

**[个人实验报告](report/README.md)**
