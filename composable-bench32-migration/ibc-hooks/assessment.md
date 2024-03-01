# Assessment

The `ibc-hooks` module `is not being affected` by the migration process.

## SetWhitelistedAddressPair
This function is called when SendPacket is called, it stores a contract address which is a bech32 address. But the contract address will be deleted when OnAckknowledgement or OnTimeout is called. So this module is not affected by the migration process.
```go
// StorePacketCallback stores which contract will be listening for the ack or timeout of a packet
func (k Keeper) StorePacketCallback(ctx sdk.Context, channel string, packetSequence uint64, contract string) {
	store := ctx.KVStore(k.storeKey)
	store.Set(GetPacketKey(channel, packetSequence), []byte(contract))
}
```

