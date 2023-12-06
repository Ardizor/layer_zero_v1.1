## Practically an AIO script for LayerZero.

The script is prepared exclusively for educational purposes.
This script was developed for educational purposes only.

## Changes in v1.1:

- Added the `MIN_STABLE_FOUND` variable in the main_config.py file, so the script does not pick up every little thing from wallets, but only works with the amount of stablecoins you need. Example: if the wallet has 2 USDT and 0.11 USDC, and if you set `MIN_STABLE_FOUND` = 1.0, then the script will only work with USDT.
- Removed single-networks, made everything primary in Stargate, so people do not get confused. Network settings are done in the **main_config.py** and in **stargate\_._settings.py**
- Optimized some inaccuracies.
- Removed USDT from the Arbitrum network, as there were several problems with swaps.
  As soon as stablecoin support is added to the zkSync and zkEVM networks, I will immediately add them to the script.

### What the script can do:

- Bridge ALL (!!!) available USDT/USDC from any network to any network (except ETH, FTM (due to pool deactivation), Metis)
- Buy STG in `POLYGON` and stake them
- Buy BTC in `AVALANCHE`, bridge them through btcbridge to `POLYGON`, bridge back and sell
- Swap ETH in `ARBITRUM` or `OPTIMISM` for GoerliETH through https://testnetbridge.com/, one-way
- Bridge USDT to `APTOS` through https://theaptosbridge.com/bridge, one-way
- Bridge USDT to `HARMONY` through https://bridge.harmony.one/erc20, one-way
- Dilute transaction activities with approvals in different networks
- Dilute transaction activities with swaps in different networks
- Work in multi-thread
- Choose random wallets and perform activities in random order
- Record general logs of all actions and separately for wallets

#### Preparation for use

Networks in the script are divided into single and non-single. Single networks - `ARBITRUM`, `BSC`, `OPTIMISM` - networks FROM which you can bridge, but TO which you cannot. This is done for savings, as bridging to these networks is about twice as expensive as from these networks. Non-single networks - `POLYGON`, `AVALANCHE` - are the main networks.
If you have no programming experience, reread the instruction - conduct the initial setup, and do not change other code.
Also, we strongly recommend first running a test of 2-3 wallets, to understand the logic of the script's operation.

# main_config.py

`WAIT_TIME = range(5, 10)` - delay between actions, in minutes
`RATIO_STARGATE_SINGLE = 0.5` - the percentage of the USDT/USDC balance that will be used for bridging from single networks (1 = 100%, 0.5 = 50%)
`STARGATE_CHAIN_LIST` - a list of non-single networks. You can add your own (those that are presented on Stargate, naturally). If you do not want to delve into the code - skip, as writing a separate guide for describing the process of addition would be required.

# max_setting.csv

A table with commission settings (in dollars).
`Activity` - activity
`MAX_GAS` - maximum commission per transaction (gas)
`MAX_VALUE` - maximum value Stargate (see screenshot below)"

# RPCs.py

Fill it with your own or public RPCs.

_Free private RPCs:_

- https://www.alchemy.com/ (no problems were encountered)
- https://www.quicknode.com/
- https://accounts.lavanet.xyz/ (sometimes they were dropping out)

# stargate_settings.py

Optional!
Here you can change the bridge routes.

For example, let's take ARBITRUM:
POLYGON and AVALANCHE are specified in `chain_to`. If we want to add the ability to bridge from ARBITRUM to OPTIMISM - we add `OPTIMISM`, on a new line.

# data.csv

The heart of the script! Pay special attention to filling out this table. In all decimal numbers (17.89) a dot (".") is used, not a comma.

- `DO` - whether to run this wallet. For the script to start running it, you need to write an English "X". Without "X" it doesn't matter what activities are selected further. After completing the run of all selected activities of the wallet, "X" will change to "DONE".
- `Name` - name of the wallet (necessary for logs).
- `Wallet` - wallet address in the EVM network.
- `Private_key` - private key of the wallet.
- `Proxy` - field for proxy. It doesn't work yet, leave it empty.
- `Stargate` - log - the total number of bridges through Stargate after starting the script, no need to change anything, needed for your reporting. Initially = 0.
- `Stargate_range` - sum from and to - how much USDT/USDC to buy in non-single networks. If your wallets don't have USDT/USDC in non-single networks, then the script will buy a random amount from the specified range in a random network (!!!).
- `STARGATE_FIRST_SWAP` - specify "DONE" if it's not necessary to purchase USDT/USDC in non-single networks (it's assumed that one of the networks already has stables for the run). If buying stablecoins is necessary - leave it empty.
- `STARGATE_POLYGON/AVALANCHE` - how many bridges from this network need to be made through StarGate. Initially 0. IMPORTANT! Should be 0, not an empty string, otherwise, it will give an error.
- `STARGATE_BSC` - analogous to STARGATE_POLYGON/AVALANCHE, but for BSC.
- `STARGATE_BSC_RANGE` - the value, how much to buy USDT/USDC in BSC, analogous to `STARGATE_RANGE`.
- `STARGATE_BSC_FIRST_SWAP` - analogous to STARGATE_FIRST_SWAP, but for BSC.
- `STARGATE_ARBITRUM` - analogous to STARGATE_POLYGON/AVALANCHE, but for ARBITRUM.
- `STARGATE_ARBITRUM_RANGE` - analogous to STARGATE_POLYGON/AVALANCHE, but for ARBITRUM.
- `STARGATE_ARBITRUM_FIRST_SWAP` - analogous to STARGATE_FIRST_SWAP, but for ARBITRUM.
- `STARGATE_LIQ` - adding liquidity, currently not working.
- `STARGATE_LIQ_VALUE` - how much liquidity to add to Stargate, currently not working.
- `STARGATE_STG` - whether to buy and stake STG. The module works on POLYGON.
- `STARGATE_STG_RANGE` - sum from and to in $, range of how much STG to buy for staking.
- `BTC_BRIDGE` - how many bridges to make from AVALANCHE to POLYGON and back with buying-selling BTC.b. IMPORTANT! Should be 0, not an empty string, otherwise, it will give an error.
- `BTC_BRIDGE_RANGE` - the range in $ equivalent to buy BTC.b for operation.
- `BTC_BRIDGE_STEP` - stages of BTC_BRIDGE, do not touch. Initially empty. With each stage, an "X" will appear, indicating at which stage the script is: One X - bought BTC.b. Two X - bridged BTC.b to avalanche from polygon. Three X - bridged BTC.b to polygon from avalanche. Empty string - the script completed the sale of BTC.b and subtracted 1 (unit) from the BTC_BRIDGE column.
- `TESTNET_BRIDGE` - how many times to swap ETH for GETH through testnetbridge.
- `TESTNET_BRIDGE_RANGE` - the range of buying GETH in $ equivalent.
- `TESTNET_BRIDGE_CHAINS` - which networks to use in testnetbridge. Selects randomly. Write with a comma, in capital letters.
- `APTOS_BRIDGE` - how many times to bridge USDT from BSC to APTOS.
- `APTOS_BRIDGE_RANGE` - the range of buying USDT/USDC.
- `APTOS_BRIDGE_WALLET` - Unique Aptos wallet to which USDT will be bridged. (Generate on Cointool for convenience)
- `HARMONY_BRIDGE` - how many times to bridge USDT from BSC to HARMONY
