# enroll-conflict：实验 1 的统一示例

> 实验要求、任务步骤、检查点和评分见 Canvas 上发布的《实验 1：从提交到可追溯制品的流水线》指导书。本仓库只提供示例代码，不含指导书。

一个仅依赖标准库的小型服务：判断两个教学班的上课时间是否冲突，对应 L-05 课程实践中“冲突检测新规则”的场景。**服务本身不是实验的重点**，它的作用是为流水线提供一个可构建、可测试、可使测试失败的对象。

## 规则

两个教学班冲突，当且仅当：同一个星期、上课周次有交集、节次区间有重叠。上课周次分三种：`all`（每周）、`odd`（单周）、`even`（双周）。单周班与双周班在同一时段上课不冲突。

## 接口

| 方法与路径 | 作用 |
|---|---|
| `GET /healthz` | 存活检查，返回 `ok` |
| `GET /version` | 返回版本号、提交号、构建号、Go 版本（JSON） |
| `POST /api/conflict` | 请求体 `{"a": 教学班, "b": 教学班}`，返回 `{"conflict": true/false}` |

教学班的字段：`id`、`weekday`（1～7）、`start`、`end`（节次，含）、`weeks`。

## 在本地运行

需要 Go 1.26 或更高版本；`make` 目标还需要 GNU make 和 GNU tar（Linux 或 WSL2）。

```bash
make fmt-check vet test                    # quality gate
go run . -addr 127.0.0.1:8080              # 直接运行
make package VERSION=1.0.0 BUILD_NO=1      # 打包到 dist/
dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict -version
```

`make package` 的参数 `VERSION`、`BUILD_NO`、`COMMIT` 是制品的三个追溯标识，`RUN_ID` 用于指向具体的那次运行；未传入时分别取 `dev`、`local`、本地 `HEAD` 和 `local`，表示“未经过流水线”。

## 从 Release 运行与回退

以下操作在小组的 Linux x86-64 主机、已克隆的小组仓库中执行，需要 Git、curl 和 GNU tar。先确认两个 Release 都已发布，再下载附件；运行已发布的程序无需安装 Go 或重新编译。

```bash
git fetch origin main --tags
mkdir -p dist/release
for v in 1.0.0 1.0.1; do
  f="enroll-conflict_${v}_linux_amd64.tar.gz"
  url="https://github.com/S1mpleWind/sbSE-lab1/releases/download/v${v}"
  curl -fL -o "dist/release/$f" "$url/$f"
  curl -fL -o "dist/release/$f.sha256" "$url/$f.sha256"
  bash scripts/trace.sh "dist/release/$f" . origin/main || exit 1
  tar -xzf "dist/release/$f" -C dist/release
done
```

`trace.sh` 检查附件校验和、读取 `BUILD_INFO`、确认提交在 main 的历史中，并给出提交和 Actions 运行链接。将首次下载 v1.0.0 时的 sha256 另行保存在 `LAB1.md`，发布 v1.0.1 后重新下载旧附件并与该值比较。

确认 `127.0.0.1:18080` 空闲后，在 Bash 中依次运行新版和旧版。`pid` 仅记录本次启动的进程；每次退出前停止它：

```bash
for v in 1.0.1 1.0.0; do
  ./dist/release/enroll-conflict_${v}_linux_amd64/enroll-conflict -addr 127.0.0.1:18080 &
  pid=$!
  trap 'kill "$pid" 2>/dev/null || true' EXIT
  sleep 1
  kill -0 "$pid" || exit 1
  curl -fsS http://127.0.0.1:18080/version || exit 1
  kill "$pid"
  wait "$pid" 2>/dev/null || true
  trap - EXIT
done
```

将两次响应记录到 `LAB1.md` 第 4 节。回退使用 v1.0.0 的原始 Release 附件；已经发布的 tag 和版本号保留，修复通过新版本发布。

## 文件

| 文件 | 说明 |
|---|---|
| `conflict.go`、`conflict_test.go` | 冲突规则及其测试 |
| `main.go`、`main_test.go` | HTTP 服务、版本信息及其测试 |
| `Makefile` | 本地与 CI 共用的构建入口 |
| `scripts/trace.sh` | 追溯脚本：由制品回答“它来自哪次提交” |
| `Dockerfile` | 选做 O1 使用 |
| `LAB1.md` | 小组的证据与分析记录，需要填写并提交 |

`.github/workflows/ci.yml` 负责 PR 检查和 main 制品，`release.yml` 负责 tag 发布；发布门禁要求 tag 指向 main 历史中的提交。
