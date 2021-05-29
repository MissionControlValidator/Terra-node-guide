# Terra-node-guide
In this guide we will go through every step to easily set up a Terra node. 
First of all you need to know the system requirements:
  * 2 or more CPU cores
  * At least 1TB of disk storage (SSDs are strongly recommended)
  * At least 16GB of memory
  * At least 100mbps network bandwidth****
#
# How to setup a Terra node
## 1) Update OS

Update everything with:
```bash
sudo apt-get update
sudo apt-get upgrade -y
```
#
## 2) Setup the firewall

Open ports `26656` and `ssh`, then enable the firewall:
```bash
sudo ufw allow ssh
sudo ufw allow 26656/tcp
```
set default firewall rules:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
enable firewall:
```bash
sudo ufw enable
```
#
## 3) Install Terra Core
Download the repository:
```bash
cd ~
git clone https://github.com/terra-project/core.git
```
Install:
```bash
cd core
make install
```
Move `terrad` and `terracli` in the path directory:
```bash
sudo mv ~/go/bin/terrad /usr/bin/terrad
sudo mv ~/go/bin/terracli /usr/bin/terracli
```
Test if `terrad` and `terracli` works:
```bash
terrad version 
terracli version
```
Expected output:
```bash
0.4.6
```
#
## 4) Download genesis and quicksync
Create all the necessary files, change **{MONIKER}** with a custom moniker
```bash
terrad init {MONIKER}
```
Download the genesis files:
```bash
curl https://columbus-genesis.s3-ap-northeast-1.amazonaws.com/columbus-4-genesis.json > ~/.terrad/config/genesis.json
```
Download the latest [quickSync](https://terra.quicksync.io/)

Command for **pruned** archive:
```bash
cd ~/.terrad
wget https://get.quicksync.io/{FILENAME} 
```
Install liblz4 to extract the file:
```bash
sudo apt-get install wget liblz4-tool -y
```
Extract the quicksync:
```bash
cd ~/.terrad
lz4 -d {FILENAME}   | tar xf -
```
> Now you can delete the quicksynck file with `rm {FILENAME}  `
#
## 5) Edit the config
Edit the `config.toml` file:
```bash
nano ~/.terrad/config/config.toml
``` 
and set the following value:
```python
seeds = "5d9b8ac70000bd4ab1de3ccaf85eb43f8e315146@seed.terra.delightlabs.io:26656,6d8e943c049a80c161a889cb5fcf3d184215023e@public-seed2.terra.dev:26656,87048bf71526fb92d73733ba3ddb79b7a83ca11e@public-seed.terra.dev:26656"
```
then edit the `app.toml` file:
```bash
nano ~/.terrad/config/app.toml
```
and set the following values:
```python
minimum-gas-prices = "0.01133uluna,0.15uusd,0.104938usdr,169.77ukrw,428.571umnt,0.125ueur,0.98ucny,16.37ujpy,0.11ugbp,10.88uinr,0.19ucad,0.14uchf,0.19uaud,0.2usgd,4.62uthb,1.25usek"
pruning = "everything"
```
Download the address book:
```bash
curl https://network.terra.dev/addrbook.json > ~/.terrad/config/addrbook.json
```
Run `terrad`:
```bash
cd ~
terrad start
```
#
## 6) Setup pm2 process manager
Install `npm`, `nodejs` and `pm2`:
```bash
sudo apt-get install nodejs npm -y
sudo npm i -g pm2
```
create the necessary files:
```bash
cd ~
touch runNode.sh
chmod +x runNode.sh
```
edit the file `runNode.sh`:
```bash
nano runNode.sh
```
and put in the following content:
```bash
terrad start --tx-gas-hard-limit 10000000
```
then create the `terra-node.ecosystem.config.js` file:
```bash
nano terra-node.ecosystem.config.js
```
and put in the following content:
```js
module.exports = { apps : [ { name: "terra-node", script: "runNode.sh", exec_mode: "fork", exec_interpreter: "bash"} ] }
```
#
## 7) Run terrad with pm2
Run terrad with pm2 process manager
```bash
cd ~
pm2 start ./terra-node.ecosystem.config.js
```
Save the process and setup auto-start at boot:
```bash
pm2 save
pm2 startup
```
Check the status:
```bash
pm2 status
```
Check the logs:
```bash
pm2 logs 0
```
Use  `ctrl` + `c` to exit from the log view
#
## 8) Wait untill the node is sincronyzed 
Wait a few hours, you can check the node status using:
```bash
curl -sS localhost:26657/status
```
expected output:
```json
{
  "jsonrpc": "2.0",
  "id": -1,
  "result": {...},
    "sync_info": {
      "latest_block_hash": "...",
      "latest_app_hash": "...",
      "latest_block_height": "lastBlock",
      "latest_block_time": "currentDate",
      "earliest_block_hash": "...",
      "earliest_app_hash": "",
      "earliest_block_height": "1",
      "earliest_block_time": "2020-10-03T15:56:10Z",
      "catching_up": false
    },
    "validator_info": {...}
  }
}
```
If you see  `"catching_up": false` the node is synchronized, otherwise you will have to wait
