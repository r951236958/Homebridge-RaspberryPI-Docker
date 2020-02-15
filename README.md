# 樹莓派安裝Docker

## 官方建議使用腳本安裝

- Step 1. 下載安裝腳本並執行

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

- Step 2. 將使用者加入docker群組 (免root權限使用docker，加入群組後需登入才能生效)

```bash
$ sudo usermod -aG docker $USER
$ logout
```

## 安裝Portainer

```
$ docker volume create portainer_data
$ docker run -d -p 9000:9000 -p 8000:8000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

參考官網說明 [Get Docker Engine - Community for Debian](https://docs.docker.com/install/linux/docker-ce/debian/#install-using-the-convenience-script)

[docker-homebridge](homebridge/README.md)
