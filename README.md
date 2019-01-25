# Docker on Raspberry Pi

## 如何在樹莓派上透過啟動 ```Docker``` 容器, 並且成功運行 ```Homebridge```

您可以在前往 [oznu/docker-homebridge][docker-homebridge-link] , 您將會在其中獲得相當豐富的知識與學會如何安裝並且使用它們.


### 這是一篇中文化簡易備忘錄

[oznu/homebridge][docker-homebridge-link] 的詳細說明中的 **[Wiki頁面][docker-homebridge-wiki]** ,
所載述的的步驟說明與內容: 使樹莓派玩家學會如何啟動Docker containers, 並在容器中啟動Homebridge.

於是將內容簡化成自己看得懂的文字與替換成自己系統角色, 來使自己方便找到筆記, 然後再將Homebridge動起來.

### 樹莓派指令備忘錄

一些常用的系統指令: [Raspberry Pi Command Line Note][rpi-link]

## Docker安裝指南

依下列步驟，從官方來源安裝

```
# Add Docker’s official GPG key:
curl -fsSL https://download.docker.com/linux/raspbian/gpg | sudo apt-key add -

# Use the following command to set up the stable repository:
echo "deb [arch=armhf] https://download.docker.com/linux/raspbian stretch stable" | sudo tee /etc/apt/sources.list.d/docker.list

# Update sources and install docker
sudo apt-get update
sudo apt-get install docker-ce
```

將使用者 `pi` 新增至docker群組中並登出

```
sudo usermod -aG docker pi && logout
```

## 安裝Docker Compose

有了 [Docker Compose][docker-compose-link] 之後, 安裝於Docker containers中變得相當容易, 首次安裝需透過Python與一些套件來完成安裝它:

```
sudo apt-get -y install python-setuptools && sudo easy_install pip  && sudo pip install docker-compose
```

在家目錄使用者內新增一個所需的資料夾, 依照以下指示建立新資料夾並且進到該資料夾中.

```
mkdir /home/pi/NAME_IT
cd /home/pi/NAME_IT
```
然後使用 ```nano``` 編寫並建立名為 ```docker-compose.yml``` 的檔案.

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

使用 ```CTRL+X``` 儲存並且關閉檔案.

在背景啟動Container容器, 執行以下指令:

```
docker-compose up -d
```

* 這個指令將會從 [oznu/homebridge][docker-homebridge-link] 下載最新版本的docker image.
* 使用 `-d` 是為了使docker-compose在背景啟動.

查看log有無異常, Portainer是否成功運作中:

```
docker-compose logs -f container_name (or container_ID)
```

[rpi-link]: pages/rpi-cli.md
[docker-homebridge-link]: https://github.com/oznu/docker-homebridge
[docker-homebridge-wiki]: https://github.com/oznu/docker-homebridge/wiki/Homebridge-on-Raspberry-Pi
