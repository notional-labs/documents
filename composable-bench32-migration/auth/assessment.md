# Assessment

The `auth` module `is being affected` by the migration process.

## SetAccount

This function is used to create various types of accounts.
```go
// x/auth/keeper/account.go
// SetAccount implements AccountKeeperI.
func (ak AccountKeeper) SetAccount(ctx sdk.Context, acc types.AccountI) {
	addr := acc.GetAddress()
	store := ctx.KVStore(ak.storeKey)

	bz, err := ak.MarshalAccount(acc)
	if err != nil {
		panic(err)
	}

	store.Set(types.AddressStoreKey(addr), bz)
	store.Set(types.AccountNumberStoreKey(acc.GetAccountNumber()), addr.Bytes())
}
```
These types are BaseAccount, ModuleAccount, BaseVestingAccount, ContinuousVestingAccount, DelayedVestingAccount, PeriodicVestingAccount, PermanentLockedAccount.
BaseAccount defines a base account type. It contains all the necessary fields for basic account functionality. And other accounts extend this type for additional functionality.
In BaseAccount, it has Address which is a bech32 address.
```go
type BaseAccount struct {
	Address       string     `protobuf:"bytes,1,opt,name=address,proto3" json:"address,omitempty"`
	PubKey        *types.Any `protobuf:"bytes,2,opt,name=pub_key,json=pubKey,proto3" json:"public_key,omitempty"`
	AccountNumber uint64     `protobuf:"varint,3,opt,name=account_number,json=accountNumber,proto3" json:"account_number,omitempty"`
	Sequence      uint64     `protobuf:"varint,4,opt,name=sequence,proto3" json:"sequence,omitempty"`
}
```
So the solution here is get all the data from AddressStoreKeyPrefix and change the Address field into new prefix