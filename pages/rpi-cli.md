---

---

# 樹莓派指令備忘錄

## 樹莓派系統指令

### 新增系統服務

在系統中新增某個SERVICE服務, 使用 ```sudo nano /etc/systemd/system/SERVICE.service``` 編輯文件, 參考homeassistant範例.

- ***SERVICE***要自行替換為新服務名稱
- 其中```ExecStart```中的路徑因服務所在位置不同, 需視情況替換.

```
[Unit]
Description=Home Assistant
After=network-online.target

[Service]
Type=simple
User=%i
ExecStart=/usr/bin/hass

[Install]
WantedBy=multi-user.target
```
### 服務加載、啟動與重啟

新增完文件後要先告知系統, 知道他知道有新服務

```
sudo systemctl --system daemon-reload
```

在系統啟動時, 自動啟動服務

```
sudo systemctl start SERVICE
```

同理, 要暫停、重啟的話, 則改為```stop```或```restart``` 查看狀態```status```

服務啟動後```status```只會顯示出短短幾行紀錄, 如果要查看完整log則要使用指令```journalctl```

顯示日誌輸出

```
sudo journalctl -f -u SERVICE
```

只查看錯誤日誌

```
sudo journalctl -f -u SERVICE | grep -i 'error'
```

系統服務重啟後顯示日誌

```
sudo systemctl restart SERVICE && sudo journalctl -f -u SERVICE
```
