# Root-My-Galaxy-Payloads 配置说明

## 配置文件

配置文件路径: `support/targets-v3.json`

应用通过 GitHub API 获取 main 分支的最新 commit SHA，然后按 commit 固定下载以下文件：
- `support/targets-v3.json` - 机型支持配置
- `artifacts/<profileId>/cve-2026-43499-app.so` - exploit payload
- `kernelsu/<file>` - KernelSU 文件

## 配置格式

```json
{
  "schemaVersion": 3,
  "payloads": [
    {
      "payloadId": "唯一ID",
      "displayName": "显示名称",
      "models": ["设备型号1", "设备型号2"],
      "kernelVersions": ["内核版本"],
      "status": "verified|unverified|failed",
      "requiresFreshP0Session": false,
      "exploit": {
        "url": "https://raw.githubusercontent.com/q1016134519/Root-My-Galaxy-Payloads/main/artifacts/xxx/cve-2026-43499-app.so",
        "size": 104128
      },
      "kernelsu": {
        "url": "https://raw.githubusercontent.com/q1016134519/Root-My-Galaxy-Payloads/main/kernelsu/ksud-xxx-kdp",
        "size": 6407096
      }
    }
  ]
}
```

## status 字段说明

| 值 | 颜色 | 含义 |
|----|------|------|
| `verified` | 绿色 | 已验证，此 payload 已测试确认可用 |
| `unverified` | 黄色 | 未验证，此 payload 尚未经过测试 |
| `failed` | 红色 | 未通过，此 payload 已测试但不可用 |

不指定 `status` 时默认为 `unverified`。

## 部署步骤

1. 确保 GitHub 仓库 `q1016134519/Root-My-Galaxy-Payloads` 已创建
2. 将 `support/targets-v3.json` 推送到 main 分支
3. 确保 artifacts 和 kernelsu 目录中的 payload 文件路径与 JSON 中的 URL 一致
4. 文件大小必须精确匹配，否则下载会验证失败
