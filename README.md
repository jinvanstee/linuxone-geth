# Go Ethereum Deployment on LinuxONE
This repo describes the steps to build and install Go Ethereum on LinuxONE or Linux on IBM Z. In addition to base installation, it will cover:
* How to setup a single-node private Ethereum network. Private Ethereum networks are good for developing and testing out smart contracts or for private consortium use cases.
* How to deploy a simple smart contract to it and transact on it using the Truffle smart contract development framework.

## Versions used in this tutorial
* Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic s390x)
* go1.16.5 linux/s390x
* geth 1.10.4-stable

## Installing Pre-requisites and Go Ethereum
Please note that there really isn't anything special about installing Go Ethereum on LinuxONE or Linux on IBM Z. After the s390x version of Go is installed, the steps to build and install Go Ethereum is exactly the same as on any other platform. These instructions were lifted straight from the official [Go Ethereum installation guide](https://geth.ethereum.org/docs/install-and-build/installing-geth#build-go-ethereum-from-source-code). I include them here for completeness.
### Installing pre-requisite tools and packages
```
$ sudo apt-get update
$ sudo apt-get install -y curl build-essential git libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev libzstd-dev libclang-10-dev cmake
```
### Installing Go and setting up Go workspace and GOPATH
```
$ curl -OL https://golang.org/dl/go1.16.5.linux-s390x.tar.gz
$ sudo tar -C /usr/local -xzf go1.16.5.linux-s390x.tar.gz
$ export PATH=$PATH:/usr/local/go/bin
$ echo "export PATH=${PATH}:/usr/local/go/bin" >> "${HOME}/.bashrc"
```
<b>Check the Go version</b>
```
$ go version
go version go1.16.5 linux/s390x
```
<b>Setup the Go workspace and GOPATH</b>
```
$ cd ~
$ mkdir -p go/src
$ mkdir -p go/bin
$ export GOPATH=$HOME/go
$ echo "export GOPATH=${HOME}/go" >> "${HOME}/.bashrc"
$ export PATH=$PATH:$GOPATH/bin
$ echo "export PATH=$PATH:$GOPATH/bin" >> "${HOME}/.bashrc"
```
### Installing and testing Go Ethereum
First download the Go Ethereum source package and its dependencies:
```
$ go get -d github.com/ethereum/go-ethereum
go: downloading github.com/ethereum/go-ethereum v1.10.4
go: downloading golang.org/x/crypto v0.0.0-20210322153248-0c34fe9e7dc2
go: downloading github.com/btcsuite/btcd v0.20.1-beta
go: downloading golang.org/x/sys v0.0.0-20210420205809-ac73e9fd8988![image](https://user-images.githubusercontent.com/8147238/125550289-78e734c8-34f7-4698-abbc-e183c786c094.png)
```
Then navigate to the Go Ethereum package directory and install geth and other developer tools:
```
$ cd $GOPATH/pkg/mod/github.com/ethereum/go-ethereum@v1.10.4/
$ go install ./...
```
You will begin to see a lot of output related to dependent packages being downloaded. This is expected. There should be no error messages during the installation process. After installation completes, verify the `geth` version with:
```
$ geth version
Geth
Version: 1.10.4-stable
Architecture: s390x
Go Version: go1.16.5
Operating System: linux
GOPATH=/home/linux1/go
GOROOT=/usr/local/go
```
Now let's run the test suite against it:
```
$ cd $GOPATH/pkg/mod/github.com/ethereum/go-ethereum@v1.10.4/
$ go test -v ./eth
```
With the verbose option you will see a lot of output as it runs through each test module but all should PASS and in the end you should see:
```
PASS
ok  	github.com/ethereum/go-ethereum/eth	13.106s
```
## Creating and starting a private network
### Setup
Create directories for your private network.
```
$ cd ~
$ mkdir private_network
$ mkdir private_network/chaindata
```
All the assets related to your private network will be kept inside the `private_network` directory. For a description of all the directories and what they mean go here.
### Create your `genesis.json` file for your private network. 
This file describes the rules associated with your private network. For a desription of all the settings you can customize in a `genesis.json` file please see here.
```
$ cd private_network
```
Use your favorite text editor and create the `genesis.json` file as follows: 
```
{
   "difficulty" : "0x20000",
   "gasLimit"   : "0x8000000",
   "alloc": {},
   "nonce": "0x0000000020210713",
   "config": {
        "chainId": 25,
        "homesteadBlock": 0,
        "eip150Block": 0,
        "eip155Block": 0,
        "eip158Block": 0,
        "byzantiumBlock": 0,
        "constantinopleBlock": 0,
        "petersburgBlock": 0,
        "istanbulBlock": 0,
        "berlinBlock": 0
    }
}
```
I'm using a very basic, generic `genesis.json` file that should meet basic dev/test needs. Feel free to customize to suit your needs. The Go Ethereum repo recommends "changing the nonce to some random value so you prevent unknown remote nodes from being able to connect to you." You should also select a random chainID that doesn't intefere with other chainIDs that you interact with to minimize confusion.
### Initialize the private network
```
$ cd ~/private_network
$ geth --datadir=./chaindata init ./genesis.json
```
You should see output similar to the following:
```
INFO [07-13|14:24:39.586] Maximum peer count                       ETH=50 LES=0 total=50
INFO [07-13|14:24:39.586] Smartcard socket not found, disabling    err="stat /run/pcscd/pcscd.comm: no such file or directory"
INFO [07-13|14:24:39.587] Set global gas cap                       cap=50,000,000
INFO [07-13|14:24:39.587] Allocated cache and file handles         database=/home/linux1/private_network/chaindata/geth/chaindata cache=16.00MiB handles=16
INFO [07-13|14:24:39.591] Writing custom genesis block
INFO [07-13|14:24:39.591] Persisted trie from memory database      nodes=0 size=0.00B time="5.369µs" gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [07-13|14:24:39.592] Successfully wrote genesis state         database=chaindata hash=e4d9fb..65bf61
INFO [07-13|14:24:39.592] Allocated cache and file handles         database=/home/linux1/private_network/chaindata/geth/lightchaindata cache=16.00MiB handles=16
INFO [07-13|14:24:39.596] Writing custom genesis block
INFO [07-13|14:24:39.596] Persisted trie from memory database      nodes=0 size=0.00B time="1.846µs" gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [07-13|14:24:39.596] Successfully wrote genesis state         database=lightchaindata hash=e4d9fb..65bf61
```
### Now you can start your network
I am starting it with very minimal parameters. For a list of all the `geth` parameters you can use please see here.
```
$ cd ~/private_network
$ geth --datadir=./chaindata --nodiscover
```
You will see output similar to the following:
```
INFO [07-13|14:44:47.305] Starting Geth on Ethereum mainnet...
INFO [07-13|14:44:47.306] Bumping default cache on mainnet         provided=1024 updated=4096
INFO [07-13|14:44:47.309] Maximum peer count                       ETH=50 LES=0 total=50
INFO [07-13|14:44:47.309] Smartcard socket not found, disabling    err="stat /run/pcscd/pcscd.comm: no such file or directory"
INFO [07-13|14:44:47.309] Set global gas cap                       cap=50,000,000
INFO [07-13|14:44:47.309] Allocated trie memory caches             clean=614.00MiB dirty=1024.00MiB
INFO [07-13|14:44:47.309] Allocated cache and file handles         database=/home/linux1/private_network/chaindata/geth/chaindata cache=2.00GiB handles=524,288
INFO [07-13|14:44:47.334] Opened ancient database                  database=/home/linux1/private_network/chaindata/geth/chaindata/ancient readonly=false
INFO [07-13|14:44:47.334] Initialised chain configuration          config="{ChainID: 25 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: 0 EIP155: 0 EIP158: 0 Byzantium: 0 Constantinople: 0 Petersburg: 0 Istanbul: 0, Muir Glacier: <nil>, Berlin: 0, London: <nil>, Engine: unknown}"
INFO [07-13|14:44:47.334] Disk storage enabled for ethash caches   dir=/home/linux1/private_network/chaindata/geth/ethash count=3
INFO [07-13|14:44:47.334] Disk storage enabled for ethash DAGs     dir=/home/linux1/.ethash count=2
INFO [07-13|14:44:47.334] Initialising Ethereum protocol           network=1 dbversion=<nil>
INFO [07-13|14:44:47.334] Loaded most recent local header          number=0 hash=e4d9fb..65bf61 td=131,072 age=52y3mo1w
INFO [07-13|14:44:47.334] Loaded most recent local full block      number=0 hash=e4d9fb..65bf61 td=131,072 age=52y3mo1w
INFO [07-13|14:44:47.334] Loaded most recent local fast block      number=0 hash=e4d9fb..65bf61 td=131,072 age=52y3mo1w
WARN [07-13|14:44:47.334] Failed to load snapshot, regenerating    err="missing or corrupted snapshot"
INFO [07-13|14:44:47.334] Rebuilding state snapshot
INFO [07-13|14:44:47.335] Resuming state snapshot generation       root=56e81f..63b421 accounts=0 slots=0 storage=0.00B elapsed="176.073µs"
INFO [07-13|14:44:47.335] Generated state snapshot                 accounts=0 slots=0 storage=0.00B elapsed="286.052µs"
INFO [07-13|14:44:47.335] Regenerated local transaction journal    transactions=0 accounts=0
INFO [07-13|14:44:47.335] Gasprice oracle is ignoring threshold set threshold=2
WARN [07-13|14:44:47.335] Error reading unclean shutdown markers   error="leveldb: not found"
INFO [07-13|14:44:47.335] Starting peer-to-peer node               instance=Geth/v1.10.4-stable/linux-s390x/go1.16.5
INFO [07-13|14:44:47.341] IPC endpoint opened                      url=/home/linux1/private_network/chaindata/geth.ipc
INFO [07-13|14:44:47.341] New local node record                    seq=1 id=9b27ff1d50395e54 ip=127.0.0.1 udp=0 tcp=30303
INFO [07-13|14:44:47.341] Started P2P networking                   self="enode://f7d7a68e02dab66bb638d54b5d3062ddbd1667a9e7acfefac8c7704ef428b2bc8fba1b30e3b949cbd8d5b8ae2fa6f5094104e36d1b5d043f2f62c772d0f21cd0@127.0.0.1:30303?discport=0"
```
Please make a note of the IPC endpoint. In the above example, it is `/home/linux1/private_network/chaindata/geth.ipc`. You will need this URL to attach a geth console to it so that you can run web3js commands to interact with your private network.
## Attach console, run some w3js commands from there to do a basic test:
From a separate terminal window, run the following command to attach a console to your geth private network. Use the IPC endpoint URL from your private network output in the previous step.
```
$ geth attach /home/linux1/private_network/chaindata/geth.ipc
```
You should see output similar to the following:
```
Welcome to the Geth JavaScript console!

instance: Geth/v1.10.4-stable/linux-s390x/go1.16.5
at block: 0 (Wed Dec 31 1969 19:00:00 GMT-0500 (EST))
 datadir: /home/linux1/private_network/chaindata
 modules: admin:1.0 debug:1.0 eth:1.0 ethash:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

To exit, press ctrl-d
> 
```
The console accepts web3js function calls. These are javascript functions written to interact with the Ethereum network. For a complete guide to all the web3js functions please see here. For now we will run through a set of basic commands, to create a couple of accounts, start the miner, check balances on the accounts, and transfer some ether from one account to another. From the `geth console`, type the following commands.
```
> eth.accounts
[]
> personal
{
  listAccounts: [],
  listWallets: [],
  deriveAccount: function(),
  ecRecover: function(),
  getListAccounts: function(callback),
  getListWallets: function(callback),
  importRawKey: function(),
  initializeWallet: function(),
  lockAccount: function(),
  newAccount: function(),
  openWallet: function(),
  sendTransaction: function(),
  sign: function(),
  signTransaction: function(),
  unlockAccount: function(),
  unpair: function()
}
> personal.newAccount();
Passphrase:
Repeat passphrase:
"0xa6f03d4e14f0897978e62df8d5f3bd0d532570b3"
> personal.newAccount();
Passphrase:
Repeat passphrase:
"0x808f02a8dbcafdab4e417215b16d36eb95109114"
> eth.accounts
["0xa6f03d4e14f0897978e62df8d5f3bd0d532570b3", "0x808f02a8dbcafdab4e417215b16d36eb95109114"]
> eth.coinbase
"0xa6f03d4e14f0897978e62df8d5f3bd0d532570b3"
> web3.eth.getBalance(eth.accounts[0]);
0
> web3.eth.getBalance(eth.accounts[1]);
0
> miner
{
  getHashrate: function(),
  setEtherbase: function(),
  setExtra: function(),
  setGasPrice: function(),
  setRecommitInterval: function(),
  start: function(),
  stop: function()
}
> miner.setEtherbase(eth.accounts[0]);
true
> miner.start(1);
null

```


