# 課題ページ表示手順
## 1.Docker 本体のインストール
```sh
sudo yum install -y docker
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -a -G docker ec2-user
```
## 2.Docker Composeのインストール
```sh
sudo mkdir -p /usr/local/lib/docker/cli-plugins/
sudo curl -SL https://github.com/docker/compose/releases/download/v2.36.0/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
```

## 3.gitのインストール
```sh
sudo yum install git -y
```
## 4.ソースコードの取得
```sh
git clone https://github.com/kazu16916/web_saisyukadai.git
```
## 5.起動
```sh
cd dockertest
docker compose build
docker compose up -d
```
## 6.アクセス
```
http://44.192.10.36/login.php
```
