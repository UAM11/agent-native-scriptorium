# GitHub Push Diagnosis Checklist

## Goal

在修改 Git 传输路径之前，先确认问题到底属于哪一类。

## 优先排查的三类问题

### 1. 基础连通性问题

常见表现：

- `github.com:443` 无法访问
- 浏览器也无法打开 GitHub
- DNS 解析异常

如果属于这一类，不要急着改 Git 配置，先处理网络问题本身。

### 2. 权限或认证问题

常见表现：

- `403`
- `404`
- repository not found
- permission denied
- SSH key 未配置或未被 GitHub 接受

如果属于这一类，不要把它误判成 HTTPS 传输不稳定。

### 3. HTTPS 上传链路不稳定

这是本 skill 主要处理的场景。

常见表现：

- `git ls-remote` 正常
- `git push --dry-run` 正常
- 真正 `git push` 失败
- 失败信息包含 reset、timeout、couldn't connect、recv failure

## 建议验证顺序

1. 确认 remote 确实是 GitHub
2. 确认仓库地址正确
3. 确认账号确实有权限
4. 确认 `github.com:443` 可达
5. 再判断是否要切换到 SSH over 443
