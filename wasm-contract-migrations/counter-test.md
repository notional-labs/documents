This test the affectionn of migrating bench32 prefix from `centuari` to `pica` to the `counter` contract

## Contract context

The `counter` program, is a program that stores a list of counters, each identified by an address.

### Specs

- State: Map<Addr, Counter>
- Functions
  - increment(sender: Addr): increment counter base on sender address
    - This results in the value of Map<Addr, Counter> increase to 1
  - get_count(addr: Addr): get the address of the counter

```rust
pub const COUNTERS: Map<Addr, Counter> = Map::new("state");

ExecuteMsg::Increment {} => execute::increment(deps, info),
QueryMsg::GetCount { addr } => to_binary(&query::count(deps, addr)?),
```

## Test context

- Running chain at `v6.4.4`.
  - Setup `wallet1` and `wallet2`
  - Deploy and instantiate a `counter contract`
  - `wallet1` : Increment the counter value
  - `wallet1`: Query the counter value
- Doing a chain upgrade, from `v6.4.4` to `v6.4.6`

- Continue running chain at `v6.4.6` after upgrade
  - Get the deployed contract
  - **Test new address prefix**
    - `wallet2`: Doing the `increment()` and the `get_count()`
    - `wallet1` queries its state again, using the `getCount()`
  - **Test old address prefix**
    - `wallet1` calls the `increment()`
    - `wallet2` queries its state again, using the `getCount()`

## Test outputs

- Pre upgrade:

```txt
running old node
executing additional pre scripts from ./scripts/upgrade/v_6_4_6/pre-script.sh

 ********** Running Pre-Scripts **********
binary value: _build/old/centaurid
wallet 1: centauri15drjsrg8xgk454w36x8ch3t2qh0duj9suyhf9x - balance: 1000000000000000000000
code id: 1
Contract address deployed at: centauri1z4udu7rhupmhuc4ywxqk79pezl644fcr5agfnrr78csvakx90s3sswq445
wallet1: call the increment() function
wallet1: call the get_count() function
{"data":{"count":1}}
Assertion passed: Counter value is 1 as expected
CONTRACT_ADDRESS = centauri1z4udu7rhupmhuc4ywxqk79pezl644fcr5agfnrr78csvakx90s3sswq445
```

Here, the `wallet1`: `centauri15drjsrg8xgk454w36x8ch3t2qh0duj9suyhf9x` call the `increment` function. The it calls the `get_count()`, in which returns `1` as expected.

- Upgrade

```txt
=> =>start upgrading
UPGRADE_HEIGHT = 43
"PROPOSAL_STATUS_VOTING_PERIOD"
BLOCK_HEIGHT = 33
"PROPOSAL_STATUS_VOTING_PERIOD"
BLOCK_HEIGHT = 35
"PROPOSAL_STATUS_VOTING_PERIOD"
BLOCK_HEIGHT = 37
"PROPOSAL_STATUS_PASSED"
BLOCK_HEIGHT = 39
"PROPOSAL_STATUS_PASSED"
BLOCK_HEIGHT = 41
BLOCK HEIGHT = 43 REACHED, KILLING OLD ONE


=> =>continue running nodes after upgrade
```

The upgrade is successful, and the chain is running at `v6.4.6`

- Post ugprade

```txt
=> =>continue running nodes after upgrade
executing additional after scripts from ./scripts/upgrade/v_6_4_6/post-script.sh

 ********** Running Post-Scripts **********
wallet 1: pica18rj75lw5a8kak6vc36vdt7tcduudwtkrjlykp8 - balance: 999999999999999775000
wallet 2: pica1s2zuxaassq7m5cfduhj2whjclw8vsp6nlhxgu0
binary value: _build/new/picad

 Fetching the new contract address (it got changed after the upgrade)
Query contract address: pica1mgp7y5hcf47fsu2xj2tfv5znmvmrwghc8ewcrl8au94vn6c5ju6sltwha8

 Testing with new wallet / wallet that has not interacted with the contract before
wallet2: init the counter
wallet2: call the increment() function
wallet2: call the get_count() function
COUNTER_VALUE_2 = 1

 Testing with wallet that has interacted with the contract before
wallet1: call the increment() function
wallet1: call the get_count() function
{"data":{"count":0}}
```

Here, we assert two cases:

- The wallet that has not interacted with the contract before, `wallet2`:

  - It calls the `increment()` function
  - It calls the `get_count()` function
  - The counter value is `1` as expected

- The wallet that has interacted with the contract before, `wallet1`:
  - It calls the `increment()` function // this equals to init the counter
  - It calls the `get_count()` function
  - The counter value is `0`, **is not as expected** since the value should be `2`

## Conclusion

The upgrade is successful, but the state of the contract is not migrated correctly. The `wallet1` should have the counter value of `2` instead of `0`. This is due to the migration of the address prefix from `centauri` to `pica` that is not handled correctly.

Doing the query to the `picad` (after upgrade), we can see that the old state is still persisted.

```txt
_build/new/picad query wasm contract-state smart pica1mgp7y5hcf47fsu2xj2tfv5znmvmrwghc8ewcrl8au94vn6c5ju6sltwha8 '{"get_count": {"addr": "centauri18rj75lw5a8kak6vc36vdt7tcduudwtkrdf20pn"}}'
data:
  count: 1
```

Then, doing the `increment()` using `wallet1`, the contract will consider `wallet1` as the completely new address. => the old data will be discarded, and the state is break, contract might not working as intended.
