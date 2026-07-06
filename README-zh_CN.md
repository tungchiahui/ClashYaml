# Clash Fallback 配置说明

中文 | [English](README.md)

这是一个基于规则分流的 Clash 配置，内置地区节点组、自动延迟测试和故障转移组，适用于使用 **Mihomo（Clash Meta）内核**的客户端。

## 1. 填写订阅链接

打开 [`clash-fallback.yaml`](clash-fallback.yaml)，找到文件开头的以下内容，把 `订阅链接` 替换成自己的机场订阅链接：

```yaml
MyProvider:
  url: "订阅链接"
```

请保持原有缩进不变。这里需要填写兼容 Clash 的节点订阅地址，不是机场官网、用户中心或付款页面。填写后的 YAML 会包含你的订阅凭据，请勿公开分享或提交到公共仓库。

### 添加多个服务商

有几个服务商，就在 `proxy-providers` 下面放几个完整的服务商配置块。每个服务商的名称和订阅地址都要唯一。建议给各家的节点添加不同前缀，防止不同服务商存在同名节点时难以区分：

```yaml
proxy-providers:
  ProviderA:
    url: "服务商 A 的订阅链接"
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
    url: "服务商 B 的订阅链接"
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

需要添加更多服务商时，继续复制完整配置块，并改成 `ProviderC` 等唯一名称和不同前缀即可。不要重复写第二个 `proxy-providers:`。现有节点组使用了 `include-all: true`，会自动纳入所有服务商的节点，不需要再修改代理组。只要原节点名称仍包含日本、台湾、`JP`、`TW` 等地区关键词，新增的前缀就不会影响地区筛选。各字段的完整说明可查看 [Mihomo 官方代理集合文档](https://wiki.metacubex.one/config/proxy-providers/)。

## 2. 导入 Clash 客户端

1. 修改完成后，保存 `clash-fallback.yaml`。
2. 打开使用 Mihomo/Clash Meta 内核的客户端，例如 Clash Verge Rev、Mihomo Party 或其他兼容客户端。
3. 进入**订阅**、**配置**或 **Profiles** 页面。
4. 选择**从文件导入**，然后选中 `clash-fallback.yaml`；部分客户端也支持把文件直接拖进该页面。
5. 启用刚导入的配置，等待节点订阅和规则集下载完成。
6. 日常使用时打开**系统代理**；只有在需要接管不遵循系统代理的软件流量时，再开启 **TUN 模式**。

如果配置无法加载，请先检查订阅链接是否有效，再确认客户端使用的是较新的 Mihomo/Clash Meta 内核。本配置使用了规则提供器和 `.mrs` 规则文件，已经停止维护的旧版 Clash 内核可能无法兼容。

## 3. 设置业务分流组

进入客户端的**代理**页面，按下表逐项选择。配置里的正式组名是 `日本-故转`，也就是日本节点的故障转移组。

| 分流组 | 选择 |
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

简单来说：Nvidia、Apple、Microsoft 和 Games 直连；Crypto 使用台湾节点；Block 拒绝连接；国内流量直连；其余业务分流组全部使用日本节点。

## 4. 给各地区故转组指定手动节点

继续在**代理**页面操作，把每个 `xx-故转` 组切换到对应的 `xx-手动`：

| 故转组 | 选择 |
| --- | --- |
| 香港-故转 | `香港-手动` |
| 澳门-故转 | `澳门-手动` |
| 台湾-故转 | `台湾-手动` |
| 日本-故转 | `日本-手动` |
| 韩国-故转 | `韩国-手动` |
| 新加坡-故转 | `新加坡-手动` |
| 美国-故转 | `美国-手动` |
| 其他-故转 | `其他-手动` |

然后依次打开每个 `xx-手动` 组，从中选择一个延迟较低、稳定性较好的节点。建议连续进行几次延迟测试，不要只看单次最低值；优先选择延迟波动小、不会超时或频繁断连的节点。至少要设置好 `日本-手动` 和 `台湾-手动`，因为上面的分流方案会实际使用这两个地区。

配置已启用 `profile.store-selected`，客户端通常会记住选择。如果机场更换或删除了节点，请回到对应的手动组重新选择。

## 补充说明

- `直连` 表示 **DIRECT**，`拒绝` 表示 **REJECT**。
- `xx-自动` 会通过定期延迟测试自动选择节点。
- `xx-故转` 是故障转移组。选择 `xx-手动` 后，它会使用你在对应手动组中指定的节点；如果希望自动选节点，也可以改选 `xx-自动`。
- 配置默认将控制器开放在 `0.0.0.0:9090`，密码为 `123456`。如果设备处于不可信的局域网中，建议修改密码并限制局域网访问。
