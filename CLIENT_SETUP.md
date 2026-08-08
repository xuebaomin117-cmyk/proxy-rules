# 客户端接入与 Codex 操作说明

更新日期：2026-07-29

## 这份文档的用途

这份文档用于交给其他电脑上的 Codex。请客户端 Codex 先阅读本文，再检查本机实际配置并完成接入。

仓库：

<https://github.com/xuebaomin117-cmyk/proxy-rules>

目标是让不同电脑上的 Surge、Clash Verge Rev 或 Shadowrocket 共用本仓库的个人分流规则，同时不把机场订阅、节点、密码、证书等隐私内容上传到 GitHub。

## 核心原则

机场订阅和个人规则必须分开：

```text
机场订阅：节点、基础策略组、机场规则
GitHub：个人域名规则
客户端本地适配层：把 GitHub 规则映射到本机策略组
```

GitHub 只同步规则内容，不会自动修改每台电脑的客户端配置。

每台客户端首次都需要安装一次“本地适配层”。完成后：

- 更新机场订阅不会删除个人规则。
- GitHub 规则更新后，客户端可自动或手动刷新。
- 不需要每次在每台电脑上重复添加域名。

## 当前公共规则

### Apple 强制直连

规则地址：

<https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/Apple-Direct.list>

目标策略：`DIRECT`

### OpenAI/ChatGPT

规则地址：

<https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/OpenAI.list>

目标策略组：`OpenAI`

### 通用代理

规则地址：

<https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/Proxy.list>

目标策略组：`Proxies`

当前包含 `DOMAIN-SUFFIX,linux.do`，它会匹配 `linux.do` 根域名及其全部子域名。

## 客户端 Codex 的安全边界

执行前必须：

1. 确认当前实际启用的客户端、配置文件和策略组名称。
2. 备份将要修改的本地配置。
3. 不输出或上传机场订阅 URL、节点服务器、端口、密码、密钥、证书及认证信息。
4. 不把完整 Surge 或 Clash 配置提交到本公开仓库。
5. 不凭空创建或引用不存在的策略组。
6. 修改完成后先做配置检查，再重新载入客户端。

## Mac：Surge 本地配置副本

### 使用模式

Mac 上的 Surge 不直接把 iCloud 目录作为配置存储路径。

每台 Mac 使用自己的 Surge 本地配置目录。iCloud 或其他私人目录只承担配置副本传递作用，最终需要把配置复制到该 Mac 的 Surge 本地目录，再重新载入。

常见本地目录：

```text
~/Library/Application Support/Surge/Profiles
```

客户端 Codex 不应仅凭这个默认路径直接覆盖文件。必须先确认 Surge 当前实际使用的配置名称和目录。

### 策略组检查

当前映射需要以下策略组：

```text
OpenAI
Proxies
```

如果本机配置中的名称不同，应先向用户报告实际策略组名称并确认映射关系。

`DIRECT` 是内置直连策略，不需要创建。

### Surge 规则映射

在本机实际使用的完整配置文件 `[Rule]` 段中加入以下内容。前三条是 ChatGPT/Codex 进程级兜底，后面三条是共享远程规则映射：

```ini
# Personal shared rules
PROCESS-NAME,/Applications/ChatGPT.app/,OpenAI
PROCESS-NAME,ChatGPT,OpenAI
PROCESS-NAME,codex,OpenAI
RULE-SET,https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/Apple-Direct.list,DIRECT,update-interval=86400,extended-matching
RULE-SET,https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/OpenAI.list,OpenAI,update-interval=86400,extended-matching
RULE-SET,https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/Proxy.list,Proxies,update-interval=86400,extended-matching
```

这些规则必须位于机场通用规则和 `FINAL` 之前。

进程规则说明：

