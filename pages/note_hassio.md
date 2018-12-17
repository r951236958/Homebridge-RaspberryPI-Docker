# HASSIO備忘錄

---

## Mac安裝方式

1. 使用Python 3.5(或以上)

```pip3 install homeassistant```

- 自動啟動

```
$ hass --script macos install
Home Assistant has been installed.         Open it here: < http://localhost:8123 >
```

- 移除安裝
```
$ hass --script macos uninstall
Home Assistant has been uninstalled.
```

### 小抄-指令用法

```
usage: hass [-h] [--version] [-c path_to_config_dir] [--demo-mode] [--debug]
            [--open-ui] [--skip-pip] [-v] [--pid-file path_to_pid_file]
            [--log-rotate-days LOG_ROTATE_DAYS] [--log-file LOG_FILE]
            [--log-no-color] [--runner] [--script ...] [--daemon]

Home Assistant: Observe, Control, Automate.

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  -c path_to_config_dir, --config path_to_config_dir
                        Directory that contains the Home Assistant
                        configuration
  --demo-mode           Start Home Assistant in demo mode
  --debug               Start Home Assistant in debug mode
  --open-ui             Open the webinterface in a browser
  --skip-pip            Skips pip install of required packages on startup
  -v, --verbose         Enable verbose logging to file.
  --pid-file path_to_pid_file
                        Path to PID file useful for running as daemon
  --log-rotate-days LOG_ROTATE_DAYS
                        Enables daily log rotation and keeps up to the
                        specified days
  --log-file LOG_FILE   Log file to write to. If not set, CONFIG/home-
                        assistant.log is used
  --log-no-color        Disable color logs
  --runner              On restart exit with code 100
  --script ...          Run one of the embedded scripts
  --daemon              Run Home Assistant as daemon
  ```


```
hass --script [COMMAND]

COMMAND:
benchmark, influxdb_import, auth, macos, ensure_config, check_config, influxdb_migrator, keyring, db_migrator, credstash
```

2. 腳本自動安裝

- 執行指令, 就會在Docker開始自動建立

```
$ curl -sL https://raw.githubusercontent.com/home-assistant/hassio-build/master/install/hassio_install | bash -s
```

## Docker安裝Volumio

還沒試過, 先寫起來放

>參考文章:

1. [Volumio_Docker映像檔](https://hub.docker.com/r/jbonjean/volumio/)
2. [其他人在GitHub](https://github.com/titilambert/docker-volumio2)
3. [其他參考](https://bbs.hassbian.com/thread-2083-1-1.html)

## 隨身碟預載設定檔

把hassio燒錄進SD卡之後, 可以準備一個隨身碟, 格式化後裝置名稱叫```CONFIG```
裡面可以建立幾個預載使用(WiFi, SSH...)的檔案夾, 細則看官方文檔說明

[預載設置](https://github.com/home-assistant/hassos/blob/dev/Documentation/configuration.md)

[網路](https://github.com/home-assistant/hassos/blob/dev/Documentation/network.md)

## HomeAssistant設置

 - [HA中文文檔參考](https://www.hachina.io/docs/321.html)

 - [HA配置文檔](https://www.home-assistant.io/getting-started/configuration/)

 - [HA HomeKit](https://www.home-assistant.io/components/homekit_controller/)

 - [iTunes](https://www.home-assistant.io/components/media_player.itunes/)

 - [Hass.io Add-ons](https://www.home-assistant.io/addons/)

---

### Spotify

設置方式有點小麻煩與囉唆, 稍微解釋一下

 - [官方文檔](https://www.home-assistant.io/components/media_player.spotify/)

```
media_player:
  - platform: spotify
    client_id: YOUR_CLIENT_ID
    client_secret: YOUR_CLIENT_SECRET
```

先到[Spotify開發者網頁](https://developer.spotify.com/dashboard/applications)中找到自己建立的App, 
把Clien ID跟Client Secret填到對應位置

再編輯App設定中的Redirect URIs: ```http://IP_or_Domain:8123/api/spotify```

如果發現callback頁面無效, 記得要在```Configuration.yaml```
把```http:```啟用```trusted_networks:``` (```base_url:```也順便設置一下)

```
http:
  trusted_networks:
    - 127.0.0.1
    - ::1
    - 192.168.0.0/24
```

---

#### 遇到pip報錯

目前找到比較有參考價值的, 有空再試試看, [參考文章](https://community.home-assistant.io/t/run-pip-install-in-docker-hass/52946/8)

I’ve got it working in HassIO using the following procedure:

Enable SSH access as described in <https://developers.home-assistant.io/docs/en/hassio_debugging.html>

Then SSH to hassio.local

```
ssh root@hassio.local -p 22222

#run shell in homeassistant docker containter
docker exec -it homeassistant /bin/bash
```

define user base directory

```
export PYTHONUSERBASE=/config/deps
#install pyaes
pip3 install --user --no-cache-dir --prefix= --no-dependencies pyaes
#install nefit-client 0.2.2
```

0.2.5 didn’t work for me, got certificate expired messages

```
pip3 install --user --no-cache-dir --prefix= --no-dependencies nefit-client==0.2.2
```

