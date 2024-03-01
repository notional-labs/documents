# Assessment

The `ibc-hooks` module `is being affected` by the migration process.

## SetWhitelistedAddressPair
This function takes contract as bech32 addresses. 
```go
// StorePacketCallback stores which contract will be listening for the ack or timeout of a packet
func (k Keeper) StorePacketCallback(ctx sdk.Context, channel string, packetSequence uint64, contract string) {
	store := ctx.KVStore(k.storeKey)
	store.Set(GetPacketKey(channel, packetSequence), []byte(contract))
}
```

So the solution here is get all the data from GetPacketKey, change the contract address into new prefix.
