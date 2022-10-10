# lambda_mainnet_setup_guide
Lambda_mainnet_setup_guide


# Lambda node setup for mainnet

Official documentation:
>- [Validator setup instructions](https://docs.lambda.im/validators/overview.html)

Explorer:
>-  https://explorer.nodestake.top/

Staking: 
>-  https://app.lambda.im/staking

🛡️Bridge:
>-  https://app.lambda.im/crosschain/ethereum_bridge

## Hardware Requirements
Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

### Minimum Hardware Requirements
 >- 4 or more physical CPU cores
 >- At least 500GB of SSD disk storage
 >- At least 32GB of memory (RAM)
 >- At least 100mbps network bandwidth

## Set up your Lambda fullnode
### Manual Installation 
```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install curl tar wget tmux htop net-tools clang pkg-config libssl-dev jq build-essential git make ncdu -y
```

```
version="1.19.2" \
&& cd $HOME \
&& wget "https://golang.org/dl/go$version.linux-amd64.tar.gz" \
&& sudo rm -rf /usr/local/go \
&& sudo tar -C /usr/local -xzf "go$version.linux-amd64.tar.gz" \
&& rm "go$version.linux-amd64.tar.gz" \
&& echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile \
&& source $HOME/.bash_profile
```

```
git clone https://github.com/LambdaIM/lambdavm.git
cd lambdavm
make install
```

```
lambdavm init Coinstamp --chain-id lambda_92000-1
lambdavm config chain-id lambda_92000-1
```


CREATE WALLET
```
lambdavm keys add wallet 
```
OR RECOVER

```
lambdavm keys add wallet --recover
```

```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ulamb\"/;" $HOME/.lambdavm/config/app.toml
```

```
pruning="custom" && \
pruning_keep_recent=100 && \
pruning_keep_every=0 && \
pruning_interval=10 && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.lambdavm/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.lambdavm/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.lambdavm/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.lambdavm/config/app.toml
```

```
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"2c4f8e193fd10ab3a2bc919b484fd1c78eceffb3@13.213.214.88:26656\"/; s/^seeds *=.*/seeds = \"2c4f8e193fd10ab3a2bc919b484fd1c78eceffb3@13.213.214.88:26656\"/" $HOME/.lambdavm/config/config.toml

```
```
indexer="null" && sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.lambdavm/config/config.toml
```
(OPTION)
```
wget https://raw.githubusercontent.com/LambdaIM/mainnet/main/lambda_92000-1/genesis.json
mv genesis.json ~/.lambdavm/config/

sudo tee /etc/systemd/system/lambdavm.service > /dev/null <<EOF
[Unit]
Description=lambdavm
After=network-online.target
[Service]
User=$USER
ExecStart=$(which lambdavm) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload && sudo systemctl enable lambdavm && sudo systemctl restart lambdavm && sudo journalctl -u lambdavm -f -o cat
```

## Post installation

When installation is finished please load variables into system
```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
lambdavm status 2>&1 | jq .SyncInfo
```

### (OPTIONAL) State Sync
You can state sync your node in minutes by running commands below. Special thanks to `NodeStake`

https://nodestake.top/lambda

```
peers="2c4f8e193fd10ab3a2bc919b484fd1c78eceffb3@13.213.214.88:26656,b772a0a8a8ee52c12ff0995ebb670a17db930489@54.225.36.85:26656,277b04415ee88113c304cc3970c88542d6d8f5d3@51.79.91.32:26656,a4ad9857ac5efdd75ec94875b19dd2f0bf562bde@47.75.111.113:26656,13e0e58efbb50df4dc5d39263bda1e432fe204f7@13.229.162.168:26656,ed4fd7dafd7df21f7152d38ee729ec33f505793d@54.254.224.222:26656,53e1c5f1783e839b1b1b51ae57ed2f05b9cdb4f3@13.229.27.15:26656,829503936e022119ce5e9cebf23c8e3a694c70f7@34.159.41.156:26656,d475be798a3b8d9eceb56b5cb276ff75d515cb7b@38.242.215.240:26656,d5bc2c509d730b5211f1e2f4cc95ffbbb6eb6944@194.163.164.52:26656,975afec2ce27ef21eea9d512f68eac8487680b09@135.181.72.187:12123,5a7e747884d496aec70495a767431410edb02167@149.102.139.69:26656,7f07d54901170270d7e7568481867535a363a1d5@65.108.129.104:26656,b029580f30c612176c81df200cf724836bba93c5@49.235.92.21:26656,b2cfe9fa02d93f3fa27cdb45272b5dcf3a075985@138.201.141.76:04656,bdeb4b00fe23900b323a3040a30b81e3c8f82803@23.88.69.167:26989"

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.lambdavm/config/config.toml
```

 
```
more ~/.lambdavm/config/config.toml | grep 'seeds'

more ~/.lambdavm/config/config.toml | grep 'persistent_peers'
```

 
```
SNAP_RPC="https://rpc.lambda.nodestake.top:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \

BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \

TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```

 
```
sudo systemctl stop lambdavm

lambdavm tendermint unsafe-reset-all --home ~/.lambdavm/
```

 
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \

s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \

s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \

s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \

s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" ~/.lambdavm/config/config.toml

more ~/.lambdavm/config/config.toml | grep 'rpc_servers'

more ~/.lambdavm/config/config.toml | grep 'trust_height'

more ~/.lambdavm/config/config.toml | grep 'trust_hash'
```

 
```
sudo systemctl restart lambdavm

journalctl -u lambdavm -f -o cat
```

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
lambdavm keys add wallet
```

(OPTIONAL) To recover your wallet using seed phrase
```
lambdavm keys add wallet --recover
```

To get current list of wallets
```
lambdavm keys list
```

### Save wallet info
Add wallet and valoper address into variables 
```
LAMBDA_WALLET_ADDRESS=$(lambdavm keys show wallet -a)
```
```
LAMBDA_VALOPER_ADDRESS=$(lambdavm keys show wallet --bech val -a)
```
Load variables into the system
```
echo 'export LAMBDA_WALLET_ADDRESS='${LAMBDA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export LAMBDA_VALOPER_ADDRESS='${LAMBDA_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Create validator
Before creating validator please make sure that you have at least 1 lamb (1 lamb is equal to 10000000000000000000 ulamb) and your node is synchronized

To check your wallet balance:
```
lambdavm query bank balances $LAMBDA_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
lambdavm tx staking create-validator \
  --amount 100000ulamb \
  --from wallet \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(lambdavm tendermint show-validator) \
  --moniker "MONIKERNAME" \
  --chain-id lambda_92000-1 \
  --fees=2500ulamb \
  --gas=250000 
```


## Useful commands

### Service management
Check logs
```
journalctl -fu lambdavm -o cat
```

Start service
```
sudo systemctl start lambdavm
```

Stop service
```
sudo systemctl stop lambdavm
```

Restart service
```
sudo systemctl restart lambdavm
```

### Node info
Synchronization info
```
lambdavm status 2>&1 | jq .SyncInfo
```

Validator info
```
lambdavm status 2>&1 | jq .ValidatorInfo
```

Node info
```
lambdavm status 2>&1 | jq .NodeInfo
```

Show node id
```
lambdavm tendermint show-node-id
```

### Wallet operations
List of wallets
```
lambdavm keys list
```

Recover wallet
```
lambdavm keys add wallet --recover
```

Delete wallet
```
lambdavm keys delete wallet
```

Get wallet balance
```
lambdavm query bank balances $LAMBDA_WALLET_ADDRESS
```

Transfer funds
```
lambdavm tx bank send $LAMBDA_WALLET_ADDRESS <TO_LAMBDA_WALLET_ADDRESS> 10000000ulamb
```

### Voting
```
lambdavm tx gov vote 1 yes --from $WALLET --chain-id=lambda_92000-1
```

### Staking, Delegation and Rewards
Delegate stake
```
lambdavm tx staking delegate $KUJIRA_VALOPER_ADDRESS 10000000ulamb --from=wallet --chain-id=lambda_92000-1 --gas=auto
```

Redelegate stake from validator to another validator
```
lambdavm tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000ulamb --from=wallet --chain-id=lambda_92000-1 --gas=auto
```

Withdraw all rewards
```
lambdavm tx distribution withdraw-all-rewards --from=wallet --chain-id=lambda_92000-1 --gas=auto
```

Withdraw rewards with commision
```
lambdavm tx distribution withdraw-rewards $LAMBDA_VALOPER_ADDRESS --from=wallet --commission --chain-id=lambda_92000-1D
```

### Validator management
Edit validator
```
lambdavm tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=lambda_92000-1 \
  --from=$WALLET
```

Unjail validator
```
lambdavm tx slashing unjail \
  --broadcast-mode=block \
  --from=wallet \
  --chain-id=lambda_92000-1 \
  --gas=auto
```
