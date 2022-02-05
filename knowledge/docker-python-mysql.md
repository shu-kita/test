# Docker上でPythonとMySQLを連携

## 使用環境
* Dockerのバージョン
    * Docker Desktop : 4.0.1
    * Engine : 20.10.8
* Python, MySQLのバージョン(Dockerの公式イメージの2022/02/05時点での最新)
    * Pythonのバージョン : 3.10.2
    * MySQLのバージョン : 8.0.28


## コンテナイメージの取得
以下のコマンドでpython, mysqlのコンテナイメージを取得
```
docker pull python
```
```
docker pull mysql
```

## ネットワークの作成
以下のコマンドでネットワークを作成
```
docker network create python-mysql
```
「python-mysql」という名前のdocker networkを作成

確認
```
PS C:\Users\Programs\docker\mysql> docker network ls
NETWORK ID     NAME           DRIVER    SCOPE
a51d54c0b2c9   bridge         bridge    local
faaad3171f2c   host           host      local
c11908b83263   none           null      local
0f5d7dd03e03   python-mysql   bridge    local
```

## コンテナの起動
以下のコマンドでコンテナを起動
* MySQLコンテナ  
「mysql_con」という名前でコンテナを起動
    ```
    docker run --name mysql_con  --network python-mysql  -e MYSQL_ROOT_PASSWORD=mysql_pass -d -p 33306:3306 mysql
    ```
* Pythonコンテナ  
「python_con」という名前でコンテナを起動
    ```
    docker run --name python_con -it  --network python-mysql  -v C:\Users\Programs\docker\mysql:/var/opt  python:latest /bin/bash
    ```

確認
```
PS C:\Users\Programs\docker\mysql> docker container ls
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                                                    NAMES
92bf3f48f0f7   python:latest   "/bin/bash"              37 minutes ago   Up 37 minutes                                                            python_con
8337420b4cad   mysql           "docker-entrypoint.s…"   38 minutes ago   Up 30 minutes   33060/tcp, 0.0.0.0:33306->3306/tcp, :::33306->3306/tcp   mysql_con
```

## MySQLのパスワード設定の変更
MySQLのバージョンが8の場合、以下の手順を実行する
1. mysql8がインストールされているdockerにログイン
2. 「echo "default_authentication_plugin= mysql_native_password" >> /etc/mysql/my.cnf」 を実行
3. コンテナをstopして、start(再起動)

## MySQLのデータベース作成
MySQLにrootでログインしてデータベースを作成する
1. 以下のコマンドでMySQLにrootでログイン
    ```
    mysql -u root -p
    ```
    パスワードは「mysql_pass」

2. 以下の文でデータベースを作成
    ```
    CREATE DATABASE test_work;
    ```
    データベース名は「test_work」

<a id=IP></a>
## MySQLコンテナのIPアドレスを確認
次手順で作成するプログラムにMySQLコンテナのIPアドレスの情報が必要なので確認  
以下のコマンドで確認する
```
docker network inspect python-mysql
```

確認
```
PS C:\Users\Programs\docker\mysql> docker network inspect python-mysql
[
    (略)

        "Containers": {
            "8337420b4cad7317a6916a5d61ea86cdcec34f61d87547d1d8610eba5e35ae6b": {
                "Name": "mysql_con",
                "EndpointID": "1e09387938126cebdc69203f930d0d0f06ab044c34be10f6c588396b1459b1f8",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
            "92bf3f48f0f79449798ce64b92b84ca406b56a381da9ba6eeef83ce989af528c": {
                "Name": "python_con",
                "EndpointID": "231043087d4bc176bf2d8df2961cdd31802caf7afcb0df1cec982b020d0ee7e2",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
            }
        },
    (略)
]
```
各種コンテナの名前、エンドポイント、Macアドレス、IPアドレスが記載されている  

確認するのは"name"キーの値が"mysql_con"のオブジェクトの"IPv4Address"キーの値  
今回の場合は「172.18.0.2」　(/16は今回は必要ない)

## PythonコンテナでMySQLとの接続にライブラリをインストール
Pythonコンテナで以下のコマンドを実行して「mysql-connector-python」をインストール
```
pip3 install mysql-connector-python
```

## Pythonのプログラムを作成
MySQLにテーブルを作成するPythonプログラムを作成  
ボリュームをマウントしているので、C:\Users\Programs\docker\mysqlの配下に  
「sample.py」を以下の内容で作成
```
# -*- coding: utf-8 -*-

import mysql.connector

db=mysql.connector.connect(host="172.18.0.2", port="3306",  user="root", password="mysql_pass")
　　　　　
cursor=db.cursor()

cursor.execute("USE test_work")
db.commit()
cursor.execute("""CREATE TABLE IF NOT EXISTS docker_make(
                id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                fruits VARCHAR(32),
                value INT);""")
db.commit()
```
5行目の「db=mysql...」の引数「host=」には[前手順](#IP)で確認したIPアドレスを入れる

## sample.pyの実行と結果の確認
1. Pythonコンテナにログインしてsample.pyを実行
    ```
    python /var/opt/sample.py
    ```

2. MySQLコンテナにログインして結果を確認
    mysqlにrootユーザーでログインして以下の文を実行
    ```
    USE test_work;
    SHOW TABLES;
    ```
    結果が以下のようになっていれば、PythonプログラムからMySQLにアクセス出来ている
    ```
    mysql> use test_work;
    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A

    Database changed
    mysql> show tables;
    +---------------------+
    | Tables_in_test_work |
    +---------------------+
    | docker_make         |
    +---------------------+
    1 row in set (0.01 sec)
    ```