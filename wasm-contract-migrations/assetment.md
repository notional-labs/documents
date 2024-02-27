When we change the `bench32-address-prefix`, it will have the following effects on `wasm contract`:

## Contract address got changed to new bench32 prefix

The contract address will be changed to the new bench32 prefix. This means that the contract address will be different from the old one.
</Br>=> it will break API, requires client to update the contract address in their application.

## Contract Creator not being updated

The contract creator will not be updated. This means that the contract creator will still be the same as the old one.

### Example

```txt
code_infos:
- code_id: "1"
  creator: centauri18rj75lw5a8kak6vc36vdt7tcduudwtkrdf20pn
  data_hash: C0E7A3B40D9710F6F72322293BA5CD871714008D9ACCD9A91C0FB08272609054
  instantiate_permission:
    address: ""
    addresses: []
    permission: Everybody
pagination:
  next_key: null
  total: "0"
```

### Propose solution

We can update the contract creator during the chain migration process.

## Contract State not being migrated

Every contract state, which use "&Addr" as a key / value, will be break. The contract state will not be migrated. This means that the contract state will still be the same as the old one, using the address with old prefix:

### Example

For example, this state in `cw20` contract, after the migrate, all the account balances of user, will not be updated correctly. Hence, any of the actions after the upgrade, like `transfer`, or `query` will be consider as the new address. Old data is basically being discarded.

[cw20/state.rs](https://github.com/CosmWasm/cw-plus/blob/main/contracts/cw20-base/src/state.rs)

```rust
pub const TOKEN_INFO: Item<TokenInfo> = Item::new("token_info");
pub const MARKETING_INFO: Item<MarketingInfoResponse> = Item::new("marketing_info");
pub const LOGO: Item<Logo> = Item::new("logo");
pub const BALANCES: Map<&Addr, Uint128> = Map::new("balance");
pub const ALLOWANCES: Map<(&Addr, &Addr), AllowanceResponse> = Map::new("allowance");
// TODO: After https://github.com/CosmWasm/cw-plus/issues/670 is implemented, replace this with a `MultiIndex` over `ALLOWANCES`
pub const ALLOWANCES_SPENDER: Map<(&Addr, &Addr), AllowanceResponse> =
    Map::new("allowance_spender");
```

### Solution

This requires the contract creator to do the migration, iterating over all the state, and update the state with the new address prefix.

[terra/cw-migration](https://docs.terra.money/develop/terrain/contract-migration/)

**TODO**: Add the example of the contract migration process.
