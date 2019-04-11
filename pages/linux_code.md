# 樹莓派與Linux常用指令與說明

- [常用指令](#常用指令)
  - [新增系統服務](#新增系統服務)
  - [服務加載、啟動與重啟](#服務加載啟動與重啟)
- [Linux檔案與目錄權限說明](#linux檔案與目錄權限說明)
  - [權限組成方式](#權限組成方式)
  - [權限轉換方式](#權限轉換方式)
  - [一般常用的權限賦予有以下幾種](#一般常用的權限賦予有以下幾種)
  - [變更權限指令chgrp chown chmod](#權限變更指令-chgrp-chown-chmod)

## 常用指令

一些常忘記的備忘錄
```
$ uname -m (查詢CPU)
$ df -h (查詢SD卡容量)
$ apt-get update (更新遠端伺服器套件清單)
$ apt-get upgrade (更新已安裝套件)
$ wget URL (適用於一般小型檔案或文件下載)
$ curl -I URL -o /path/to ([-I]可讀取網頁header, [-o]檔案儲放路徑)

#### SSH密鑰 ####
產生SSH密鑰
$ ssh-keygen -t rsa -b 4096 -C "example@example.com"

透過SSH密鑰登入遠端伺服器驗證, 往後SSH就不用再輸入密碼
$ ssh-copy-id USERNAME@SSH_IP_ADDRESS

將SSH密鑰加入ssh-agent
1. 在後台啟動ssh-agent
$ eval "$(ssh-agent -s)"
> Agent pid 59566

2. SSH密鑰添加到ssh-agent
$ ssh-add ~/.ssh/id_rsa
```

要以root權限執行
```
service SERVICE restart

systemctl [OPTIONS...] {COMMAND} ... SERVICE

{COMMAND}
    start   啟動服務
    stop    停止服務
    status  查詢狀態
```

### 新增系統服務

在系統中新增某個SERVICE服務, 使用 ```sudo nano /etc/systemd/system/SERVICE.service``` 編輯文件, 參考homeassistant範例.

- **SERVICE** 要自行替換為新服務名稱
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

---

## Linux檔案與目錄權限說明

### 權限組成方式


```[-][rwx][r-x][r--]```

1. [-] 檔案的屬性 
    - [ d ] 目錄, 
    - [ - ] 檔案 , 
    - [ l ] 連結檔, 
    - [ b ] 裝置檔裡面的可供儲存的周邊設備 
    - [ c ] 裝置檔裡面的序列埠設備，例如鍵盤、滑鼠
2. [rwx] 擁有者的權限
3. [r-x] 群組的權限
4. [r--] 其他非群組內使用者的權限

[rwx] 分別代表: 讀(read) / 寫(write) / 執行(execute), [-] 表示沒有權限

### 權限轉換方式

權限 | 二進制 | 十進制 | 權限說明
:---------:|:---------:|:---------:|:---------
 --- | 000 | 0 | 無法讀/寫/執行
 --r | 001 | 1 | 執行
 -w- | 010 | 2 | 寫
 -wx | 011 | 3 | 寫/執行
 r-- | 100 | 4 | 讀
 r-x | 101 | 5 | 讀/執行
 rw- | 110 | 6 | 讀/寫
 rwx | 111 | 7 | 讀/寫/執行

 [r]=4, [w]=2, [x]=1, [-]=0

### 一般常用的權限賦予有以下幾種

```
-rw------- (600) 只有使用者具有 「讀/寫」 權限

-rw-r--r-- (644) 表示使用者具有 「讀/寫」 權限, 同群組與其他使用者只有 「讀」 的權限

-rwx------ (700) 只有使用者具有 「讀/寫/執行」 權限

-rwxr-xr-x (755) 表示使用者具有 「讀/寫/執行」 權限, 同群組與其他使用者具有 「讀/寫」 權限

-rwx--x--x (711) 表示使用者具有 「讀/寫/執行」 權限, 同群組與其他使用者具有 執行 權限

-rw-rw-rw- (666) 所有使用者具有 「讀/寫」 權限

-rwxrwxrwx (777) 所有使用者具有 「讀/寫/執行」 權限
```

- 資料夾通常給予755的權限
- 檔案通常給予644的權限

### 權限變更指令 chgrp chown chmod

* chgrp ：改變檔案所屬群組
* chown ：改變檔案所屬人
* chmod ：改變檔案的屬性、 SUID 、等等的特性

chgrp指令說明
```
$ chgrp [OPTION]... GROUP FILE...
$ chgrp [OPTION]... GROUP FILE... chgrp [OPTION]... --reference=RFILE FILE...

[OPTION]
    -R, --recursive 以遞歸的方式變更，包含目錄內的子目錄與檔案

範例：
root@linux:~ $ chgrp users test-file.txt
root@linux:~ $ ls -l
-rw-r--r--  1 root users 68495 Jun 25 08:53 test-file.txt
root@linux:~ $ chgrp testing 
chgrp: invalid group name `testing' # 發生錯誤,  沒有testing這個群組
```

chown指令說明
```
$ chown [OPTION]... [OWNER][:[GROUP]] FILE...

[OPTION]
    -R, --recursive 以遞歸的方式變更，包含目錄內的子目錄與檔案

範例:
root@linux:~ $ chown [-R] 帳號名稱 檔案或目錄
root@linux:~ $ chown [-R] 帳號名稱:群組名稱/檔案或目錄
root@linux:~ $
root@linux:~ $ chown bin test-file.txt
root@linux:~ $ ls -l
-rw-r--r--  1 bin  users 68495 Jun 25 08:53 test-file.txt
root@linux:~ $ chown root:root test-file.txt
root@linux:~ $ ls -l
-rw-r--r--  1 root root 68495 Jun 25 08:53 test-file.txt
```

chmod指令說明

```
$ chmod [OPTION]... MODE[,MODE]... FILE...

[OPTION]
    -R, --recursive 以遞歸的方式變更，包含目錄內的子目錄與檔案
    MODE: 就是剛剛提到的數字類型的權限屬性，為 rwx 屬性數值的相加。

範例:
root@linux:~ $ chmod [-R] 檔案或目錄
root@linux:~ $ ls -al .bashrc
-rw-r--r--  1 root root 395 Jul  4 11:45 .bashrc
root@linux:~ $ chmod 777 .bashrc
root@linux:~ $ ls -al .bashrc
-rwxrwxrwx  1 root root 395 Jul  4 11:45 .bashrc

root@linux:~ $chmod a+x filename #即可讓該程式擁有執行權限
```