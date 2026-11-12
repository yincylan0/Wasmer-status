# Wasmer 状态页

无第三方依赖的只读 Python 状态页，提供 `/`、`/api/status` 与 `/healthz`。页面为中文深色主题，展示当前进程在 Wasmer Edge 沙箱中可见的系统信息、常用 CLI 和运行时版本；页面打开后立即向同源服务读取状态，此后每 60 秒刷新一次。轮询失败会保留最近一次成功显示的数据，并在后续周期重试；浏览器后台标签页可能对定时器节流，因此间隔并非实时性保证。容量按 1024 进位，运行时间以易读格式显示。源码位于 [yincylan0/Wasmer-status](https://github.com/yincylan0/Wasmer-status)。本项目改编自 [mathowvan/orkestr-status](https://github.com/mathowvan/orkestr-status)。

## 部署到 Wasmer Edge

需要先安装 Wasmer CLI 并登录（`wasmer login`），然后在仓库根目录执行：

```sh
wasmer deploy
```

应用名为 `wasmer-status`，包名为 `karlbushby11/wasmer-status`，部署后地址形如 `https://wasmer-status-karlbushby11.wasmer.app`。应用使用 `wasmer/python`（Python 3.12，WASI）运行 `src/wasmer_status.py`，监听 `HOST`（默认 `127.0.0.1`）与 `PORT`（默认 `80`）环境变量；健康检查端点 `/healthz` 可独立访问。

Wasmer Edge 不用 Dockerfile：部署配置是仓库根目录的 `wasmer.toml`（包定义：`wasmer/python` 依赖、`/src` 文件映射）与 `app.yaml`（应用名、包、启动参数）。

## 本地运行与测试

```sh
PORT=8123 python3 -I -B -u src/wasmer_status.py
python3 -I -B -m unittest -v
node --test poll.test.js
```

也可以用 Wasmer 本地运行（会下载 `wasmer/python` 包）：

```sh
wasmer run . --env PORT=8123
```

Node.js 仅用于运行前端行为测试，不是应用运行依赖。轮询请求在 15 秒后超时，保留旧数据并等待下一周期重试。

## 指标范围与安全提示

这些是沙箱内进程可见的指标，不代表 Wasmer 账户、服务商层面的可用性、运行时间或 uptime，也不是平台监控或告警保证。WASI 沙箱没有 `/proc`、`/sys/fs/cgroup` 与 `/etc/os-release`，也无法执行子进程，因此内存、CPU 配额、负载与命令行工具版本等字段在 Wasmer 上通常显示为“不可用”或“未安装”，这是预期行为；页面本身、JSON API 与健康检查均正常工作。网络详情默认关闭，只有显式设置 `STATUS_NETWORK_DETAILS=1` 才进行本地地址解析和固定 HTTPS 公网 IP 查询。

状态 API 未提供身份认证；若公开服务地址，任何人都可能读取系统指标和已识别的软件版本。请勿与敏感业务或凭据共置。
