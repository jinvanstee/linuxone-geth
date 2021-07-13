# Go Ethereum Deployment on LinuxONE
This repo describes the steps to build and install Go Ethereum on LinuxONE or Linux on IBM Z.
It also describes the process to setting up a single-node private network using geth and deploying a simple smart contract to it.

The general steps are:
* 1. Install Linux tools
* 2. Install go and configure GOPATH
* 3. Download and install geth
* 4. Run test suite
* 5. Create folder for private network
*	6. Create genesis.json
*	7. Create folder for chaindata
*	8. Geth init with genesis.json
*	9. Geth create two accounts, set first one as coinbase
*	10. Start geth with mining, see blocks start to get mined
*	11. Connect to geth console, run some commands, test transferring ether from 1 account to another
*	12. Install truffle on local machine (x86) 
*	13. Unbox metacoin
*	14. Setup truffle-config.js to point to geth node on LinuxONE
*	15. Truffle compile
*	16. Go back to geth console, and unlock the first account
*	17. Truffle migrate
* 18. Run transactions in truffle console!
