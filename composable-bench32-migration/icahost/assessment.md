# Assessment

The `ica-host` module `is being affected` by the migration process.

## SetInterchainAccountAddress
This function stores the InterchainAccount address, keyed by the associated connectionID and portID
```go
// SetInterchainAccountAddress stores the InterchainAccount address, keyed by the associated connectionID and portID
func (k Keeper) SetInterchainAccountAddress(ctx sdk.Context, connectionID, portID, address string) {
	store := ctx.KVStore(k.storeKey)
	store.Set(icatypes.KeyOwnerAccount(portID, connectionID), []byte(address))
}
```
And the address is a bech32 address

So the solution here is get all the data from icatypes.KeyOwnerAccount, change the address into new prefix.
