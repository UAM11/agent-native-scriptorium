# GitHub Push Stable Setup Skill

- 创建时间：2026-03-23
- 更新时间：2026-03-23
- 类型：skill
- 状态：verified
- 标签：github, git, ssh, networking, windows
- Model：GPT-5.4
- Hardness：Codex
- 来源：由 `playbooks/github/github-push-stable-setup.md` 提炼而来

## 适用场景

当在 Windows 环境中向 GitHub 执行 `git push` 时，如果 `HTTPS` 路径不稳定，出现如下症状，可使用本 skill：

- `Recv failure: Connection was reset`
- `Couldn't connect to server`
- `git push --dry-run` 正常，但真实 push 失败

## 目标

将 GitHub 的 Git 传输从不稳定的 `HTTPS` 切换到更稳定的 `SSH over 443`，并优先使用 Windows 自带 `OpenSSH`。

## 前置条件

- 已安装 Git
- Windows 系统可用 `C:\Windows\System32\OpenSSH\ssh.exe`
- 本机已有可用于 GitHub 的 SSH key，或允许后续补配

## 执行步骤

1. 先验证问题是否真的出在 `HTTPS push` 链路，而不是仓库权限错误。
2. 检查 `github.com:443` 是否可达，确认不是纯粹的网络中断。
3. 将 GitHub 远程改为 SSH 形式，或通过 Git 全局配置把 `https://github.com/...` 自动改写为 `git@github.com:...`。
4. 通过包装脚本或 `core.sshCommand`，强制 GitHub 连接转到 `ssh.github.com:443`。
5. 强制使用 Windows 自带 `OpenSSH`，避免依赖环境中不稳定的 SSH 实现。
6. 使用 `git ls-remote` 和真实 `git push` 做验证。

## 推荐验证命令

```bash
git config --global --get core.sshCommand
git config --global --get-regexp '^url\\..*\\.insteadOf$'
git ls-remote https://github.com/<owner>/<repo>.git
git ls-remote git@github.com:<owner>/<repo>.git
git push
```

## 成功判据

- `git ls-remote` 能正常返回提交哈希
- `git push` 能真正完成，而不是只在 dry-run 下成功

## 回滚

```bash
git config --global --unset core.sshCommand
git config --global --unset-all url.git@github.com:.insteadOf
```

如果使用了包装脚本，还应手动删除对应脚本文件。

## 与 playbook 的关系

本 skill 是对以下知识条目的执行化封装：

- `playbooks/github/github-push-stable-setup.md`
