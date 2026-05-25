# 大象接入 Handoff

## 当前只保留 WebSocket 版 `daxiang`

`cc-connect` 接入 `/Users/liuhaodong/rc-project/ugc-agent-tools` 时走 WebSocket bridge，不直接承接大象开放平台 Thrift callback。

拓扑：

```text
大象 -> ugc-agent-tools Thrift callback -> ugc-agent-tools WebSocket -> cc-connect
cc-connect -> ugc-agent-tools WebSocket -> ugc-agent-tools 大象 OpenAPI/HTTP -> 大象
```

关键文件：

- `platform/daxiang/`
- `cmd/cc-connect/plugin_platform_daxiang.go`
- `docs/daxiang.md`
- `docs/bridge-protocol.md`
- `docs/bridge-protocol.zh-CN.md`

核心行为：

- cc-connect 主动连接 `ws_url`
- 发送 `client.register`
- 用 `client_id + client_secret + timestamp` 做 AES-GCM 注册凭证
- 收到 `client.registered` 后处理 `bridge.event.message`
- Agent 回复通过 `agent.reply.*` 帧写回 bridge

## 已移除原生 Thrift 版实现

原 `platform/daxiang` 曾经是 cc-connect 直接承接大象开放平台 Thrift callback 的实现。

当前实际链路由 `ugc-agent-tools` 承接 Thrift callback，再通过 WebSocket 转发给 cc-connect，所以原生 Thrift 版实现已删除。现在的 `platform/daxiang` 是原 `platform/daxiangbridge` 改名后的 WebSocket bridge 实现。

- 原生 Thrift callback server
- 手写 `xmopencallback` thrift processor
- `go.mod/go.sum` 中仅由 `daxiang` 引入的 thrift 依赖

后续大象相关开发只看 `daxiang`。

## 本次 CI lint 修复

GitHub Actions job：

```text
CI / lint
```

失败原因：

```text
platform/daxiang/client.go
errcheck: Error return value of conn.WriteMessage is not checked
```

已做修复：

- `platform/daxiang/client.go` 中 close frame 写入补错误处理
- `platform/daxiang/daxiang_test.go` 中测试 server 的 `WriteMessage` 也补错误处理

验证命令：

```bash
go test ./platform/daxiang
git diff --check
go test ./...
```

注意：`daxiang` 现在仍然走 WebSocket bridge；Thrift callback 在 `ugc-agent-tools` 侧。

## 当前工作区状态

本次相关修改：

- 将 `platform/daxiangbridge/` 改名为 `platform/daxiang/`
- 将 `cmd/cc-connect/plugin_platform_daxiangbridge.go` 改名为 `cmd/cc-connect/plugin_platform_daxiang.go`
- 修改 `Makefile`
- 修改 `go.mod`
- 修改 `go.sum`
- 修改 `platform/daxiang/client.go`
- 修改 `platform/daxiang/daxiang_test.go`
- 新增/更新 `docs/daxiang-handoff.md`

另有未跟踪目录：

- `docs/superpowers/`

该目录不是本次工作产生的改动，未处理。
