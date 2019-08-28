# Miniflux
docker自建Miniflux 轻便好用的 RSS 服务

安装docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum-config-manager --enable docker-ce-edge
sudo yum-config-manager --enable docker-ce-test
sudo yum install docker-ce -y
sudo systemctl start docker
sudo systemctl enable docker
sudo docker run hello-world

安装Docker Compose
yum -y install docker-compose

vim docker-compose.yml
version: '3'
services:
  miniflux:
    image: miniflux/miniflux:2.0.16
    ports:
      - "80:8080"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://miniflux:secret@db/miniflux?sslmode=disable
      - RUN_MIGRATIONS=1
      - POLLING_FREQUENCY=60
      - CREATE_ADMIN=1
      - ADMIN_USERNAME=admin
      - ADMIN_PASSWORD=test123
    restart: unless-stopped
  db:
    image: postgres:10.1
    environment:
      - POSTGRES_USER=miniflux
      - POSTGRES_PASSWORD=secret
    volumes:
      - miniflux-db:/var/lib/postgresql/data
volumes:
  miniflux-db:


docker-compose up -d
访问 http://<ip地址>:<端口号>使用配置文件中填写的账号密码即可登陆了


域名和 HTTPS 支持
安装caddy
wget -N --no-check-certificate https://www.moerats.com/usr/shell/Caddy/caddy_install.sh && chmod +x caddy_install.sh && bash caddy_install.sh

vim /usr/local/caddy/Caddyfile
https://<解析到该ip的域名> {
 tls <你的邮箱>
 proxy / <ip地址:miniflux对外端口> {
    header_upstream Host {host}
    header_upstream X-Real-IP {remote}
    header_upstream X-Forwarded-For {remote}
    header_upstream X-Forwarded-Proto {scheme}
  }
 log /var/log/caddy.log
 gzip
}

保存退出后，重启 Caddy 就可以用 https 访问域名了
/etc/init.d/caddy restart
