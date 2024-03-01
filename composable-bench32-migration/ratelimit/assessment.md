# Assessment

The `ratelimit` module `is being affected` by the migration process.

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

So the solution here is get all the data from GetAddressWhitelistKey, change the Sender and Receiver into new prefix. And we also need to change its components into new prefix.
