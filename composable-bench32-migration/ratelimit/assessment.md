# Assessment

The `ratelimit` module `is not being affected` by the migration process.

## SetWhitelistedAddressPair
This function takes WhitelistedAddressPair as a parameter. It has Sender and Receiver as bech32 addresses. 
```go
// Adds an pair of sender and receiver addresses to the whitelist to allow all
// IBC transfers between those addresses to skip all flow calculations
func (k Keeper) SetWhitelistedAddressPair(ctx sdk.Context, whitelist types.WhitelistedAddressPair) {
	store := prefix.NewStore(ctx.KVStore(k.storeKey), types.AddressWhitelistKeyPrefix)
	key := types.GetAddressWhitelistKey(whitelist.Sender, whitelist.Receiver)
	value := k.cdc.MustMarshal(&whitelist)
	store.Set(key, value)
}

type WhitelistedAddressPair struct {
	Sender   string `protobuf:"bytes,1,opt,name=sender,proto3" json:"sender,omitempty"`
	Receiver string `protobuf:"bytes,2,opt,name=receiver,proto3" json:"receiver,omitempty"`
}
```

This function is triggered in InitGenesis function. But in genesis file, WhitelistedAddressPair is empty so the migration does not affect this module.
