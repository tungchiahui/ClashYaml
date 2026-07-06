[中文](README-zh_CN.md) | English

# Clash Fallback Configuration

This repository provides a rule-based Clash configuration with regional node groups, automatic latency tests, and fallback groups. It is intended for clients using the **Mihomo (Clash Meta) kernel**.

## 1. Add your subscription

Open [`clash-fallback.yaml`](clash-fallback.yaml), find the following section near the top, and replace `Subscription URL` with your proxy-provider subscription URL:

```yaml
MyProvider:
  url: "Subscription URL"
```

Keep the indentation unchanged. The URL must be a Clash-compatible provider subscription, not a web dashboard or payment page. Treat the completed YAML file as private because it contains your subscription credential.

### Using multiple providers

Add one complete block under `proxy-providers` for each provider. Every provider name and URL must be unique. Adding a short prefix is recommended so nodes with the same name from different providers remain distinguishable:

```yaml
proxy-providers:
  ProviderA:
    url: "Provider A subscription URL"
    type: http
    interval: 86400
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 300
    override:
      additional-prefix: "[A] "
    proxy: 直连

  ProviderB:
    url: "Provider B subscription URL"
    type: http
    interval: 86400
    health-check:
      enable: true
      url: https://www.gstatic.com/generate_204
      interval: 300
    override:
      additional-prefix: "[B] "
    proxy: 直连
```

Repeat the block as needed, using names such as `ProviderC` and a different prefix for each one. Do not add another `proxy-providers:` line. The existing node groups use `include-all: true`, so nodes from all configured providers are included automatically; no proxy-group edits are needed. The prefixes do not interfere with the regional filters as long as the original node names still contain recognizable location keywords. See the official [Mihomo proxy-provider documentation](https://wiki.metacubex.one/en/config/proxy-providers/) for the supported fields.

## 2. Import the configuration into Clash

1. Save `clash-fallback.yaml` after editing it.
2. Open a Mihomo/Clash Meta client, such as Clash Verge Rev, Mihomo Party, or another compatible client.
3. Go to **Profiles** (sometimes named **Subscriptions** or **Configurations**).
4. Choose **Import from file**, then select `clash-fallback.yaml`. You can also drag the file into the Profiles page if your client supports it.
5. Activate the imported profile and wait for the provider and rule sets to finish downloading.
6. Enable **System Proxy** for normal proxy use. Enable **TUN mode** only when you need traffic from applications that do not follow the system proxy.

If the profile fails to load, first verify the subscription URL and confirm that the client uses a recent Mihomo/Clash Meta kernel. The configuration uses rule providers and `.mrs` rule files that may not work with the discontinued original Clash core.

## 3. Set the routing groups

Open the client's **Proxies** page and set each business routing group as follows. The label `日本-故转` means the Japanese fallback group.

| Routing group | Select |
| --- | --- |
| AI | `日本-故转` |
| Stream Media | `日本-故转` |
| GitHub | `日本-故转` |
| Reddit | `日本-故转` |
| Nvidia | `直连` |
| Apple | `直连` |
| Microsoft | `直连` |
| Games | `直连` |
| Crypto | `台湾-故转` |
| Test | `日本-故转` |
| Block | `拒绝` |
| 国外 | `日本-故转` |
| 国内 | `直连` |
| 其他 | `日本-故转` |

In short: Nvidia, Apple, Microsoft, and Games connect directly; Crypto uses Taiwan; Block is rejected; domestic traffic connects directly; all remaining business groups use Japan.

## 4. Choose the manual nodes used by fallback groups

For each regional `xx-故转` group, select its matching `xx-手动` group:

| Fallback group | Select |
| --- | --- |
| 香港-故转 | `香港-手动` |
| 澳门-故转 | `澳门-手动` |
| 台湾-故转 | `台湾-手动` |
| 日本-故转 | `日本-手动` |
| 韩国-故转 | `韩国-手动` |
| 新加坡-故转 | `新加坡-手动` |
| 美国-故转 | `美国-手动` |
| 其他-故转 | `其他-手动` |

Then open every `xx-手动` group and select a node with relatively low latency and good stability. Run the client's latency test several times rather than choosing only the lowest result from a single test. Prefer a node whose delay varies little and which does not time out or lose connections. At minimum, configure `日本-手动` and `台湾-手动`, because the routing setup above actively uses them.

Your selections are retained by the configuration's `profile.store-selected` setting. If a provider changes or removes a node, choose a replacement in the corresponding manual group.

## Notes

- `直连` means **DIRECT** and `拒绝` means **REJECT**.
- `xx-自动` selects a node using periodic latency tests.
- `xx-故转` is a fallback group. With `xx-手动` selected, it uses the node chosen in that manual group; you can switch it to `xx-自动` if you prefer automatic selection.
- The configuration exposes its controller on `0.0.0.0:9090` with the default secret `123456`. If the controller is reachable from an untrusted network, change the secret and restrict LAN access.
