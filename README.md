![Electra (1)](https://github.com/Alkia/electra/raw/master/vue/public/Electra.png)

# The electra-testnet-0 guide
 
- **Recommended hardware requirements**:

| Network   |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet 0   |  2 | 8GB | 0.25TB SSD/NVMe |
| Mainnet   |  2 | 16GB | 0.5TB SSD/NVMe |

- **testnet-0 official ports**:   

| Service | testnet-0 Port | Description       |
|----------|----------------|-------------------|
| rpc       |      26659      |                   |
| p2p       |      26658      |                   |
| prof      |       6061      |                   |
| grp c     |       9092      |                   |
| grpc-web  |       9093      |                   |
| api       |       1318      |                   |
| rosetta   |       8080      |                   |

### Preparing the server 

    sudo apt update && sudo apt upgrade -y && \
    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

## Install the binary
There are 2 options to get the binary:

### Option 1: Download (the easier softer way)

#### 1.0) Direct download
```
wget http://electra01.alkia.net:3000/electrad.7z
p7zip -d electrad.7z
```
#### 1.1) Download from this repo the 2 splits of the file
Github limit http upload to 25M max and we are a bit over so we needed to split in 2 files (command: <i>split -b 20M electra_linux_amd64.tar.gz </i>):

 * electra_linux_amd64.tar.gz.part_aa
 * electra_linux_amd64.tar.gz.part_ab

    ##### 1.1.2) Reconstruct the binary
    You join the files using the cat command. Employing cat is the most efficient and reliable method of performing a joining operation. 
    ```
    cat electra_linux_amd64.tar.gz.part_* > electra_linux_amd64.tar.gz
    ```
    #####Security note: Please check that the checksum provided match the reconstructed electra_linux_amd64.tar.gz

    ##### 1.1.3) Untar
    ```
    tar -xvf electra_linux_amd64.tar.gz 
    ``` 
This gives you the binary *electrad*

####  2) Copy *electrad* to your favorite binary directory
```
cp electrad /usr/local/bin/
chmod a+x   /usr/local/bin/electrad
```

### Option 2: Recompile

#### 2.1) Install GO 18.7 (one command)
            wget https://golang.org/dl/go1.18.7.linux-amd64.tar.gz; \
            rm -rv /usr/local/go; \
            tar -C /usr/local -xzf go1.18.7.linux-amd64.tar.gz && \
            rm -v go1.18.7.linux-amd64.tar.gz && \
            echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
            source ~/.bash_profile && \
            go version

#### 2.2) Build    (07.11.22)
            git clone https://github.com/alkia/electra
            cd electra
            git checkout v0.1.4
            make install
        `electrad version`
        + version 0.1.4

## Initilize the chain
```
sudo rm ~/.electra/ -r
moniker="electra03"
electrad tendermint unsafe-reset-all
electrad init $moniker --chain-id electra-testnet-0    
```
NOTES:
* Use only alphanumeric characters, hyphens, and underscores for your node name.
* If you are using secondary storage:
    You need to add a --home flag where you direct the command to the path of your secondary storage. By default, und init uses --home /home/$USER/.und_mainchain if you do not specify otherwise.

## Create/recover wallet
```
electrad keys add <walletname>
electrad keys add <walletname> --recover
```
##### when creating, do not forget to write down the seed phrase   

Check that your keys are recorded properly:
```
electrad keys list
```

## Genesis
```
    wget https://raw.githubusercontent.com/Alkia/electra-testnet-0/main/genesis.json
    mv genesis.json ~/.electra/config/    
```    
Seeds format:
seeds = "[node_id@ip:port]"
    
## Set up the minimum gas price $HOME/.electra/config/app.toml as well as seed and peers

    sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"1uelectra\"/;" ~/.electra/config/app.toml
    
    sed -i.bak -e "s/^pruning *=.*/pruning = \"everything\"/;" ~/.electra/config/app.toml
    
    sed -i.bak -e "s/^create_empty_blocks *=.*/create_empty_blocks = false/" $HOME/.electra/config/config.toml
    
    # moniker="electra01" 
    
    sed -i.bak -e "s/^moniker *=.*/moniker = \"$moniker\"/" $HOME/.electra/config/config.toml
    
    external_address=$(wget -qO- eth0.me)
    
    sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26659\"/" $HOME/.electra/config/config.toml
    
    peers="E276114F7F6AC8E6566E22683186AB657BAF7C3E@66.42.50.244:26658"
    peers="62504544f96a078cb70709486e318035c048ebc6@66.42.50.244:26658"
    
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.electra/config/config.toml
    
    seeds=""
    
    sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.electra/config/config.toml 
    
    sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.electra/config/config.toml
    
    sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.electra/config/config.toml
    

    
## Create a service file
    sudo tee /etc/systemd/system/electrad.service > /dev/null <<EOF
    [Unit]
    Description=electra
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which electrad) start
    Restart=on-failure
    RestartSec=3
    LimitNOFILE=65535

    [Install]
    WantedBy=multi-user.target
    EOF
    
 [Snapshot](https://polkachu.com/tendermint_snapshots/electra)    (optional) \
    or \
 [RPC](https://nodejumper.io/electra/sync) (optional)
 
# Start
```
# Soft start
    electrad start --log_level info
```
At first, you will see an error message, e.g.:

E[2020-03-19|09:42:06.072] Stopping peer for error                      module=p2p peer="Peer{MConn{1.2.3.4:26656} dcff5de69dcc170b28b6628a1336d420f7eb60c0 out}" err=EOF
This is normal. Wait. Eventually, you will see your node downloading blocks. Once you are happy seeing the blocks roll in, hit CTRL+C to stop the node.

# Service start 
```
    sudo systemctl daemon-reload
    sudo systemctl enable electrad
    sudo systemctl restart electrad
    sudo journalctl -u electrad -f -o cat
```       
# Notes on Managing Validator

Running a production-quality validator node with a robust architecture and security features requires an extensive setup.

Electra chain is powered by the Tendermint consensus. Validators run full nodes, participate in consensus by broadcasting votes, commit new blocks to the blockchain, and participate in governance of the blockchain.

Validators earn the following fees:
- Gas: Fees added on to each transaction to avoid spamming and pay for computing power. Validators set minimum gas prices and reject transactions that have implied gas prices below this threshold.
- Validators get coins as rewards proportional to their shares

`If validators double sign or frequently offline, their credibility will be slashed. Penalties can vary depending on the severity of the violation.`


## Create validator
```
electrad tx staking create-validator --amount 100000000uelectra --commission-max-change-rate "0.10" --commission-max-rate "0.20" --commission-rate "0.10" --min-self-delegation "1" --identity="" --details="" --website="" --pubkey $(electrad tendermint show-validator) --moniker electra01 --chain-id electra-testnet-0 -y --from <walletName> 
```

# References    
    * [Website](https://electra.alkia.net/)       
    * [Block Explorer](https://www.mintscan.io/electra/validators)

# Options

## Pruning (optional)
    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.electra/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.electra/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.electra/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.electra/config/app.toml
    
## Indexer (optional)    
    indexer="null" && \
    sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.electra/config/config.toml    
   
## Remove

    sudo systemctl stop electrad && \
    sudo systemctl disable electrad && \
    rm /etc/systemd/system/electrad.service && \
    sudo systemctl daemon-reload && \
    cd $HOME && \
    rm -rf .electrad && \
    rm -rf electra && \
    rm -rf $(which electrad)    

### electrad chain init
```
electrad keys add validator --keyring-backend test
electrad add-genesis-account electra1ta52mxc8xfmaaatq88ycr6pyuruhqxux6hmdyh 100000000uelectra --keyring-backend test
electrad gentx validator 100000000uelectra  --chain-id electra-testnet-0  --keyring-backend test
electrad collect-gentxs
electrad validate-genesis
```
