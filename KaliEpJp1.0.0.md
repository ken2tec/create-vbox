### VAGRANT BOX作成ノート（KaliEpJp v1.0.0）
[Devian8Jpdev.box](https://app.vagrantup.com/ken2tec/boxes/Debian8JpDev) に続きこれから使い続けるディストリビューションとしてベネレーションテストツールの揃ったKali Linuxを選択。最終的にエンタープライズ環境に仕上げるからKaliEpJpと命名。

 |機能|内容|
:---|:---
|Version|1.0.0|
||Kali2018.1Full+日本語環境+マルチメディアアプリケーション|
|Vagrant box size|4.85GB|
||展開後vmdk約12GB|
|HDD|VMDK500GB|
|Memory|2048MB|
|Video Memory|128MB|
|ロケール|ja/JP|
|Desktop|LXDE(M), LXQT, XFCE, CINAMON, GNOME|
|Host Name|kali|
|Domain Name|Non|
|Root User:Pass|root : toor|
|User:Pass|vagrant : vagrant|

## 作成開始
Kaliは他のディストリビューションとネットワーク、自動起動の制限など微妙に異なる箇所があり何度も試行錯誤を繰り返した。最終的にはVirtualBox GuestAdditionでトラブルのない2017.3のovaをベースにしている。
## ベースISO(kali-linux-2017.3-vbox-amd64.ova)をKALIサイトよりダウンロード。VirtualBoxでHDD(Kali20173base.vmdk)にインポート。
## VboxManageでディスクを可変容量500GBに変換
```bash
	CD /D C:\Program Files\Oracle\VirtualBox
	.\vboxmanage.exe clonehd {vmdkパス}\Kali20173base.vmdk {vmdkパス}\Kali20173base.vdi --format vdi
	.\VBoxManage.exe modifyhd {vmdkパス}\Kali20173base.vdi --resize 512000
	.\vboxmanage.exe clonehd {vmdkパス}\Kali20173base.vdi {vmdkパス}\Kali20173base.vmdk --format vmdk
```
##  GPartedの入っている適当なLive Linuxを起動して変換したvmdkパーティションを操作
```bash
	#スワップパーティションを削除
	/dev/sda5	linux-swap
	/dev/sda2	extended
	#スワップ領域分を残して主パーティションをリサイズ
	Resize /dev/sda1 58GB to 498GB								
	#スワップ領域を再アロケート
	/dev/sda2	extended	2GB
	/dev/sda5	linux-swap	2GB
	linux-swap	activate
	#UUIDが変化しているため修復
	blkid
	vim /etc/fstab

	shutdown -r now

```
## リポジトリ操作keyが古くapt-get update失敗するため更新してから一旦update
```bash
	wget -q -O - https://archive.kali.org/archive-key.asc | apt-key add 
```
## Guest Addition未導入の場合
```bash
	apt-get install -y virtualbox-guest-x11
	#reboot
	apt-get update
	apt-get install -y linux-headers-$(uname -r)
	#Media insert
	cp /media/cd-rom/VBoxLinuxAdditions.run /root/
	chmod 755 /root/VBoxLinuxAdditions.run
	cd /root
	./VBoxLinuxAdditions.run
```
## 日本語化(関連パッケージ導入、設定)
```bash
	apt-get install -y task-japanese task-japanese-desktop
	#右上▼アイコンから設定画面
		Details - Date & Time , TimeZone to JST
		Region & LANGUAGE , 日本語
		Input SourcesでJapaneseの追加と順序の変更
```
## 初期設定関連(Kaliリポジトリに入ってないアプリケーションがあるからDebian標準リポジトリを追加しておく)
```bash
	vim /etc/apt/sources.list
	+	deb http://ftp.jp.debian.org/debian/stretch main contrib non-free
	apt-get update
	#フォルダー名を英語に変更
	apt-get install module-assistant xdg-user-dirs-gtk
	m-a prepare
	env LANGUAGE=C LC_MESSAGES=C xdg-user-dirs-gtk-update
	#デスクトップ導入
	tasksel
		GNOME CINAMON XFCE LXDE LXQT Desktop Select.
```
## サーバー再起動、LXDEデスクトップをチョイスしてログイン
## スクリーンセーバー無効化
	設定 > 設定マネージャー > スクリーンセーバー
## Mozc無効化
	アプリケーション > 設定 > 入力メソッド(青アイコン) > 使用可能にする入力方式を編集 > Mozcを無効に
## 日本語フォント
```bash
	apt-get install fonts-ipamj-mincho fonts-takao fonts-mplus
		fonts-migmix nts-mmcedar fonts-umeplus fonts-horai-umefont
		fonts-motoya-l-maruberi fonts-motoya-l-cedar
```
## 各種アプリケーション
```bash
	apt-get install tree gnome-system-monitor bleachbit conky conky-all
		geany geany-plugins gwenview chromium darktable scribus audacity
		mypaint blender librecad jackd2 jack-tools rdour qtractor hydrogen
		yoshimi yoshimi-data rakarrack guitarix gladish musescore w3m
		hexedit medit clamtk lmms
	#haroopadはdebから
	cd /tmp
	wget https://bitbucket.org/rhiokim/haroopad-download/downloads/haroopad-v0.13.2-x64.deb
	dpkg -i haroopad-v0.13.2-x64.deb

	apt-get upgrade

```
## vagrantユーザー作成
```bash
	useradd -m vagrant
	passwd vagrant
	usermod -a -G sudo vagrant
```
## SSH認証鍵の取得とSSHサーバーのインストール、システム起動の高速化
```bash
	su vagrant
	mkdir $HOME/.ssh
	chmod 700 $HOME/.ssh
	cd $HOME/.ssh
	curl -k -L -o authorized_keys 'https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub'
	chmod 600 $HOME/.ssh/authorized_keys
	su
		toor
	systemctl enable ssh.service
	sudo vim /etc/ssh/sshd_config
	+	Port 22
	+	PermitRootLogin yes		#ssh root password login ok
	+	PubKeyAuthentication yes
	+	AuthorizedKeysFile %h/.ssh/authorized_keys
	+	PermitEmptyPasswords no
	+	UseDNS no
	#Kali基本設定要注意点1 update-rc.d ssh enable
	vim /usr/sbin/update-rc.d
	-	ssh disable
	+	ssh enable
	#Kali基本設定要注意点2 network/interfaces
	vim /etc/network/interfaces
	+	# The primary network interface
	+	allow-hotplug eth0
	+	auto eth0
	+	iface eth0 inet dhcp
	+
	+	allow-hotplug eth1
	#システム起動時のウェイト短縮
	sudo vim /etc/default/grub
		+	GRUB_HIDDEN_TIMEOUT=0
	sudo update-grub
	#vagrant up時にpassword無しでsudo出来るように設定
	sudo visudo
	*	%sudo ALL=(ALL:ALL) NOPASSWD:ALL
	+	vagrant ALL=(ALL)   NOPASSWD: ALL
	*	# requiretty # コメントアウトでttyなくsudoできる
	# 一旦cacheクリアなどを行い再起動
	apt autoremove
	apt-get autoclean
	apt-get clean
	shutdown -r now

```
## Kali2018.1Fullへのアップグレード(2018/2/13時点で)
```bash
	apt-get update
	apt-get install kali-linux-full
	apt update && apt full-upgrade
	apt autoremove
	apt-get autoclean
	apt-get clean
	shutdown -r now
```
## 仕上げ
bleachbitをrootでフロント実行してから「vagrant package」でBOX化。
```bash
	#VirtualBox設定でUSBコントローラーが2.0以上なら1.1に変更しておく
	vagrant package --base KaliEpJp --output KaliEpJpV10.box
```
