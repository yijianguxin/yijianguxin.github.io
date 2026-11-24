---
title: "从零搭建 GitHub Pages 个人博客，从账号授权到发布第一篇笔记"
categories: [建站笔记]
tags: [GitHub Pages, Git, GitHub Actions]
---

我想给 Unity 学习、游戏开发和随笔找一个固定的地方。比如，跟着 Catlike Coding 做完一次练习，我可以记下当时遇到的问题、试过的几种办法，以及最后是怎么验证的。过几个月再遇到类似问题，还能通过搜索或标签翻出当时的笔记。

这次我用 **GitHub Pages、Jekyll 和 Chirpy** 搭建了「一剑孤心」。AI 帮我完成了不少设置，真正动手时，我发现最容易混在一起的是三件事：**本地 Git 怎么连接 GitHub，GitHub 怎么把 Markdown 变成网页，以及网页什么时候才算发布成功。** 这篇就按这个顺序拆开。

## 一、为什么用 GitHub Pages？

我想要一个公开的个人站点，也想把文章保存在自己电脑上。用这套方式，我可以在本地写 Markdown，用 Git 查看修改和管理历史，然后把变更推到 GitHub。GitHub Free 支持从公开仓库托管 Pages 站点，个人站点自带 `用户名.github.io` 网址，之后也可以绑定自己的域名。 [GitHub Pages 快速入门](https://docs.github.com/en/pages/quickstart)

每个工具负责的事可以这样理解：

| 工具 | 在这个博客里做什么 |
| --- | --- |
| Git | 在电脑上管理文件和提交历史，把提交推到远程仓库 |
| GitHub 仓库 | 保存 Markdown 原稿、图片、主题配置和部署配置 |
| Jekyll | 读取 Markdown 和模板，生成浏览器能打开的网页 |
| Chirpy | 提供博客排版、简体中文界面、标签、归档和搜索 |
| GitHub Actions | 在云端运行 Jekyll、检查生成结果并部署 |
| GitHub Pages | 提供站点网址，让读者访问构建好的网页 |

这种静态网站适合公开笔记、文章、作品集和项目文档。如果想做一个登录后才能看、还能在网页里写私密笔记的应用，登录和存储需要另外实现。我把可以公开的文章放进博客仓库，私人草稿留在仓库外。 [什么是 GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/what-is-github-pages)

## 二、在 GitHub 上创建博客仓库

如果只是想试试 GitHub Pages，可以建一个叫 `你的用户名.github.io` 的公开仓库，勾选 **Add README**，再到 **Settings → Pages** 选择 **Deploy from a branch**，发布 `main` 分支的 `/ (root)`。这条路线足够验证网站能打开。 [GitHub Pages 快速入门](https://docs.github.com/en/pages/quickstart)

如果一开始就想要我现在用的博客样式，可以直接走 **Chirpy Starter** 这条路线：

1. 登录 GitHub，打开 Chirpy Starter，点击 **Use this template → Create a new repository**。
2. **Owner** 选择自己的个人账号，仓库名填写 `你的用户名.github.io`，可见性选择 **Public**。
3. 创建后，在仓库的 **Code** 页确认能看到 `_config.yml` 和 `.github/workflows/`。前者保存站点配置，后者保存自动构建和部署的流程。
4. 到 **Settings → Pages → Build and deployment**，把 **Source** 设为 **GitHub Actions**。Chirpy Starter 已经带了部署流程，可以继续用模板里的工作流。 [Chirpy Starter](https://github.com/cotes2020/chirpy-starter) · [Chirpy 部署说明](https://chirpy.cotes.page/posts/getting-started/#deploy-using-github-actions)

我原来先试了分支发布，换成 Chirpy 时才切到 GitHub Actions。如果已经有同名的个人站点仓库，可以把模板里的文件整合到原仓库，再切换部署来源。

## 三、本地 Git 怎么和 GitHub 账号关联？

这里有三个东西，分别解决不同的问题：

| 设置 | 它回答的问题 |
| --- | --- |
| `origin` 远程地址 | 我要把提交推到哪个仓库？ |
| `user.name`、`user.email` | 这次提交记录显示是谁写的？ |
| GitHub 登录凭据 | 当前操作的人有没有权限往这个仓库推送？ |

浏览器里已经登录 GitHub，说明你可以访问网页登录后的界面。第一次在 PowerShell 里执行 `git push` 时，Git 仍可能需要单独完成一次授权。

### 1. 克隆仓库，检查远程地址

在 PowerShell 中运行以下命令，把示例用户名 `octocat` 换成自己的 GitHub 用户名。Windows 上可以先运行 `git --version`，确认已安装 Git。

```powershell
git clone https://github.com/octocat/octocat.github.io.git
Set-Location .\octocat.github.io
git remote -v
git status --short --branch
```

`git clone` 会把仓库和提交历史下载到当前目录，并自动配置名为 `origin` 的远程地址。`git remote -v` 会显示拉取和推送对应的网址。我的仓库地址是 `https://github.com/yijianguxin/yijianguxin.github.io.git`，网站地址则是 `https://yijianguxin.github.io/`。

### 2. 设置提交署名

如果第一次提交时 Git 提示 `Author identity unknown`，可以在博客仓库里设置署名：

```powershell
git config --local user.name "你的显示名"
git config --local user.email "你的 GitHub 提交邮箱"
```

`--local` 只作用于当前仓库。公开博客可以在 GitHub 的 **Settings → Emails** 找到自己的 `noreply` 邮箱，把它填进第二条命令。这样，公开提交里的作者邮箱可以使用 GitHub 提供的地址。署名设置完成后，推送权限仍由下面的授权步骤决定。 [GitHub 提交邮箱说明](https://docs.github.com/en/account-and-profile/how-tos/email-preferences/setting-your-commit-email-address)

### 3. 用 HTTPS 完成推送授权

我这个站点用 **HTTPS + Git Credential Manager**。第一次 `git push` 如果弹出登录窗口，按提示登录拥有该仓库写入权限的账号，并查看授权页列出的权限。成功后，Git Credential Manager 会保存凭据，后续推送会复用有效凭据。使用这个 HTTPS 远程地址，可以直接走这套流程。 [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager)

我实际遇到过一次卡点。浏览器里点了同意，Edge 跳回本机地址时却显示 `ERR_UNSAFE_PORT`，导致 Git 并没有收到授权结果。后来改用设备登录：

```powershell
git credential-manager github login --device
```

运行后，Git Credential Manager 会给出临时设备码。打开 [GitHub 设备激活页](https://github.com/login/device)，输入刚才窗口里的设备码，核对账号和权限并完成授权。等命令结束后，再回到仓库执行 `git push`。设备登录避开了那次出问题的浏览器本地回调。 [Git Credential Manager 命令说明](https://github.com/git-ecosystem/git-credential-manager/blob/main/docs/usage.md) · [GitHub 认证模式](https://github.com/git-ecosystem/git-credential-manager/blob/main/docs/configuration.md#credentialgithubauthmodes)

## 四、GitHub Actions 到底是怎么部署的？

我这个博客的部署文件在 `.github/workflows/pages-deploy.yml`。这个文件是仓库里的一份 YAML 配置，它告诉 GitHub **什么时候运行，以及运行哪些步骤**。

以我现在这个站点为例，推送包含文章变更的提交到 `main` 后，会触发名为 **Build and Deploy** 的工作流：

| 步骤 | 发生了什么 |
| --- | --- |
| **Checkout** | GitHub 的构建机器取出本次提交里的文件 |
| **Setup Pages / Setup Ruby** | 读取站点部署参数，准备 Ruby 和所需依赖 |
| **Build site** | Jekyll 读取 `_posts`、`_config.yml` 和主题，把 Markdown 转成 HTML，输出到 `_site` |
| **Test site** | 检查生成的页面和站内链接 |
| **Upload site artifact** | 把构建产物打包，交给后续部署任务 |
| **Deploy to GitHub Pages** | 将这份构建产物发布到 Pages 网址 |

整条流程是在 GitHub 的构建机器上跑的。平时写文章，我的 Windows 电脑上有编辑器和 Git 就能提交。想在自己电脑上提前预览网页时，再配置本地 Jekyll 环境。工作流部署时用的是 GitHub 为这次运行提供的权限，个人的 GitHub 密码和本地推送凭据都不用写进 YAML。 [GitHub Pages 自定义工作流](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages) · [Chirpy 本地预览说明](https://chirpy.cotes.page/posts/getting-started/#start-the-jekyll-server)

**怎么确认发布成功？** 在 GitHub 仓库打开 **Actions → Build and Deploy**，点进对应本次提交的运行记录。等部署任务显示成功，再访问博客首页和文章链接。我的站点第一次改头像时，部署已经成功，Edge 却因为离线缓存还显示旧版。碰到这种情况，可以先试一次 `Ctrl + Shift + R` 强制刷新，同时用 Actions 的记录确认部署的是哪次提交。

## 五、怎样写、提交和发布一篇笔记？

在本地仓库的 `_posts` 目录新建文件，按 `年-月-日-英文短名.md` 命名。例如 `_posts/2026-11-24-unity-study-notes.md`。开头两条 `---` 之间是文章配置，下面就是正文。 [Chirpy 写作指南](https://chirpy.cotes.page/posts/write-a-new-post/)

```markdown
---
title: "我的第一篇 Unity 学习笔记"
categories: [游戏开发]
tags: [Unity, Catlike Coding]
---

## 这次想验证什么？

写下练习目标，以及开始前预期会看到的结果。

## 我做了什么？

记录关键操作、代码变化和复现条件。

## 结果如何？

写下实际观察到的现象，以及还有哪些问题没解决。
```

我会用「游戏开发」这样比较宽的词做分类，用「Unity」「Catlike Coding」「调试」这样具体的词做标签。同一篇笔记可以有多个标签，以后找起来方便。

写完后，在仓库目录执行：

```powershell
git status --short --branch
git add -- _posts/2026-11-24-unity-study-notes.md
git diff --cached
git commit -m "添加第一篇 Unity 学习笔记"
git push
```

`git add` 指定这一篇文章，避免顺手把目录里其他草稿一起提交。`git diff --cached` 可以在提交前确认具体会公开什么。`git commit` 把变更记录在本地，`git push` 才会把这次提交送到 GitHub。随后等 Actions 部署，打开文章检查标题、正文和标签。

如果某一步没走通，可以按看到的现象定位：

| 现象 | 先检查哪里 |
| --- | --- |
| 提交时报作者身份错误 | 检查当前仓库的 `user.name` 和 `user.email` |
| `git push` 提示权限不足 | 用 `git remote -v` 核对地址，再确认授权的是有写入权限的账号 |
| GitHub 仓库里有新文章，Actions 没有新任务 | 检查推送的分支、工作流触发条件和 Pages 的 Source 设置 |
| Actions 有红叉 | 点开失败步骤，查看是构建、检查链接还是部署报错 |
| 部署成功，首页还是旧版 | 核对部署对应的提交，再强制刷新浏览器 |

第一次搭站需要理解这些环节。之后写笔记，我每天面对的主要还是一个 Markdown 文件。想公开时，提交、推送，等部署成功，再打开文章确认一次。下次回头看一段代码或一个调试问题，也能顺着原来的记录找回当时的思路。
