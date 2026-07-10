# Personal Proxy Rules

用于在 Surge 和 Shadowrocket（小火箭）之间共享个人分流规则。

本仓库只保存域名匹配规则，不包含机场订阅地址、代理节点、密码、证书或其他私密配置。

## 文件

- `rules/OpenAI.list`：OpenAI、ChatGPT 和 `chatgpt.site` 相关域名。
- `rules/Apple-Direct.list`：需要强制直连的 Apple 服务域名。
- `clients/Surge.snippet.conf`：Surge 引用示例。
- `clients/Shadowrocket.snippet.conf`：小火箭引用示例。

## Surge

将下面两行放在机场规则和 `FINAL` 之前：

```ini
RULE-SET,https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/Apple-Direct.list,DIRECT,update-interval=86400,extended-matching
RULE-SET,https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/OpenAI.list,OpenAI,update-interval=86400,extended-matching
```

`OpenAI` 必须是当前 Surge 配置中存在的策略组。

## Shadowrocket

将下面两行放在 `GEOIP` 和 `FINAL` 之前：

```ini
RULE-SET,https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/Apple-Direct.list,DIRECT
RULE-SET,https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/OpenAI.list,OpenAI
```

当前示例假定小火箭和 Surge 都存在名为 `OpenAI` 的代理策略组。

## 更新规则

修改 `rules/*.list` 后提交到 `main` 分支。客户端会根据自身的外部资源刷新机制获取新版本，也可以在客户端中手动刷新外部资源。

## 注意

- 不要向仓库提交完整机场配置或订阅 URL。
- 规则从上到下匹配，个人规则必须位于通用规则和 `FINAL` 之前。
- `DOMAIN-SUFFIX,chatgpt.site` 同时覆盖根域名和所有子域名。
