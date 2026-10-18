# Windows 操作参考

## 只读检查

```powershell
netsh winhttp show proxy
Get-NetIPConfiguration
Get-DnsClientServerAddress
Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' |
  Select-Object ProxyEnable,ProxyServer,AutoConfigURL,ProxyOverride
```

确认 Ninja 实际 HTTP/Mixed 端口后，再检查 Windows 用户代理是否一致。WinHTTP 代理和用户代理是两套配置，不要混为一谈。

## 应用重启与显式代理

```powershell
Stop-Process -Name Claude -ErrorAction SilentlyContinue
Start-Process "$env:LOCALAPPDATA\Programs\Claude\Claude.exe" `
  -ArgumentList '--proxy-server=http://127.0.0.1:7897'
```

先正常退出；只有进程无响应时才加 `-Force`。路径和端口以本机实际安装位置、Ninja UI 为准。ChatGPT/Codex 同理，只在应用不继承系统代理时使用 `--proxy-server`，先用任务管理器或 `Get-Process` 定位实际可执行文件。

## 最小连通性检查

```powershell
curl.exe --proxy http://127.0.0.1:7897 -I --max-time 15 https://chatgpt.com/
curl.exe --proxy http://127.0.0.1:7897 -I --max-time 15 https://claude.ai/
```

这只能证明代理下的 HTTP/TLS 可达；最终仍要以桌面端 UI 和日志为准。

## 安全修复顺序

先重启 Ninja 内核和桌面应用，再检查 DNS/IPv6。不要先执行 `netsh winsock reset`、`netsh int ip reset`、删除代理注册表或清空应用数据；这些操作会影响所有应用并需要重启。
