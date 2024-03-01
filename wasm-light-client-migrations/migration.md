# Migration

This document describes the migration process for the wasm-light-client module, from using `notional-ibc` version to `ibc-light-client` version.

- [notional-labs/ibc-go](https://github.com/notional-labs/ibc-go/tree/6cf43006971fe5862ca420c278a408d189918b98)
- [cosmos/ibc-go](https://github.com/cosmos/ibc-go)

# State changes

Client store is persisted.

# API Changes

## Msgs and Queries

View [differences.md](differences.md)

## Sudo vs Execute

For `notional/ibc`, the `core module`, uses the `execute` entry point to call the contract

```rust
func (c ClientState) UpdateState(ctx sdk.Context, cdc codec.BinaryCodec, store sdk.KVStore, clientMsg exported.ClientMessage) []exported.Height {
	...
	_, err := call[contractResult](payload, &c, ctx, store)
	...
}
```

For `cosmos/ibc`, the `core module` uses the `sudo` entry point to call the contract

```rust
func (cs ClientState) UpdateState(ctx sdk.Context, cdc codec.BinaryCodec, clientStore storetypes.KVStore, clientMsg exported.ClientMessage) []exported.Height {
	result, err := wasmSudo[UpdateStateResult](ctx, cdc, clientStore, &cs, payload)
	if err != nil {
		panic(err)
	}

	return heights
}
```

# Migration path

Since the `client_store` persisted, we need to care the migration connection endppoints between the new `cosmos-ibc` vs `light-client` contract◊

The `wasm client` need to be migrate, to accomodate the new `cosmos-ibc` entrypoint interfaces.

</Br>
Prepare the `wasmClient` code bytes, and update using this methods:

```go
func (k Keeper) migrateContractCode(ctx sdk.Context, clientID string, newChecksum, migrateMsg []byte) error
```

## Add fields update for messages

Update expect struct for expected fields for msgs and queriers. Most of them is to remove unused fields from the `notional-ibc` version. Logic is changed only, state is `not break`

## Using sudo messages instead of execute messages

Migrate `wasm-client`, to support sudo messages.
Instead of using `execute` entrypoints, move to use `sudo` entrypoints for:

- clientState.UpdateState
- clientState.UpdateStateOnMisbehaviour
- clientState.VerifyMembership
- clientState.VerifyNonMembership
- clientState.VerifyUpgradeAndUpdateState
