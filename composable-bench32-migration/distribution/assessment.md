# Assessment

The `distribution` module `is not being affected` by the migration process.

This module have no setter funtion that update (store.Set()) any field related to bech32 address.

These functions take sdk.ConsAddress, sdk.AccAddress as their inputs. So the migration does not affect this module.
```go
// set the delegator withdraw address
func (k Keeper) SetDelegatorWithdrawAddr(ctx sdk.Context, delAddr, withdrawAddr sdk.AccAddress) {
	store := ctx.KVStore(k.storeKey)
	store.Set(types.GetDelegatorWithdrawAddrKey(delAddr), withdrawAddr.Bytes())
}

// set the proposer public key for this block
func (k Keeper) SetPreviousProposerConsAddr(ctx sdk.Context, consAddr sdk.ConsAddress) {
	store := ctx.KVStore(k.storeKey)
	bz := k.cdc.MustMarshal(&gogotypes.BytesValue{Value: consAddr})
	store.Set(types.ProposerKey, bz)
}

// set the starting info associated with a delegator
func (k Keeper) SetDelegatorStartingInfo(ctx sdk.Context, val sdk.ValAddress, del sdk.AccAddress, period types.DelegatorStartingInfo) {
	store := ctx.KVStore(k.storeKey)
	b := k.cdc.MustMarshal(&period)
	store.Set(types.GetDelegatorStartingInfoKey(val, del), b)
}

// set historical rewards for a particular period
func (k Keeper) SetValidatorHistoricalRewards(ctx sdk.Context, val sdk.ValAddress, period uint64, rewards types.ValidatorHistoricalRewards) {
	store := ctx.KVStore(k.storeKey)
	b := k.cdc.MustMarshal(&rewards)
	store.Set(types.GetValidatorHistoricalRewardsKey(val, period), b)
}

// set current rewards for a validator
func (k Keeper) SetValidatorCurrentRewards(ctx sdk.Context, val sdk.ValAddress, rewards types.ValidatorCurrentRewards) {
	store := ctx.KVStore(k.storeKey)
	b := k.cdc.MustMarshal(&rewards)
	store.Set(types.GetValidatorCurrentRewardsKey(val), b)
}

// set accumulated commission for a validator
func (k Keeper) SetValidatorAccumulatedCommission(ctx sdk.Context, val sdk.ValAddress, commission types.ValidatorAccumulatedCommission) {
	var bz []byte

	store := ctx.KVStore(k.storeKey)
	if commission.Commission.IsZero() {
		bz = k.cdc.MustMarshal(&types.ValidatorAccumulatedCommission{})
	} else {
		bz = k.cdc.MustMarshal(&commission)
	}

	store.Set(types.GetValidatorAccumulatedCommissionKey(val), bz)
}

// set validator outstanding rewards
func (k Keeper) SetValidatorOutstandingRewards(ctx sdk.Context, val sdk.ValAddress, rewards types.ValidatorOutstandingRewards) {
	store := ctx.KVStore(k.storeKey)
	b := k.cdc.MustMarshal(&rewards)
	store.Set(types.GetValidatorOutstandingRewardsKey(val), b)
}

// set slash event for height
func (k Keeper) SetValidatorSlashEvent(ctx sdk.Context, val sdk.ValAddress, height, period uint64, event types.ValidatorSlashEvent) {
	store := ctx.KVStore(k.storeKey)
	b := k.cdc.MustMarshal(&event)
	store.Set(types.GetValidatorSlashEventKey(val, height, period), b)
}
```