---
name: ai-desktop-network
description: 诊断和修复 Ninja/Ninjia/NinjaDesktop（Mihomo）、ChatGPT/Codex 与 Claude 桌面端的网络、代理、TUN、DNS、流式连接和区域访问问题，支持 macOS 与 Windows。用户提到这些应用无法连接、503、403、重连、stream disconnected、error decoding response body、代理错误或网络优化时使用。
---

# AI 桌面端网络优化

默认用中文回复。目标是恢复可验证的稳定连接，优先保留用户当前节点和网络选择；除非用户明确要求或现有出口被服务端拒绝，不自动切换节点。

## 工作流

1. **复现并分层**：记录操作系统、应用版本、Ninja 内核状态、当前节点、系统代理/TUN 状态和原始错误。分别测试浏览器、Ninja 检测、桌面端，不能把“网页可达”当成“桌面端可达”。
2. **先查根因**：区分本机网络不可达、代理未继承、DNS/IPv6、连接被中断、服务端或网关的 403/区域判定、应用缓存/会话卡死。先读应用日志和响应正文/状态，再改一个变量。
3. **保持配置一致**：确认系统代理指向 Ninja 当前 HTTP/Mixed 端口（常见 7897，必须以 UI 实际值为准），SOCKS 仅在应用明确要求时使用；TUN 与系统代理可以同时存在，但要确认只有一个有效流量入口，避免双重代理或回环。
4. **应用专项处理**：
   - ChatGPT/Codex：验证 `chatgpt.com` 与其后端请求，重点检查 SSE/HTTP 流式响应、代理连接复用和 403/503；优先重启内核与应用。
   - Claude：先用同一节点的浏览器对比。若浏览器可用而桌面端失败，优先完整退出并重开桌面端，必要时让 Electron 显式使用现有本地代理；`region_unavailable` 通常是服务端或网关的区域判定，需结合响应正文和同节点浏览器结果判断。
5. **最小修复**：按证据尝试重启内核、在用户允许的地区范围内固定已验证节点、重启桌面端、关闭不稳定 IPv6、清理应用缓存（先备份，保留 Cookies/登录数据）。不要先登出、重置订阅、删除配置或改 DNS；终止进程前先正常退出，强制结束只作兜底。
6. **验证完成**：分别用 HTTP/Mixed 代理做一次 DNS/TLS/HTTP 连通性检查，再确认 Ninja 的 ChatGPT/Claude 检测和桌面端 UI。日志不再出现对应的 403、`ERR_CONNECTION_CLOSED`、`Failed to fetch`、流式解码错误。记录改动和回滚方式。

## 平台操作

按需读取：

- macOS：见 [references/macos.md](references/macos.md)
- Windows：见 [references/windows.md](references/windows.md)

## 约束

- 不泄露或打印代理 Token、WebUI Token、内核 Secret、Cookie、OAuth 凭证。
- 不把用户偏好的节点改成自动选择；发现出口被拒绝时，先报告证据，再询问是否允许换到用户指定的地区。
- 不执行破坏性的网络重置（如删除所有代理配置、重置 Winsock、删除应用数据）除非用户明确要求且已保存回滚信息。
