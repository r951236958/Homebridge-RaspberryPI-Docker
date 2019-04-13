# 樹莓派與Linux指令與權限說明

<!-- TOC -->

- [樹莓派與Linux指令與權限說明](#樹莓派與linux指令與權限說明)
    - [SSH遠端連線](#ssh遠端連線)
        - [檢查現有的SSH密鑰](#檢查現有的ssh密鑰)
        - [生成新的SSH密鑰](#生成新的ssh密鑰)
        - [將SSH密鑰加入ssh-agent](#將ssh密鑰加入ssh-agent)
            - [ssh-add -K發生錯誤](#ssh-add--k發生錯誤)
        - [複製SSH公鑰到遠端主機](#複製ssh公鑰到遠端主機)
            - [ssh-add指令說明](#ssh-add指令說明)
    - [Linux檔案與目錄權限說明](#linux檔案與目錄權限說明)
        - [權限組成方式](#權限組成方式)
        - [權限轉換方式](#權限轉換方式)
        - [一般常用的權限賦予有以下幾種](#一般常用的權限賦予有以下幾種)
        - [權限變更指令 chgrp chown chmod](#權限變更指令-chgrp-chown-chmod)
            - [chgrp指令說明](#chgrp指令說明)
            - [chown指令說明](#chown指令說明)
            - [chmod指令說明](#chmod指令說明)
    - [系統服務](#系統服務)
        - [加載/啟動/重啟 系統服務](#加載啟動重啟-系統服務)

<!-- /TOC -->

**常用指令備忘錄:**

```sh
~ $ uname -m #查詢CPU
~ $ df -h #查詢SD卡容量
~ $ apt-get update #更新遠端伺服器套件清單
~ $ apt-get upgrade #更新已安裝套件
~ $ wget URL #適用於一般小型檔案或文件下載
~ $ curl -I URL -o /path/to #[-I]可讀取網頁header, [-o]檔案儲放路徑
```

## SSH遠端連線

如何生成與使用SSH密鑰, 詳細說明可參考[使用SSH連線至GitHub][github-ssh-link]

### 檢查現有的SSH密鑰

在生成SSH密鑰前, 可以先確認系統中是否已存在任何SSH密鑰

1. 輸入 `ls -al ~/.ssh` 檢查是否有任何SSH密鑰:

    ```sh
    pi@moode:~ $ ls -al ~/.ssh
    total 24
    drwx------  2 pi pi 4096 Apr 13 04:48 .
    drwxr-xr-x 11 pi pi 4096 Apr 13 03:02 ..
    -rw-------  1 pi pi 1497 Apr 10 23:04     authorized_keys
    -rw-------  1 pi pi 3243 Apr 10 00:51     id_rsa
    -rw-------  1 pi pi  746 Apr 10 00:51     id_rsa.pub
    -rw-r--r--  1 pi pi  884 Apr 13 04:48     known_hosts
    # 列出 .ssh 目錄中的檔案
    ```

2. 檢查目錄列表以查看您是否已擁有公共SSH密鑰

    在預設情況下, 公鑰文件名稱是下列之一:

    * _id_dsa.pub_
    * _id_ecdsa.pub_
    * _id_ed25519.pub_
    * _id_rsa.pub_
    
    * 如果沒有現有的公鑰與密鑰, 則可以[生成新的SSH密鑰](#生成新的SSH密鑰)
    * 如果已有現有的公鑰與密鑰(例如id_rsa.pub和id_rsa), 則可以[將SSH密鑰加入ssh-agent](#將SSH密鑰加入ssh-agent)

    > 若系統提示 `~/.ssh` 不存在, 將會在[生成新的SSH密鑰](#生成新的SSH密鑰)時創建它

### 生成新的SSH密鑰

1. 複製以下指令, 並將 _E-mail_ 替換為自己的電子郵件

    ```sh
    ~ $ ssh-keygen -t rsa -b 4096 -C    "E-mail@example.com"
    ```

    這將使用提供的電子郵件作為標籤創建一個新的ssh密鑰。

    ```sh
    > Generating public/private rsa key pair.
    ```

2. 當系統提示您"輸入要保存密鑰的文件"時, 按Enter鍵。這接受預設文件位置。

    ```sh
    > Enter a file in which to save the key (/  Users/you/.ssh/id_rsa): [Press enter]
    ```

3. 在系統提示下, 輸入安全密碼。有關更多說明, 請參閱"使用SSH密鑰密碼"

    ```sh
    > Enter passphrase (empty for no    passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type    passphrase again]
    ```

### 將SSH密鑰加入ssh-agent

在將新的SSH密鑰添加到ssh-agent以管理密鑰之前, 您應該檢查[現有的SSH密鑰](#檢查現有的SSH密鑰)並[生成新的SSH密鑰](#生成新的SSH密鑰)。將SSH密鑰添加到代理時, 請使用預設的macOS `ssh-add` 指令, 而不是macports, homebrew或其他外部源安裝的應用程式。

1. 在後台啟動ssh-agent

    ```sh
    ~ $ eval "$(ssh-agent -s)"
    > Agent pid 59566
    ```

2. 如果使用的是macOS Sierra 10.12.2或更高版本, 則需要修改 `~/.ssh/config` 文件以自動將密鑰加載到ssh-agent中並在鑰匙圈中儲存密碼。

    ```
    Host *
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/id_rsa
    ```

3. 將SSH私鑰添加到ssh-agent並將密碼儲存在鑰匙圈中。如果使用其他名稱創建密鑰, 或者要添加具有不同名稱的現有密鑰, 請將指令中的 _id_rsa_ 替換為私鑰文件的名稱。

    ```sh
    ~ $ ssh-add -K ~/.ssh/id_rsa
    ```

    > 注意：該 `-K` 選項是Apple的標準版本 `ssh-add` , 當您向ssh-agent添加ssh密鑰時, 會將鑰匙圈儲存在您的鑰匙串中。
    > 
    > 如果您沒有安裝Apple的標準版本, 則可能會收到錯誤訊息。有關解決此錯誤的詳細說明, 請參閱 " [Error: ssh-add: illegal option -- K](#ssh-add -K發生錯誤) "。

#### ssh-add -K發生錯誤

"**Error: ssh-add: illegal option -- K**" 此錯誤表示目前使用的macOS版本 `ssh-add` 不支持macOS keychain集成, 這允許您將密碼儲存在鑰匙串中。

該 `-K` 選項在Apple的標準版本中 `ssh-add` , 當您向ssh-agent添加ssh密鑰時, 它會將密碼鏈儲存在您的鑰匙圈中。如果您安裝了不同版本 `ssh-add` , 則可能缺乏支持`-K`。

**解決方法:**

要將SSH私鑰添加到ssh-agent, 您可以指定Apple版本的路徑 `ssh-add` :

```sh
~ $ /usr/bin/ssh-add -K ~/.ssh/id_rsa
```

> 注意：如果使用其他名稱創建密鑰, 或者要添加具有不同名稱的現有密鑰, 請將命令中的id_rsa替換為私鑰文件的名稱。

### 複製SSH公鑰到遠端主機

使用指令 `ssh-copy-id` 將本機SSH公鑰複製到遠端主機 `~/.ssh/authorized_keys` 文件中

```sh
~ $ ssh-copy-id [-i [identity_file]] [user@]server_hostname #-i: 指定公鑰文件

#範例:
~ $ ssh-copy-id -i ~/.ssh/id_rsa.pub [user@]server_hostname 
```

#### ssh-add指令說明

```sh
~ $ ssh-add ssh-add [options] [file ...]
Options:
    -D：刪除ssh-agent中的所有密鑰.
    -d：從ssh-agent中的刪除密鑰
    -e pkcs11：刪除PKCS#11共享庫pkcs1提供的鑰匙。
    -s pkcs11：添加PKCS#11共享庫pkcs1提供的鑰匙。
    -L：顯示ssh-agent中的公鑰
    -l：顯示ssh-agent中的密鑰
    -t life：對加載的密鑰設置超時時間, 超時ssh-agent將自動卸載密鑰
    -X：對ssh-agent進行解鎖
    -x：對ssh-agent進行加鎖
```

---

## Linux檔案與目錄權限說明

### 權限組成方式

常見的權限表示由 ```[-][rwx][r-x][r--]``` 所組成

1. [-] 檔案的屬性 
    - [ d ] 目錄, 
    - [ - ] 檔案 , 
    - [ l ] 連結檔, 
    - [ b ] 裝置檔裡面的可供儲存的周邊設備 
    - [ c ] 裝置檔裡面的序列埠設備, 例如鍵盤、滑鼠
2. [rwx] 擁有者的權限
3. [r-x] 群組的權限
4. [r--] 其他非群組內使用者的權限
    
> [rwx] 分別代表: 讀(read) / 寫(write) / 執行(execute), [-] 表示沒有權限

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

 **[r]=4, [w]=2, [x]=1, [-]=0**

### 一般常用的權限賦予有以下幾種

```
-rw------- (600) #只有擁有者具有 「讀/寫」 權限

-rw-r--r-- (644) #表示擁有者具有 「讀/寫」 權限, 同群組與其他使用者只有 「讀」 的權限

-rwx------ (700) #只有擁有者具有 「讀/寫/執行」 權限

-rwxr-xr-x (755) #表示擁有者具有 「讀/寫/執行」 權限, 同群組與其他使用者具有 「讀/執行」 權限

-rwx--x--x (711) #表示擁有者具有 「讀/寫/執行」 權限, 同群組與其他使用者具有 「執行」 權限

-rw-rw-rw- (666) #所有擁有者具有 「讀/寫」 權限

-rwxrwxrwx (777) #所有擁有者具有 「讀/寫/執行」 權限
```

- 資料夾通常給予755的權限
- 檔案通常給予644的權限

### 權限變更指令 chgrp chown chmod

* chgrp ：改變檔案所屬群組
* chown ：改變檔案所屬人
* chmod ：改變檔案的屬性、 SUID 、等等的特性

#### chgrp指令說明

```sh
~ $ chgrp [Options] group file ...
~ $ chgrp [Options] group file ... 
~ $ chgrp [Options] --reference=RFILE FILE...

Options:
    -R, --recursive #以遞歸的方式變更, 包含目錄內的子目錄與檔案

範例：
~ $  chgrp users test-file.txt
~ $ ls -l
-rw-r--r--  1 root users 68495 Jun 25 08:53 test-file.txt

~ $ chgrp testing 
chgrp: invalid group name `testing' #發生錯誤,  沒有testing這個群組
```

#### chown指令說明

```sh
~ $ chown [Options] [OWNER][:[GROUP]] FILE...

Options:
    -R, --recursive #以遞歸的方式變更, 包含目錄內的子目錄與檔案

範例:
~ $ chown [-R] 帳號名稱 檔案或目錄
~ $ chown [-R] 帳號名稱:群組名稱/檔案或目錄
~ $ chown bin test-file.txt
~ $ ls -l
-rw-r--r--  1 bin  users 68495 Jun 25 08:53 test-file.txt

~ $ chown root:root test-file.txt
~ $ ls -l
-rw-r--r--  1 root root 68495 Jun 25 08:53 test-file.txt
```

#### chmod指令說明

```sh
~ $ chmod [Options] MODE[,MODE]... FILE...

Options:
    -R, --recursive #以遞歸的方式變更, 包含目錄內的子目錄與檔案
    MODE #就是剛剛提到的數字類型的權限屬性, 為 rwx 屬性數值的相加。

範例:
~ $ chmod [-R] 檔案或目錄
~ $ ls -al .bashrc
-rw-r--r--  1 root root 395 Jul  4 11:45 .bashrc

~ $ chmod 777 .bashrc
~ $ ls -al .bashrc
-rwxrwxrwx  1 root root 395 Jul  4 11:45 .bashrc

~ $ chmod a+x filename #即可讓該程式擁有執行權限
```

---

## 系統服務

在 `/etc/systemd/system/` 路徑下新增 `SERVICE_NAME.service` 系統服務檔(以下為文件範例)

```sh
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

### 加載/啟動/重啟 系統服務

新增完文件後要先告知系統, 知道他知道有新服務

```sh
~ $ sudo systemctl --system daemon-reload
```

在系統啟動時, 自動啟動服務

```sh
~ $ sudo systemctl start SERVICE
```

同理, 要暫停、重啟的話, 則改為 `stop` 或 `restart`  查看狀態 `status` 

要以root權限執行

```sh
~ $ service SERVICE restart
~ $ systemctl [Options] {COMMAND} ... SERVICE

{COMMAND}
    start   #啟動服務
    stop    #停止服務
    status  #查詢狀態
```

服務啟動後 `status` 只會顯示出短短幾行紀錄, 如果要查看完整log則要使用指令 `journalctl` 

```sh
# 顯示日誌輸出
~ $ sudo journalctl -f -u SERVICE

# 只查看錯誤日誌
~ $ sudo journalctl -f -u SERVICE | grep -i 'error'

# 系統服務重啟後顯示日誌
~ $ sudo systemctl restart SERVICE && ~ $ sudo journalctl -f -u SERVICE
```



[github-ssh-link]: https://help.github.com/en/articles/connecting-to-github-with-ssh