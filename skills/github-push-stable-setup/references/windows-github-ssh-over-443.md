# Windows GitHub SSH over 443 Pattern

## 适用目标

在 Windows 上，把 GitHub 的 Git 传输从不稳定的 HTTPS 路径切换到更稳定的 SSH over 443。

## 推荐做法

### 1. 使用 SSH 形式的 GitHub remote

推荐 remote 形式：

```text
git@github.com:OWNER/REPO.git
```

### 2. 将 GitHub SSH 流量转到 `ssh.github.com:443`

可选实现方式：

- `~/.ssh/config`
- `core.sshCommand`
- 一个专用包装脚本

目标效果都是一致的：

- GitHub 仍然以 SSH 访问
- 实际走 443 端口

### 3. 在有需要时优先使用 Windows 自带 OpenSSH

如果 Git for Windows 自带的 SSH 不稳定，可改用：

```text
C:\Windows\System32\OpenSSH\ssh.exe
```

### 4. 对 HTTPS remote 做自动改写

如果希望 `https://github.com/...` 也自动走 SSH，可用 Git 全局配置：

```ini
[url "git@github.com:"]
    insteadOf = https://github.com/
    insteadOf = http://github.com/
```

## 验证顺序

```bash
git ls-remote https://github.com/OWNER/REPO.git
git ls-remote git@github.com:OWNER/REPO.git
git push
```

注意：

- `git push --dry-run` 不能代替真实 `git push`
- 最终一定要用一次真实推送验证
