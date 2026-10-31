# Claude Kill Switch for macOS

一个面向代理用户的 macOS 网络防泄漏工具：代理或 TUN 失效时，让 Claude 断网，而不是悄悄回落到物理网卡直连 Anthropic。

> This project prevents Claude traffic from falling back to a physical network interface when a proxy or TUN tunnel fails. The documentation is currently Chinese-first.

## 它解决什么问题

代理正常时：

```text
Claude Code / Claude Desktop → localhost 或 utun → 代理 → Internet
```

代理失效时，macOS 可能重新选择 Wi-Fi、以太网或雷雳桥作为出口。本工具使用系统原生 `pf`，在物理接口上：

- 阻断 Anthropic 官方固定入站 IPv4 网段；
- 阻断公网 IPv6 与 NAT64，防止 IPv6 绕过仅支持 IPv4 的代理；
- 保留回环、局域网、链路本地和组播 IPv6；
- 不阻断 `utun` 和本机代理端口。

Anthropic 网段来源：[Anthropic IP addresses](https://platform.claude.com/docs/en/api/ip-addresses)。

## 安全语义

本工具采用 **fail-closed** 设计：

- 代理正常：Claude 经代理使用；
- 代理退出或崩溃：规则继续存在，Claude 无法回落到家宽直连；
- 代理重新启动：流量重新经过代理，无需重启本工具；
- 守护进程退出：`launchd` 自动重启；
- `pf` 被关闭、规则丢失、`pf.conf` 引用被覆盖或出现新物理网卡：守护进程自动修复。

它不会修改代理客户端、Claude、账号、钥匙串、登录凭据或系统代理设置。

## 系统要求

- macOS；
- 管理员权限；
- 使用本机代理或 TUN 的网络环境；
- `/etc/pf.conf` 可由系统 `pfctl` 正常解析。

## 安装

下载仓库后执行：

```bash
chmod +x ./claude-killswitch
./claude-killswitch check
sudo ./claude-killswitch install
sudo claude-killswitch verify
sudo claude-killswitch status
```

安装过程会：

1. 自动识别 `en*` 和 `bridge*` 物理接口；
2. 在写入前验证候选 `pf` 配置；
3. 备份原 `/etc/pf.conf`；
4. 写入独立 anchor；
5. 安装 root `LaunchDaemon`；
6. 保留安装前 `pf` 是否启用的状态，以便卸载时恢复。

## 验证

```bash
sudo claude-killswitch verify
```

验证不会连接 Anthropic。它会从多个非 Anthropic 诱饵地址中选择一个可达基线，完成：

```text
同一物理出口、同一目标：可达 → 加载同构阻断规则后不可达 → 恢复后再次可达
```

如果基线本身不可达，验证会明确失败，不会把普通网络故障误判成规则生效。

## 状态检查

```bash
sudo claude-killswitch status
```

状态页会显示：安装状态、`pf` 状态、自动守护、物理接口覆盖、本地 HTTP 代理、IPv4/IPv6 路由类别。

## 卸载

```bash
sudo claude-killswitch uninstall
```

卸载会：

- 停止并移除 LaunchDaemon；
- 删除本项目的 anchor 和 `pf.conf` 管理片段；
- 恢复安装前 `pf` 的启用状态；
- 保留配置备份供人工审计。

备份目录：`/var/backups/io.github.claude-killswitch.macos/`。

## 威胁边界

本工具能保护：

- 任意进程经物理接口直连 Anthropic 官方固定 IPv4 网段；
- 任意进程经物理接口访问公网 IPv6 或 NAT64。

本工具不能保证：

- 阻断所有第三方托管的遥测、MCP、插件或未来新增地址；
- 判断当前代理究竟使用了哪个出口节点；
- 防止代理服务自身泄漏流量；
- 避免账号风控、停用或封禁；
- 在第三方安全软件完全替换或禁用 macOS `pf` 时继续工作。

macOS `pf` 不提供适合此场景的可靠“按应用路径”出站过滤，因此本项目没有声称完整复刻 Windows 按进程规则。

## 为什么不在代理退出时撤掉规则

主动退出和崩溃在进程层面可能完全相同。如果代理一消失就撤防，恰好会在最危险的时刻开放直连。本项目因此始终保持核心规则，只有显式执行 `uninstall` 才撤销。

## 致谢

项目思路受 Windows 版 [yoruuuchan/claude-desktop-killswitch](https://github.com/yoruuuchan/claude-desktop-killswitch) 启发。macOS 实现为独立编写，使用 `pf` 与 `launchd`，未直接移植 PowerShell 代码。

## License

[MIT](LICENSE)
