+++
title = "使用 Docker 在 Mac 上裝 Mysql Server"
author = "Allen Hsieh"
description = "由於公司的 Package Managment 都是客製化的，外加維護的服務沒有 Container 化，所以以前很難在本機上開發。但最近公司開始導入了 Spring Boot ，所以現在想在本機架起一個環境也不像以前這麼繁瑣，也就順便研究一下直接用 Docker 在本機架起一個 Mysql Server，這邊紀錄一下。"
featured = true
categories = ["CONTAINER"]
tags = [
    "Docker"
]
date = "2022-01-28"
aliases = ["create_mysql8_server_by_docker"]
images = ["images/docker.png"]
+++

由於公司的 Package Managment 都是客製化的，外加維護的服務沒有 Container 化，所以以前很難在本機上開發。但最近公司開始導入了 Spring Boot ，所以現在想在本機架起一個環境也不像以前這麼繁瑣，也就順便研究一下直接用 Docker 在本機架起一個 Mysql Server，這邊紀錄一下。


## 安裝 Mysql Client
---
透過 Brew 安裝 Mysql Client
```Bash
brew install mysql8-client
```

安裝好以後，Brew 有提醒要 Export 環境變數，別忘了
```bash
If you need to have mysql-client first in your PATH, run:
  echo 'export PATH="/usr/local/opt/mysql-client/bin:$PATH"' >> ~/.zshrc

For compilers to find mysql-client you may need to set:
  export LDFLAGS="-L/usr/local/opt/mysql-client/lib"
  export CPPFLAGS="-I/usr/local/opt/mysql-client/include"
```

## 安裝與運行 Mysql8 
---
```Bash
$ docker pull mysql/mysql-server:8.0
$ docker run -itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql/mysql-server:8.0
```

* name: Container 的名字
* p: 本機的 port mapping 到 container 裡面的 port 
* MYSQL_ROOT_PASSWORD: Mysql Root 的密碼

## 確認 Mysql8 運行正常
---
```Bash
$ docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS                    PORTS                                                        NAMES
8ec7a52cc3ea   mysql/mysql-server:8.0   "/entrypoint.sh mysq…"   29 minutes ago   Up 26 minutes (healthy)   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060-33061/tcp   mysql
```

要確認 Container 的 Status 是 healthy 喔 

## 連線 Mysql Server
---
Container 中已經安裝好 Mysql Client 所以可以直接進入到 Container 中連線
```Bash
$ docker exec -it mysql8 mysql -uroot -p
```

## 讓 Container 本機可以連線
---
為了讓 Container 以外的本機可以連線，所以需修改 Mysql User 的權限， Root User 可以接受外面的連線。
```
UPDATE mysql.user SET HOST='%' WHERE USER = 'root' LIMIT 1;
```

注意，這邊要重啟 Mysql Server 才會生效，艾倫是直接重啟整個 Container 
```Bash
docker restart mysql
```

## Mac 本機測試連線
---
這邊要注意，由於 Mysql Server 是在 Contaienr 中運行的，導致本機沒有 mysq.sock 的 file，所以直接運行 mysql client 沒指定 Host 會出現以下錯誤。
```Bash
$ mysql -u root -p
Enter password:
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
```

所以這邊需要特別指定 Host 為 127.0.0.1
```Bash
mysql -h 127.0.0.1 -u root -p
```
