INITIA Validator Node


•HARDWARE REQUIREMENT :
```
- Memory: 16 GB RAM
- CPU: 4 cores
- Disk: 1 TB SSD
- Bandwidth: 1 Gbps
- Linux amd64 arm64 (Ubuntu LTS release)
```

•INSTALL DEPENDENCIES :
```
sudo apt update && sudo apt upgrade -y
sudo apt install screen curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

•INSTALL GO : 
```
cd $HOME
VER="1.22.2"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

•SET VARIABLE : 
```
echo "export WALLET="yournamewallet"" >> $HOME/.bash_profile
echo "export MONIKER="yournamenode"" >> $HOME/.bash_profile
echo "export INITIA_CHAIN_ID="initiation-1"" >> $HOME/.bash_profile
echo "export INITIA_PORT="21"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
Please Change "yournamewallet" & "yournamenode"

•INSTALL INITIA LIBRARY :
```
cd $HOME
rm -rf initia
git clone https://github.com/initia-labs/initia.git
cd initia
git switch -c v0.2.14
git checkout v0.2.14
make install
```

•CONFIG INIT, DOWNLOAD Genesis.json & addrbook.json :
```
initiad init $MONIKER
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${INITIA_PORT}657\"|" $HOME/.initia/config/client.toml
wget -O $HOME/.initia/config/genesis.json https://testnet-files.itrocket.net/initia/genesis.json
wget -O $HOME/.initia/config/addrbook.json https://testnet-files.itrocket.net/initia/addrbook.json
```

•SET SEEDS, PEERS, CUSTOM PORTS On app.toml & config.toml , SET PRUNING WITH GAS SPEND : 
```
SEEDS="cd69bcb00a6ecc1ba2b4a3465de4d4dd3e0a3db1@initia-testnet-seed.itrocket.net:51656"
PEERS="aee7083ab11910ba3f1b8126d1b3728f13f54943@initia-testnet-peer.itrocket.net:11656,d17d2d48b4741b21b16cba7aa5a0496151dec2e3@65.109.37.125:26656,767fdcfdb0998209834b929c59a2b57d474cc496@207.148.114.112:26656,9f0ae0790fae9a2d327d8d6fe767b73eb8aa5c48@176.126.87.65:22656,e43ce5800e48df7917942191c95276cb88bdd699@212.90.121.127:51656,7317b8c930c52a8183590166a7b5c3599f40d4db@185.187.170.186:26656,626e082b9a5a1cf99dbf8cbc1cb702ee7c1e9991@64.225.102.23:51656,b79874ca9607e5d4a3fd730617cca863ff9f590e@5.78.116.66:26656,b8fcc8886246b3bd6058583a8017a7f987d7437e@185.182.186.46:26656,00bf6d94bc8bae9d75c29a9bb198eaa401d34f4d@95.216.216.74:15656,a45314423c15f024ff850fad7bd031168d937931@162.62.219.188:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.initia/config/config.toml
sed -i.bak -e "s%:1317%:${INITIA_PORT}317%g;
s%:8080%:${INITIA_PORT}080%g;
s%:9090%:${INITIA_PORT}090%g;
s%:9091%:${INITIA_PORT}091%g;
s%:8545%:${INITIA_PORT}545%g;
s%:8546%:${INITIA_PORT}546%g;
s%:6065%:${INITIA_PORT}065%g" $HOME/.initia/config/app.toml
sed -i.bak -e "s%:26658%:${INITIA_PORT}658%g;
s%:26657%:${INITIA_PORT}657%g;
s%:6060%:${INITIA_PORT}060%g;
s%:26656%:${INITIA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${INITIA_PORT}656\"%;
s%:26660%:${INITIA_PORT}660%g" $HOME/.initia/config/config.toml
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.initia/config/app.toml
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.15uinit,0.01uusdc"|g' $HOME/.initia/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.initia/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.initia/config/config.toml
```

•CREATING SERVICE FILE :
```
sudo tee /etc/systemd/system/initiad.service > /dev/null <<EOF
[Unit]
Description=Initia node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.initia
ExecStart=$(which initiad) start --home $HOME/.initia
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

•DOWNLOAD SNAPSHOT :
```
initiad tendermint unsafe-reset-all --home $HOME/.initia
if curl -s --head curl https://testnet-files.itrocket.net/initia/snap_initia.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://testnet-files.itrocket.net/initia/snap_initia.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.initia
    else
  echo no have snap
fi
```

•RUNNING SERVICE :
```
sudo systemctl daemon-reload
sudo systemctl enable initiad
sudo systemctl restart initiad
```
Using Screen to run Log
```
screen -R in
sudo journalctl -u initiad -f
```
use Ctrl+A+D to close screen

•CREATE WALLET & RECOVER
```
initiad keys add $WALLET
```
```
initiad keys add $WALLET --recover
```

•SET VARIABLE FOR WALLET & VALOPER :
```
WALLET_ADDRESS=$(initiad keys show $WALLET -a)
VALOPER_ADDRESS=$(initiad keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile
```

•CHECKING NODE SYNC :
```
initiad status 2>&1 | jq
```
if catching_up = false , Your Node is Fully Sync

•CHECK BALANCE : 
```
initiad query bank balances $WALLET_ADDRESS
```

CREATE VALIDATOR, AFTER NODE FULLY SYNC :
```
initiad tx staking create-validator \
--amount 1000000uinit \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(initiad tendermint show-validator) \
--moniker "yourmoniker" \
--identity "keybase ID" \
--website "Use your twitter / your github" \
--details "I love INITIA ❤️" \
--chain-id initiation-1 \
--gas auto --fees 80000uinit \
-y
```
Please change "yourmoniker" with your node name, "keybase id" create on https://keybase.io paste your id,
for website you can use twitter / github or etc.

GoodLuck!! you can Delegate To Me : https://app.testnet.initia.xyz/validator/initvaloper1wnpv6359y4exw0gnx56gt3uvlgq870m0fly0as




