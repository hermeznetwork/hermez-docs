```toml
#[API]
### Url and port where the API will listen
#Address = "0.0.0.0:8086"
### Enables the Explorer API endpoints
#Explorer = true
### Interval between updates of the API metrics
#UpdateMetricsInterval = "10s"
### Interval between updates of the recommended fees
#UpdateRecommendedFeeInterval = "10s"
### Maximum concurrent connections allowed between API and SQL
#MaxSQLConnections = 100
### Maximum amount of time that an API request can wait to establish a SQL connection
#SQLConnectionTimeout = "2s"

[Debug]
### If it is set, the debug api will listen in this address and port
APIAddress = "0.0.0.0:12345"
### Enables meddler debug mode, where unused columns and struct fields will be logged
MeddlerLogs = true
### Sets the web framework Gin-Gonic to run in debug mode
GinDebugMode = true

[StateDB]
### Path where the synchronizer StateDB is stored
Path = "/tmp/hermez/statedb"
### Number of checkpoints to keep
#Keep = 256

[PostgreSQL]
## Port of the PostgreSQL write server
PortWrite     = 5432
## Host of the PostgreSQL write server
HostWrite     = "localhost"
## User of the PostgreSQL write server
UserWrite     = "hermez"
## Password of the PostgreSQL write server
PasswordWrite = "yourpasswordhere"
## Name of the PostgreSQL write server database
NameWrite     = "hermez"

[Web3]
## Url of the web3 Ethereum-node RPC server. Only geth is officially supported
URL = "http://localhost:8545"

#[Synchronizer]
### Interval between attempts to synchronize a new block from an Ethereum node
#SyncLoopInterval = "1s"
### Threshold of a number of Ethereum blocks left to synchronize, such that if there are more blocks to sync than the defined value synchronizer can aggressively skip calling UpdateEth to save network bandwidth and time. After reaching the threshold UpdateEth is called on each block. This value only affects the reported % of synchronization of blocks and batches, nothing else.
#StatsUpdateBlockNumDiffThreshold = 100
### While having more blocks to sync than updateEthBlockNumThreshold, UpdateEth will be called once in a defined number of blocks. This value only affects the reported % of synchronization of blocks and batches, nothing else
#StatsUpdateFrequencyDivider = 100

[SmartContracts]
## Smart contract address of the rollup Hermez.sol
Rollup   = "0x10465b16615ae36F350268eb951d7B0187141D3B"


[Coordinator]
## Ethereum address that the Coordinator is using to forge batches
ForgerAddress = "0xDcC5dD922fb1D0fd0c450a0636a8cE827521f0eD" # Non-Boot Coordinator
# ForgerAddressPrivateKey = "0x705df2ae707e25fa37ca84461ac6eb83eb4921b653e98fdc594b60bea1bb4e52"
### Minimum balance the forger address needs to start the Coordinator in Wei. If it is set to 0, the Coordinator will not check the balance
#MinimumForgeAddressBalance = "0"
### Number of confirmation blocks to be sure that the tx has been mined correctly
#ConfirmBlocks = 5
### Portion of the range before the L1Batch timeout that will trigger a schedule to forge an L1Batch
#L1BatchTimeoutPerc = 0.00001
### Number of delay blocks to wait before starting the pipeline when a slot in which the Coordinator can forge is reached
#StartSlotBlocksDelay = 0
### Number of blocks ahead used to decide when to stop scheduling new batches
#ScheduleBatchBlocksAheadCheck = 0
### Number of margin blocks used to decide when to stop sending batches to the smart contract
#SendBatchBlocksMarginCheck = 0
### Interval between calls to the ProofServer to check the status
#ProofServerPollInterval = "1s"
### Interval between forge retries after an error
#ForgeRetryInterval = "10s"
### Interval between calls to the main handler of a synced block after an error
#SyncRetryInterval = "1s"
#Delay after which a batch is forged if the slot is already committed.  If It is set to "0s", the Coordinator will continuously forge at the maximum rate
ForgeDelay = "0s"
### Delay after a forged batch if there are no txs to forge. If It is set to 0s, the Coordinator will continuously forge even if the batches are empty
ForgeNoTxsDelay = "1s"
### Interval between calls to the PurgeByExternalDelete function of the l2db which deletes pending txs externally marked by the column `external_delete`
#PurgeByExtDelInterval = "1m"
### Enables the Coordinator to forge in slots if the empty slots reach the slot deadline
#MustForgeAtSlotDeadline = true
### It will make the Coordinator forge at most one batch per slot, only if there are included txs in that batch, or pending l1UserTxs in the smart contract.  Setting this parameter overrides `ForgeDelay`, `ForgeNoTxsDelay`, `MustForgeAtSlotDeadline` and `IgnoreSlotCommitment`.
#IgnoreSlotCommitment = true
### This parameter will make the Coordinator forge at most one batch per slot, only if there are included txs in that batch, or pending l1UserTxs in the smart contract.  Setting this parameter overrides `ForgeDelay`, `ForgeNoTxsDelay`, `MustForgeAtSlotDeadline` and `IgnoreSlotCommitment`.
#ForgeOncePerSlotIfTxs = false

[Coordinator.FeeAccount]
## Ethereum address of the account that will receive the fees
Address = "0xCfDe8f47215a147e3876efa0c059771159c4FC70"
# PrivateKey = "0xfdb75ceb9f3e0a6c1721e98b94ae451ecbcb9e8c09f9fc059938cb5ab8cc8a7c"
## BJJ is the baby jub jub public key of the account that will receive the fees
BJJ = "0x1b176232f78ba0d388ecc5f4896eca2d3b3d4f272092469f559247297f5c0c13"
# BJJPrivateKey = "0xb556862fb60e7cf4c0a8a7f44baf2ab06a4c90ac39decc4eef363b6142d07a34"

[Coordinator.L2DB]
### Number of batches after which non-pending L2Txs are deleted from the pool
#SafetyPeriod = 10
### Maximum number of pending L2Txs that can be stored in the pool
#MaxTxs       = 1000000
### Minimum fee in USD that a tx must pay in order to be accepted into the pool
MinFeeUSD    = 0.0
### Maximum fee in USD that a tx must pay in order to be accepted into the pool
#MaxFeeUSD    = 10.00
### Time To Live for L2Txs in the pool. L2Txs older than TTL will be deleted.
#TTL          = "24h"
### Delay between batches to purge outdated transactions. Outdated L2Txs are those that have been forged or marked as invalid for longer than the SafetyPeriod and pending L2Txs that have been in the pool for longer than TTL once there are MaxTxs
#PurgeBatchDelay = 10
### Delay between batches to mark invalid transactions due to nonce lower than the account nonce
#InvalidateBatchDelay = 20
### Delay between blocks to purge outdated transactions. Outdated L2Txs are those that have been forged or marked as invalid for longer than the SafetyPeriod and pending L2Txs that have been in the pool for longer than TTL once there are MaxTxs.
#PurgeBlockDelay = 10
### Delay between blocks to mark invalid transactions due to nonce lower than the account nonce
#InvalidateBlockDelay = 20

[Coordinator.TxSelector]
### Path where the TxSelector StateDB is stored
Path = "/tmp/hermez/txselector"

[Coordinator.BatchBuilder]
### Path where the BatchBuilder StateDB is stored
Path = "/tmp/hermez/batchbuilder"

[Coordinator.ServerProofs]
## Server proof API URLs
URLs = ["http://localhost:3000/api"]

[Coordinator.Circuit]
## Maximum number of txs supported by the circuit
MaxTx = 512
## Maximum number of merkle tree levels supported by the circuit
NLevels = 32

#[Coordinator.EthClient]
### Interval between receipt checks of Ethereum transactions in the TxManager
#CheckLoopInterval = "500ms"
### Number of attempts to do an Eth client RPC call before giving up
#Attempts = 4
### Delay between attempts do do an Eth client RPC call
#AttemptsDelay = "500ms"
### Timeout after which a non-mined Ethereum transaction will be resent (reusing the nonce) with a newly calculated gas price
#TxResendTimeout = "2m"
### Disables reusing nonces of pending transactions for new replacement transactions
#NoReuseNonce = false
### Maximum gas price allowed for Ethereum transactions in Gwei
#MaxGasPrice = 500
### Minimum gas price allowed for Ethereum transactions in Gwei
#MinGasPrice = 5
### Percentage increased of gas price set in an Ethereum transaction from the suggested gas price by the Ethereum node
#GasPriceIncPerc = 5

[Coordinator.EthClient.Keystore]
### Path where the keystore is stored
Path = "/tmp/hermez/ethkeystore"
## Password used to decrypt the keys in the keystore
Password = "yourpasswordhere"

#[Coordinator.EthClient.ForgeBatchGasCost]
### Gas needed to forge an empty batch
#Fixed = 900000
### Gas needed per L1 tx
#L1UserTx = 15000
### Gas needed for a Coordinator L1 tx
#L1CoordTx = 7000
### Gas needed for an L2 tx
#L2Tx = 600

#[Coordinator.API]
### Enables Coordinator API endpoints
#Coordinator = true

[Coordinator.Debug]
## If this parameter is set, specifies the path where batchInfo is stored in JSON in every step/update of the pipeline
BatchPath = "/tmp/hermez/batchesdebug"
## If lightScrypt is set, uses light parameters for the Ethereum keystore encryption algorithm
LightScrypt = false
## RollupVerifierIndex is the index of the verifier to use in the Rollup smart contract. The verifier chosen by index must match with the Circuit parameters. Only for debug purposes. It can't be used as env variable
# RollupVerifierIndex = 0

[Coordinator.Etherscan]
## If this parameter is set, specifies the Etherscan endpoint to get the gas estimations for that momment
URL = "https://api.etherscan.io"
## APIKey parameter allows access to Etherscan services
APIKey = "FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF"

#[RecommendedFeePolicy]
### Strategy used to calculate the recommended fee that the API will expose.
### Available options:
### - Static: always return the same value (StaticValue) in USD
### - AvgLastHour: calculate using the average fee of the forged transactions during the last hour
### Selects the mode. "Static" or "AvgLastHour"
#PolicyType = "Static"
### If PolicyType is "static" defines the recommended fee value
#StaticValue = 0.10
```
