# README



Docker環境でWordpressの環境を作るとき際に実際の本番環境ではWordpressをサブディレクトリに作成するときが多くてサイト自体のトップページはLaravelだったり別のフレームワークだったり、あるいはcomposerを使ったautoloadだけを読み込む簡易的なライブラリーだったりすることが多いのでWordpressの開発環境といいつつ結局はLAMP環境になるというのが僕の結論です。公式でpullできる

```bash
% docker pull wordpress
```

は、あんまり使えないというが僕の判断です。高度な開発になってくるとWordpressのセッションとLaravelのセッションを共有したりあるいは双方ともREST APIとして使ってフロント側は全く別のフレームワーク（vueやreact）で構築してしまうというのもこういった開発環境を作成する理由です。

PHPは最新の8を選びました。大きくバージョンアップしましたが、今度PHP8へのバージョンアップをしないことには開発に耐えられないでしょう。
[https://hub.docker.com/r/oberd/php-8.0-apache](https://hub.docker.com/r/oberd/php-8.0-apache)
MySQLは5.7にします。MySQLの8.0.23はまだまだ普及していなくて実用には耐えられないかなという判断です。もう何年か経ってから考えた方がいいかもです。
PHPMyAdminは比較的なんでもよいので適当にイメージを引っ張り込みます。



```bash
ugryhacks
├── app
├── mysql
│   ├── mysql
│   ├── performance_schema
│   ├── sys
│   └── test
├── php.ini
└── phpmyadmin
    └── sessions
```



Docker-compose.yml

```yml
version: '3'

services:
  php:
    image: php:8.0-apache
    volumes:
      - ./php.ini:/usr/local/etc/php/php.ini
      - ./app:/var/www/html
    ports:
      - 8080:80
  mysql:
    image: mysql:5.7
    volumes:
      - ./mysql:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=test
      - MYSQL_USER=test
      - MYSQL_PASSWORD=test
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    environment:
      - PMA_ARBITRARY=1
      - PMA_HOST=mysql
      - PMA_USER=test
      - PMA_PASSWORD=test
    links:
      - mysql
    ports:
      - 4040:80
    volumes:
      - ./phpmyadmin/sessions:/sessions
```

php.ini
```
[Date]
date.timezone = "Asia/Tokyo"
[mbstring]
mbstring.internal_encoding = "UTF-8"
mbstring.language = "Japanese"
```



dockerを起動
```
$ docker-compose up -d --build
```

dockerの確認
```
$ docker-compose ps
         Name                       Command               State                  Ports
------------------------------------------------------------------------------------------------------
ugryhacks_mysql_1        docker-entrypoint.sh mysqld      Up      3306/tcp, 33060/tcp
ugryhacks_php_1          docker-php-entrypoint apac ...   Up      0.0.0.0:8080->80/tcp,:::8080->80/tcp
ugryhacks_phpmyadmin_1   /docker-entrypoint.sh apac ...   Up      0.0.0.0:4040->80/tcp,:::4040->80/tcp
```



app
http://localhost:8080/

phpmyadmin
http://localhost:4040/







