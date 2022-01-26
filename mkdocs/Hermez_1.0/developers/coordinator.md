# Hermez Node
This tutorial describes how to launch a Hermez node. It starts by explaining how to launch a Boot Coordinator in localhost.
Next, it describes how to initialize a Proof Server and how to connect it to the Boot Coordinator.
The next section describes how to spin up a second Hermez node in synchronizer mode to track the rollup status independently from the Boot Coordinator. This second node will be launched in Rinkeby testnet.
The last part of the tutorial includes an explanation on how to add a second Coordinator node to Hermez testnet Rinkeby that bids for the right to forge batches.
 
1. [Preparing the Environment](#preparing-the-environment) 
2. [Launching the Boot Coordinator](#launching-the-boot-coordinator)
3. [Launching a Proof Server](#launching-a-proof-server)
4. [Launching a Price Updater](#launching-a-price-updater)
5. [Launching a Synchronizer Node](#launching-a-synchronizer-node)
6. [Launching a Second Coordinator](#launching-a-second-coordinator-node)

## Preparing the Environment
Hermez node requires a PostgreSQL database and connectivity to an Ethereum node. In this part, we describe how you can set this environment
up using docker containers.

### Dependencies
- [golang 1.16+](https://golang.org/doc/install) 
   - [golangci-lint](https://golangci-lint.run/usage/install/)
   - [packr utility](https://github.com/gobuffalo/packr) to bundle the database migrations. Make sure your `$PATH` contains `$GOPATH/bin`, otherwise the packr utility will not be found.
```shell
cd /tmp && go get -u github.com/gobuffalo/packr/v2/packr2 && cd -
```
- docker and docker-compose without sudo permission (optional if you want to use the provided PostgreSQL and Geth containers)
   - [docker](https://docs.docker.com/engine/install/ubuntu/)
   - [docker-compose](https://docs.docker.com/compose/install/)
- [aws cli 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) (optional if you want to use the provided Geth container)

### Setup
1. Clone [hermez-node](https://github.com/hermeznetwork/hermez-node.git) repository
```shell
git clone https://github.com/hermeznetwork/hermez-node.git
```
2. Build `hermez-node` executable
```shell
cd hermez-node
make 
```
The executable can be found in `dist/heznode`


3. Deploy PostgreSQL database and Geth node containers. For this step we provide a docker-compose file example. Copy [file](/coord-files/docker-compose.sandbox.md) to `docker-compose.sandbox.yaml`.

  Login to AWS public ECR to be able to download the Geth docker image:
```shell
export AWS_REGION=eu-west-3
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/r7d5k1t8
```
Ensure that port 5432 is not being used. Otherwise, PostgreSQL docker will fail.

  To start start database and Geth node containers:
```shell
DEV_PERIOD=3 docker-compose -f docker-compose.sandbox.yaml up -d
```
This command will start a Geth node mining a block every 3 seconds. 
Database is available at port 5432. Geth node is available at port 8545.

  To stop containers:
```shell
docker-compose -f docker-compose.sandbox.yaml down
```

  The Geth container comes with pre-deployed Hermez contracts and with 200 funded accounts.
The relevant information about the contract deployment can be found below

```json
 "hermezAuctionProtocolAddress": "0x317113D2593e3efF1FfAE0ba2fF7A61861Df7ae5"
 "hermezAddress": "0x10465b16615ae36F350268eb951d7B0187141D3B"
 "withdrawalDelayeAddress": "0x8EEaea23686c319133a7cC110b840d1591d9AeE0"
 "HEZTokenAddress": "0x5E0816F0f8bC560cB2B9e9C87187BeCac8c2021F"
 "hermezGovernanceIndex": 1
 "hermezGovernanceAddress": "0x8401Eb5ff34cc943f096A32EF3d5113FEbE8D4Eb"
 "emergencyCouncilIndex": 2
 "emergencyCouncilAddress": "0x306469457266CBBe7c0505e8Aad358622235e768"
 "donationIndex": 3
 "donationAddress": "0xd873F6DC68e3057e4B7da74c6b304d0eF0B484C7"
 "bootCoordinatorIndex": 4
 "mnemonic": "explain tackle mirror kit van hammer degree position ginger unfair soup bonus"
 "chainId" : 1337
}
```


4. Customize Hermez Node configuration file. For this example, we can use [this configuration file](/coord-files/cfg.sandbox.boot-coordinator.md). Just copy this file to `cmd/heznode/cfg.sandbox.boot.coordinator.toml`

  For more information on the parameters in the configuration file, read the [configuration parameters description](https://github.com/hermeznetwork/hermez-node/blob/master/config/config.go#L57).

5. Ensure correct permissions are granted to /var/hermez folder
```sh
sudo mkdir -p /var/hermez
sudo chown $USER:$USER /var/hermez
```

## Launching the Boot Coordinator
It is recommended to run the Coordinator node in a server with 8+ cores, 16 GB+ of RAM and 250GB of disk (AWS c5a.2xlarge or equivalent).

1. Import the Coordinator and Fee Ethereum accounts private keys into the keystore. 
```shell
./dist/heznode importkey --mode coord --cfg ./cmd/heznode/cfg.sandbox.boot-coordinator.toml --privatekey 0x705df2ae707e25fa37ca84461ac6eb83eb4921b653e98fdc594b60bea1bb4e52
./dist/heznode importkey --mode coord --cfg ./cmd/heznode/cfg.sandbox.boot-coordinator.toml --privatekey 0xfdb75ceb9f3e0a6c1721e98b94ae451ecbcb9e8c09f9fc059938cb5ab8cc8a7c
```
The Coordinator account is used to pay the gas required to forge batches. The  Fee account is used to collect the fees paid by users submitting transactions to Hermez Network.
You only need to import these keys once.
 
2. Start a mock proof server. 
```shell
cd test/proofserver/cmd
go build -o proof-server
./proof-server -d 15s -a 0.0.0.0:3000
```
The `hermez-node` repository provides a mock proof server that generates proofs every 15 seconds. The mock prover is launched at http://localhost:3000, and it exports two endpoints:
- GET /api/status: Queries the prover's status.
- POST /api/input: Starts the generation of a new proof.

3. Wipe SQL database

Before starting the Coordinator node, you may want to wipe the pre-existing SQL database. This command will wipe the pre-existing database if it exists, and it will force the Coordinator to resynchronize the full state.
```shell
./dist/heznode wipedbs --mode coord --cfg cmd/heznode/cfg.sandbox.boot-coordinator.toml 
```

4. Launch the Hermez Node

```shell
./dist/heznode run --mode coord --cfg cmd/heznode/cfg.sandbox.boot-coordinator.toml
```

Once the Hermez Node is launched, the API can be queried at `localhost:8086/v1`. You can find more information on the API [here](http://localhost:3000/#/developers/api)

## Launching a Proof Server
We will use [rapidsnark](https://github.com/iden3/rapidsnark) as the Hermez proof server. `rapidsnarks` is a zkSnark proof generator written in C++.
It is recommended to run the proof server in servers with 48+ cores, 96 GB+ of RAM and 250GB of disk (AWS c5a.12xlarge or equivalent).

> rapidsnark requires a host CPU that supports ADX extensions. You can check this with `cat /proc/cpuinfo | grep adx`

### Dependencies
- [node v16+](https://nodejs.org/en/download/)
- npm
```shell
apt install npm
```
- npx
```shell
npm i -g npx
```

- Install gcc, libsodium, gmp, cmake
```
sudo apt install build-essential
sudo apt-get install libgmp-dev libsodium-dev nasm cmake
```

### Circuit Files
Download circuit and auxiliary files. These files are extremely large (20GB+), so make sure you have enough bandwidth and disk space.

There are two Hermez circuits that have undergone the Trusted Setup Ceremony. 
- circuit-2048-32-256-64 with 2048 transactions per batch (~2^27 constraints)
- circuit-400-32-256-64 with 400 transactions per batch (~2^25 constraints)

For each type of circuit, you will need the following files:
- C++ source file (extension .cpp)
- Data file (extension .dat)
- Verification and Proving Key files (extension .zkey)

[circuit-400-32-256-64.cpp](https://hermez.s3-eu-west-1.amazonaws.com/circuit-400-32-256-64.cpp)

[circuit-400-32-256-64.dat](https://hermez.s3-eu-west-1.amazonaws.com/circuit-400-32-256-64.dat)

[circuit-400-32-256-64_hez4_final.zkey](https://hermez.s3-eu-west-1.amazonaws.com/circuit-400-32-256-64_hez4_final.zkey)

[circuit-2048-32-256-64.cpp](https://hermez.s3-eu-west-1.amazonaws.com/circuit-2048-32-256-64.cpp)

[circuit-2048-32-256-64.dat](https://hermez.s3-eu-west-1.amazonaws.com/circuit-2048-32-256-64.dat)

[circuit-2048-32-256-64_hez4_final.zkey](https://hermez.s3-eu-west-1.amazonaws.com/circuit-2048-32-256-64_hez4_final.zkey)


More information on Trusted Setup can be found [here](https://github.com/hermeznetwork/phase2ceremony_4).

### Setup
1. Clone [rapidsnark](https://github.com/iden3/rapidsnark.git) repository
```shell
git clone  https://github.com/iden3/rapidsnark.git
```
2. Compile the prover.

  In this example we are building the 400 transactions prover.
```shell
cd rapidsnark
npm install
git submodule init
git submodule update
npx task createFieldSources
npx task buildPistche
npx task buildProverServer ../circuit-400-32-256-64.cpp
```
3. Launch prover
```shell
cd ..
./rapidsnark/build/proverServer circuit-400-32-256-64.dat circuit-400-32-256-64_hez4_final.zkey
```
Prover is deployed at port 9080.

4. Check prover status
```shell
curl -i -H "Accept: application/json" -H "Content-Type: application/json" -X GET http://localhost:9080/status
```

### Generate a Prover Input File

1. Clone [circuits](https://github.com/hermeznetwork/circuits.git) repository
```shell
git clone https://github.com/hermeznetwork/circuits.git
```

2. Install dependencies
```shell
cd circuits
npm install
cd tools
```

In this example we are working with `circuit-400-32-236-64_hez1.zkey`, which corresponds to a circuit with 400 transactions, 32 levels, 256 maxL1Tx and 64 maxFeeTx.

3. Generate Input file

To generate a new input file with empty transactions:

```shell
node build-circuit.js input 400 32 256 64
```
This command generates a new input file `rollup-400-32-256-64/input-400-32-256-64.json`

To generate a new input file with random transactions
```shell
node generate-input.js 256 144 400 32 256 64
``` 
This will create a new input file called `inputs-256.json` 

### Generate a New Proof

You can use curl to post any of the inputs generated in the previous step.
```shell
curl -X POST -d @inputs-256.json http://localhost:9080/input
```
or
```shell
curl -X POST -d @input-400-32-256-64.json http://localhost:9080/input
```

You check the status of the prover by querying the `/status` endpoint.
```shell
curl -i -H "Accept: application/json" -H "Content-Type: application/json" -X GET http://localhost:9080/status
```
`/status` returns if the prover is ready to accept a new input as well as the proof result and input data of the previous iteration. An example is shown below.

```json
{"proof":"{\"pi_a\":[\"15669797899330531899539165505099185328127025552675136844487912123159422688332\",\"4169184787514663864223014515796569609423571145125431603092380267213494033234\",\"1\"],\"pi_b\":[[\"15897268173694161686615535760524608158592057931378775361036549860571955196024\",\"7259544064908843863227076126721939493856845778102643664527079112408898332246\"],[\"11114029940357001415257752309672127606595008143716611566922301064883221118673\",\"11641375208941828855753998380661873329613421331584366604363069099895897057080\"],[\"1\",\"0\"]],\"pi_c\":[\"3069279014559805068186938831761517403137936718184152637949316506268770388068\",\"17615095679439987436388060423042830905459966122501664486007177405315943656120\",\"1\"],\"protocol\":\"groth16\"}","pubData":"[\"18704199975058268984020790304481139232906477725400223723702831520660895945049\"]","status":"success"}
```

### Connect Prover to Coordinator Node
Once you have verified the prover is working, you can connect it to the Hermez Coordinator by configuring the `cfg.sandbox.boot-coordinator.toml` configuration file.
You need to substitute sections `ServerProofs` with the updated URLs where prover is deployed, and the `Circuit` section where the verifier smart contract is specified.

```
[Coordinator.ServerProofs]
#TODO: Add Prover URL
#URLs = ["http://localhost:9080"]
```

```
[Coordinator.Circuit]
MaxTx = 400
NLevels = 32
```

At this point, you can stop the mock server if it is still running, and re-launch the coordinator as we saw in the previous section. The new prover will be running at http://localhost:9080 (or at the configured URL), and the two endpoints are `/status` and `/input`


## Launching a Price Updater
Price Updater service is used to consult and updater the tokens and fiat currency used by Hermez Node. Once Hermez Node has been deployed, the Price Updater service can be deployed. Follow these [instructions](../developers/price-updater.md#price-updater) to set up the Price Updater service.

## Launching a Synchronizer Node
In synchronizer mode, the node is capable of keeping track of the rollup and consensus smart contracts, storing all the history of events, and keeping the rollup
state updated, handling reorgs when they happen. This mode is intended for entities that want to gather all the rollup data by themselves and not rely on third party APIs.
For this part of the tutorial, we are going to deploy the syncrhonizer node in testnet on Rinkeby.

1. Stop Coordinator node launched in localhost in previous steps.

  Stop prover, coordinator node and containers from previous phases as you will be working in testnet with a real Boot Coordinator node.
```shell
docker-compose -f docker-compose.sandbox.yaml down
```

2. Launch PostgreSQL database.

  The Hermez node in synchronizer mode needs to run on a separate database
```shell
docker run --rm --name hermez-db -p 5432:5432 -e POSTGRES_DB=hermez -e POSTGRES_USER=hermez -e POSTGRES_PASSWORD="yourpasswordhere" -d postgres
```

3. Start an Ethereum node in Rinkeby

You will need to run your own Ethreum node on Rinkeby. We recommend using Geth.
- Pre-built binaries for all platforms on our downloads page (https://geth.ethereum.org/downloads/).
- Ubuntu packages in our Launchpad PPA repository (https://launchpad.net/~ethereum/+archive/ubuntu/ethereum).
- OSX packages in our Homebrew Tap repository (https://github.com/ethereum/homebrew-ethereum).

Sync this node with Rinkeby testnet where all of Hermez's smart contracts are deployed. 

4. Get contract addresses

  Query Testnet API for the addresses of the Hermez smart contracts. You can use a web browser or the command below.

```shell
curl -i -H "Accept: application/json" -H "Content-Type: application/json" -X GET api.testnet.hermez.io/v1/config
```
  At this moment, Hermez Network is deployed in this address:

```
"Rollup":"0x679b11e0229959c1d3d27c9d20529e4c5df7997c"
```

5. Copy configuration [file](/coord-files/cfg.testnet.coord.md) to `hermez-node/cmd/heznode/cfg.testnet.sync.toml`. You will need to edit the following sections:
- **PostgreSQL** Values provided are valid for docker postgreSQL container. You will need to supply the actual values for your database.  
- **Web3** URL of your Rinkeby Ethereum node
- **SmartContracts** Double check that the address provided in the configuration file corresponds to the current Hermez Network contract deployed in Rinkeby

6. Launch `hermez-node` in synchronizer mode
```shell
./dist/heznode run --mode sync --cfg cmd/heznode/cfg.testnet.sync.toml
```
7. Kill and relaunch Price Updater service in testnet
Follow [instructions](../developers/price-updater.md) to set up the Price Updater service.

Once the Hermez node is launched, the API can be queried at the location specified in the configuration file in `API.Address` section, as well as at https://api.testnet.hermez.io/v1/ serviced by the Boot Coordinator node.

## Launching a Second Coordinator Node
In this part of the tutorial we will start a second Coordinator Node in testnet that will bid for the right to forge batches.

### Dependencies
- node 14+

### Start Coordinator in Testnet
1. Stop Synchronizer node and PostgreSQL container launched in previous steps.

2. Launch PostgreSQL database

```shell
docker run --rm --name hermez-db -p 5432:5432 -e POSTGRES_DB=hermez -e POSTGRES_USER=hermez -e POSTGRES_PASSWORD="yourpasswordhere" -d postgres
```
3. Launch Prover as shown [here](#launching-a-proof-server)

4. Create two Ethereum accounts in Rinkeby using Metamask wallet. One account is the `forger` account (needs to pay for gas to forge batches in Ethereum and for bids in auction in HEZ), and the second is the `fee` account (receives the HEZ fees). The fees are collected in L2. You can convert from ETH to HEZ in [Uniswap](https://app.uniswap.org/#/swap?use=V2)

5. Create a Wallet with `fee` account Ethereum Private Key. 

This wallet is needed to generate a Baby JubJub address where fees will be collected. There is an example code in the [SDK](https://github.com/hermeznetwork/hermezjs/blob/main/examples/create-wallet.js) that can be used. Simply substitute `EXAMPLES_WEB3_URL` by your Rinkeby Node URL and `EXAMPLES_PRIVATE_KEY1` by `fee` account private key.

This script will generate a similar output:
```json
{
  privateKey: <Buffer 3e 12 35 91 e9 99 61 98 24 74 dc 9c 09 70 0a cb d1 a5 c9 6f 34 2f ab 35 ca 44 90 01 31 f4 dc 19>,
  publicKey: [
    '554747587236068008597553797728983628103889817758448212785555888433332778905',
    '5660923625742030187027289840534366342931920530664475168036204263114974152564'
  ],
  publicKeyHex: [
    '139f9dba06599c54e09934b242161b80041cda4be9192360b997e4751b07799',
    'c83f81f4fce3e2ccc78530099830e29bf69713fa11c546ad152bf5226cfc774'
  ],
  publicKeyCompressed: '5660923625742030187027289840534366342931920530664475168036204263114974152564',
  publicKeyCompressedHex: '0c83f81f4fce3e2ccc78530099830e29bf69713fa11c546ad152bf5226cfc774',
  publicKeyBase64: 'hez:dMfPJlK_UtFqVByhP3FpvykOg5kAU3jMLD7OTx_4gwzO',
  hermezEthereumAddress: 'hez:0x74d5531A3400f9b9d63729bA9C0E5172Ab0FD0f6'
}
```
The Baby JubJub address is `publicKeyCompressedHex`. In this case, `0x0c83f81f4fce3e2ccc78530099830e29bf69713fa11c546ad152bf5226cfc774`.

6. Copy configuration [file](/coord-files/cfg.testnet.coord.md) to `hermez-node/cmd/heznode/cfg.testnet.coord.toml`. You will need to edit the following sections:
- **PostgreSQL** Values provided are valid for docker postgreSQL container. You will need to supply the actual values for your database.  
- **Web3** URL of your Rinkeby Ethereum node
- **SmartContracts** Double check that the address provided in the configuration file corresponds to the current Hermez Network contract deployed in Rinkeby
- **Coordinator.ForgerAddress** Ethereum account in Ethereum Rinkeby. This account is used to bid during the slots auction and to pay the gas to forge batches in Ethereum
- **Coordinator.FeeAccount** You need to supply the Fee account in Ethereum Rinkeby and the Baby JubJub address computed in previous step. This account is used to colled the fees paid by transactions.
- **Coordinator.ServerProofs** Provide a valid URL for the proof server.
- **Coordinator.Circuit** Ensure the `MaxTx` parameters matches with the circuit size configed in the proof server.

7. Import the `forger` and `fee` Ethereum private keys into the keystore. 
```shell
./dist/heznode importkey --mode coord --cfg cmd/heznode/cfg.coord.toml --privatekey <FORGER ACCOUNT_PRIVATE KEY>
./dist/heznode importkey --mode coord --cfg cmd/heznode/cfg.coord.toml --privatekey <FEE_ACCOUNT PRIVATE KEY>
```
This private key corresponds to the new Coordinator node 

8. Launch New Coordinator Node
```shell
./dist/heznode run --mode coord --cfg cmd/heznode/cfg.testnet.coord.toml
```
The node will start synchronizing with the Hermez Network in testnet. This may take a while.

9. Launch new Price Updater service and connect it to the newly launched Hermez node
Follow [instructions](../developers/price-updater.md) to set up the Price Updater service.

### Bidding Process
Once the node is synchronized, you can start bidding for the right to forge a batch.

1. Install cli-bidding

  `cli-bidding` is a tool that allows to register a Coordinator in Hermez Network and place bids in the auction.
```shell
git clone https://github.com/hermeznetwork/cli-bidding.git
```
Once downloaded, follow the installation steps in the [README](https://github.com/hermeznetwork/cli-bidding/blob/master/readme.md).
>NOTE that `PRIVATE_KEY_CLI_BIDDING` corresponds to the `forger` private key.

2. Approve HEZ transfers.

  Before the coordinator can start bidding, it needs to approve the use of HEZ tokens. To do this go to [HEZ address in Etherscan](https://rinkeby.etherscan.io/token/0x2521bc90b4f5fb9a8d61278197e5ff5cdbc4fbf2), select `Contract` -> `Write Contract` -> `Approve` and set `spender address` to Coordinator address and `value` to quantity you want to approve. The recommendation is to set this quantity value very high.

3. Register Forger

  Using `cli-bidding`, you need to register the new Coordinator API URL. In our case, we have the Coordinator node running at `http://134.255.190.114:8086`
```
node src/biddingCLI.js register --url http://134.255.190.114:8086
```
>NOTE. In order for the wallet-ui to be able to forward transactions to this coordinator, the API needs to be accessible from a https domain.

4. Get Current Slot and Minimum Bid in Hermez bid

  Take a look at the current slot being bid in Hermez. When bidding, you need to bid at least 2 slots after the curent slot
```shell
node src/biddingCLI.js slotinfo
```

  In our case, minimum bidding is set to 11.0 HEZ, and first biddable slot is 4200.

5. Bidding Process

  Send a simple bid of $11 \times 10^{18}$ HEZ for slot 4200. 

```shell
node src/biddingCLI.js bid --amount 11 --slot 4200 --bidAmount 11
```
  Parameter `amount` is the quantity to be transferred to the auction smart contract, and  `bidAmount` is the actual bid amount. 

  If the bidding process is successful, an Etherscan URL with the transaction id is returned to verify transaction.

  You can check the allocated nextForgers using /v1/state endpoint

`cli-bidding` provides additional mechanisms to bid in multple slots at once. Check the [README file](https://github.com/hermeznetwork/cli-bidding)


