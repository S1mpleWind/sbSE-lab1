# 实验 1 记录：第 6 组

本组使用统一示例 `enroll-conflict`，完成合并门禁、发布门禁、双版本追溯与回退，并据实记录数据和分析。GitHub 时间统一换算为北京时间（UTC+8）；提交、审批、PushEvent 和作业的秒级时间使用 GitHub API 与页面、日志交叉核对。任务 0 的完整前后对照和问题 3 的本地构建于 2026-10-21 补测，与初次实验分别注明。main 已受保护，本报告通过 [PR #9](https://github.com/S1mpleWind/sbSE-lab1/pull/9) 提交，待他人审批并合入。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 6 组，共 5 人 |
| 仓库地址 | <https://github.com/S1mpleWind/sbSE-lab1> |
| 代码载体 | 统一示例 `enroll-conflict`，Go，功能为教学班上课时段冲突检测 |
| runner 与语言版本 | GitHub Actions：`ubuntu-24.04`（x64），Go `1.26.9`；第二轮本地验证：macOS、`darwin/arm64`、Go `1.26.9`；本地构建补测：WSL2 Ubuntu 24.04、Linux `6.6.87.2-microsoft-standard-WSL2`、`linux/amd64`、Go `1.26.9` |
| 合并方式 | squash；仓库仅启用 squash 合并，合并后自动删除分支，PR #1～#5 的 main 历史与此一致 |

## 2. 任务 0：本地基线

任务 0 初次实验中，修改后执行 `make package VERSION=1.0.0 BUILD_NO=1` 的结果为：`BUILD_INFO 没有变化`，二进制 sha256 为 `4abc817eb94bec30525e33e87e60dfee242dc55aaad78234e085d1fad55668c9`，`vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`、`vcs.time=2026-10-12T16:26:07Z`、`vcs.modified=true`。初次实验未完整留存首次二进制哈希、BUILD_INFO 内容和实际 Go 版本，完整前后对照由下方补测给出。

下表和两个代码块保留 2026-10-21 19:45（UTC+8）的完整补测：在独立临时克隆中使用同一[课程模板提交](https://github.com/zilinglius/operations-2026-lab1-sample/commit/7b63b41c13f061439997b0e6e4eaa2af922a7aa1)，执行 `make package VERSION=1.0.0 BUILD_NO=1`；然后在 `conflict.go` 追加一行注释、不提交，用相同命令再次打包。两次实验中 BUILD_INFO 不变、工作区变脏的观察一致；初次实验与补测的哈希分别记录，不混作一对。该模板的提交号不能用于追溯本组仓库的 Release。

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | 见下方原文；第二次与第一次逐字相同 |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？ | BUILD_INFO 不变；二进制 sha256 改变。第一次：`f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e`；第二次：`27b19a2488c82311263d3dfdf164727fc910062cfa83b3e739848816ef61ff66` |
| 两次 `go version -m` 输出中的 vcs 行有何不同？ | `vcs`、`vcs.revision`、`vcs.time` 均相同，`vcs.modified` 从 `false` 变为 `true`，原始行见下方 |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | 能读出名称、版本、构建号、Go 版本和声称使用的提交；没有机器与构建时间字段，`run_id=local` 也不能指向流水线。未提交修改后的包仍声称来自同一个 commit，仅凭该值把它等同于该提交的干净源码就会出错；`vcs.time` 是提交时间，不是构建时间 |

第一次 BUILD_INFO（2026-10-21 补测）：

```text
name=enroll-conflict
version=1.0.0
build_no=1
run_id=local
commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
go=go1.26.9
```

两次 `go version -m` 的 vcs 行（2026-10-21 补测）：

```text
# 第一次，工作区干净
build vcs=git
build vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
build vcs.time=2026-10-12T16:26:07Z
build vcs.modified=false
# 第二次，仅追加注释、未提交
build vcs=git
build vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1
build vcs.time=2026-10-12T16:26:07Z
build vcs.modified=true
```

注：上面仅将 `go version -m` 输出的制表符规范为空格；此外模块版本从 `v0.0.0-20261012162607-7b63b41c13f0` 变为带 `+dirty` 的版本。即使只改注释，Go 嵌入的 VCS 元数据也会改变二进制。补测后已在临时克隆中执行 `git checkout -- conflict.go` 恢复。

## 3. 证据

| 项 | 链接或文本 |
|---|---|
| 规则集里启用的规则 | [Main Branch Rule](https://github.com/S1mpleWind/sbSE-lab1/rules/25219306)，Active，作用于默认分支；Restrict deletions（`deletion`）、Block force pushes（`non_fast_forward`）、Require a pull request before merging（1 人审批）、Require status checks to pass（必需检查 `ci`，来源 GitHub Actions）。组长直接推送 main 同样被门禁拒绝，验证输出见下方 |
| 组长直接 `git push origin main` 被拒的终端输出 | 空提交 `a14b654` 推送 main 时收到 `GH013`，提示必须经 PR 且必需检查 `ci` 未满足，远程拒绝该次推送。关键输出见下方 |
| 第一轮：失败的 PR 运行 | [PR #1](https://github.com/S1mpleWind/sbSE-lab1/pull/1)，[ci #2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856650142)，head `619e544a5a724ed04c4ff48a83692966e776bcd1`。将 `weeksOverlap` 的非每周分支改为总返回 `true`，现有“单周对双周不冲突”用例及 HTTP 接口测试失败 |
| 第一轮：修复后通过的 PR 运行 | [ci #3](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856745614)，修复提交 `54724f5dd965acd4718ea0c797f85a00402e7c8c`，恢复 `return a == b`，所有检查通过 |
| 第一轮：合入 main 后的运行（含制品） | [ci #4](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38857824225)，main 的 squash 提交 `34d9e65b2af66ab481d4504f384d34f2ba27df7a`；Package、Print checksums、Upload artifact 均成功，制品名为 `build-34d9e65b2af66ab481d4504f384d34f2ba27df7a-4`，保留 14 天 |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | [PR #2](https://github.com/S1mpleWind/sbSE-lab1/pull/2)：把 `Section.Valid` 中 `s.End < s.Start` 改为 `s.End <= s.Start`，误拒绝合法的单节课（`start == end`）。原测试的合法输入均为多节课，非法输入未覆盖这个应当合法的边界，故测试仍通过 |
| 第二轮：仅含缺陷时仍然通过的运行 | [ci #5](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38860438084)，head `366876350af7390fc59a4ee4d51474e675287137`；自动检查未拦住缺陷，此时未审批、未合并 |
| 第二轮：补测试后失败的运行 | [ci #6](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38860537459)，head `90546e77aa31c5d985ac6ef3ee349c0e6287b70a`；新增 `TestValidSinglePeriod`，`all`、`odd`、`even` 三个子测试全部失败 |
| 第二轮：撤销缺陷后通过的运行 | [ci #7](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38860647442)，修复提交 `f3c6600aadc702f12064c6eb6bcffd43a1f6e77e`；恢复 `<`，保留新增测试，经 S1mpleWind 审批后 squash 合入；[main ci #8](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38862738456) 通过 |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | [release #2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39231936952)：`v0.0.1` 指向未合入 main 的空提交 `83b5ab7`，在 `Check that the tag points to main` 步骤失败，后续构建和发布步骤均跳过，未产生 Release |
| Release v1.0.0 | [v1.0.0](https://github.com/S1mpleWind/sbSE-lab1/releases/tag/v1.0.0)，发布于 2026-10-19 17:41:16（UTC+8） |
| Release v1.0.1 | [v1.0.1](https://github.com/S1mpleWind/sbSE-lab1/releases/tag/v1.0.1)，发布于 2026-10-21 10:01:34（UTC+8） |

规则证据可由[规则集 API](https://api.github.com/repos/S1mpleWind/sbSE-lab1/rulesets/25219306)复核。规则集允许的合并方法字段包含三种方法，但仓库级设置仅启用 squash，因此实际使用 squash；必需检查未启用严格要求分支与 main 同步，推送新提交后也未自动撤销已有审批，不能把当前配置解释成已检查所有并发合并情形。

组长直接推送被拒的原始输出节选：

```text
[main a14b654] test: direct push
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: Review all repository rules at https://github.com/S1mpleWind/sbSE-lab1/rules?ref=refs%2Fheads%2Fmain
remote:
remote: - Changes must be made through a pull request.
remote:
remote: - Required status check "ci" is expected.
remote:
To github.com:S1mpleWind/sbSE-lab1.git
 ! [remote rejected] main -> main (push declined due to repository rule violations)
error: failed to push some refs to 'github.com:S1mpleWind/sbSE-lab1.git'
```

第二轮演练各阶段的 PR 状态和本地验证结果如下：

| 阶段 | CI 与 PR 状态快照 | 本地验证 |
|---|---|---|
| 仅含缺陷，原测试 | ci #5 为 `SUCCESS`，`reviewDecision=REVIEW_REQUIRED`、`mergeStateStatus=BLOCKED`；自动检查未发现缺陷，但尚未获人工审批 | 错误代码配原有测试，`make fmt-check vet test` 退出码为 0 |
| 保留缺陷，补充测试 | ci #6 为 `FAILURE`，`mergeStateStatus=BLOCKED`；`TestValidSinglePeriod` 的 all、odd、even 三个子测试均失败 | 错误代码配新增测试，退出码为 2 |
| 修复缺陷，保留测试 | ci #7 为 `SUCCESS`，记录时仍为 `REVIEW_REQUIRED`、`BLOCKED`，等待他人审批 | 修复代码配新增测试，退出码为 0；最终再次检查退出码也为 0 |

第二轮最终本地检查的输出为：

```text
go vet ./...
go test -count=1 ./...
ok  	enroll-conflict	0.488s
```

最终改动仅为 `conflict_test.go` 增加 11 行，`conflict.go` 已恢复。上表的待审批状态为演练时的状态快照；最终审批、合并和 main CI 结果见本节前表。

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

以下时间均为北京时间（UTC+8）。PR 创建与合入时间来自 PR 页面/API；提交时间取提交的 committer 时间；实际推送时间取 GitHub 仓库事件中的 PushEvent，避免把本地提交时间当作服务器收到推送的时间。运行结束按该次运行完成时的 `updated_at` 记录，作业 `completed_at` 偶尔早 1 秒；下表始终使用同一口径。

第一轮 gate 演练的时间线（2026-10-16）：

| 事件 | 时间 | 链接 |
|---|---|---|
| 分支第一个提交 | 13:36:05 | [619e544](https://github.com/S1mpleWind/sbSE-lab1/commit/619e544a5a724ed04c4ff48a83692966e776bcd1) |
| 发起 PR | 13:36:19 | [PR #1](https://github.com/S1mpleWind/sbSE-lab1/pull/1) |
| 第一次 `ci` 开始 | 13:36:24 | [ci #2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856650142) |
| 第一次 `ci` 失败（运行结束） | 13:36:40 | [ci #2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856650142) |
| 完成修复提交 | 13:37:23 | [54724f5](https://github.com/S1mpleWind/sbSE-lab1/commit/54724f5dd965acd4718ea0c797f85a00402e7c8c) |
| 推送修复提交（PushEvent） | 13:37:27 | [仓库事件 API](https://api.github.com/repos/S1mpleWind/sbSE-lab1/events?per_page=100)，`ref=refs/heads/test/gate-round1`、`head=54724f5…` |
| 修复后的 `ci` 开始 | 13:37:32 | [ci #3](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856745614) |
| `ci` 通过（运行结束） | 13:37:42 | [ci #3](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856745614) |
| 另一位组员批准 | 13:48:57 | [wgy2006 的审批](https://github.com/S1mpleWind/sbSE-lab1/pull/1#pullrequestreview-5543572544) |
| 合入 main | 13:49:48 | [PR #1](https://github.com/S1mpleWind/sbSE-lab1/pull/1)，main 提交 `34d9e65…` |
| main 的 `ci` 通过（运行结束） | 13:50:17 | [ci #4](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38857824225) |

两轮演练的 job 时间如下，均已换算为北京时间（2026-10-16）。这些是 job 的开始至结束时间，与上文按工作流完成时间统计的口径分别列出；例如 ci #2 的 job 于 13:36:39 完成，工作流最后更新于 13:36:40，并不矛盾。上方时间线的实际修复推送时间采用 GitHub PushEvent 时间，未拿 commit 时间替代。

| 运行 | job 开始 | job 完成 | job 时长 | 结果 |
|---|---|---|---|---|
| [ci #2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856650142) | 13:36:27 | 13:36:39 | 12 秒 | failure |
| [ci #3](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856745614) | 13:37:34 | 13:37:42 | 8 秒 | success |
| [ci #5](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38860438084) | 14:18:13 | 14:18:30 | 17 秒 | success |
| [ci #6](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38860537459) | 14:19:17 | 14:19:27 | 10 秒 | failure |
| [ci #7](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38860647442) | 14:20:27 | 14:20:38 | 11 秒 | success |

提交到发布的间隔：以 PR 中第一个提交到**首个包含该 PR 的正式 Release**的发布时间相减，不以 PR 创建或 main 的 squash 提交代替第一个提交；因此三个 PR 都对应 v1.0.0。PR #4 另作补充，PR #5 尚未进入已发布版本，不将 v1.0.1 误配给 #5。

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| [#1](https://github.com/S1mpleWind/sbSE-lab1/pull/1) | 2026-10-16 13:36:05 | 2026-10-16 13:49:48 | [v1.0.0](https://github.com/S1mpleWind/sbSE-lab1/releases/tag/v1.0.0)：2026-10-19 17:41:16 | 3 天 4 小时 5 分 11 秒（273,911 秒） |
| [#2](https://github.com/S1mpleWind/sbSE-lab1/pull/2) | 2026-10-16 14:17:50 | 2026-10-16 14:42:31 | v1.0.0：2026-10-19 17:41:16 | 3 天 3 小时 23 分 26 秒（271,406 秒） |
| [#3](https://github.com/S1mpleWind/sbSE-lab1/pull/3) | 2026-10-16 18:47:22 | 2026-10-18 14:55:25 | v1.0.0：2026-10-19 17:41:16 | 2 天 22 小时 53 分 54 秒（255,234 秒） |
| [#4](https://github.com/S1mpleWind/sbSE-lab1/pull/4) | 2026-10-19 17:45:59 | 2026-10-19 18:51:31 | [v1.0.1](https://github.com/S1mpleWind/sbSE-lab1/releases/tag/v1.0.1)：2026-10-21 10:01:34 | 1 天 16 小时 15 分 35 秒（144,935 秒） |

`ci` 工作流的运行时长：列出取数时的全部 14 次原始运行，以及本次为问题 3 执行的第 2 次尝试；不混入 `release` 工作流。时长为 `updated_at - run_started_at`，包含作业启动等开销，不只统计 `go test` 本身；第 14 次原始尝试的时间在重跑前已保存。

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| [ci #1](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38852616255) | push | 37 秒 | 初次配置 CI，main 基线，通过并上传制品 |
| [ci #2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856650142) | PR | 16 秒 | PR #1，引入周次缺陷，失败 |
| [ci #3](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856745614) | PR | 10 秒 | PR #1，修复后通过 |
| [ci #4](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38857824225) | push | 26 秒 | PR #1 合入 main，通过并上传制品 |
| [ci #5](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38860438084) | PR | 20 秒 | PR #2，仅含单节课缺陷，仍通过 |
| [ci #6](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38860537459) | PR | 13 秒 | PR #2，补边界测试后失败 |
| [ci #7](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38860647442) | PR | 14 秒 | PR #2，修复后通过 |
| [ci #8](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38862738456) | push | 21 秒 | PR #2 合入 main，通过并上传制品 |
| [ci #9](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38889572768) | PR | 22 秒 | PR #3，增加 release 工作流，通过 |
| [ci #10](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39105761578) | push | 16 秒 | PR #3 合入 main，通过并上传制品 |
| [ci #11](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39232620390) | PR | 13 秒 | PR #4，补发布与回退文档，通过 |
| [ci #12](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39240448796) | push | 21 秒 | PR #4 合入 main，通过并上传制品 |
| [ci #13](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39531029157) | PR | 17 秒 | PR #5，补双版本追溯与回退证据，通过 |
| [ci #14](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39532499316) | push | 18 秒 | PR #5 合入 main，尝试 1，通过并上传制品 |
| [ci #14，尝试 2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39532499316/attempts/2) | push（Re-run all jobs） | 44 秒 | 2026-10-21 19:34:48～19:35:32，通过，供问题 3 比较；不新增构建号 |

原始 14 次运行合计 264 秒，平均 18.86 秒，中位数 17.5 秒，范围 10～37 秒；加上重跑后共 15 次尝试、合计 308 秒、平均 20.53 秒。上述样本来自小型标准库项目及 GitHub 托管 runner，不能直接推断其他项目的用时，也不足以单独得出缓存因果结论。

## 6. 分析问题

**问题 1：gate 的收益与成本。**

所用数据：第 3 节的 ci #2/#3、#5/#6/#7；第 5 节的第一轮时间线、14 次原始 CI 用时，以及 PR #1～#3 到 v1.0.0 的间隔。时间分解如下：

| 量 | 计算 | 结果 |
|---|---|---|
| PR 创建到首次失败结果 | 13:36:40 − 13:36:19 | 21 秒（含触发等待，运行本身 16 秒） |
| 失败结果到完成修复提交 | 13:37:23 − 13:36:40 | 43 秒 |
| 修复提交到推送被服务器记录 | 13:37:27 − 13:37:23 | 4 秒 |
| 修复推送到重新通过 | 13:37:42 − 13:37:27 | 15 秒（触发等待 5 秒、运行 10 秒） |
| 失败后恢复绿灯的耗时 | 13:37:42 − 13:36:40 | 62 秒 = 人工修复与推送窗口 47 秒 + 等待 CI 15 秒 |
| PR 创建到恢复绿灯 | 13:37:42 − 13:36:19 | 83 秒 = 等待两次结果 36 秒 + 修复与推送窗口 47 秒 |
| CI 通过到合入 | 13:49:48 − 13:37:42 | 12 分 6 秒 |

第一轮 gate 拦住了现有测试可发现的单周/双周误判，第二轮最初却放过了未覆盖的合法单节课缺陷，补上边界测试后才拦住它。以“首次失败到恢复绿灯”为失败带来的可观察处理窗口，本轮多经历了 62 秒，其中 47 秒处于人工修复与推送窗口、15 秒等待重新检查；这不是与一次未实际发生的成功运行相比的严格净延迟，也不能把 47 秒全当作键盘编辑时间。从 PR 创建起，两次检查结果共等待 36 秒，加上这 47 秒得到 83 秒，说明计算时须包含事件触发等待，而不能只相加两次 CI 的 16 秒和 10 秒。一次 CI 在原始 14 次样本中平均约 18.86 秒，分别只占 PR #1～#3 提交到发布间隔的约 0.0069%、0.0069%、0.0074%，且 CI 通过后仍等待了 12 分 6 秒才合并，说明本组交付历时主要落在人工协作和发布安排上。这些占比只比较一次平均 CI 与整个历时，不代表一个 PR 全部 CI 的累计占比，也不能证明移除 gate 就能缩短相同时间。即使补充单节课测试，错误的需求、遗漏的其他边界、真实流量下的性能、生产配置和部署兼容问题仍可能未被当前测试发现，需要人工审批、相应测试和运行观测。上述判断以门禁实际强制执行、必需检查来自正确工作流、测试断言符合规则为条件，不能将绿灯解释为软件无缺陷。

**问题 2：PR 中的构建与 main 上的构建。**

所用数据：[PR #1 的 ci #3 日志](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38856745614/job/116829808983)、[合入后的 ci #4](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/38857824225)、main 的 `git rev-list` 结果及 squash 配置。

| 对象 | 实际值 | 是否在本组 main 历史中 |
|---|---|---|
| PR 运行的 `github.ref` | `refs/pull/1/merge` | GitHub 的临时合并引用，不是 main 分支 |
| PR 运行的 `github.sha` | `f8c572fd88878fa24186c27c2e3fc98da3dcc9e1` | 否 |
| PR 运行的 `pull_request.head.sha` | `54724f5dd965acd4718ea0c797f85a00402e7c8c` | 否 |
| squash 合入后的 main 提交 | `34d9e65b2af66ab481d4504f384d34f2ba27df7a` | 是，ci #4 在此提交构建并上传 |

PR 日志中的两个 SHA 不相同：默认 `pull_request` 检查运行在 GitHub 将分支与当时基线合成的临时 merge commit 上，head SHA 则是分支作者实际推送的提交。本组使用 squash，合并时把分支净改动生成一个新提交 `34d9e65…`，所以这两个 PR SHA 均未成为 main 的祖先，main 上可追溯的正式身份是新的 squash SHA。若使用 merge commit 合并，分支 head 通常会保留为 main 的祖先；rebase 则通常改写提交身份，因此不能脱离合并方式回答这个问题。PR 的临时合并结果可能随基线变化，测试通过并不使该临时 SHA 自动成为 main 的正式提交，也不能用它冒充最终发布源码。因此本组只在 push 到 main 时产出主干制品，并从指向 main 历史提交的 tag 构建正式 Release，让制品记录的 SHA 能在受保护主干中复核。这一结论以默认 checkout、`pull_request` 触发和本组 squash 流程为条件；“来自 main”是本实验的发布来源约束，并不是 Git 在技术上不能编译其他提交。关于默认 PR SHA 的含义，参见 [GitHub 的 pull_request 事件文档](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#pull_request)。

**问题 3：可重复构建。**

所用数据：2026-10-21 19:45（UTC+8）起在独立临时克隆上补测；源码固定为本组 main 提交 `989ebe3e77af41d810261a71ec4df0877ffc28e7`，Go `1.26.9`、`CGO_ENABLED=0`、`GOOS=linux`、`GOARCH=amd64`，使用同一 GNU tar/gzip 与 Git 元数据。①②固定参数如下；③仅把 `BUILD_NO` 从 `7` 改为 `8`，④⑤恢复 `7` 并仅临时去掉 tar 参数中的 `--mtime=@0`，⑤距④至少 3 秒。补测版本号是实验参数，未创建新 tag 或 Release。

```bash
make package VERSION=1.0.0 BUILD_NO=7 RUN_ID=12345 COMMIT=989ebe3e77af41d810261a71ec4df0877ffc28e7
# ③：同上，仅 BUILD_NO=8
# ④⑤：BUILD_NO=7，临时去掉 Makefile 中 tar 命令的 --mtime=@0
# 测完在临时克隆中恢复：
git checkout -- Makefile
git status --porcelain     # 实际输出为空
```

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | 固定 VERSION=1.0.0、BUILD_NO=7、RUN_ID=12345、COMMIT=989ebe3… | `2588969890508b61588b5c0b184c8ca219bda7f8db578ef9a17f4601fc9c3005` | `9978be711435a9cd396fff889d5f961688f4622aee6b1f2be64a684a1e9434b8` |
| ② | 与①完全相同，再构建一次 | `2588969890508b61588b5c0b184c8ca219bda7f8db578ef9a17f4601fc9c3005` | `9978be711435a9cd396fff889d5f961688f4622aee6b1f2be64a684a1e9434b8` |
| ③ | 仅 BUILD_NO 改为 8 | `66b98542e6f572cf427ae4a2cb81eea51b2b8b4779c9b288b4a6f3ae1d2fcd73` | `adb9add7fcc608489966943cafb03297571c03d08ad7ce8552592c6bfbd38318` |
| ④ | 去掉 --mtime=@0，其余同①；Makefile 未提交 | `09949734de3f3f303366fafafcc3a50c99649a8a202a30bb106fbcf51d531322` | `c0536fdb1eab848588248afd238cc44b11ab5de98ad31c3dd7fdcd678e124149` |
| ⑤ | 同④，至少间隔 3 秒 | `09949734de3f3f303366fafafcc3a50c99649a8a202a30bb106fbcf51d531322` | `a6d951ab5561db08a4e1d44bb2042658476415cb8bd83ce0337f358a0fbdb449` |
| Actions | main ci #14 的尝试 1 → 尝试 2 | `1694a045b308303b563494672b16d66284a356fd91a83e741fc35271942049e3` → `1694a045b308303b563494672b16d66284a356fd91a83e741fc35271942049e3` | `658ce54c50855c9cca864a338ee1fe6dd31483c86b8ef4b0a7de493aa88b723d` → `658ce54c50855c9cca864a338ee1fe6dd31483c86b8ef4b0a7de493aa88b723d` |

Actions 对照来自 [ci #14 尝试 1](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39532499316/attempts/1)（2026-10-21 10:20:50～10:21:08）和 [Re-run all jobs 尝试 2](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39532499316/attempts/2)（19:34:48～19:35:32）。两次都为 `VERSION=0.0.0-main`、`BUILD_NO=14`、`RUN_ID=39532499316`、`COMMIT=989ebe3e77af41d810261a71ec4df0877ffc28e7`；`run_attempt` 从 1 变为 2，但未写入制品，因此这两次输入不随尝试号变化。这里记录的是日志中 `dist/.../enroll-conflict` 与 `dist/*.tar.gz` 的哈希，不是 upload-artifact 外层 ZIP 的摘要。

①②的二进制和归档都相同，说明固定输入及环境时，本组这套构建可以复现字节；③的 `BUILD_NO` 同时进入链接参数和 BUILD_INFO，因此两种哈希都改变。④⑤的二进制彼此相同，但归档不同，因为去掉 `--mtime=@0` 后，重写 BUILD_INFO 等操作产生的文件时间进入 tar，`gzip -n` 不能消除 tar 内部的时间差。④的二进制与①也不同，`go version -m` 实测显示 `vcs.modified=false` 变成 `true`，模块版本附加 `+dirty`，原因是修改 Makefile 使工作区变脏，而不是 tar 参数直接影响了编译逻辑。制品应包含名称、版本、构建号、准确提交、Go 版本及可定位运行的 run_id，并保留或明确检查 VCS 状态；干净提交和固定参数仍不足以涵盖全部输入，工具链、依赖、Git/tag 元数据、文件权限和归档规则也须保持一致，机器绝对路径与每次变化的墙钟时间不应无必要地写入制品，运行主机和构建时刻可由 Actions 外部记录追查。本组 main 的 CI 编入 `0.0.0-main`，release 则要把正式版本号通过 `-X main.version=…` 写入二进制，且有自己的构建号与 run_id，所以 release 必须按当前设计重新编译并重新检查，不能原样发布主干 CI 的包。这在 CI 到 release 之间偏离了“一次构建、多处部署”，代价是额外编译、测试和新字节可能引入的环境漂移风险；正式 Release 生成以后，各环境部署和回退仍应复用同一个已验证附件。若要贯穿整个交付链路坚持一次构建，可提前确定版本或把可变发布标识外置后按摘要提升同一制品，而本次哈希相等仅证明记录条件下的两次输出，不能保证任意机器、任意参数的重新构建都相同。

**问题 4：发布与回退。**

所用数据：第 4 节的两次 `trace.sh`、`/version` 与旧包复校验记录；v1.0.0 归档摘要 `aeb05ee8c3a27ba4f191bb4e4d05e41c1d2ece3bacb01daa58192c2d5dcbe4df`；问题 3 的五次构建；本组两个 Release 的 API 字段均为 `immutable=false`，不能声称已启用不可变发布。

若 v1.0.1 上线后发现缺陷，应按第 4 节的实际过程停止新版、切回已保存并校验过的 v1.0.0 原始附件，随后通过新 PR 修复、测试和审批，以新版本例如 v1.0.2 发布，并在 v1.0.1 的说明中记录问题。不应删除 v1.0.1 的 tag 后复用同一版本号，因为这会让同名版本对应不同字节，破坏下载缓存、部署审计和故障定位时的共同依据。任务 6 实际运行的回退文件是 `/home/sbse/lab1-d-evidence/enroll-conflict_1.0.0_linux_amd64/enroll-conflict`，来自原始 v1.0.0 Release 归档，无需重新构建；两次 `/version` 从 `1.0.1 / 52c647c… / build_no=3` 变成 `1.0.0 / a845dd3… / build_no=1`，与两份 BUILD_INFO 一致。若改为从 v1.0.0 源码重建，只有工具链、构建参数、Git/VCS 状态、依赖与归档条件都与原发布一致时才有望得到原字节，问题 3 已说明换构建号、工作区变脏或归档时间变化都会改变结果，因此回退优先复用原附件。若有人同时替换附件和配套 `.sha256`，并保持 BUILD_INFO 中的提交仍在 main 上，现有 `trace.sh` 仍可能通过，因为它不拿可信旧摘要对照，也不验证二进制确实由声称的源码构成。第 4 节独立记录的原始 sha256 能揭示被替换包与原包不一致，前提是报告中的旧值本身可信且未被一起篡改。GitHub 的 [Immutable releases](https://docs.github.com/en/code-security/concepts/supply-chain-security/immutable-releases) 可锁定已发布附件及关联 tag，并生成发布证明，适合加强这种保证；本组当前未启用，且这个结论仍以完整可信的校验记录、兼容的运行环境以及没有不可逆数据迁移等回退障碍为条件。

### 实验结论

两轮演练表明，门禁能强制执行已编码的检查与审批要求，覆盖不足时仍会放过缺陷；新增边界测试后，同一类回归可以在合并前被拦截。squash 后的 main SHA、正式版本、构建号与 run_id 共同建立追溯路径，独立保存的摘要用于确认制品字节是否变化。固定输入和归档元数据支持可重复构建，回退则直接复用已校验的旧 Release；第 4 节的双版本运行已完成这条链路。直接推送触发 GH013 拒绝，三阶段 PR、CI 和本地验证结果证明了门禁限制与测试覆盖变化的作用。E 已发起最终报告 [PR #9](https://github.com/S1mpleWind/sbSE-lab1/pull/9)，待他人审批并合入；未合入前不计作已合入的 PR 贡献。

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | 未做；仓库保留示例 Dockerfile，但没有构建、推送和运行镜像的证据，不将模板文件视为完成 |
| O2 缓存对反馈时间的影响 | 已完成：有缓存 [PR #6](https://github.com/S1mpleWind/sbSE-lab1/pull/6)、无缓存 [PR #7](https://github.com/S1mpleWind/sbSE-lab1/pull/7) 各运行 5 次，实验 PR 均保留未合并；job 平均用时分别为 13.8 秒、24.8 秒。[报告 PR #8](https://github.com/S1mpleWind/sbSE-lab1/pull/8) 已合入，完整方法与原始数据见 [O2 实验报告](https://github.com/S1mpleWind/sbSE-lab1/blob/242bacffd182bd28ab0f3b0f2bf18fa71ffd2459/docs/O2-cache-experiment.md)，本节列出全部测量与结论 |
| O3 迁移到小组项目 | 已完成核心流程迁移至 [IssueScope](https://github.com/S1mpleWind/IssueScope)；[O3 证据材料](https://github.com/S1mpleWind/IssueScope/blob/course-material/docs/O3-CI-release-evidence.md)记录合并门禁、CI 失败与修复、双版本发布及追溯，迁移差异见下表 |

O2 缓存对照实验（2026-10-22）：

测量前预期：示例没有第三方依赖，模块下载收益应较小，但编译输出缓存仍可能缩短 vet 和测试准备时间；恢复缓存也有开销，净收益以实测为准。[预期原始记录](https://github.com/S1mpleWind/sbSE-lab1/blob/242bacffd182bd28ab0f3b0f2bf18fa71ffd2459/docs/o2-cache/pre-experiment.txt)早于首次测量。

两组从同一基线 `989ebe3e77af41d810261a71ec4df0877ffc28e7` 创建，业务代码相同，只改变 `setup-go` 的 `cache: true/false`；均为 PR 检查，不执行打包上传。使用 `ubuntu-24.04`、Go `1.26.9`，交替测量，首次运行后各重跑 4 次。下表口径为 `ci` job 的 `completed_at - started_at`，不含排队等待，与第 5 节工作流运行时长的口径不同；十次均成功，有缓存组五次均命中。

| 尝试 | 有缓存：ci #15（秒） | 无缓存：ci #16（秒） |
|---|---:|---:|
| 1 | [14](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714012994/attempts/1) | [29](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714097159/attempts/1) |
| 2 | [12](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714012994/attempts/2) | [32](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714097159/attempts/2) |
| 3 | [15](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714012994/attempts/3) | [19](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714097159/attempts/3) |
| 4 | [15](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714012994/attempts/4) | [23](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714097159/attempts/4) |
| 5 | [13](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714012994/attempts/5) | [21](https://github.com/S1mpleWind/sbSE-lab1/actions/runs/39714097159/attempts/5) |
| 平均值 | 13.8 | 24.8 |
| 中位数 | 14 | 23 |
| 范围 | 12～15 | 19～32 |

本次样本中，热缓存组平均少用 11 秒，相对无缓存组缩短约 44.4%；主要差别在 vet，其平均用时分别为 4.4 秒和 15.0 秒，而 test 两组均为 3.0 秒，与复用编译输出的作用一致，不能解释成节省了第三方依赖下载。[原始 CSV](https://github.com/S1mpleWind/sbSE-lab1/blob/242bacffd182bd28ab0f3b0f2bf18fa71ffd2459/docs/o2-cache/results.csv)及[原始 JSON](https://github.com/S1mpleWind/sbSE-lab1/blob/242bacffd182bd28ab0f3b0f2bf18fa71ffd2459/docs/o2-cache/records.json)保留时间、步骤耗时、缓存状态和日志链接。

每组仅 5 次，runner 负载、网络和缓存解压速度会带来波动，两组分配到的 runner 镜像版本比例也不同，不能将全部差额精确归因于缓存。结果只反映本次 PR job 的热缓存与关闭缓存对照，不代表首次创建缓存、端到端反馈时间或其他项目的固定收益；无缓存组也未清空 runner 预装的 Go 工具链缓存。

O3 迁移说明与验证（材料更新于 2026-10-21）：

| 项 | 实施情况与证据 |
|---|---|
| 构建与测试适配 | 从 Go 单服务迁移到 Python 3.12 / FastAPI 后端和 React / TypeScript 前端，分别通过 uv、npm 安装锁定依赖，执行 pytest、前端 lint、Vitest 和 Vite build |
| 合并与发布门禁 | [main-branch 规则集](https://github.com/S1mpleWind/IssueScope/rules/25532883)已启用：PR、至少 1 人审批、必需检查 `ci`，禁止删除和强制推送；发布由版本 tag 触发，验证提交属于 main 历史后再检查、打包 |
| 门禁演练 | [PR #51](https://github.com/S1mpleWind/IssueScope/pull/51)：前端测试[失败](https://github.com/S1mpleWind/IssueScope/actions/runs/39570727566)后修复[通过](https://github.com/S1mpleWind/IssueScope/actions/runs/39570870332)；[PR #53](https://github.com/S1mpleWind/IssueScope/pull/53)：后端测试[失败](https://github.com/S1mpleWind/IssueScope/actions/runs/39572929615)后修复[通过](https://github.com/S1mpleWind/IssueScope/actions/runs/39573041544) |
| 发布与追溯 | 已发布 [v0.1.0](https://github.com/S1mpleWind/IssueScope/releases/tag/v0.1.0) 和 [v0.1.1](https://github.com/S1mpleWind/IssueScope/releases/tag/v0.1.1)，均附归档与 SHA-256 文件；材料记录下载校验通过，项目追溯脚本与 `/api/version` 返回的版本、构建号、commit 一致 |

制品包含后端源码和前端生产构建产物，元数据采用归档根目录的 `BUILD_INFO.json`（`version`、`build`、`commit`），运行接口改为 `/api/version`。项目的 `scripts/trace.sh` 已适配 JSON；其文件名、字段和目录布局与统一示例不同，若按自有项目的制品格式逐项验收，仍需适配。上述迁移差异及详细校验值见 [O3 原始材料](https://github.com/S1mpleWind/IssueScope/blob/course-material/docs/O3-CI-release-evidence.md)。

## 8. 成员贡献

按组内分工图记录 A～E 的任务；仅在下表填写 GitHub 用户名，不填写姓名。PR 归属和审批按 GitHub 实际记录：E 已审批 #3、#8，并发起本报告的 [PR #9](https://github.com/S1mpleWind/sbSE-lab1/pull/9)，待他人审批并合入，当前不计入已合入贡献。B 的 #6、#7 是按 O2 要求保留未合并的实验 PR，不计入已合入贡献；#8 是其已合入的 O2 报告 PR。D 的 #5 是任务 6 的补充证据 PR，不能算作 E 的报告 PR。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| S1mpleWind | [#1](https://github.com/S1mpleWind/sbSE-lab1/pull/1) | [#2](https://github.com/S1mpleWind/sbSE-lab1/pull/2) |
| wgy2006 | [#2](https://github.com/S1mpleWind/sbSE-lab1/pull/2)、[#8](https://github.com/S1mpleWind/sbSE-lab1/pull/8) | [#1](https://github.com/S1mpleWind/sbSE-lab1/pull/1)、[#4](https://github.com/S1mpleWind/sbSE-lab1/pull/4)、[#5](https://github.com/S1mpleWind/sbSE-lab1/pull/5) |
| AllenTwillight05 | [#3](https://github.com/S1mpleWind/sbSE-lab1/pull/3) | 暂无已提交的 APPROVED 记录 |
| yollsplus | [#4](https://github.com/S1mpleWind/sbSE-lab1/pull/4)、[#5](https://github.com/S1mpleWind/sbSE-lab1/pull/5) | 暂无已提交的 APPROVED 记录 |
| yangjunqiqq | 暂无已合入；已发起最终报告 [#9](https://github.com/S1mpleWind/sbSE-lab1/pull/9)，待审批并合入 | [#3](https://github.com/S1mpleWind/sbSE-lab1/pull/3)、[#8](https://github.com/S1mpleWind/sbSE-lab1/pull/8) |

具体分工与工作内容（按提供的图片）：

| 成员（GitHub 用户名） | 负责任务 | 具体工作 | 对应 PR / 记录 |
|---|---|---|---|
| S1mpleWind（组长） | Task 0–3、Task 4 第一轮 | 负责本地基线、仓库与成员、CI 和规则集；第一轮引入现有测试能发现的周次缺陷，经历 CI 失败、修复、审批与合入，完成直接推送拒绝验证与第一轮时间记录 | [#1](https://github.com/S1mpleWind/sbSE-lab1/pull/1)、第 2、3、5 节；任务 0 的完整前后对照见第 2 节补测 |
| wgy2006 | Task 4 第二轮、O2 | 找到现有测试未覆盖的单节课缺陷，完成“通过—补测试失败—修复通过”的三阶段演练及本地验证，保留新增测试；另完成 O2 缓存对照实验与已记录的审批 | [#2](https://github.com/S1mpleWind/sbSE-lab1/pull/2)、[#8](https://github.com/S1mpleWind/sbSE-lab1/pull/8)、第 3、5、7 节 |
| AllenTwillight05 | Task 5 | 编写 `release.yml`，验证 Release Gate，发布 v1.0.0 | [#3](https://github.com/S1mpleWind/sbSE-lab1/pull/3)、release #1/#2、v1.0.0 |
| yollsplus | Task 6 | 提交文档小改动 PR，发布 v1.0.1，完成双版本追溯、运行、旧附件复校验与回退，并补充服务器复核记录 | [#4](https://github.com/S1mpleWind/sbSE-lab1/pull/4)、[#5](https://github.com/S1mpleWind/sbSE-lab1/pull/5)、release #3、第 4 节 |
| yangjunqiqq | Task 7、全部文档 | 完成可重复构建补测、任务 0 缺失字段补测、CI 重跑比较、GitHub 时间与审批统计；汇总实验记录及选做材料，撰写四道分析题和实验结论，整理 LAB1.md；审批发布工作流与 O2 报告 PR | 最终报告 [#9](https://github.com/S1mpleWind/sbSE-lab1/pull/9) 已发起，待审批并合入，暂不计作已合入贡献 |
