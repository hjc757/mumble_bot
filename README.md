# mumble_bot

一个运行在 [Mumble](https://www.mumble.info/) 服务器上的音乐机器人，fork 自 [azlux/botamusique](https://github.com/azlux/botamusique)。本分支针对中国大陆网络环境做了调整：移除了国内无法直接访问的来源，补充了中文命令说明。

## 功能

1. **本地音乐播放**：播放本地文件夹内的音频文件，支持通过网页界面上传、浏览音乐库、编辑标签。
2. **国内网络电台**：内置可直连的央广 HLS 电台（中国之声、经济之声），也可在配置中替换为其他可访问的电台流；聊天命令与网页界面均可添加电台地址。
3. **Flask 网页控制台**：播放列表管理、音乐库管理（上传、浏览文件、编辑标签等）。
4. **命令系统**：命令与机器人发言文案均可自定义，支持命令部分匹配，中文说明见 `!help`。
5. **闪避（Ducking）**：有人说话时自动降低音乐音量。
6. **多语言**：文案见 `lang/` 目录，默认 `zh_CN`（简体中文）。

> 相比上游，本分支**已移除**以下在国内难以使用的来源：
> - YouTube / SoundCloud 的链接与播放列表添加命令；
> - radio-browser.info 在线电台检索与按 ID 播放命令。
>
> 底层兼容旧播放列表中已有的链接条目，历史数据仍可正常展示。

## 依赖

- Python 3.10 及以上；
- `ffmpeg`（用于拉流与转码）；
- Opus 编解码库：`apt install libopus0`；
- `libmagic`：`apt install libmagic1`（或安装 `python-magic-bin`）；
- `git`（安装 pymumble 时需要，从 GitHub 直接拉取）。

## 安装（源码方式）

```bash
git clone https://github.com/hjc757/mumble_bot.git
cd mumble_bot
python3 -m venv venv
venv/bin/pip install wheel
venv/bin/pip install -r requirements.txt
```

依赖中的 pymumble 取自 [hjc757/pymumble](https://github.com/hjc757/pymumble)：支持 Mumble 1.3~1.6 协议与 TLS 1.2。
PyPI 上的 `pymumble` 1.6.1 仅支持 Mumble 1.2，且依赖 Python 3.12 已移除的 `ssl.wrap_socket`，**不要使用**。

前端资源若需重新构建：

```bash
(cd web && npm install && npm run build)
```

## 配置

复制示例配置为实际配置文件，并按需修改：

```bash
cp configuration.example.ini configuration.ini
```

注意：**不要直接修改** `configuration.default.ini`；`configuration.ini` 中未定义的项会回退到默认值。

常用配置段：

- `[server]`：Mumble 服务器地址、端口、频道、接入令牌、证书。
- `[bot]`：机器人名称、自我介绍、音乐目录、临时目录、默认音量、默认语言等。
- `[webinterface]`：网页控制台开关、监听地址与端口、登录账号密码。
- `[commands]`：各命令的别名（例如 `help = helpme`，机器人即响应 `!helpme`）。
- `[radio]`：默认电台列表，格式 `名称 = 流地址 "备注"`。失效时可在此替换为其他可访问的电台流，也可在频道内用 `!radio <地址>` 临时添加。
- `[debug]`：ffmpeg / pymumble 调试输出。

最小可用示例：

```ini
[server]
host = 127.0.0.1
port = 64738

[bot]
music_folder = music_folder/
tmp_folder = /tmp/
language = zh_CN

[webinterface]
enabled = True
listening_addr = 127.0.0.1
listening_port = 8181
```

> 自动版本检查默认关闭（`auto_check_update = False`）：上游的更新源在国内不可达，本分支以源码方式自行维护。

## 运行

```bash
venv/bin/python mumbleBot.py --config configuration.ini
```

或通过命令行参数指定服务器：

```bash
venv/bin/python mumbleBot.py -s HOST -u BOTNAME -P PASSWORD -p PORT -c CHANNEL -C /path/to/cert.pem
```

## 命令

在频道内发送 `!help` 可查看完整命令清单（中文）。默认所有命令以 `!` 开头，主要分组：

- **播放控制**：播放 / 暂停 / 停止 / 下一首 / 音量 / 播放模式 / 闪避 / 回到默认频道；
- **播放列表**：当前曲目、队列、按标签加入、本地文件与目录、电台、候选短列表、删除、单曲循环、随机打乱；
- **音乐库**：搜索、列出文件、打标签 / 去标签、按标签查找、删除；
- **其他**：加入发言者所在频道、修改网页登录密码。

## 常见问题

- `ImportError: libtiff.so.5`：缺少图像库，`apt install libtiff5`。
- `Could not find opus library`：`apt install libopus0`。
- macOS 提示找不到 `libmagic`：`brew install libmagic`，或改用 `python-magic-bin`。
- 音乐文件较多（>1000）时启动较慢：可在 `[bot]` 设置 `refresh_cache_on_startup=False`，再通过 `!rescan` 命令或网页按钮手动扫描。
- Alpine 编译 Pillow 需额外依赖：`python3-dev musl-lib libmagic jpeg-dev zlib-dev gcc`。

## 许可

沿用上游授权，见 `LICENSE`。
