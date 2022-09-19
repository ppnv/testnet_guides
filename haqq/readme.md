![](https://pbs.twimg.com/profile_images/1538207991631171585/GpvwlBhe_400x400.jpg)
# HAQQ
- Website: https://islamiccoin.net/
- Twitter: https://twitter.com/Islamic_coin
- Discord: https://discord.gg/anTje2ze9c
- Github: https://github.com/haqq-network
## Explorers
NodesGuru - https://haqq.explorers.guru <br>
## Official documentation
[Validator setup instructions](https://github.com/haqq-network/haqq?ysclid=l6txaje24v893082750)
## Set up your teritori fullnode ( manual install )
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
cd $HOME \
wget https://github.com/haqq-network/haqq \
cd haqq \
git checkout v1.0.3 \
make install
```
### Config app
```
haqqd config chain-id haqq_54211-2
haqqd config keyring-backend test
haqqd config node tcp://localhost:26657
```
### Init app (change nodename)
```
haqqd init <nodename> --chain-id haqq_54211-2
```
### Download genesis and addrbook
```
wget -O $HOME/.haqqd/config/genesis.json "https://raw.githubusercontent.com/haqq-network/validators-contest/master/genesis.json"
wget -O $HOME/.haqqd/config/addrbook.json "https://raw.githubusercontent.com/StakeTake/guidecosmos/main/haqq/haqq_54211-2/addrbook.json"
```
### Set seeds and peers
```
SEEDS=""
PEERS=""; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.haqqd/config/config.toml
```
### Config pruning (optional)
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.teritorid/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.teritorid/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.teritorid/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.teritorid/config/app.toml
```
### Disable indexing (optional)
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.haqqd/config/config.toml
```
### Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utori\"/" $HOME/.haqqd/config/app.toml
```
### Reset chain data
```
haqqd tendermint unsafe-reset-all --home $HOME/.haqqd
```
### Create service
```
sudo tee /etc/systemd/system/haqqd.service > /dev/null <<EOF
[Unit]
Description=haqqd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which haqqd) start --home $HOME/.haqqd
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
sudo systemctl enable haqqd
sudo systemctl restart haqqd
```
### Load variables into system
```
source $HOME/.bash_profile
```
***
## After the node is fully synchronized, you need to create (or restore) a wallet and create a validator
### Check synchronization status
```
haqqd status 2>&1 | jq .SyncInfo
```
### Create wallet
```
haqqd keys add wallet
```
or recover wallet
```
haqqd keys add wallet --recover
```
### Add wallet and valoper address into variables
```
HAQQD_WALLET_ADDRESS=$(haqqd keys show <wallet> -a)
HAQQD_VALOPER_ADDRESS=$(haqqd keys show <wallet> --bech val -a)
echo 'export HAQQD_WALLET_ADDRESS='${HAQQD_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export HAQQD_VALOPER_ADDRESS='${HAQQD_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Create validator
*Test tokens to create a validator must be requested in https://testedge2.haqq.network/*
#### Check your wallet balance
```
haqqd query bank balances <address>
```
*If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue*
### To create your validator run command below (change name)
```
haqqd tx staking create-validator \
  --amount 1000000aISML \
  --from wallet \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(haqqd tendermint show-validator) \
  --moniker <name> \
  --chain-id haqq_54211-2
  ```
  ***
  ## Other commands
  ### Check logs
  ```
  journalctl -fu haqqd -o cat
  ```
  ### Restart service
  ```
  sudo systemctl restart haqqd
  ```
  ### Synchronization info
  ```
  haqqd status 2>&1 | jq .SyncInfo
  ```
  ### Unjail validator
  ```
  haqqd tx slashing unjail \
  --broadcast-mode=block \
  --from wallet \
  --chain-id=haqq_54211-2 \
  --gas=auto
  ```
  ## State Sync
  ```
  sudo systemctl stop haqqd
haqqd tendermint unsafe-reset-all --home $HOME/.haqqd
SEEDS=""
PEERS=""; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.haqqd/config/config.toml
wget -O $HOME/.haqqd/config/addrbook.json "https://raw.githubusercontent.com/StakeTake/guidecosmos/main/haqq/haqq_54211-2/addrbook.json"
SNAP_RPC="https://haqq-t.rpc.manticore.team:443"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.haqqd/config/config.toml
sudo systemctl restart haqqd && journalctl -u haqqd -f -o cat
```
  ## Delete node
  ```
  sudo systemctl stop haqqd
  sudo systemctl disable haqqd
  sudo rm /etc/systemd/system/haqqd* -rf
  sudo rm $(which haqqd) -rf
  sudo rm $HOME/.haqqd* -rf
  sudo rm $HOME/haqqd -rf
  sed -i '/haqqd_/d' ~/.bash_profile
  ```
