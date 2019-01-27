---
---

# 樹莓派上安裝Hassio

## 必要條件

已經安裝好系統的樹莓派1台以外, 還要安裝下列相依套件

```
docker-ce
bash
jq
curl
avahi-daemon
dbus
```

## 使用方式

先提取raspberrypi3-homeassistant鏡像檔下來 (因為檔案很大)

```
docker pull homeassistant/raspberrypi3-homeassistant
```

必須以 `root` 身分執行, 所以先 ```sudo su ```

```
curl -sL https://raw.githubusercontent.com/home-assistant/hassio-build/master/install/hassio_install | bash -s -- -m raspberrypi3
```

然後執行完看到 ```Run Hass.io ```就完成了!

### 系統服務 `systemctl` 指令

```
# 查看狀態
sudo systemctl status hassio-supervisor

# 停止服務
sudo systemctl stop hassio-supervisor

# 重啟服務
sudo systemctl restart hassio-supervisor
```

### Docker指令

查看容器啟用情況

```
# 查詢正在運作的容器
docker ps

# 查詢所有容器 (包含未啟用)
docker ps -a
```

其他常用指令

```
docker [Options] [container_name/id]

Options:
stop
restart
kill

# 詳情查詢使用
dock --help

# 查詢鏡像檔
docker images

# 移除鏡像檔
docker rmi Image_Id

# 查看log
docker logs -f Conatiner_Name
```

## 其他

hassio路徑: `/use/share/hassio`
homeassistant路徑： `/use/share/hassio/homeassistant`
hassio-supervisor路徑: `/etc/systemd/system/hassio-supervisor`

後續有想到再補上
