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

## Install Cosmovisor

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
sudo tee /etc/systemd/system/initia.service > /dev/null << EOF
[Unit]
Description=initia node service
After=network-online.target
 
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.initia"
Environment="DAEMON_NAME=initiad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
 
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable initia
```

## Initialize Node
* Replace `YourName` with your Moniker
```
MONIKER="YourName"
```
```
initiad init $MONIKER --chain-id initation-1
```
