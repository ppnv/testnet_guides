![](https://pbs.twimg.com/profile_images/1569281955971072000/1TVeJn3w_400x400.jpg)
# Nois Network - Get randomness on the fly
- Website: https://nois.network/
- Twitter: https://twitter.com/NoisNetwork
- Discord: https://discord.gg/JvUrJJeGXn
- Docs: https://docs.nois.network/
## Explorers
https://explorer.nodestake.top/nois-testnet
## Official documentation
[Validator setup instructions](https://docs.nois.network/use-cases/for-validators)
## Set up your teritori fullnode (manual install)
### Update packages
```
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```
sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop wget jq build-essential nano unzip net-tools build-essential git unzip curl wget nano htop make gcc screen tar clang pkg-config libssl-dev build-essential bsdmainutils ncdu jq chrony liblz4-tool curl build-essential git wget jq make gcc tmux htop nvme-cli tar clang bsdmainutils ncdu unzip cargo screen curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y
```
### Install GO
```
cd $HOME && \
ver="1.18.4" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
### Download and build binaries
```
git clone https://github.com/noislabs/full-node.git 
cd full-node/full-node/
./setup.sh
mv out/noisd $HOME/go/bin/
```
### Config app
```
noisd config chain-id nois-testnet-002
noisd config keyring-backend test
noisd config node tcp://localhost:26657
```
### Init app (change nodename)
```
noisd init <nodename> --chain-id nois-testnet-002
```
### Download genesis
```
cd $HOME/.noisd/config/
rm genesis.json
curl -O https://raw.githubusercontent.com/noislabs/testnets/main/nois-testnet-002/genesis.json
```
### Set peers
```
PEERS=2df500525826199afc25665ee7cc45ceb86d68d7@35.193.237.242:26656,a1222dfb8641e0cb55615b75e0122d5695be1f35@35.224.189.139:26656,61be6aa87471196757ea0f7b1d7897e97b4e09c2@34.171.234.115:26656,cf16671c00eec9a9a047a5c6aa8510cb681b64b8@34.171.67.167:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.noisd/config/config.toml
```
### Config pruning (optional)
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.noisd/config/app.toml
```
### Disable indexing (optional)
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.noisd/config/config.toml
```
### Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.05unois\"/" $HOME/.noisd/config/app.toml
```
### Reset chain data
```
noisd tendermint unsafe-reset-all --home $HOME/.noisd
```
### Create service
```
sudo tee /etc/systemd/system/noisd.service > /dev/null <<EOF
[Unit]
Description=noisd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which noisd) start --home $HOME/.noisd
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
### Register and start service
```
sudo systemctl daemon-reload
sudo systemctl enable noisd
sudo systemctl restart noisd
```
### Load variables into system
```
source $HOME/.bash_profile
```
***
## After the node is fully synchronized, you need to create (or restore) a wallet and create a validator
### Check synchronization status
```
noisd status 2>&1 | jq .SyncInfo
```
### Create wallet
```
noisd keys add wallet
```
or recover wallet
```
noisd keys add wallet --recover
```
### Add wallet and valoper address into variables
```
NOIS_WALLET_ADDRESS=$(noisd keys show wallet -a)
NOIS_VALOPER_ADDRESS=$(noisd keys show wallet --bech val -a)
echo 'export NOIS_WALLET_ADDRESS='${NOIS_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export NOIS_VALOPER_ADDRESS='${NOIS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Create validator
*Test tokens to create a validator must be requested special command* (change wallet)
```
curl --header "Content-Type: application/json" \
  --request POST \
  --data '{"address": "<wallet>","denom": "unois"}' \
  http://faucet.noislabs.com/credit
```
#### Check your wallet balance
```
noisd query bank balances <address>
```
*If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue*
### To create your validator run command below (change name)
```
noisd tx staking create-validator \
  --amount=99000000unois \
  --pubkey=$(noisd tendermint show-validator) \
  --moniker=<name> \
  --chain-id=nois-testnet-002 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation "1" \
  --from=wallet \
  --fees=16000unois \
  --gas=300000 \
  -y
  ```
  ***
  ## Other commands
  ### Check logs
  ```
  journalctl -u noisd -f -o cat
  ```
  ### Restart service
  ```
  sudo systemctl restart noisd
  ```
  ### Synchronization info
  ```
  noisd status 2>&1 | jq .SyncInfo
  ```
  ### Unjail validator
  ```
  noisd tx slashing unjail \
  --broadcast-mode=block \
  --from wallet \
  --chain-id=nois-testnet-002 \
  --gas=auto
  ```
  ## State Sync
  ```
  NA
```
  ## Delete node
  ```
  sudo systemctl stop noisd
  sudo systemctl disable noisd
  sudo rm /etc/systemd/system/nois* -rf
  sudo rm $(which noisd) -rf
  sudo rm $HOME/.noisd* -rf
  sudo rm $HOME/full-node -rf
  ```
