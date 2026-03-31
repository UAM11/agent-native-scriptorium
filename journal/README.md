# journal

这个目录是当前仓库的过程层入口。

它不负责保存所有长期资产，而是负责记录真实推进：

- 今天做了什么
- 这周推进了什么
- 卡在哪里
- 哪些内容值得晋升为长期资产

由于当前默认维护的是公开仓，这里的日志应默认保持 public-safe。

## 结构

- `daily/`：按天记录
- `weekly/`：按周汇总

## 命名建议

- `journal/daily/YYYY-MM-DD.md`
- `journal/weekly/YYYY-Www.md`

## 使用方式

1. 先记录当天真实 session
2. 再在 `Promotions` 中标出哪些内容值得晋升
3. 周末在 `weekly/` 汇总

## 相关模板

- `templates/daily-journal-template.md`
- `templates/weekly-review-template.md`
