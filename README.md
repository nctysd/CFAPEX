CFAPEX 1.3 更新日志

更新范围：自定义优选 IP 支持一次保存 1000 条、GitHub Raw 列表拉取超时修复之后的改动。 

## 功能与兼容性

- 新增 **Trojan/WS** 和 **VLESS/XHTTP（stream-one）**，与原有 VLESS/WS 共存；Trojan 和 XHTTP 只在 TLS 端口下发，均支持 TCP。Trojan 可单独设置密码，两个新协议均可通过环境变量关闭。
- 概览保留四个订阅入口：VLESS/WS YAML、Trojan/WS YAML、通用订阅、VLESS/XHTTP YAML。通用入口可切换 URI 与 sing-box JSON；显式 JSON 地址避免 sing-box 桌面客户端因请求头未被识别而收到 URI。XHTTP 查询转为 sing-box JSON 时提供兼容的 VLESS/WS 节点。
- 参考 CFNext V3.0.2 的地区选择，将内置节点池从 234 条更新为 555 条去重地址；用户保存的自定义优选列表仍优先。修正“CF 移动优选”备注被误识别为国家代码的问题。
- Clash YAML 修复 `MATCH` 规则的代理组名称；默认选择组包含自动测速，最多抽样 24 条测速，全部节点仍可手动选择。增加 UDP/443 拒绝规则，让视频流量的 QUIC 请求回退 TCP。

## 转发与资源控制

- 自定义 Proxy IP 仅接管 Cloudflare IPv4 目标的 TLS 流量；候选需完成 TLS 首响应验证，失败时尝试同地区下一条，最终恢复原有出站。非 Cloudflare 目标沿用原有出站。
- 错误 VLESS UUID 与 Trojan 密码立即拒绝；未认证 WebSocket 首包上限降至 64 KiB。默认限制目标端口；WS/XHTTP 单连接最长 15 分钟，上行最多 64 MiB、下行最多 128 MiB。
- 修复无关请求也读取 KV 的问题：无关路径和无效订阅地址在读取前返回；同一 Worker 实例 10 秒内复用并合并 KV 读取，保存或还原后立即失效。面板停止每 15 秒自动轮询，重新显示时刷新状态。
- Worker 字符串及面板 HTML 使用运行时解码表交付，Workers 与 Pages 保持相同的单文件入口。代码混淆不会改变网络行为，也不能消除 Cloudflare 的滥用通报。

## 验证与待确认事项

- 本地 KV 计数：40 个无关请求从 40 次读取降为 **0 次**；同一时窗内 30 次订阅请求共读取 **3 次**。协议、Proxy IP、真实出站及实际 workerd 回归通过；Workers、Pages 和 ZIP 内入口逐字节一致。
- 内置节点抽样 21 条进行 TLS/WebSocket 握手，12 条返回 101。该结果不能证明移动网络下全部节点可用，也不能保证 4K 视频速度。sing-box 桌面程序实际导入及 Cloudflare 线上 KV 告警变化仍需部署后验证。


# CFAPEX
项目是可部署在 Cloudflare 上的代理面板，支持 VLESS/WS、Trojan/WS 和 VLESS/XHTTP。用户可在面板中管理优选 IP、按地区设置 Proxy IP、自定义真实出站和节点 UUID，并获取 Clash YAML、通用 URI 或 sing-box JSON 订阅。项目提供 KV 设置存储、订阅访问控制及短时读取缓存。
