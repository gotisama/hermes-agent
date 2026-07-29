# CLAUDE.md

本仓库是 **NousResearch/hermes-agent 的本地 fork**，用途：自己改代码（主要是飞书/钉钉
平台适配器）+ 持续跟进上游更新。上游动得很快（每天都有合并），分支纪律必须守住。

## 分支约定

| 分支 | 用途 | 纪律 |
|---|---|---|
| `main` | **纯上游镜像** | 永远不在这里提交自己的改动，只 `git fetch origin && git merge --ff-only origin/main` |
| `feature/feishu-custom` | **自己的补丁集** | 所有本地修改都提交在这里，基于某个上游提交 |

## 跟上游的节奏

```bash
git fetch origin
git checkout main && git merge --ff-only origin/main   # 刷新镜像
git checkout feature/feishu-custom
git rebase origin/main                                  # 把补丁搬到新基底
```

用 **rebase 不用 merge**：补丁集永远是"上游 + 干净的几个自己提交"，
冲突只在被上游改到的文件上出现，好解；也方便日后给上游提 PR。
rebase 后老提交号作废属正常，本仓库没有共享分支，随便 force。

## 部署（改完怎么生效）

运行中的 hermes 在 `%LOCALAPPDATA%\hermes\hermes-agent`（独立的浅克隆安装，
**不是本仓库**）。本仓库的改动要拷过去才生效：

```bash
cp plugins/platforms/feishu/adapter.py \
   "$LOCALAPPDATA/hermes/hermes-agent/plugins/platforms/feishu/adapter.py"
# 重启网关（必须用这个脚本，裸启动会解析到系统 Python）：
%LOCALAPPDATA%\hermes\start-gateway.cmd
```

- 插件是纯 Python，拷完重启即生效，无需编译。
- **`hermes update` 会冲掉安装目录里的手工改动**——本仓库的 commit 才是真身，
  安装目录只是部署目标。升级后：先在这里 rebase 到新上游，再重新拷贝。
- 改动涉及多个文件时逐个拷，或 `git diff --name-only origin/main | grep -v CLAUDE` 列清单。

## 当前基底

fork 建立时（2026-07-29）基点 `d71033a`，与当时运行中的安装同版本。
rebase 之后以 `git merge-base HEAD origin/main` 为准。

## 已知的值得改的点

- **钉钉适配器不支持 file 消息**：`plugins/platforms/dingtalk/adapter.py` 的
  `_extract_media()` 只读 `image_content`/`rich_text_content`，msgtype=file 的
  downloadCode 落在 `ChatbotMessage.extensions['content']` 无人读，消息被
  "Empty message, skipping" 静默丢弃。修法：加分支取 downloadCode 接进现成的
  `_fetch_download_url()` 链路。修好可给上游提 PR。
- 飞书侧无需改：file 消息原生支持；群内免 @ 靠的是开放平台
  「获取群组中所有消息」权限（已开）+ config.yaml 的 `group_rules`，不是代码问题。

## 相关环境

- 运行配置/密钥在 `%LOCALAPPDATA%\hermes\`（config.yaml / .env），不在本仓库，勿提交。
- 本机 hermes 的完整使用说明：`D:\project\ccProject\harnessAgent\HERMES.md`。
