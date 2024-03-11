# Composable Bench-32 prefix address migration

This is the documentation for the migration process of the `bench32` prefix, from `centauri` to `pica`

## Criteria

- Assetment: documents how migration affects the state / interaction with the module / contracts
- Migration: Provide migration path if there is any migration required

## Module affected
- Auth [Link Text](./auth/assessment.md) @dzmitry-lahoda +1
- Gov [Link Text](./gov/assessment.md) @dzmitry-lahoda +1
- Interchain-account host [Link Text](./icahost/assessment.md) @dzmitry-lahoda +1
- Mint [Link Text](./mint/assessment.md) @dzmitry-lahoda +1
- Slashing [Link Text](./slashing/assessment.md)
- Staking [Link Text](./staking/assessment.md)
- Transfermiddleware [Link Text](./transfermiddleware/assessment.md) @dzmitry-lahoda +1
- Wasmd [Link Text](./wasm-contracts/assessment.md) @dzmitry-lahoda +1

## Module not affected
- Alliance [Link Text](./alliance/assessment.md)
- Authz [Link Text](./authz/assessment.md) @dzmitry-lahoda +1
- Bank [Link Text](./bank/assessment.md) @dzmitry-lahoda +1
- Capability [Link Text](./capability/assessment.md)
- Consensus [Link Text](./consensus/assessment.md) @dzmitry-lahoda +1
- Crisis [Link Text](./crisis/assessment.md)
- Distribution [Link Text](./distribution/assessment.md)
- Evidence [Link Text](./evidence/assessment.md)
- Feegrant [Link Text](./feegrant/assessment.md)
- Genunil [Link Text](./genunil/assessment.md)
- Group [Link Text](./group/assessment.md)
- IBC-Tranfer [Link Text](./ibc-transfer/assessment.md) @dzmitry-lahoda +1
- Params [Link Text](./params/assessment.md)
- Stakingmiddleware [Link Text](./stakingmiddleware/assessment.md)
- Upgrade [Link Text](./upgrade/assessment.md)
- IBC-Hooks [Link Text](./ibc-hooks/assessment.md) @dzmitry-lahoda +1
- Ratelimit [Link Text](./ratelimit/assessment.md) @dzmitry-lahoda +1


## List of actionable items
### Before chain upgrade
Need PR before chain upgrade, merge after chain upgrade
1) Keplr

    a) Keplr-chain-registry 
    - [Update bench32 config in keplr](https://github.com/chainapsis/keplr-chain-registry/blob/main/cosmos/centauri.json#L22)
    - Update chainName -> Picasso
    - Move file centauri.json -> pica.json
    
    b) Keplr-app-registry
    - [Update appName from Composable -> Picasso ](https://github.com/chainapsis/keplr-app-registry/blob/main/apps/composable/app.json)
    - Update folder composable -> picasso

2) Cosmos chain-registry
    - Update folder composable -> picasso
    - Update chain_name in assetlist.json
    - Update info (name, bench32 prefix, ...) in chain.json

3) Ping.pub
    - [Move file to pica.json ](https://github.com/ping-pub/ping.pub/blob/main/chains/mainnet/centauri.json)
    - Update chain_name and addr_prefix

### After chain upgrade
Merge keplr fist, and config relayer 

4) Relayer
    - Update config account-prefix to pica