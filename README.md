# binance-trade-bot
> 自动化加密货币交易机器人

![github](https://img.shields.io/github/workflow/status/edeng23/binance-trade-bot/binance-trade-bot)
![docker](https://img.shields.io/docker/pulls/edeng23/binance-trade-bot)
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/edeng23/binance-trade-bot)

[![Deploy to DO](https://mp-assets1.sfo2.digitaloceanspaces.com/deploy-to-do/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/coinbookbrasil/binance-trade-bot/tree/master&refcode=a076ff7a9a6a)


## 为什么使用这个机器人？

这个项目的灵感来自观察到的一个现象：大多数加密货币的走势相似。当某个币种上涨时，其他币种也通常会上涨；同理，某个币种下跌时，其他币种也会随之下跌——虽然并非总是如此。而且，所有币种通常会跟随比特币的走势，差异主要表现在它们的波动周期上。

因此，如果币种的波动是相互关联的，那么利用上涨的币种兑换下跌的币种，当局势反转时再兑换回来，可能是一个不错的交易策略。

## 如何运作？

本交易机器人运行在 Binance 市场平台上，利用市场中的桥梁货币进行交易，因为 Binance 并不是每个币对都支持直接交易。默认的桥梁货币是 Tether（USDT），它是稳定的且几乎支持所有币种。

<p align="center">
  币种A → USDT → 币种B
</p>

机器人利用这一观察到的规律，始终从“强势”币种兑换为“弱势”币种，假设某个币种最终会反弹。然后，再将其兑换回原来的币种，最终持有更多的原始币种。在此过程中，还会考虑交易手续费。

<div align="center">
  <p><b>币种A</b> → USDT → 币种B</p>
  <p>币种B → USDT → 币种C</p>
  <p>...</p>
  <p>币种C → USDT → <b>币种A</b></p>
</div>

机器人在配置的币种列表中循环跳跃，但不会重复兑换同一币种，除非这样做能够带来利润。这样，我们永远不会损失某种币。唯一的风险是，某个币种可能突然暴跌，触发我们的反向贪婪算法。

## Binance 配置

-   创建一个 [Binance 账户](https://www.binance.com/en/register?ref=13222128)（附带我的推荐链接，感谢您的支持）。
-   启用双重身份验证。
-   创建一个新的 API 密钥。
-   获取一种加密货币。如果币种符号不在默认列表中，可以手动添加。

## 工具配置

### 安装 Python 依赖

在终端中运行以下命令：`pip install -r requirements.txt`。

### 创建用户配置文件

根据 `.user.cfg.example` 文件创建一个名为 `user.cfg` 的文件，并添加您的 API 密钥和当前选择的币种。

**配置文件包括以下字段：**

-   **api_key** - Binance 账户中生成的 API 密钥。
-   **api_secret_key** - Binance 账户中生成的 API 密钥。
-   **current_coin** - 起始币种，应该是支持的币种之一。如果您想从桥梁币种开始，请留空，机器人会从支持的币种列表中随机选择一个并购买。
-   **bridge** - 桥梁货币选择。不同的桥梁币种支持不同的币对。例如，可能存在某个特定币种/USDT 交易对，但没有该币种/BUSD 交易对。
-   **tld** - 'com' 或 'us'，取决于您所在的地区，默认为 'com'。
-   **hourToKeepScoutHistory** - 控制在数据库中保留多少小时的历史数据，超过指定时间后数据会被删除。
-   **scout_sleep_time** - 控制每次数据采集间隔多少秒。
-   **use_margin** - 如果使用 margin 交易，请设置为 'yes'，否则为 'no'。
-   **scout_multiplier** - 控制在当前币种比率与历史比率之间差异的乘数。设置较大的值会使机器人等待较大的波动再进行交易。
-   **scout_margin** - 每次交易的最低币种涨幅百分比。例如，0.8 表示 0.1% 手续费时，`scout_multiplier` 为 5。
-   **strategy** - 选择交易策略，更多信息见 [`binance_trade_bot/strategies`](binance_trade_bot/strategies/README.md)。
-   **buy_timeout/sell_timeout** - 设置买卖订单的超时等待时间，超过时间则自动取消。

#### 环境变量

`user.cfg` 文件中的所有配置项也可以通过环境变量进行配置。

```
CURRENT_COIN_SYMBOL:
SUPPORTED_COIN_LIST: "XLM TRX ICX EOS IOTA ONT QTUM ETC ADA XMR DASH NEO ATOM DOGE VET BAT OMG BTT"
BRIDGE_SYMBOL: USDT
API_KEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
API_SECRET_KEY: NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j
SCOUT_MULTIPLIER: 5
SCOUT_SLEEP_TIME: 1
TLD: com
STRATEGY: default
BUY_TIMEOUT: 0
SELL_TIMEOUT: 0
```

### 使用 BNB 支付手续费

您可以使用 [BNB 代币支付 Binance 平台的手续费](https://www.binance.com/en/support/faq/115000583311-Using-BNB-to-Pay-for-Fees)，这将使所有手续费减少 25%。为了支持这一功能，机器人会自动执行以下操作：
-   自动检测您是否启用了 BNB 支付手续费。
-   确保您的账户中有足够的 BNB 来支付交易的手续费。
-   在计算交易阈值时，考虑到 BNB 币种的优惠。

### 使用 Apprise 进行通知

Apprise 允许机器人将通知发送到各大流行的通知服务，例如：Telegram、Discord、Slack、Amazon SNS、Gotify 等。

要设置此功能，请在配置目录中创建一个 `apprise.yml` 文件。

如果您希望运行 Telegram 机器人，请参考 [Telegram 官方文档](https://core.telegram.org/bots)了解更多信息。

### 运行

```shell
python -m binance_trade_bot
```

### 使用 Docker

官方 Docker 镜像可以从 [这里](https://hub.docker.com/r/edeng23/binance-trade-bot) 获取，并会随着新更新自动同步。

```shell
docker-compose up
```

如果只想启动 SQLite 浏览器，可以使用：

```shell
docker-compose up -d sqlitebrowser
```

## 回测

您可以在历史数据上测试机器人，以查看其表现。

```shell
python backtest.py
```

您可以修改该文件来测试不同的设置和时间段。

## 开发

为了确保您的代码在提交前格式正确，请安装 [pre-commit](https://pre-commit.com/)：

```shell
pip install pre-commit
pre-commit install
```

如果您希望贡献一个新的策略，请参考 [添加新策略](binance_trade_bot/strategies/README.md)。

## 相关项目

感谢一些才华横溢的开发者，现在有一个 [Telegram 机器人](https://github.com/lorcalhost/BTB-manager-telegram) 可以用来远程管理这个项目。

## 支持项目

<a href="https://www.buymeacoffee.com/edeng" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## 加入讨论

-   **Discord**: [邀请链接](https://discord.gg/m4TNaxreCN)

## 常见问题

常见问题的答案可以在我们的 Discord 服务器中找到，具体频道中有详细的解答。

<p align="center">
  <img src = "https://usercontent2.hubstatic.com/6061829.jpg">
</p>

## 免责声明

本项目仅供信息参考，不能作为法律、税务、投资、金融或其他建议。如果您打算使用真实资金，务必自行承担风险。

在任何情况下，我都不对任何损失或损害负责，包括但不限于直接或间接损失。