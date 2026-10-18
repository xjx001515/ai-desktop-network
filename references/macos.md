# macOS 操作参考

## 只读检查

```bash
scutil --proxy
networksetup -listallnetworkservices
service='Wi-Fi'  # 替换为 listallnetworkservices 中实际启用的服务
networksetup -getwebproxy "$service"
networksetup -getsecurewebproxy "$service"
networksetup -getsocksfirewallproxy "$service"
```

以 Ninja UI 显示的端口为准；常见 HTTP/Mixed 端口是 `127.0.0.1:7897`。

## 代理继承与应用重启

先从输出中选当前启用的服务（可能是 Wi-Fi、Ethernet、USB 或其他名称），不要假定服务名是 Wi-Fi。

正常退出 Claude 后再执行；`pkill` 只作应用无响应时的兜底。

```bash
pkill -f '/Applications/Claude.app/Contents/MacOS/Claude' || true
open -a Claude
```

若 Electron 应用仍不继承系统代理，可在保持同一节点的前提下临时显式指定：

```bash
env HTTP_PROXY=http://127.0.0.1:7897 \
    HTTPS_PROXY=http://127.0.0.1:7897 \
    ALL_PROXY=http://127.0.0.1:7897 \
    /Applications/Claude.app/Contents/MacOS/Claude \
    --proxy-server=http://127.0.0.1:7897
```

只在确认端口后替换 `7897`。应用恢复后不需要改节点。

## 最小连通性检查

```bash
curl --proxy http://127.0.0.1:7897 -I --max-time 15 https://chatgpt.com/
curl --proxy http://127.0.0.1:7897 -I --max-time 15 https://claude.ai/
```

这只能证明代理下的 HTTP/TLS 可达；最终仍要以桌面端 UI 和日志为准。

## 缓存原则

优先正常退出并重启应用。只有日志显示渲染器/会话缓存损坏时，才先复制备份后清理 `~/Library/Application Support/Claude/Cache`、`Code Cache`、`GPUCache`、`DawnGraphiteCache`；保留 `Cookies`、`Local Storage`、`IndexedDB` 和会话凭证。
