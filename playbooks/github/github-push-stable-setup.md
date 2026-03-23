# GitHub Push 稳定方案

- 创建时间：2026-03-23
- 更新时间：2026-03-23
- 类型：playbook
- 状态：verified
- 标签：github, git, ssh, networking, windows
- 来源：一次在 Windows 环境下真实排障并验证成功的修复过程

## 背景

这台机器上，向 GitHub 执行 `git push` 时，如果走 `HTTPS`，会出现间歇性失败。GitHub 本身可达，账号权限也正常，但真正上传时会断开。

典型报错：

```text
fatal: unable to access 'https://github.com/...': Recv failure: Connection was reset
```

## 诊断结论

这个问题不是远程地址写错，也不是仓库权限不足。

已确认的事实：

- `github.com:443` 可以连通
- `git ls-remote` 正常
- `git push --dry-run` 正常
- 真正的 `git push` 在上传阶段会被重置连接

因此，更接近“`HTTPS` Git 传输链路不稳定”，而不是“GitHub 账号配置有问题”。

## 提炼后的修复方案

把 GitHub 的 Git 传输切到 `SSH over 443`，并强制使用 Windows 自带的 `OpenSSH`，不要依赖 Git for Windows 自带的 SSH。

这样做有效的原因：

- `443` 端口通常比标准 SSH 的 `22` 更容易穿过网络限制
- `SSH over 443` 可以绕开不稳定的 `HTTPS push` 链路
- 在这个环境中，Windows 自带的 `OpenSSH` 表现更稳定

## 已实施的全局配置

### 全局 Git 配置

写入位置：`C:\Users\UAMC\.gitconfig`

```ini
[core]
    sshCommand = C:/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -NoProfile -ExecutionPolicy Bypass -File C:/Users/UAMC/git-tools/git-ssh-github-443.ps1

[url "git@github.com:"]
    insteadOf = https://github.com/
    insteadOf = http://github.com/
```

作用：

- 所有 `https://github.com/...` 形式的 GitHub 远程地址都会自动改写成 `git@github.com:...`
- 所有 Git SSH 调用都会走 GitHub 专用包装脚本

### SSH 包装脚本

脚本路径：

```text
C:\Users\UAMC\git-tools\git-ssh-github-443.ps1
```

脚本行为：

- 如果目标主机是 `github.com` 或 `ssh.github.com`
- 自动把连接改到 `git@ssh.github.com`
- 自动指定端口 `443`
- 自动附加 `StrictHostKeyChecking=accept-new`
- 最终调用 `C:\Windows\System32\OpenSSH\ssh.exe`

对于非 GitHub 的 SSH 目标，脚本会保持原样透传，不强行改端口。

## 验证命令

```bash
git config --global --get core.sshCommand
git config --global --get-regexp '^url\\..*\\.insteadOf$'
git ls-remote https://github.com/UAM11/Master-Thesis.git
git ls-remote git@github.com:UAM11/Master-Thesis.git
```

如果最后两条命令都能返回提交哈希，说明 GitHub 的 SSH over 443 路径已经生效。

## 已验证结果

目标仓库在切换到该方案后，成功完成推送：

```text
To ssh.github.com:UAM11/Master-Thesis.git
   2d24ace..3bd4bff  main -> main
```

## 回滚方法

如果未来不再需要这套全局配置，可以执行：

```bash
git config --global --unset core.sshCommand
git config --global --unset-all url.git@github.com:.insteadOf
```

如果还要彻底移除包装脚本，再手动删除：

```text
C:\Users\UAMC\git-tools\git-ssh-github-443.ps1
```

## 注意事项

- 这套方案是针对 GitHub 定制的，不适用于所有 SSH 主机。
- 网络环境如果变化了，这套方案依然可以继续保留，不影响正常使用。
- 如果本机 SSH key 变了，仍然需要保证 GitHub 侧的 SSH 认证是有效的。
