### VAGRANT BOX作成ノート（その１）
[Devian8Jpdev.box](https://app.vagrantup.com/ken2tec/boxes/Debian8JpDev) virtualbox  Hosted by Vagrant Cloud (1.23 GB)
当時はサイトにアップしないと再利用できないと勘違いしていた。 fish shellなんてものまで入れてしまい少し汚くなったけど、それでも展開後のHDDが3GB前後。
ApacheからNginxに変更してmariadbやmongodbとかサーバーアプリは一通り揃えてからPHPは当時最新の7.1を結構苦労してインストール。vagrant upするだけですぐに遊べるから結構イケてると思う。

 |機能|内容|
:---|:---
|HDD|VMDK500GB|
|Memory|1024MB|
|Video Memory|32MB|
|Base ISO|Debian-Jessie net-install ISO|
|Root User:Pass|root:vagrant|
|User:Pass|vagrant:vagrant|
|ロケール|ja/JP|
|Desktop|XFCE|
|nginx|1.10.3|
|php|7.1.3|
|mariadb|10.0.30|
|mongod|3.4.2|
|redis-server|2.8.17|
|Adminer(root/password)|http://127.0.0.1/Adminer|
|AdminerEditor(root/password)|http://127.0.0.1/AdminerEditor|
|RockMongo(admin/admin)|http://127.0.0.1/rockmongo|
|phpRedisAdmin|http://127.0.0.1/phpredisadmin|


## 作成開始
まずはnetInstall ISOからVirtual Boxで普通にインストール。aptitudeをインストールしたのはapt-getから見つからないアプリがあったから。あとは初期のセッティングに必要なものをゲット。
```bash
	apt-get update & apt-get dist-upgrade
	apt-get install aptitude
	aptitude update
	aptitude upgrade
	aptitude install sudo build-essential module-assistant vim tree curl git gnome-system-monitor xdg-user-dirs-gtk zip
	m-a prepare
```
## VirtualBox でDevices > Install Guest Additions
```bash
	mount /media/cdrom0
	bash /media/cdrom0/VBoxLinuxAdditions.run
```
## vagrantユーザに権限設定
```bash
	vim /etc/sudoers
		+	vagrant ALL=(ALL) NOPASSWD: ALL
```
## 再起動してフォルダ名表記の英語化とJSTへの設定変更
```bash
	env LANGUAGE=C LC_MESSAGES=C xdg-user-dirs-gtk-update
	date
	strings /etc/localtime
	cp /etc/sysconfig/clock /etc/sysconfig/clock.org
	echo -e 'ZONE="Asia/Tokyo"\nUTC=false' > /etc/sysconfig/clock
	cp /etc/localtime /etc/localtime.org
	ln -sf  /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
	date
	strings /etc/localtime
```
## SSH認証鍵の取得とSSHサーバーのインストール、システム起動の高速化
```bash
	mkdir .ssh
	chmod 700 .ssh
	cd .ssh
	curl -L -o authorized_keys http://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub
	chmod 600 authorized_keys
	sudo aptitude install openssh-server

	sudo vim /etc/ssh/sshd_config
		+	UseDNS no
	sudo vim /etc/default/grub
		+	GRUB_HIDDEN_TIMEOUT=0
	sudo update-grub
```
## NginX
Debian/jessie標準アプリケーションにNginxが含まれていなかった
```bash
	wget "http://nginx.org/keys/nginx_signing.key"
	sudo apt-key add nginx_signing.key
	sudo cp /etc/apt/sources.list /etc/apt/sources.list.org
	sudo vim /etc/apt/sources.list
	+	deb http://nginx.org/packages/debian/ jessie nginx
	+	deb-src http://nginx.org/packages/debian/ jessie nginx
	sudo apt-get update
	apt-cache show nginx
	sudo apt-get install nginx
	sudo nginx -v
```
## PHP7.1
PHP7.1も同様
```bash
	sudo apt update
	sudo apt install apt-transport-https lsb-release ca-certificates -y
	sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
	echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
	sudo apt update
	sudo apt install php7.1 -y
	sudo apt-get -y install php7.1-gd php7.1-fpm php7.1-mysql php7.1-mbstring php7.1-dev php7.1-mongodb php-pear
```
## MariaDb
```bash
	sudo apt-get -y install mariadb-server
	sudo mysql_secure_installation
	#	rootパスワードを空白からpasswordに変更、以下デフォルトで進行
	sudo vim /etc/mysql/conf.d/mariadb.cnf
		[client]
	*	default-charactor-set = utf8
		[mysqld]
	*	character-set-server = utf8
	sudo systemctl restart mysql
	mysql -u root -p
		select user,host,password from mysql.user;
		show databases;
		exit
```
## FISHシェルの導入
面白そうなので入れてみたけど使わないかな、ややこしいからbashでいい。
```bash
	sudo wget -qO - http://download.opensuse.org/repositories/shells:fish:release:2/Debian_8.0/Release.key | sudo apt-key add -
	echo 'deb http://download.opensuse.org/repositories/shells:/fish:/release:/2/Debian_8.0/ /' | sudo tee /etc/apt/sources.list.d/fish.list
	sudo apt-get update
	sudo apt-get install -y fish
	git clone https://github.com/junegunn/fzf.git ~/.fzf
	~/.fzf/install
# install fisherman, oh-my-fish
	curl -Lo ~/.config/fish/functions/fisher.fish --create-dirs git.io/fisher
	curl -L http://get.oh-my.fish | fish
	chsh -s /usr/bin/fish
	exit
	fish
# fisherman install
	fisher install z fzf decors/fish-ghq fnm nvm shark anicode choices shellder bass menu spin

# Plot a sequence of numbers test
	seq 8 | shark
# Plot two set of numbers and treat each as an individual dataset test
	printf "%s\n" "1 2 3 4 5" "100 200 300 400 500" | shark -n
# Draw a random sequence of numbers test
	for i in (seq 42)
	    random
	end | shark
```
## FISH入れたら合わせ技でPowerLineFontsも入れるといいらしい
```bash
	wget https://github.com/powerline/powerline/raw/develop/font/PowerlineSymbols.otf
	wget https://github.com/powerline/powerline/raw/develop/font/10-powerline-symbols.conf
	mkdir /,fonts
	mv PowerlineSymbols.otf ~/.fonts/
	fc-cache -vf ~/.fonts/
	mv 10-powerline-symbols.conf ~/.config/fontconfig/conf.d/
# change bobthefish theme on oh-my-fish
	omf install bobthefish
```
## FNMでNODEバージョン管理をすると…
```bash
	fnm u latest
	node -v
	npm -v
```
## HTTPDをapache2からnginxに変更
```bash
	sudo systemctl disable apache2
	sudo nginx
	sudo systemctl unmask nginx.service
	sudo systemctl enable nginx
# httpd check
	ab -n 100 -c 1- http://127.0.0.1/
```
## NGINX設定の調整
```bash
	sudo vim /etc/php/7.1/fpm/pool.d/www.conf
	*	user = nginx
	*	group = nginx
	*	listen.owner = nginx
	*	listen.group = nginx
```
default.confファイルをいろいろ調整するけど詳細は省略
「root /var/www/html」と「fastcgi_pass unix:run/php/php7.1-fpm.sock」あたりに注意
```bash
	sudo vim /etc/nginx/conf.d/default.conf
```
## WEB操作環境を一通り揃える
```bash
	sudo apt-get install redis-server
# mongodb
	sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
	echo "deb http://repo.mongodb.org/apt/debian jessie/mongodb-org/3.4 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
	sudo apt-get update
	sudo apt-get install -y mongodb-org
	sudo service mongod start
# Adminer
	sudo mkdir /usr/local/src/Adminer
	cd /usr/local/src/Adminer
	sudo wget https://github.com/vrana/adminer/releases/download/v4.3.0/adminer-4.3.0.php
	sudo mv adminer-4.3.0.php index.php
	sudo ln -s /usr/local/src/Adminer /var/www/Adminer
# AdminerEditor
	sudo mkdir /usr/local/src/AdminerEditor
	cd /usr/local/src/AdminerEditor
	sudo wget https://github.com/vrana/adminer/releases/download/v4.3.0/editor-4.3.0.php
	sudo mv editor-4.3.0.php index.php
	sudo ln -s /usr/local/src/AdminerEditor /var/www/AdminerEditor
# phpRedisAdmin
	cd /usr/local/src
	sudo composer create-project -s dev erik-dubbelboer/php-redis-admin phpredisadmin
	sudo ln -s /usr/local/src/phpredisadmin /var/www/phpredisadmin
# rockmongo
	sudo mkdir /usr/local/src
	cd /usr/local/src
	sudo wget https://github.com/iwind/rockmongo/archive/master.zip
	sudo unzip master.zip
	sudo mv rockmongo-master rockmongo
	mongo
		use admin
		db.createUser({user: "admin",  pwd: "admin", roles:[{role: "userAdminAnyDatabase", db: "admin"}]})
		exit
	sudo vim config.php
	# 29行目をtrueに。
		$MONGO["servers"][$i]["mongo_auth"] = true; //false;
	# 31行目をfalseに。
		$MONGO["servers"][$i]["control_auth"] = false; //true;
	sudo ln -s /usr/local/src/rockmongo /var/www/rockmongo
```
## RockMongoにはPHP拡張インターフェースが必要
```bash
	curl -sS https://getcomposer.org/installer | php
	sudo mv composer.phar /usr/local/bin/composer
	composer self-update
# pecl
	sudo apt-get install -y autoconf g++ make openssl libssl-dev libcurl4-openssl-dev
	sudo apt-get install -y libcurl4-openssl-dev pkg-config libsasl2-dev
	sudo pecl install mongodb
# mongo extension
	cd /usr/local/src/rockmongo
	composer require mongodb/mongodb
```
## SDKMAN(Java系)は下記で一発、GVM使うときはaliasに注意
```bash
	curl -s api.sdkman.io | bash
```
## 最後にconky、それからゴミファイルのクリーン用にbleachbitを導入
```bash
	sudo apt-get install bleachbit conky conky-all lm-sensors hddtemp
```
## 仕上げ
ここまでやったらbleachbitをrootでフロント実行してから「vagrant package」でBOXにする。
- 「キャッシュ、メモリー」のチェックは外しましょう
- 時間がかかるからシステムモニターを表示しておくとHDD空容量の増減が確認できて安心
