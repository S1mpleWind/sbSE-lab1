# 实验 1 记录：第 <组号> 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 <组号> 组 |
| 仓库地址 | <https://github.com/…> |
| 代码载体 | 统一示例 / 自己的项目（<项目名与语言>） |
| runner 与语言版本 | `ubuntu-24.04`，<Go 或其他语言的版本> |
| 合并方式 | <squash / merge commit / rebase> |

## 2. 任务 0：本地基线

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | <填写> |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | <填写> |
| 两次 `go version -m` 输出中的 vcs 行有何不同？ | <填写> |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | <填写> |

## 3. 证据

| 项 | 链接或文本 |
|---|---|
| 规则集里启用的规则 | <列出规则名、审批人数和必需检查的名称> |
| 组长直接 `git push origin main` 被拒的终端输出 | <粘贴文本> |
| 第一轮：失败的 PR 运行 | <链接> |
| 第一轮：修复后通过的 PR 运行 | <链接> |
| 第一轮：合入 main 后的运行（含制品） | <链接> |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | <一两句话> |
| 第二轮：仅含缺陷时仍然通过的运行 | <链接> |
| 第二轮：补测试后失败的运行 | <链接> |
| 第二轮：撤销缺陷后通过的运行 | <链接> |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | [release #2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39231936952)：`v0.0.1` 指向未合入 main 的空提交 `83b5ab7`，在 `Check that the tag points to main` 步骤失败，后续构建和发布步骤均跳过，未产生 Release。 |
| Release v1.0.0 | [v1.0.0](https://github.com/S1mpleWind/sbSE-lab1/releases/tag/v1.0.0)，发布于 2026-10-19 17:41:16（UTC+8） |
| Release v1.0.1 | [v1.0.1](https://github.com/S1mpleWind/sbSE-lab1/releases/tag/v1.0.1)，发布于 2026-10-21 10:01:34（UTC+8） |

## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | `1.0.0` | `1.0.1` |
| 构建号 | `1`（release 工作流） | `3`（release 工作流；第 2 次运行是门禁拒绝演练） |
| 提交号（SHA） | `a845dd34214418516d01f77fbd7796498a7866da` | `52c647c0cef046d70c144a6d0c63e81461daddb7` |
| 引入该提交的 PR | [#3](https://github.com/S1mpleWind/sbSE-lab1/pull/3) | [#4](https://github.com/S1mpleWind/sbSE-lab1/pull/4)，由 `yollsplus` 发起，`wgy2006` 审批后 squash 合入 |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | [release #1](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39231899813) | [release #3](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39530639353) |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | `aeb05ee8c3a27ba4f191bb4e4d05e41c1d2ece3bacb01daa58192c2d5dcbe4df` | `d44e22d96743e5e175e59dbac49ae30bccaa2c924d4172e6b936f9f91d9c4cf5` |

仅凭构建号不能唯一确定运行：`github.run_number` 在每个工作流中独立计数。v1.0.0 的构建号为 `1`，对应 [release #1](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39231899813)；CI 中也有 [ci #1](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38852616255)。`BUILD_INFO` 中的 `run_id=39231899813` 才能直接定位构建该 Release 的运行。

发布 v1.0.1 之后，于 2026-10-21 10:04（UTC+8）重新下载并校验 v1.0.0 的附件，计算得到的 sha256：`aeb05ee8c3a27ba4f191bb4e4d05e41c1d2ece3bacb01daa58192c2d5dcbe4df`，与上表中发布当时的值完全一致。两个版本的归档哈希均与各自的 Release `.sha256` 一致，服务器上对两个附件执行 `trace.sh` 的退出码均为 0。

`trace.sh` 输出（v1.0.0）：

```text
== 1. 校验和
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.0
build_no=1
run_id=39231899813
commit=a845dd34214418516d01f77fbd7796498a7866da
go=go1.26.9
== 3. 提交 a845dd34214418516d01f77fbd7796498a7866da
作者: Mingyang Cui <159035665+AllenTwillight05@users.noreply.github.com>
时间: 2026-10-18T14:55:24+08:00
标题: ci: add release workflow (#3)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
版本号: 1.0.0    构建号: 1
提交页面（含关联的 PR）: https://github.com/S1mpleWind/sbSE-lab1/commit/a845dd34214418516d01f77fbd7796498a7866da
构建它的运行: https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39231899813
```

以上命令在小组华为云 Linux x86-64 主机执行，退出码为 0。服务器直连 GitHub 克隆时发生 TLS 错误，因此从本机下载 Release 原始附件和 `.sha256`，通过 SCP 传到服务器；源码历史通过包含 main 和 v1.0.0 的 Git bundle 克隆，并将 origin 设置为小组仓库地址。服务器上的附件校验通过，校验值与本机下载值、Release `.sha256` 三者一致。现场复核目录为 `/home/sbse/lab1-d-evidence/repo`，附件位于其上一级目录。发布门禁演练使用的远程测试 tag 和临时分支已清理，失败的 Actions 记录保留。

`trace.sh` 输出（v1.0.1）：

```text
== 1. 校验和
enroll-conflict_1.0.1_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.1
build_no=3
run_id=39530639353
commit=52c647c0cef046d70c144a6d0c63e81461daddb7
go=go1.26.9
== 3. 提交 52c647c0cef046d70c144a6d0c63e81461daddb7
作者: Yang Kezhuo <miner_04282024@qq.com>
时间: 2026-10-19T18:51:30+08:00
标题: docs: record initial release evidence and rollback procedure (#4)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.1
版本号: 1.0.1    构建号: 3
提交页面（含关联的 PR）: https://github.com/S1mpleWind/sbSE-lab1/commit/52c647c0cef046d70c144a6d0c63e81461daddb7
构建它的运行: https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39530639353
```

任务 6 第 4 步，两次 `/version` 的输出：

```text
GET /version (v1.0.1): {"version":"1.0.1","commit":"52c647c0cef046d70c144a6d0c63e81461daddb7","build_no":"3","go":"go1.26.9"}
GET /version (v1.0.0): {"version":"1.0.0","commit":"a845dd34214418516d01f77fbd7796498a7866da","build_no":"1","go":"go1.26.9"}
```

运行与回退在 2026-10-21 10:04（UTC+8）于同一台 Linux x86-64 主机完成，监听 `127.0.0.1:18080`。先运行 `/home/sbse/lab1-d-evidence/enroll-conflict_1.0.1_linux_amd64/enroll-conflict`，请求 `/version` 和 `/healthz` 后停止该进程，再启动同目录下 v1.0.0 的原始 Release 二进制并重复请求；两次 `/healthz` 均返回 `ok`。过程中没有重新构建；结束时两个演示进程均已停止。服务器仓库历史已通过新的 Git bundle 更新至 PR #4 的 main 提交，并包含两个版本的 tag。

现场可在服务器上复核：

```bash
cd /home/sbse/lab1-d-evidence/repo
bash scripts/trace.sh ../enroll-conflict_1.0.0_linux_amd64.tar.gz . origin/main
bash scripts/trace.sh ../enroll-conflict_1.0.1_linux_amd64.tar.gz . origin/main
```

## 5. 数据

时间均从 GitHub 页面读取，并注明时区。

第一轮 gate 演练的时间线：

| 事件 | 时间 | 链接 |
|---|---|---|
| 发起 PR | <填写> | <PR 链接> |
| 第一次 `ci` 失败（运行结束） | <填写> | <链接> |
| 推送修复提交 | <填写> | <链接> |
| `ci` 通过（运行结束） | <填写> | <链接> |
| 合入 main | <填写> | <链接> |

提交到发布的间隔，至少记录三个合入的 PR：

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| #<编号> | <填写> | <填写> | <填写> | <填写> |
| #<编号> | <填写> | <填写> | <填写> | <填写> |
| #<编号> | <填写> | <填写> | <填写> | <填写> |

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| #<编号> | <填写> | <填写> | <填写> |

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

<填写>

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

<填写>

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | `make package`，`VERSION`、`BUILD_NO`、`RUN_ID`、`COMMIT` 均指定固定值 | <填写> | <填写> |
| ② | 与 ① 完全相同，再构建一次 | <填写> | <填写> |
| ③ | 仅将 `BUILD_NO` 换成另一个值 | <填写> | <填写> |
| ④ | 去掉 `--mtime=@0`，其余同 ① | <填写> | <填写> |
| ⑤ | 同 ④，间隔几秒后再构建一次 | <填写> | <填写> |
| Actions | 对 main 上的一次 push 运行执行 Re-run all jobs，比较两次尝试中 package 步骤打印的值 | <两次的值> | <两次的值> |

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

<填写>

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

<填写>

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | <未做 / 链接、镜像摘要（digest）、运行容器的主机架构> |
| O2 缓存对反馈时间的影响 | 已完成两组各 5 次 PR CI 对照：[完整记录与分析](docs/O2-cache-experiment.md)、[逐次数据](docs/o2-cache/results.csv)。有缓存组均值 13.8 秒，无缓存组 24.8 秒；记录了缓存命中、runner 镜像差异和样本局限。[基准 PR #6](https://github.com/S1mpleWind/sbSE-lab1/pull/6) 与 [对照 PR #7](https://github.com/S1mpleWind/sbSE-lab1/pull/7) 均不合并。 |
| O3 迁移到小组项目 | <未做 / 仓库链接> |

## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| yollsplus | [#4](https://github.com/S1mpleWind/sbSE-lab1/pull/4) | 暂无 |
| <用户名> | <编号> | <编号> |
