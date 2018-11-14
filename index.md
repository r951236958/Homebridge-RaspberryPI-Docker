---
layout: Homebridge-RasspberryPI-Docker
---
# 在樹莓派上部署Docker並運行Homebridge

## 1. 安裝Docker

開啟終端機並執行以下指令:

```
# Add Docker’s official GPG key:
curl -fsSL https://download.docker.com/linux/raspbian/gpg | sudo apt-key add -

# Use the following command to set up the stable repository:
echo "deb [arch=armhf] https://download.docker.com/linux/raspbian stretch stable" | sudo tee /etc/apt/sources.list.d/docker.list

# Update sources and install docker
sudo apt-get update
sudo apt-get install docker-ce
```

將樹莓派使用者 ```volumio``` 新增到 ```docker``` 群組 
>因本次系統使用為Volumio.

```
sudo usermod -aG docker volumio && logout
```

## 2. 安裝Docker Compose

有了 [Docker Compose][docker-homebridge-link] 之後, 安裝於Docker containers中變得相當容易, 首次安裝需透過Python與一些套件來完成安裝它:

```
sudo apt-get -y install python-setuptools && sudo easy_install pip  && sudo pip install docker-compose
```

## 3. 創建一個Docker Compose主要檔案

在本例中使用者資料夾位於 ```volumio``` 資料夾, 依照以下指示建立新資料夾並且進到該資料夾中.

建立新資料夾, 並且於建立後進入資料夾:

```
mkdir /home/volumio/homebridge
cd /home/volumio/homebridge
```

然後使用 ```nano``` 編寫並建立名為 ```docker-compose.yml``` 的檔案.

```
nano docker-compose.yml
```

檔案內容應該如下列所示:

```js
version: '2'
services:
  homebridge:
    image: oznu/homebridge:raspberry-pi
    restart: always
    network_mode: host
    volumes:
      - ./config:/homebridge
    environment:
      - PGID=1000
      - PUID=1000
      - HOMEBRIDGE_CONFIG_UI=1
      - HOMEBRIDGE_CONFIG_UI_PORT=8080
```

使用 ```CTRL+X``` 儲存並且關閉檔案.

## 4. 啟動Homebridge

在Docker container容器中啟動Homebridge, 執行以下指令:

```
docker-compose up -d
```

*   這個指令將會從 [oznu/homebridge][docker-homebridge-link] 下載最新版本的docker image.
*   使用 ```-d``` 是為了使docker-compose在背景啟動.

當Homebridge logs沒有顯示出任何問題, 並且成功運作中, 將會看到iOS的配對碼:

```
docker-compose logs -f
```

你的Homebridge ```config.json``` , 安裝外掛plugins時, 要能於 `家庭APP` 存取應用, 必須將外掛安裝於 ```config``` 資料夾中.

## 5. 管理Homebridge

在本地電腦端開啟瀏覽器, 輸入下列網址並前往 ```http://<樹莓派ip>:8080```, 例: 我安裝於Volumio系統中, 我可以透過 ```http://volumio.local:8080``` 打開Homebridge管理頁面, 在此管理頁面中無論是安裝, 移除或升級任何外掛, 都務必要確保 ```config.json``` 檔案是正確的, 並且要重啟Homebridge.

可以透過執行下列指令來重啟或啟動這個容器:

```
docker-compose restart homebridge
```

### 與iOS的家庭APP連結

當以上都已經正常啟動也顯示了 `打開家庭APP來執行配對` , 那就打開它並且對準QR Code掃描吧.

#### 預設配對碼: `**031-45-154**`

如果APP顯示無法掃描, 那就手動輸入配對碼吧.

## 更新Homebridge

版本依然是由 [oznu/homebridge][docker-homebridge-link] 提供的最新版本作為更新.

下載最後一版 [oznu/homebridge][docker-homebridge-link] 的image檔案:

```
docker-compose pull homebridge
```

如果發現了有較新的版本, 將會自動下載image, 並將使用最新版本檔案作為container容器使用的image, 必須透過執行以下指令完成:

```
docker-compose up -d
```

## 以Shell形式訪問

如果有這個必要的話, 可以透過執行以下指令, 直接以 shell access 到 正在運行的container容器中:

```
docker-compose exec homebridge sh
```

[docker-homebridge-link]: https://hub.docker.com/r/oznu/homebridge/
[docker-homebridge-wiki]: https://github.com/oznu/docker-homebridge.wiki.git