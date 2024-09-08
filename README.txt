##### Install BP

sudo apt-get update
sudo apt-get install -y psmisc zip unzip curl jq libncurses5
sudo apt-get update

git clone git@github.com:GlobalForceIO/bp.git /var/server/bp
cd /var/server/bp && git pull origin master

##### If error
libicuuc.so.60: cannot open shared
wget http://security.ubuntu.com/ubuntu/pool/main/i/icu/libicu60_60.2-3ubuntu3.2_amd64.deb
sudo apt-get install ./libicu60_60.2-3ubuntu3.2_amd64.deb

error while loading shared libraries: libssl.so.1.1
wget http://nz2.archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.22_amd64.deb
sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2.22_amd64.deb

##### From snapshot
rm -r /var/server/bp/traces/* -R
rm -r /var/server/bp/blocks/* -R
rm -r /var/server/bp/state-history/* -R
rm -r /var/server/bp/datadir/* -R
/var/server/bp/nodeos --snapshot /var/server/bp/snapshots/snapshot-5.04.24.bin --config /var/server/bp/config.ini --data-dir /var/server/bp/datadir --verbose-http-errors

cd /var/server/bp
mkdir ~/eosio-wallet
### Make dirs for user
mkdir ~/.config
mkdir ~/.config/systemd
mkdir ~/.config/systemd/user
cp -rf /var/server/bp/*.service ~/.config/systemd/user/
mkdir ~/bin
cp -rf /var/server/bp/cleos ~/bin/
### Check exist path in PATH
	echo $PATH
	##If not exist - add
	export PATH=$PATH:~/bin/
systemctl --user daemon-reload
systemctl --user enable KEOSD
systemctl --user restart KEOSD
systemctl --user enable NODEOS
systemctl --user restart NODEOS

journalctl --user -f -u NODEOS

# Sync from another node
/var/server/bp/nodeos --config /var/server/bp/config.ini --data-dir /var/server/bp/datadir --verbose-http-errors --delete-all-blocks --disable-replay-opts