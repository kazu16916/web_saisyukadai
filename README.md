# Kadai ページ表示方法

## エラー情報 
## ”Can't open file for writing”や”Permission denied”と出力が出る可能性があるので 
```bash
sudo chown -R ec2-user:ec2-user public/
```
## など現在書き込もうとしているディレクトリを書き込み権限追加してください 

## 1. EC2 インスタンスの作成

## 2. EC2 インスタンスへ PC から SSH でログイン
```bash
ssh ec2-user@{IPアドレス} -i {秘密鍵ファイルのパス}
```

## 3. Vim のインストール
```bash
sudo yum install vim -y
```

## 4. Vim 設定の変更
```bash
vim ~/.vimrc
```

内容：
```vim
set number
set expandtab
set tabstop=2
set shiftwidth=2
set autoindent
```

## 5. screen のインストール
```bash
sudo yum install screen -y
```

## 6. screen の利用
タブ分け（docker compose, php 編集画面, sql 入力画面など）がおすすめ。

## 7. screen 設定の変更
```bash
vim ~/.screenrc
```

内容：
```
hardstatus alwayslastline "%{= bw}%-w%{= wk}%n%t*%{-}%+w"
```

## 8. Docker のインストール
```bash
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user
```

## 9. Docker Compose のインストール
```bash
sudo mkdir -p /usr/local/lib/docker/cli-plugins/
sudo curl -SL https://github.com/docker/compose/releases/download/v2.36.0/docker-compose-linux-x86_64   -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
docker compose version
```

## 10. プロジェクトディレクトリの作成
```bash
mkdir dockertest
cd dockertest
```

## 11. docker-compose.yml 作成
```bash
vim compose.yml
```

内容：
```yaml
services:
  web:
    image: nginx:latest
    ports:
      - 80:80
    volumes:
      - ./nginx/conf.d/:/etc/nginx/conf.d/
      - ./public/:/var/www/public/
      - image:/var/www/upload/image/
    depends_on:
      - php
  php:
    container_name: php
    build:
      context: .
      target: php
    volumes:
      - ./public/:/var/www/public/
      - image:/var/www/upload/image/
  mysql:
    container_name: mysql
    image: mysql:8.4
    environment:
      MYSQL_DATABASE: example_db
      MYSQL_ALLOW_EMPTY_PASSWORD: 1
      TZ: Asia/Tokyo
    volumes:
      - mysql:/var/lib/mysql
    command: >
      mysqld
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_unicode_ci
      --max_allowed_packet=4MB
  redis:
    container_name: redis
    image: redis:latest
    ports:
      - 6379:6379
volumes:
  mysql:
  image:
```

## 12. Nginx 設定
```bash
mkdir -p nginx/conf.d
vim nginx/conf.d/default.conf
```

内容：
```
nginx
server {
    listen       0.0.0.0:80;
    server_name  _;
    charset      utf-8;
    client_max_body_size 6M;

    root /var/www/public;

    location ~ \.php$ {
        fastcgi_pass  php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include       fastcgi_params;
    }

    location /image/ {
        root /var/www/upload;
    }
}
```

## 13. PHP Dockerfile 作成
```bash
vim Dockerfile
```

内容：
```dockerfile
FROM php:8.4-fpm-alpine AS php

RUN apk add --no-cache autoconf build-base \
    && yes '' | pecl install redis \
    && docker-php-ext-enable redis

RUN docker-php-ext-install pdo_mysql

RUN install -o www-data -g www-data -d /var/www/upload/image/

COPY ./php.ini ${PHP_INI_DIR}/php.ini
```
## SSHを一度ログアウトし、もう一度ログイン
## 14. MySQL テーブル作成
```bash
docker compose exec mysql mysql example_db
```

```sql
CREATE TABLE `access_logs` (
  `id` INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
  `user_agent` TEXT NOT NULL,
  `remote_ip` TEXT NOT NULL,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

```sql
CREATE TABLE `bbs_entries` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `user_id` INT UNSIGNED NOT NULL,
  `body` TEXT NOT NULL,
  `image_filename` TEXT DEFAULT NULL,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

```sql
CREATE TABLE `user_relationships` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `followee_user_id` INT UNSIGNED NOT NULL,
  `follower_user_id` INT UNSIGNED NOT NULL,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

```sql
CREATE TABLE `users` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `name` TEXT NOT NULL,
  `email` TEXT NOT NULL,
  `password` TEXT NOT NULL,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);

ALTER TABLE `users` ADD COLUMN icon_filename TEXT DEFAULT NULL;

ALTER TABLE `users` ADD COLUMN introduction TEXT DEFAULT NULL;

ALTER TABLE `users` ADD COLUMN cover_filename TEXT DEFAULT NULL;

ALTER TABLE `users` ADD COLUMN birthday DATE DEFAULT NULL;
```

## 15. phpファイルの作成（新しくタブを作って行う。）
### ※sshでEC2インスタンスに入らず、powershell上で行ってください。
#### ①githubにあるリポジトリをzipで圧縮し、解凍する。
#### ②解凍後、publicディレクトリのパスをコピー
```bash
scp -i {秘密鍵のファイルパス} -r {publicディレクトリのファイルパス} ec2-user@{IPアドレス}:/home/ec2-user/dockertest
```
## 16.権限変更
```bash
chmod 755 public/
chmod 644 public/*.php
chmod 755 public/setting/
chmod 644 public/setting/*.php
```

## 17. コンテナ起動
```bash
docker compose up --build
```
（必要に応じて SSH 再ログイン推奨）

## 18. ページ確認
```
http://{IPアドレス}/signup.php
```
