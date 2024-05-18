# Initia Node Installation Guide
![Octocat](https://lh3.googleusercontent.com/-dysDpoAiAThTgPyiWS-DcKMoXU_lroKdn_6yiKlWzKturY2f2XVxDI--av1xg6KzeTRjcmuyWVo0Amv8sJ4UVsLzQ=s1280-w1280-h800 "Github logo") 

I made a little guide on how to install Node on the Initia service. Follow and pay attention to every command!

## Please Stake to my Validator & Thank you so much about it :)

<https://app.testnet.initia.xyz/validator/initvaloper1lfeqcpturraf3767akhkfprwdvntwtkf4j6dp0>

## Minimum Requirements
 * **CPU: 4 cores or more**
 
 * **Memory: 8GB RAM or more**
 
 * **Disk: 200GB SSD Storage**

## Install dependencies Required

```
sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## Install Golang

```
ver="1.22.2"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

## Build Binary

```
git clone -b v0.2.14 https://github.com/initia-labs/initia.git
cd initia
git checkout v0.2.14
make install
mkdir -p $HOME/.initia/cosmovisor/genesis/bin
cp $HOME/go/bin/initiad $HOME/.initia/cosmovisor/genesis/bin/
sudo ln -s $HOME/.initia/cosmovisor/genesis $HOME/.initia/cosmovisor/current -f
sudo ln -s $HOME/.initia/cosmovisor/current/bin/initiad /usr/local/bin/initiad -f
cd $HOME
```

## Initialize Node
* Replace `YourName` with your Moniker
```
MONIKER="YourName"
```
```
initiad init $MONIKER --chain-id initation-1
```
## Download Genesis & Addrbook
```
wget https://initia.s3.ap-southeast-1.amazonaws.com/initiation-1/genesis.json -O $HOME/.initia/config/genesis.json
wget https://rpc-initia-testnet.trusted-point.com/addrbook.json -O $HOME/.initia/config/addrbook.json
```
## Configure
```
seeds=$(curl -s https://raw.githubusercontent.com/refangga1337/Initia-Testnet/main/seed.txt)
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/" $HOME/.initia/config/config.toml
peers=$(curl -s https://raw.githubusercontent.com/refangga1337/Initia-Testnet/main/peers.txt)
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.initia/config/config.toml
sed -i.bak -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.initia/config/app.toml
sed -i.bak -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.initia/config/app.toml
sed -i.bak -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" $HOME/.initia/config/app.toml
sed -i "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uinit\"/" $HOME/.initia/config/app.toml
sed -i "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.initia/config/config.toml
sed -i \
  -e 's|^chain-id *=.*|chain-id = "initation-1"|' \
  -e 's|^keyring-backend *=.*|keyring-backend = "test"|' \
  -e 's|^node *=.*|node = "tcp://localhost:10657"|' \
  $HOME/.initia/config/client.toml
```
## Custom Port
```
echo 'export initia="106"' >> ~/.bash_profile
source $HOME/.bash_profile
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://0.0.0.0:${initia}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${initia}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${initia}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${initia}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${initia}60\"%" $HOME/.initia/config/config.toml
sed -i -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://0.0.0.0:${initia}17\"%; s%^address = \":8080\"%address = \":${initia}80\"%; s%^address = \"localhost:9090\"%address = \"0.0.0.0:${initia}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${initia}91\"%; s%:8545%:${initia}45%; s%:8546%:${initia}46%; s%:6065%:${initia}65%" $HOME/.initia/config/app.toml
```
## Config Pruning
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.initia/config/app.toml
```
## set minimum gas price, enable prometheus and disable indexing
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.15uinit,0.01uusdc\"|" $HOME/.initia/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.initia/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.initia/config/config.toml
```
## Create Service File
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
## Enable & Start Service
```
sudo systemctl daemon-reload
sudo systemctl enable initiad
sudo systemctl start initiad && sudo journalctl -fu initiad -f -o cat
```

# Please wait until your nodes sync well

## Create Wallet 
```
initiad keys add wallet
```
## If you want to import your pharse
```
initiad keys add $WALLET --recover
```
# Request Test Tokens
You can Request your tokens [Here](https://faucet.testnet.initia.xyz/)

## Save wallet and validator address
```
WALLET_ADDRESS=$(initiad keys show $WALLET -a)
VALOPER_ADDRESS=$(initiad keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Check Node Status
```
initiad status 2>&1 | jq
```
## Check your Balance
```
initiad q bank balances $(initiad keys show wallet -a)
```
# *If your node is fully synchronized and your balance is sufficient, you can create a validator*

```
initiad tx mstaking create-validator \
--amount 1000000uinit \
--pubkey $(initiad tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id initiation-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.15uinit \
-y
```
