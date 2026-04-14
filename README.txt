##### Install BP

sudo apt-get update
sudo apt-get install -y psmisc zip unzip curl jq libncurses5
sudo apt-get update

##### Process manager
ls /var/lib/systemd/linger
loginctl list-users
loginctl enable-linger root
loginctl user-status root

git clone git@github.com:GlobalForceIO/bp.git --branch v5.0.3c /var/server/bp
cd /var/server/bp

##### If error
libicuuc.so.60: cannot open shared
wget http://security.ubuntu.com/ubuntu/pool/main/i/icu/libicu60_60.2-3ubuntu3.2_amd64.deb
sudo apt-get install ./libicu60_60.2-3ubuntu3.2_amd64.deb

error while loading shared libraries: libssl.so.1.1
wget http://nz2.archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.22_amd64.deb
sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2.22_amd64.deb

##### From snapshot
rm -r /var/server/bp/datadir/* -R
/var/server/bp/nodeos --snapshot /var/server/bp/datadir/snapshots/snapshot-5.04.24.bin --config /var/server/bp/config.ini --data-dir /var/server/bp/datadir --genesis-json /var/server/nodeos/genesis.json --verbose-http-errors

cd /mnt/bp
mkdir ~/eosio-wallet
### Make dirs for user
mkdir ~/.config
mkdir ~/.config/systemd
mkdir ~/.config/systemd/user
cp -rf /mnt/bp/*.service ~/.config/systemd/user/
mkdir ~/bin
cp -rf /mnt/bp/cleos ~/bin/
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

systemctl --user stop NODEOS

# Replay this node
/var/server/bp/nodeos --config /var/server/bp/config.ini --data-dir /var/server/bp/datadir --verbose-http-errors --replay-blockchain --disable-replay-opts

# Sync from another node
/var/server/bp/nodeos --config /var/server/bp/config.ini --data-dir /var/server/bp/datadir --verbose-http-errors --delete-all-blocks --disable-replay-opts

## Test API
curl -X 'POST' 'http://127.0.0.1:18880/v1/trace_api/get_block' -d '{"block_num": 1}' | jq
curl -X 'POST' 'http://127.0.0.1:18880/v1/chain/get_block' -d '{"block_num_or_id": 1}' | jq
curl -X 'POST' 'http://127.0.0.1:18880/v1/history/get_transaction' -d '{"id": "a7cdbb87465514511e80acb20237f40d0e3a362042a64cbc32a5f9720a1b1299"}' | jq