- WebSocket/WSS 不需要单独的 Surge 协议规则；连接仍由普通路由规则决定。
- `PROCESS-NAME,ChatGPT,OpenAI` 强制 ChatGPT 主进程走 `OpenAI`。
- `PROCESS-NAME,codex,OpenAI` 强制名为 `codex` 的子进程走 `OpenAI`。
- `/Applications/ChatGPT.app/` 是 App Bundle 路径前缀匹配写法，官方标注为 Surge Mac 6.0+。
- 在 Surge Mac 6.0 以下版本，主要依靠 `ChatGPT` 和 `codex` 两条文件名规则；保留 App Bundle 条目可兼容未来升级。
- 这些规则只适用于 Surge Mac，不应加入供 Clash Verge Rev、Shadowrocket 共用的 `rules/OpenAI.list`。
- 副作用是 ChatGPT 应用和 `codex` 进程的全部网络请求都会走 `OpenAI`，不仅是远程控制 WebSocket。

如当前网络无法直接读取 GitHub Raw，可在个人规则之前加入：

```ini
DOMAIN-SUFFIX,githubusercontent.com,Proxies
DOMAIN-SUFFIX,github.com,Proxies
```

### 防止机场订阅覆盖

不要只修改带 `#!MANAGED-CONFIG` 的机场订阅文件，因为机场刷新后修改可能消失。

推荐保持两个本地文件：

```text
WestData.conf：机场更新产生的基础配置
My-Surge.conf：Surge 实际使用的完整本地配置
```

`My-Surge.conf` 应由本地生成脚本从 `WestData.conf` 重新构建，并在 `[Rule]` 开头持续插入上面的个人规则。生成脚本、`My-Surge.conf` 和 `WestData.conf` 都只能保存在私人目录。

如果本机已经存在相同的完整配置生成机制，应合并规则，不要另建一套冲突流程。

### Surge 执行步骤

1. 确认当前启用的配置。
2. 备份本机 `My-Surge.conf` 或等效完整配置。
3. 检查 `OpenAI` 和 `Proxies` 策略组存在。
4. 将三条远程规则映射加入完整本地配置。
5. 同步更新本机的配置生成脚本，确保下次生成不会丢失域名规则和三条进程兜底规则。
6. 将修改后的配置放入 Surge 本地配置目录。
7. 在 Surge 中重新载入该配置。
8. 手动刷新外部规则。
9. 访问 `chatgpt.site` 和 `linux.do`，通过请求记录确认：
   - `chatgpt.site` 命中 `OpenAI.list → OpenAI`。
   - `linux.do` 命中 `Proxy.list → Proxies`。
   - ChatGPT/Codex 后台连接命中 `PROCESS-NAME → OpenAI`，没有回落为 `DIRECT`。

## Windows：Clash Verge Rev 2.5.2

### 不要直接编辑订阅 YAML

直接修改机场下载的 YAML 会在订阅更新时被覆盖。

应使用当前订阅卡片的“扩展脚本”。脚本独立于订阅原文件，并在每次生成最终配置时加入个人规则。

### 策略组检查

默认使用：

```text
OpenAI
Proxies
```

如果 Clash 中显示的名称不同，必须把脚本中的目标组常量改为本机真实名称，包括 Emoji、空格和大小写。

### Clash Verge Rev 扩展脚本

在“订阅”页面找到当前启用的订阅，右键选择“编辑扩展脚本”或含义相同的入口。

如果已有扩展脚本，先备份并把以下逻辑合并进去，不要直接覆盖其他有效功能。

