# zksync-era-withdrawal-finalizer

A Withdrawal Finalizer in Rust.

## Purpose

Withdrawal Finalizer is a component of `zksync-era` responsible for monitoring and finalizing [L2->L1 withdrawals](https://github.com/matter-labs/zksync-era/blob/main/docs/advanced/03_withdrawals.md). It does so by continiously monitoring events happening on both L2 and L1, keeping some state in persistent storage (which is PostgreSQL) and sending withdrawal finalization transactions whenever necessary.

## Building

Building the project is straightforward:

```
cargo build
```

## Deploying

To deploy this service you will need the following prerequisites:

1. Websocket RPC endpoint on Ethereum.
2. Websocket RPC endpoint on zkSync Era.
3. An instance of PostgreSQL database.

### Running DB migrations

Prior to deployment of the service the database migrations have to be run with [`sqlx-cli`](https://github.com/launchbadge/sqlx/tree/main/sqlx-cli) component of [`sqlx`](https://github.com/launchbadge/sqlx):

```
$ cd ./storage
$ env DATABASE_URL=postgres://mycreds@myhost/mydb sqlx database create
$ env DATABASE_URL=postgres://mycreds@myhost/mydb sqlx migrate run
```
### Configuration

Configuration is done via enviromnent variables that can also be read from `.env` file if it is present. 
Deployment is done by deploying a dockerized image of the service.

| Variable | Description |
| -------- | ----------- |
| `ETH_CLIENT_WS_URL` | The address of Ethereum WebSocket RPC endpoint |
| `ETH_CLIENT_HTTP_URL` | The address of Ethereum HTTP RPC endpoint |
| `CONTRACTS_L1_ERC20_BRIDGE_PROXY_ADDR` | Address of the L1 ERC20 bridge contract** |
| `CONTRACTS_L2_ERC20_BRIDGE_ADDR` | Address of the L2 ERC20 bridge contract** |
| `CONTRACTS_DIAMOND_PROXY_ADDR` | Address of the L1 diamond proxy contract** |
| `CONTRACTS_WITHDRAWAL_FINALIZER_CONTRACT` | Address of the Withdrawal Finalizer contract ** |
| `API_WEB3_JSON_RPC_WS_URL` | Address of the zkSync Era WebSocket RPC endpoint |
| `API_WEB3_JSON_RPC_HTTP_URL` | Address of the zkSync Era HTTP RPC endpoint |
| `DATABSE_URL` | The url of PostgreSQL database the service stores its state into |
| `GAS_LIMIT` | The gas limit of a single withdrawal finalization within the batch of withdrawals finalized in a call to `finalizeWithdrawals` in WithdrawalFinalizerContract |
| `BATCH_FINALIZATION_GAS_LIMIT` | The gas limit of the finalizastion of the whole batch in a call to `finalizeWithdrawals` in Withdrawal Finalizer Contract |
| `WITHDRAWAL_FINALIZER_ACCOUNT_PRIVATE_KEY` | The private key of the account that is going to be submit finalization transactions |
| `TX_RETRY_TIMEOUT_SECS` | Number of seconds to wait for a potentially stuck finalization transaction before readjusting its fees |
| `TOKENS_TO_FINALIZE` | Configures the sets of tokens this instance of finalizer will finalize. It may be configured as a whitelist, a blacklist, a wildcard or completely disable any finalization. For more info see below. |
| `FINALIZE_ETH_TOKEN` | (Optional) Configure, whether the Ethereum withdrawal events should be monitored. Useful to turn off for custom bridges that are only interested in a particular ERC20 token and have nothing to do with main Ethereum withdrawals |
| `CUSTOM_TOKEN_DEPLOYER_ADDRESSES` | (Optional) Normally ERC20 tokens are deployed by the bridge contract. However, in custom cases it may be necessary to override that behavior with a custom set of addresses that have deployed tokens |
| `CUSTOM_TOKEN_ADDRESSES` | (Optional) Adds a predefined list of tokens to finalize. May be useful in case of custom bridge setups when the regular technique of finding token deployments does not work. |
| `ENABLE_WITHDRAWAL_METERING` | (Optional, default: `"true"`) By default Finalizer collects metrics about withdrawn token volumens. Users may optionally switch off this metering. |

The configuration structure describing the service config can be found in [`config.rs`](https://github.com/matter-labs/zksync-withdrawal-finalizer/blob/main/bin/withdrawal-finalizer/src/config.rs)

** more about zkSync contracts can be found [here](https://github.com/matter-labs/era-contracts/blob/main/docs/Overview.md)

## Configuring Tokens to finalize.

It may be handy to limit a set of tokens the Finalizer is finalizing. This
configuration may be specified by setting a rule in the `TOKENS_TO_FINALIZE` value.
If this enviromnent variable is not set then by default Finalizer will only finalize
ETH token (`0x000...0800a`).

You may specify `All`, `None`, `BlackList` or `WhiteList` as json documents:

1. `TOKENS_TO_FINALIZE = '"All"'` - Finalize everything
1. `TOKENS_TO_FINALIZE = '"None"'` - Finalize nothing
1. `TOKENS_TO_FINALIZE = '{ "WhiteList":[ "0x3355df6D4c9C3035724Fd0e3914dE96A5a83aaf4" ] }'` - Finalize only these tokens
1. `TOKENS_TO_FINALIZE = '{ "BlackList":[ "0x3355df6D4c9C3035724Fd0e3914dE96A5a83aaf4" ] }'` - Finalize all tokens but these

## License

zkSync Withdrawal Finalizer is distributed under the terms of either

- Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or <http://www.apache.org/licenses/LICENSE-2.0>)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or <http://opensource.org/licenses/MIT>)

at your option.

## Official Links

- [Website](https://zksync.io/)
- [GitHub](https://github.com/matter-labs)
- [Twitter](https://twitter.com/zksync)
- [Twitter for Devs](https://twitter.com/zkSyncDevs)
- [Discord](https://discord.gg/nMaPGrDDwk)

## Disclaimer

zkSync Era has been through lots of testing and audits. Although it is live, it is still in alpha state and will go
through more audits and bug bounties programs. We would love to hear our community's thoughts and suggestions about it!
It is important to state that forking it now can potentially lead to missing important security updates, critical
features, and performance improvements.
