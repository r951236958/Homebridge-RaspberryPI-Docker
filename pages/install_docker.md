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
docker run -d -p 9000:9000 --restart=always --name portainer -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
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
-o '/media/SANDISK/%(uploader_id)s/%(title)s.%(ext)s'

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

### Docker CE for Raspbian 安裝指南

[Docker CE官網連結][docker-ce-debian-link]

依下列步驟，從官方來源安裝

```
# 新增 Docker 官方 GPG key:
curl -fsSL https://download.docker.com/linux/raspbian/gpg | sudo apt-key add -

# 使用以下指令來新增 穩定版來源
echo "deb [arch=armhf] https://download.docker.com/linux/raspbian stretch stable" | sudo tee /etc/apt/sources.list.d/docker.list

# 更新來源並安裝
sudo apt-get update
sudo apt-get install docker-ce
```

將使用者 `pi` 新增至docker群組中並登出

```
sudo usermod -aG docker pi && logout
```

> 安裝完 `Docker` , 後續要繼續安裝Homeassistant/Hassio, [前往此連結][install-hassio-link]查看安裝說明.

### 安裝Docker Compose

有了 [Docker Compose][docker-compose-link] 之後, 安裝於Docker containers中變得相當容易, 首次安裝需透過Python與一些套件來完成安裝它:

```
sudo apt-get -y install python-setuptools && sudo easy_install pip  && sudo pip install docker-compose
```

在家目錄使用者內新增一個所需的資料夾, 依照以下指示建立新資料夾並且進到該資料夾中.

```
mkdir /home/pi/NAME_IT
cd /home/pi/NAME_IT
```
然後使用 `nano` 編寫並建立名為 `docker-compose.yml` 的檔案.

```
nano docker-compose.yml
```

內容範例以Portainer作為示範:

```js
version: '2'

services:
  portainer:
    image: portainer/portainer
    command: -H unix:///var/run/docker.sock
    restart: always
    ports:
      - 9000:9000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  portainer_data:
```

使用 `CTRL+X` 儲存並且關閉檔案.

在背景啟動Container容器, 執行以下指令:

```
docker-compose up -d
```

- 這個指令將會從 [oznu/homebridge][docker-homebridge-link] 下載最新版本的docker image.
- 使用 `-d` 是為了使docker-compose在背景啟動.

查看log有無異常, Portainer是否成功運作中:

```
docker-compose logs -f container_name (or container_ID)
```

[rpi-link]: pages/rpi-cli.md
[docker-ce-rasbian-link]: https://docs.docker.com/install/linux/docker-ce/debian/#install-using-the-convenience-script
[docker-compose-link]: https://docs.docker.com/compose/
[docker-homebridge-link]: https://github.com/oznu/docker-homebridge
[docker-homebridge-wiki]: https://github.com/oznu/docker-homebridge/wiki/Homebridge-on-Raspberry-Pi
[install-hassio-link]: install_hassio.md
[hypriot-link]: https://hypriot.com/
[hypriot-instructions-link]: https://blog.hypriot.com/post/your-number-one-source-for-docker-on-arm/