```javascript
const OPENAI_PROVIDER = "personal-openai";
const GENERAL_PROVIDER = "personal-proxy";
const OPENAI_TARGET_GROUP = "OpenAI";
const GENERAL_TARGET_GROUP = "Proxies";
const OPENAI_RULE_URL =
  "https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/OpenAI.list";
const GENERAL_RULE_URL =
  "https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/Proxy.list";

function main(config) {
  const groups = Array.isArray(config["proxy-groups"])
    ? config["proxy-groups"]
    : [];

  [OPENAI_TARGET_GROUP, GENERAL_TARGET_GROUP].forEach(function (target) {
    const groupExists = groups.some(function (group) {
      return group && group.name === target;
    });

    if (!groupExists) {
      throw new Error(
        "没有找到策略组：" + target +
        "。请把目标策略组常量改成 Clash 中真实存在的策略组名称。"
      );
    }
  });

  if (
    !config["rule-providers"] ||
    typeof config["rule-providers"] !== "object"
  ) {
    config["rule-providers"] = {};
  }

  config["rule-providers"][OPENAI_PROVIDER] = {
    type: "http",
    behavior: "classical",
    format: "text",
    url: OPENAI_RULE_URL,
    path: "./ruleset/personal-openai.list",
    interval: 86400,
    proxy: OPENAI_TARGET_GROUP
  };

  config["rule-providers"][GENERAL_PROVIDER] = {
    type: "http",
    behavior: "classical",
    format: "text",
    url: GENERAL_RULE_URL,
    path: "./ruleset/personal-proxy.list",
    interval: 86400,
    proxy: GENERAL_TARGET_GROUP
  };

  const personalRules = [
    "RULE-SET," + OPENAI_PROVIDER + "," + OPENAI_TARGET_GROUP,
    "RULE-SET," + GENERAL_PROVIDER + "," + GENERAL_TARGET_GROUP
  ];

  const oldRules = Array.isArray(config.rules) ? config.rules : [];

  config.rules = personalRules.concat(
    oldRules.filter(function (rule) {
      return personalRules.indexOf(rule) === -1;
    })
  );

  return config;
}
```

### Clash Verge Rev 执行步骤

1. 备份当前配置或 Clash Verge Rev 设置。
2. 确认当前启用的订阅。
3. 检查两个目标策略组的真实名称。
4. 添加或合并订阅扩展脚本。
5. 保存并重新激活订阅。
6. 确认存在规则集：
   - `personal-openai`
   - `personal-proxy`
7. 确认最终规则前部存在：

```text
RULE-SET,personal-openai,OpenAI
RULE-SET,personal-proxy,Proxies
```

8. 访问 `chatgpt.site` 和 `linux.do`，在连接页面核对命中策略。
9. 手动更新一次机场订阅并重新激活。
10. 再次确认两个个人规则集仍然存在，以验证抗覆盖能力。

## Shadowrocket

在机场通用规则、`GEOIP` 和 `FINAL` 之前加入：

```ini
RULE-SET,https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/Apple-Direct.list,DIRECT
RULE-SET,https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/OpenAI.list,OpenAI
RULE-SET,https://raw.githubusercontent.com/xuebaomin117-cmyk/proxy-rules/main/rules/Proxy.list,Proxies
```

必须先确认 Shadowrocket 中存在 `OpenAI` 和 `Proxies` 策略组，或将它们替换为用户确认的实际策略组。

## 如何判断以后是否需要重新设置

只修改 `rules/*.list` 中的域名时：

- 不需要重新安装客户端适配层。
- 客户端刷新远程规则即可。

以下情况需要客户端 Codex重新检查：

- 策略组名称改变。
- 更换机场订阅或客户端。
- Surge 的完整本地配置被重新生成但生成脚本未保留个人映射。
- Clash Verge Rev 的扩展脚本被删除或更换。
- GitHub Raw 下载失败。

## 客户端 Codex 完成后报告

请向用户报告：

- 客户端名称和版本。
- 当前启用的配置显示名称，不要报告订阅 URL。
- 实际使用的 `OpenAI` 和通用代理策略组名称。
- 三个 GitHub 规则集是否加载成功。
- `chatgpt.site` 和 `linux.do` 的实际命中结果。
- 更新机场订阅后个人规则是否仍然存在。
- 是否修改了本地生成脚本或扩展脚本。

禁止在报告中显示订阅 URL、节点地址、端口、密码、密钥和证书。
