# artela-node-setup
SETUP
```console
sudo apt update
sudo apt install -y curl git jq lz4 build-essential

# İnstalling GO

sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile

# repo clone
cd && rm -rf artela
git clone https://github.com/artela-network/artela
cd artela
git checkout v0.4.7-rc6

# make binary(this place can be a bit of a wait.)

make install

# node CLI configuration

artelad config chain-id artela_11822-1
artelad config keyring-backend test
artelad config node tcp://localhost:26657

# Initialize the node

artelad init "name" --chain-id artela_11822-1

# Download genesis and addrbook.

curl -L https://snapshots-testnet.nodejumper.io/artela-testnet/genesis.json > $HOME/.artelad/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/artela-testnet/addrbook.json > $HOME/.artelad/config/addrbook.json
```
required settings
```console
# seeds
sed -i -e 's|^seeds *=.*|seeds = "211536ab1414b5b9a2a759694902ea619b29c8b1@47.251.14.47:26656,d89e10d917f6f7472125aa4c060c05afa78a9d65@47.251.32.165:26656,bec6934fcddbac139bdecce19f81510cb5e02949@47.254.24.106:26656,32d0e4aec8d8a8e33273337e1821f2fe2309539a@47.88.58.36:26656,1bf5b73f1771ea84f9974b9f0015186f1daa4266@47.251.14.47:26656"|' $HOME/.artelad/config/config.toml

# gas price
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "20000000000uart"|' $HOME/.artelad/config/app.toml

# prunings
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.artelad/config/app.toml

# download snapshot(optional)
curl "https://snapshots-testnet.nodejumper.io/artela-testnet/artela-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.artelad"
```
node initialization 
```console
sudo tee /etc/systemd/system/artelad.service > /dev/null << EOF
[Unit]
Description=Artela node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which artelad) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable artelad.service
sudo journalctl -u artelad -f --no-hostname -o cat
```
thats all.
