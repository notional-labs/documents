# Assessment

The `transfermiddleware` module `is being affected` by the migration process.

## SetAllowRlyAddress
This function takes rlyAddress as a bech32 address. And it is used as a key to store data.
```go
func (keeper Keeper) SetAllowRlyAddress(ctx sdk.Context, rlyAddress string) {
	store := ctx.KVStore(keeper.storeKey)
	store.Set(types.GetKeyByRlyAddress(rlyAddress), []byte{1})
}
```

So the solution here is get all the data from GetKeyByRlyAddress, change the rlyAddress into new prefix, change new key and set to old value.
