# Assessment

The `mint` module `is being affected` by the migration process.

## SetAllowedAddress
This function sets fund allowed address to state.
```go
// SetAllowedAddress sets fund allowed address to state.
func (k Keeper) SetAllowedAddress(ctx sdk.Context, address string) {
	store := ctx.KVStore(k.storeKey)
	key := types.GetAllowedAddressStoreKey(address)
	store.Set(key, []byte{1})
}
```
And the address is get from MsgAddAccountToFundModuleSet, which is a bech32 address
```go
// MsgAddAccountToFundModuleSet add account in to allowed fund module set
type MsgAddAccountToFundModuleSet struct {
	// authority is the address that controls the module (defaults to x/gov unless
	// overwritten).
	Authority      string `protobuf:"bytes,1,opt,name=authority,proto3" json:"authority,omitempty" yaml:"authority"`
	AllowedAddress string `protobuf:"bytes,2,opt,name=allowed_address,json=allowedAddress,proto3" json:"allowed_address,omitempty" yaml:"allowed_address"`
}
```
So the solution here is get all the data from AllowedAddressKey, change the address into new prefix, change new key and set to old value.
