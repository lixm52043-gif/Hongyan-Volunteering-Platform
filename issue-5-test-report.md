# Issue #5 时长导入复审测试报告

2026 年 11 月 20 日在 Windows、Chrome 和独立 PostgreSQL 测试库上复验。业务与测试版本为 `7e8c223`，基于团队主分支 `1ac5067`。测试账号、学生、活动和工作簿数据均为合成数据。

## 执行记录

以下命令在仓库根目录执行，均通过。测试命令在 `apps/api` 执行。

| 范围 | 实际命令 | 结果 |
| --- | --- | --- |
| 契约 | `corepack pnpm --filter @hongyan/contracts generate` | 客户端生成成功 |
| 数据模型 | `corepack pnpm --filter @hongyan/api exec prisma validate`、`corepack pnpm --filter @hongyan/api db:generate` | 校验与客户端生成通过 |
| 应用构建 | `corepack pnpm -r --if-present typecheck`、`corepack pnpm -r --if-present build` | 类型检查与构建通过 |
| 账号、工作簿、预览规则 | `node --env-file=.env --test test/demo-accounts.test.mjs test/hour-workbook.test.mjs test/hour-preview.test.mjs` | 25/25 通过 |
| HTTP 与 PostgreSQL | `node --env-file=.env --test test/activity-flow.test.mjs` | 13/13 通过 |
| 三端页面 | `node --env-file=.env --test test/pages.test.mjs`，`HY_BROWSER=chrome` | 22/22 通过 |

自动测试共 60 项通过，跳过 0 项。服务端回归单独验证重复预览的明细、全批合计、历史安排及正式时长一致性。Chrome 回归从负责人页面上传工作簿、筛选及翻页、下载报告、修正重传和重试原批次。

## 四项修复

| 场景 | 验收结果 |
| --- | --- |
| 计划 4 小时，同一已报名学生填两条 5 小时记录，第二行普通备注为空 | 首行保留超计划警告和说明，第二行跳过。有效 1、跳过 1、错误 0，预计 +5.0 小时。单独超时缺说明和同键冲突用例均通过。 |
| H202、H804 含未知列，E804 为 1.25 | 上传后批次为 `invalid`。同一份下载报告列出两处未知列和原表第 804 行的认证时长精度错误。按行计为两条错误，第 805 行仍可预览，正式时长记录一致。 |
| Excel 合计行开关 | 当前合计行数为 0 且历史显示标记为 1 时可导入；当前合计行数为 1 时拒绝。 |
| 同一张有效表另存为 Strict 和 Transitional | 已在本机 Excel 16.0 实际另存，并将两个文件送入解析和业务预览。均保留十位学号，服务日期为 2026-11-18，预计 +5.0 小时，原表第 2 行的超计划警告和说明一致。 |

## 随附样本

`samples/` 包含 7 份 Excel 和 2 张截图。合成混合工作簿用于查看四类行及 +9.0 小时的全批差额，修正工作簿预览为 +10.2 小时。其余四份输入用于复验行序、冲突中的违纪上报和多处未知列，另附一份导出的错误报告。桌面和手机截图分别展示原表错误行、违纪与时长警告。

11 月 19 日提交人确认四份手动复验样本通过。11 月 20 日运行了上述自动回归及两种 Excel 文件对照。此次浏览器回归使用 Chrome；未另跑 Safari、Firefox，四项修改集中于服务端解析、预览及测试数据准备。正式送审、批准、累计入账属于后续需求。
