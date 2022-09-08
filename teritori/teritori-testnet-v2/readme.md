![](https://teritori.com/logo.svg)
# Teritori Network
- Website: https://teritori.com/
- Twitter: https://twitter.com/TeritoriNetwork
- Discord: https://discord.gg/qYg9ex5Vjk
- Github: https://github.com/TERITORI
## Explorers
NodesGuru - https://teritori.explorers.guru <br>
PingPub - https://explorer.stake-take.com/teritori-testnet/staking
## Official documentation
[Validator setup instructions](https://github.com/TERITORI/teritori-chain/blob/main/testnet/teritori-testnet-v2/README.md)
## Set up your teritori fullnode (manual install)
### Update packages
```
sudo apt update && sudo apt upgrade -y
```
### Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
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
git clone https://github.com/TERITORI/teritori-chain
cd teritori-chain
git checkout teritori-testnet-v2
make install
```
### Config app
```
teritorid config chain-id teritori-testnet-v2
teritorid config keyring-backend test
teritorid config node tcp://localhost:26657
```
### Init app (change nodename)
```
teritorid init <nodename> --chain-id teritori-testnet-v2
```
### Download genesis and addrbook
```
wget -qO $HOME/.teritorid/config/genesis.json "https://raw.githubusercontent.com/TERITORI/teritori-chain/main/testnet/teritori-testnet-v2/genesis.json"
wget -qO $HOME/.teritorid/config/addrbook.json "https://raw.githubusercontent.com/StakeTake/guidecosmos/main/teritori/teritori-testnet-v2/addrbook.json"
```
### Set seeds and peers
```
SEEDS=""
PEERS="96b55467792a143e0169684f0e8da9a9aeb97af4@23.88.77.188:28656,4eb8b8bed6aecc52dccf21fd1e9432e071659db2@38.242.154.39:36656,986cdc276eea5fcb205ea3c66503c0610f99895d@95.216.140.117:26656,545b1fe982b92aeb9f1eadd05ab0954b38eba402@194.163.177.240:26656,0d19829b0dd1fc324cfde1f7bc15860c896b7ac1@65.108.121.240:27656,34df38933c32ee21078c1d79787d76668f398b9e@89.163.231.30:36656,0248e2989a8a4f6ad87cbe0490c08908a2c2da7f@5.199.133.165:26656,691efb2bee7b585c1f434d934abf18428d0b8ff1@161.97.91.254:26656,cd363d841f4dab90f290aab21c97f8d80a93a028@38.242.154.35:36656,a1c845585abbd8490ecbbcc7f96ff3b027cbed88@38.242.154.40:36656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.teritorid/config/config.toml
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
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.teritorid/config/config.toml
```
### Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utori\"/" $HOME/.teritorid/config/app.toml
```
### Reset chain data
```
teritorid tendermint unsafe-reset-all --home $HOME/.teritorid
```
### Create service
```
sudo tee /etc/systemd/system/teritorid.service > /dev/null <<EOF
[Unit]
Description=teritori
After=network-online.target

[Service]
User=$USER
ExecStart=$(which teritorid) start --home $HOME/.teritorid
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
sudo systemctl enable teritorid
sudo systemctl restart teritorid
```
### Load variables into system
```
source $HOME/.bash_profile
```
***
## After the node is fully synchronized, you need to create (or restore) a wallet and create a validator
### Check synchronization status
```
teritorid status 2>&1 | jq .SyncInfo
```
### Create wallet
```
teritorid keys add wallet
```
or recover wallet
```
teritorid keys add wallet --recover
```
### Add wallet and valoper address into variables
```
TERITORI_WALLET_ADDRESS=$(teritorid keys show $WALLET -a)
TERITORI_VALOPER_ADDRESS=$(teritorid keys show $WALLET --bech val -a)
echo 'export TERITORI_WALLET_ADDRESS='${TERITORI_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export TERITORI_VALOPER_ADDRESS='${TERITORI_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Create validator
*Test tokens to create a validator must be requested in discord*
#### Check your wallet balance
```
teritorid query bank balances <address>
```
*If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue*
### To create your validator run command below (change name)
```
teritorid tx staking create-validator \
  --amount 1000000utori \
  --from wallet \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(teritorid tendermint show-validator) \
  --moniker <name> \
  --chain-id teritori-testnet-v2
  ```
  ***
  ## Other commands
  ### Check logs
  ```
  journalctl -fu teritorid -o cat
  ```
  ### Restart service
  ```
  sudo systemctl restart teritorid
  ```
  ### Synchronization info
  ```
  teritorid status 2>&1 | jq .SyncInfo
  ```
  ### Unjail validator
  ```
  teritorid tx slashing unjail \
  --broadcast-mode=block \
  --from wallet \
  --chain-id=teritori-testnet-v2 \
  --gas=auto
  ```
  ## State Sync
  ```
  SNAP_RPC="http://teritori.stake-take.com:26657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.teritorid/config/config.toml
teritorid tendermint unsafe-reset-all --home $HOME/.teritorid
sudo systemctl restart teritorid && journalctl -fu teritorid -o cat
```
  ## Delete node
  ```
  sudo systemctl stop teritorid
  sudo systemctl disable teritorid
  sudo rm /etc/systemd/system/teritori* -rf
  sudo rm $(which teritorid) -rf
  sudo rm $HOME/.teritorid* -rf
  sudo rm $HOME/teritori -rf
  sed -i '/TERITORI_/d' ~/.bash_profile
  ```
