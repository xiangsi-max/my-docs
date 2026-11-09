# 我的技术文档

欢迎访问我的个人文档网站。本网站使用 MkDocs 构建，通过 GitHub Actions 自动部署到 GitHub Pages。

!!! info "站点简介"
    这是一个用于练习「静态文档自动化发布」的站点。所有页面内容都写在 `docs/` 目录的 Markdown 文件里，
    推送代码后由 GitHub Actions 自动构建成静态网站并发布，不需要手工上传任何 HTML 文件。

## 本站信息

| 项目 | 内容 |
| --- | --- |
| 作者 | xiangsi-max |
| 实践主题 | 静态文档自动部署 |
| 构建工具 | MkDocs 1.6 + Material 主题 |
| 部署方式 | GitHub Actions + GitHub Pages |
| 文档页数 | 4 页 |

## 快速导航

- [关于本项目](about.md)——说明项目目标、技术组成与目录职责。
- [使用指南](guide.md)——从本地预览到线上更新的完整流程。
- [API 参考](api.md)——用 Markdown 描述接口文档的写法示例。

## 为什么做这个站

我以前改文档的流程是「本地写完 → 导出 HTML → 拖到服务器」，每次都要重复一遍，而且线上文件和源码很容易对不上。
这次我把文档当成代码来管理：Markdown 就是源文件，构建和发布交给流水线，我要做的只剩下一件事——`git push`。

下面这段命令就是我日常更新文档的全部操作：

```bash
# 1. 修改 docs/ 里的 Markdown
git add docs/index.md
git commit -m "更新首页内容"
# 2. 推送后 Actions 自动构建并部署
git push origin main
```

## 站点结构

```text
my-docs/
├── docs/            # 网站正文，四页 Markdown
├── report/          # 实践报告与截图
├── mkdocs.yml       # 站点配置：站点名、导航、主题
└── requirements.txt # 构建依赖版本范围
```

> 本页的一句话实践体会：**把重复的手工发布变成一次 push，是把「写文档」坚持下来的关键。**

## 自动部署验证

自动部署验证：这是我第二次推送后更新的内容。
