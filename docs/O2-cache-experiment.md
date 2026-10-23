# O2：缓存对 CI 反馈时间的影响

实验仓库：[S1mpleWind/sbSE-lab1](https://github.com/S1mpleWind/sbSE-lab1)。执行账号：`wgy2006`。正文表格中的时间为北京时间（UTC+08:00），耗时单位为秒；原始 CSV、JSON 保留 API 的 UTC 时间戳。

## 测量前的预期

记录时间：2026-10-22T10:54:18.468491+08:00，早于两个实验 PR 的首次运行。原始记录见 [pre-experiment.txt](o2-cache/pre-experiment.txt)，两个 PR 的初始说明也保留了这段预期。

示例没有第三方依赖，预计模块下载方面的收益很小；缓存仍可能通过复用编译输出缩短 vet 和测试的准备时间。缓存恢复也有开销，整个 ci job 是否更快、节省多少秒，以两组各至少五次的实测结果为准。

## 对照方法

两个临时 PR 均从提交 `989ebe3e77af41d810261a71ec4df0877ffc28e7` 创建：[PR #6](https://github.com/S1mpleWind/sbSE-lab1/pull/6) 显式设置 `cache: true`，[PR #7](https://github.com/S1mpleWind/sbSE-lab1/pull/7) 设置 `cache: false`。两组业务代码相同，工作流只相差这个开关。两个实验 PR 都不合并。

两组均使用 `pull_request` 事件，不执行 main 专属的打包和上传步骤。按“有缓存、无缓存”交替运行，每组首次运行后执行四次 Re-run all jobs，共十次测量。测量期间没有更改分支提交或依赖文件，十次 CI 均成功。

主指标为 `ci` job 的 `completed_at - started_at`，不包含排队等待。另记录 setup-go、vet、test 和 post setup-go 的步骤时长。数据来自 Actions jobs API，时间戳精度为整秒；表中 0 秒表示开始、结束时间落在同一秒或不足以由该口径分辨，不能理解为没有执行工作。job 总时长还包含 checkout、格式检查和收尾等步骤，不能只用表内四项相加代替。

固定配置为 `ubuntu-24.04`、Go `1.26.9`，十次 setup-go 解析到的提交均为 `b7ad1dad31e06c5925ef5d2fc7ad053ef454303e`。托管 runner 实际分配的镜像版本分布如下。

| runner 镜像版本 | 有缓存次数 | 无缓存次数 |
|---|---:|---:|
| `20261012.334.1` | 3 | 1 |
| `20261018.339.1` | 2 | 4 |

有缓存组五次均实际命中缓存；无缓存组确认 `cache: false` 且没有 setup-go 模块/编译缓存恢复记录。两组都可以使用 runner 的 Go 工具链缓存，这与实验开关控制的模块和编译输出缓存不同。

## 全部运行数据

同一 workflow run 的重跑保留运行编号和 run ID，因此同时记录 attempt，链接指向对应尝试。

| 顺序 | 组别 | 运行 | job 开始 | job 总时长 | setup-go | vet | test | post setup-go | 缓存 |
|---|---|---|---|---:|---:|---:|---:|---:|---|
| 1 | 有缓存 | [ci #15 / attempt 1](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714012994/attempts/1) | 10-22 10:55:25 | 14 | 2 | 5 | 2 | 1 | 命中 |
| 2 | 无缓存 | [ci #16 / attempt 1](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714097159/attempts/1) | 10-22 10:56:20 | 29 | 1 | 19 | 4 | 0 | 关闭 |
| 3 | 有缓存 | [ci #15 / attempt 2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714012994/attempts/2) | 10-22 10:58:06 | 12 | 1 | 3 | 3 | 1 | 命中 |
| 4 | 无缓存 | [ci #16 / attempt 2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714097159/attempts/2) | 10-22 10:58:36 | 32 | 1 | 21 | 3 | 0 | 关闭 |
| 5 | 有缓存 | [ci #15 / attempt 3](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714012994/attempts/3) | 10-22 10:59:33 | 15 | 2 | 4 | 3 | 0 | 命中 |
| 6 | 无缓存 | [ci #16 / attempt 3](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714097159/attempts/3) | 10-22 11:00:02 | 19 | 1 | 10 | 3 | 0 | 关闭 |
| 7 | 有缓存 | [ci #15 / attempt 4](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714012994/attempts/4) | 10-22 11:00:53 | 15 | 2 | 5 | 4 | 0 | 命中 |
| 8 | 无缓存 | [ci #16 / attempt 4](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714097159/attempts/4) | 10-22 11:01:29 | 23 | 0 | 13 | 3 | 0 | 关闭 |
| 9 | 有缓存 | [ci #15 / attempt 5](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714012994/attempts/5) | 10-22 11:02:16 | 13 | 1 | 5 | 3 | 0 | 命中 |
| 10 | 无缓存 | [ci #16 / attempt 5](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714097159/attempts/5) | 10-22 11:02:42 | 21 | 0 | 12 | 2 | 0 | 关闭 |

原始汇总见 [results.csv](o2-cache/results.csv) 和 [records.json](o2-cache/records.json)。CSV 包含每次 job 的开始、结束时间、提交号、run ID、attempt、缓存状态和原始链接。

## 结果

| 组别 | 次数 | job 平均值 | 中位数 | 范围 | 样本标准差 |
|---|---:|---:|---:|---|---:|
| 有缓存 | 5 | 13.8 | 14.0 | 12 至 15 | 1.30 |
| 无缓存 | 5 | 24.8 | 23.0 | 19 至 32 | 5.50 |

本次样本中，有缓存组的 job 平均用时比无缓存组少 11.0 秒，按无缓存组平均值计算缩短约 44.4%。计算式为 `(无缓存均值 - 有缓存均值) / 无缓存均值`。

各步骤的平均用时如下：

| 步骤 | 有缓存 | 无缓存 |
|---|---:|---:|
| setup-go | 1.6 | 0.6 |
| go vet | 4.4 | 15.0 |
| go test | 3.0 | 3.0 |
| post setup-go | 0.4 | 0.0 |

示例虽然没有第三方依赖，缓存仍能复用 Go 的编译输出。`go vet` 位于测试之前，也需要加载、分析和准备包；本次观测中 vet 的平均用时从 15.0 秒降至 4.4 秒，是 job 时长差异的主要部分。这个结果与复用编译输出的作用一致，不能把收益解释成减少了第三方依赖下载。Makefile 使用 `go test -count=1`，测试仍会执行，不是直接复用上一次测试的通过结果。

缓存恢复和收尾也有开销，已计入 setup-go、post setup-go 以及 job 总时长。因此上面的均值比较包含这部分成本。

## 波动与局限

每组只有五次，样本量较小。GitHub 托管 runner 的负载、缓存传输和解压速度、网络以及机器调度都会引入波动。两组交替测量，Go 版本和 setup-go 提交相同；但实际 runner 镜像版本存在分配差异，也没有固定到同一台物理机器。因此，环境并非完全受控，不能把两组均值的全部差额都精确归因于缓存。

有缓存组五次均命中已存在的缓存，因此这里比较的是热缓存与关闭缓存，不代表首次创建缓存时的收益。关闭缓存仅指 setup-go 的模块和编译输出缓存，不代表清空 runner 上的一切预装工具或底层缓存。主指标没有包含排队等待，也没有包含 main 打包和上传制品的耗时，因此结论只适用于本次 PR 检查的 job 反馈时间。

本次样本中，热缓存组的 PR 检查平均更短，vet 步骤的变化与编译输出复用的作用一致。这是在上述 runner 波动条件下得到的观察，不能当作缓存必然节省固定秒数的保证。其他语言、依赖规模或 runner 环境需要重新测量，不能直接套用这里的比例。

## 配置依据与证据

- [setup-go v7 缓存说明](https://github.com/actions/setup-go/blob/v7/README.md#caching-dependency-files-and-build-outputs)：缓存默认开启，覆盖 Go 模块与编译输出；`cache: false` 关闭这部分缓存。
- [有缓存实验 PR #6](https://github.com/S1mpleWind/sbSE-lab1/pull/6)、[无缓存对照 PR #7](https://github.com/S1mpleWind/sbSE-lab1/pull/7)：保留配置和全部 Actions 尝试，不合并、不删除证据。
- [两组配置差异](o2-cache/cache-only-difference.patch)：只相差 `cache: true/false`。
- 实验记录及分析通过单独的文档 PR 提交，不会把关闭缓存的实验配置带入 main。
