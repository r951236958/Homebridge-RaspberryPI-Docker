---
---
# Docker on Raspberry Pi

## 如何在樹莓派上安裝Docker CE & Docker Compose

- 在樹莓派上透過啟動 `Docker` 容器, 並且成功運行 `Homebridge`
- 您可以在前往 [Docker官網的快速建構中][docker-ce-rasbian-link] , 學會如何安裝並且使用它們.

### 第一步安裝 Docker CE

```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

安裝完成別忘了將使用者加入docker群組中

```
sudo usermod -aG docker pi && logout
```

### 第二步安裝Docker Compose

官方建議使用pip安裝docker-compose

```
sudo pip install docker-compose
```

但是有可能會安裝失敗, 可以再嘗試看看下面這條指令

```
sudo apt-get -y install python-setuptools && sudo easy_install pip  && sudo pip install docker-compose
```

如果還是未能成功, 官方建議透過 [Hypriot][hypriot-link] 儲存庫查詢可供安裝的docker-compose版本安裝, 詳閱 [設定Hypriot儲存庫說明][hypriot-instructions-link]

```
curl -s https://packagecloud.io/install/repositories/Hypriot/Schatzkiste/script.deb.sh | sudo bash
sudo apt-get update
sudo apt-cache madison docker-compose
```

將會打印出可安裝版本有哪些

```
apt-cache madison docker docker-compose |    1.8.0-2 | http://raspbian.raspberrypi.org/raspbian stretch/main armhf Packages docker-compose |    1.8.0-2 | http://raspbian.raspberrypi.org/raspbian stretch/main armhf Packages
```

以此例來說, 則可執行 ```apt-get install docker-compose=1.8.0-2``` 來安裝 Docker Compose

## 安裝 Portainer

### 拉取 Portainer 鏡像檔

```
docker pull portainer/portainer
```

### 創建 Volume 後運行 portainer 容器

```
docker volume create portainer_data
docker run -d -p 9000:9000 --restart=always --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

## 安裝 youtube-dl

安裝方式十分簡單, 只需執行兩條指令即可安裝完成

### 安裝指令

```
sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
sudo chmod a+rx /usr/local/bin/youtube-dl
```

### Configuration

建議路徑為 /etc/youtube-dl.conf , 配置內容如下

```
# Lines starting with # are comments

# Always extract audio
-x

# 儲存路徑與檔名格式
-o '/media/SANDISK/SCDL/%(uploader)s/%(title)s.%(ext)s'

# 新增ID3標籤 
--add-metadata

# 以縮圖作為封面
--embed-thumbnail
```

### 執行 youtube-dl 並指定設定檔路徑

因為檔案儲存路徑設定為外接USB中, 因此需要root權限才能儲存, 故需使用 `sudo`

```
sudo youtube-dl --config-location /etc/youtube-dl.conf https://soundcloud.com/djseanlee/jolin-tsai-sean-lees-circuit-mashup
```

若下載成功後，往後只需執行 `sudo youtube-dl [URL]` 即可下載


> 樹莓派指令備忘錄: [Raspberry Pi Command Line Note][rpi-link]



[rpi-link]: pages/rpi-cli.md
[docker-ce-rasbian-link]: https://docs.docker.com/install/linux/docker-ce/debian/#install-using-the-convenience-script
[docker-compose-link]: https://docs.docker.com/compose/
[docker-homebridge-link]: https://github.com/oznu/docker-homebridge
[docker-homebridge-wiki]: https://github.com/oznu/docker-homebridge/wiki/Homebridge-on-Raspberry-Pi
[install-hassio-link]: install_hassio.md
[hypriot-link]: https://hypriot.com/
[hypriot-instructions-link]: https://blog.hypriot.com/post/your-number-one-source-for-docker-on-arm/
