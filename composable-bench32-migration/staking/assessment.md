# Assessment

The `staking` module `is being affected` by the migration process.

## SetDelegation
This function sets a delegation.
```go
// SetDelegation sets a delegation.
func (k Keeper) SetDelegation(ctx sdk.Context, delegation types.Delegation) {
	delegatorAddress := sdk.MustAccAddressFromBech32(delegation.DelegatorAddress)

	store := ctx.KVStore(k.storeKey)
	b := types.MustMarshalDelegation(k.cdc, delegation)
	store.Set(types.GetDelegationKey(delegatorAddress, delegation.GetValidatorAddr()), b)
}
```
This function takes types.Delegation as its argument, which have DelegatorAddress and ValidatorAddress as bech32 addresses
```go
// Delegation represents the bond with tokens held by an account. It is
// owned by one delegator, and is associated with the voting power of one
// validator.
type Delegation struct {
	// delegator_address is the bech32-encoded address of the delegator.
	DelegatorAddress string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty"`
	// validator_address is the bech32-encoded address of the validator.
	ValidatorAddress string `protobuf:"bytes,2,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty"`
	// shares define the delegation shares received.
	Shares github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,3,opt,name=shares,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"shares"`
}
```
So the solution here is get all the data from types.GetDelegationKey(), change DelegatorAddress and ValidatorAddress into new prefix

## SetUnbondingDelegation
This function sets the unbonding delegation and associated index.
```go
// SetUnbondingDelegation sets the unbonding delegation and associated index.
func (k Keeper) SetUnbondingDelegation(ctx sdk.Context, ubd types.UnbondingDelegation) {
	delAddr := sdk.MustAccAddressFromBech32(ubd.DelegatorAddress)

	store := ctx.KVStore(k.storeKey)
	bz := types.MustMarshalUBD(k.cdc, ubd)
	valAddr, err := sdk.ValAddressFromBech32(ubd.ValidatorAddress)
	if err != nil {
		panic(err)
	}
	key := types.GetUBDKey(delAddr, valAddr)
	store.Set(key, bz)
	store.Set(types.GetUBDByValIndexKey(delAddr, valAddr), []byte{}) // index, store empty bytes
}
```
This function takes types.UnbondingDelegation as its argument, which have DelegatorAddress and ValidatorAddress as bech32 addresses
```go
// UnbondingDelegation stores all of a single delegator's unbonding bonds
// for a single validator in an time-ordered list.
type UnbondingDelegation struct {
	// delegator_address is the bech32-encoded address of the delegator.
	DelegatorAddress string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty"`
	// validator_address is the bech32-encoded address of the validator.
	ValidatorAddress string `protobuf:"bytes,2,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty"`
	// entries are the unbonding delegation entries.
	Entries []UnbondingDelegationEntry `protobuf:"bytes,3,rep,name=entries,proto3" json:"entries"`
}
```
So the solution here is get all the data from types.GetUBDByValIndexKey(), change DelegatorAddress and ValidatorAddress into new prefix

## SetRedelegation
This function set a redelegation and associated index.
```go
// SetRedelegation set a redelegation and associated index.
func (k Keeper) SetRedelegation(ctx sdk.Context, red types.Redelegation) {
	delegatorAddress := sdk.MustAccAddressFromBech32(red.DelegatorAddress)

	store := ctx.KVStore(k.storeKey)
	bz := types.MustMarshalRED(k.cdc, red)
	valSrcAddr, err := sdk.ValAddressFromBech32(red.ValidatorSrcAddress)
	if err != nil {
		panic(err)
	}
	valDestAddr, err := sdk.ValAddressFromBech32(red.ValidatorDstAddress)
	if err != nil {
		panic(err)
	}
	key := types.GetREDKey(delegatorAddress, valSrcAddr, valDestAddr)
	store.Set(key, bz)
	store.Set(types.GetREDByValSrcIndexKey(delegatorAddress, valSrcAddr, valDestAddr), []byte{})
	store.Set(types.GetREDByValDstIndexKey(delegatorAddress, valSrcAddr, valDestAddr), []byte{})
}
```
This function takes types.Redelegation as its argument, which have DelegatorAddress, ValidatorDstAddress and ValidatorSrcAddress as bech32 addresses
```go
// Redelegation contains the list of a particular delegator's redelegating bonds
// from a particular source validator to a particular destination validator.
type Redelegation struct {
	// delegator_address is the bech32-encoded address of the delegator.
	DelegatorAddress string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty"`
	// validator_src_address is the validator redelegation source operator address.
	ValidatorSrcAddress string `protobuf:"bytes,2,opt,name=validator_src_address,json=validatorSrcAddress,proto3" json:"validator_src_address,omitempty"`
	// validator_dst_address is the validator redelegation destination operator address.
	ValidatorDstAddress string `protobuf:"bytes,3,opt,name=validator_dst_address,json=validatorDstAddress,proto3" json:"validator_dst_address,omitempty"`
	// entries are the redelegation entries.
	Entries []RedelegationEntry `protobuf:"bytes,4,rep,name=entries,proto3" json:"entries"`
}
```
So the solution here is get all the data from types.GetREDKey(), change DelegatorAddress, ValidatorSrcAddress and ValidatorDstAddress into new prefix. We also need to change the key of GetREDKey, GetREDByValSrcIndexKey and GetREDByValDstIndexKey into new prefix components

## SetRedelegationQueueTimeSlice
This function sets a specific redelegation queue timeslice.
```go
// SetRedelegationQueueTimeSlice sets a specific redelegation queue timeslice.
func (k Keeper) SetRedelegationQueueTimeSlice(ctx sdk.Context, timestamp time.Time, keys []types.DVVTriplet) {
	store := ctx.KVStore(k.storeKey)
	bz := k.cdc.MustMarshal(&types.DVVTriplets{Triplets: keys})
	store.Set(types.GetRedelegationTimeKey(timestamp), bz)
}
```
This function takes types.DVVTriplet as its argument, which have DelegatorAddress, ValidatorDstAddress and ValidatorSrcAddress as bech32 addresses
```go
// DVVTriplet is struct that just has a delegator-validator-validator triplet
// with no other data. It is intended to be used as a marshalable pointer. For
// example, a DVVTriplet can be used to construct the key to getting a
// Redelegation from state.
type DVVTriplet struct {
	DelegatorAddress    string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty"`
	ValidatorSrcAddress string `protobuf:"bytes,2,opt,name=validator_src_address,json=validatorSrcAddress,proto3" json:"validator_src_address,omitempty"`
	ValidatorDstAddress string `protobuf:"bytes,3,opt,name=validator_dst_address,json=validatorDstAddress,proto3" json:"validator_dst_address,omitempty"`
}
```
So the solution here is get all the data from types.GetRedelegationTimeKey(), change DelegatorAddress, ValidatorSrcAddress and ValidatorDstAddress into new prefix.

## SetHistoricalInfo
This function sets the historical info at a given height
```go
// SetHistoricalInfo sets the historical info at a given height
func (k Keeper) SetHistoricalInfo(ctx sdk.Context, height int64, hi *types.HistoricalInfo) {
	store := ctx.KVStore(k.storeKey)
	key := types.GetHistoricalInfoKey(height)
	value := k.cdc.MustMarshal(hi)
	store.Set(key, value)
}
```
This function takes types.HistoricalInfo as its argument. It has Validator which has OperatorAddress as bech32 addresses
```go
// HistoricalInfo contains header and validator information for a given block.
// It is stored as part of staking module's state, which persists the `n` most
// recent HistoricalInfo
// (`n` is set by the staking module's `historical_entries` parameter).
type HistoricalInfo struct {
	Header types.Header `protobuf:"bytes,1,opt,name=header,proto3" json:"header"`
	Valset []Validator  `protobuf:"bytes,2,rep,name=valset,proto3" json:"valset"`
}

// Validator defines a validator, together with the total amount of the
// Validator's bond shares and their exchange rate to coins. Slashing results in
// a decrease in the exchange rate, allowing correct calculation of future
// undelegations without iterating over delegators. When coins are delegated to
// this validator, the validator is credited with a delegation whose number of
// bond shares is based on the amount of coins delegated divided by the current
// exchange rate. Voting power can be calculated as total bonded shares
// multiplied by exchange rate.
type Validator struct {
	// operator_address defines the address of the validator's operator; bech encoded in JSON.
	OperatorAddress string `protobuf:"bytes,1,opt,name=operator_address,json=operatorAddress,proto3" json:"operator_address,omitempty"`
	// consensus_pubkey is the consensus public key of the validator, as a Protobuf Any.
	ConsensusPubkey *types1.Any `protobuf:"bytes,2,opt,name=consensus_pubkey,json=consensusPubkey,proto3" json:"consensus_pubkey,omitempty"`
	// jailed defined whether the validator has been jailed from bonded status or not.
	Jailed bool `protobuf:"varint,3,opt,name=jailed,proto3" json:"jailed,omitempty"`
	// status is the validator status (bonded/unbonding/unbonded).
	Status BondStatus `protobuf:"varint,4,opt,name=status,proto3,enum=cosmos.staking.v1beta1.BondStatus" json:"status,omitempty"`
	// tokens define the delegated tokens (incl. self-delegation).
	Tokens github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,5,opt,name=tokens,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"tokens"`
	// delegator_shares defines total shares issued to a validator's delegators.
	DelegatorShares github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,6,opt,name=delegator_shares,json=delegatorShares,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"delegator_shares"`
	// description defines the description terms for the validator.
	Description Description `protobuf:"bytes,7,opt,name=description,proto3" json:"description"`
	// unbonding_height defines, if unbonding, the height at which this validator has begun unbonding.
	UnbondingHeight int64 `protobuf:"varint,8,opt,name=unbonding_height,json=unbondingHeight,proto3" json:"unbonding_height,omitempty"`
	// unbonding_time defines, if unbonding, the min time for the validator to complete unbonding.
	UnbondingTime time.Time `protobuf:"bytes,9,opt,name=unbonding_time,json=unbondingTime,proto3,stdtime" json:"unbonding_time"`
	// commission defines the commission parameters.
	Commission Commission `protobuf:"bytes,10,opt,name=commission,proto3" json:"commission"`
	// min_self_delegation is the validator's self declared minimum self delegation.
	//
	// Since: cosmos-sdk 0.46
	MinSelfDelegation github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,11,opt,name=min_self_delegation,json=minSelfDelegation,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"min_self_delegation"`
	// strictly positive if this validator's unbonding has been stopped by external modules
	UnbondingOnHoldRefCount int64 `protobuf:"varint,12,opt,name=unbonding_on_hold_ref_count,json=unbondingOnHoldRefCount,proto3" json:"unbonding_on_hold_ref_count,omitempty"`
	// list of unbonding ids, each uniquely identifing an unbonding of this validator
	UnbondingIds []uint64 `protobuf:"varint,13,rep,packed,name=unbonding_ids,json=unbondingIds,proto3" json:"unbonding_ids,omitempty"`
}
```
So the solution here is get all the data from types.GetHistoricalInfoKey(), change OperatorAddress into new prefix.

## SetUnbondingDelegationByUnbondingID
This function sets an index to look up an UnbondingDelegation by the unbondingID of an UnbondingDelegationEntry that it contains.
```go
// SetUnbondingDelegationByUnbondingID sets an index to look up an UnbondingDelegation by the unbondingID of an UnbondingDelegationEntry that it contains
// Note, it does not set the unbonding delegation itself, use SetUnbondingDelegation(ctx, ubd) for that
func (k Keeper) SetUnbondingDelegationByUnbondingID(ctx sdk.Context, ubd types.UnbondingDelegation, id uint64) {
	store := ctx.KVStore(k.storeKey)
	delAddr := sdk.MustAccAddressFromBech32(ubd.DelegatorAddress)
	valAddr, err := sdk.ValAddressFromBech32(ubd.ValidatorAddress)
	if err != nil {
		panic(err)
	}

	ubdKey := types.GetUBDKey(delAddr, valAddr)
	store.Set(types.GetUnbondingIndexKey(id), ubdKey)

	// Set unbonding type so that we know how to deserialize it later
	k.SetUnbondingType(ctx, id, types.UnbondingType_UnbondingDelegation)
}
```
This function takes types.UnbondingDelegation as its argument, which have DelegatorAddress, ValidatorAddress as bech32 addresses
```go
// UnbondingDelegation stores all of a single delegator's unbonding bonds
// for a single validator in an time-ordered list.
type UnbondingDelegation struct {
	// delegator_address is the bech32-encoded address of the delegator.
	DelegatorAddress string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty"`
	// validator_address is the bech32-encoded address of the validator.
	ValidatorAddress string `protobuf:"bytes,2,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty"`
	// entries are the unbonding delegation entries.
	Entries []UnbondingDelegationEntry `protobuf:"bytes,3,rep,name=entries,proto3" json:"entries"`
}
```
So the solution here is get all the data from types.GetUBDKey(), change DelegatorAddress, ValidatorAddress into new prefix.

## SetRedelegationByUnbondingID
This function sets an index to look up an Redelegation by the unbondingID of an RedelegationEntry that it contains
```go
// SetRedelegationByUnbondingID sets an index to look up an Redelegation by the unbondingID of an RedelegationEntry that it contains
// Note, it does not set the redelegation itself, use SetRedelegation(ctx, red) for that
func (k Keeper) SetRedelegationByUnbondingID(ctx sdk.Context, red types.Redelegation, id uint64) {
	store := ctx.KVStore(k.storeKey)

	delAddr := sdk.MustAccAddressFromBech32(red.DelegatorAddress)

	valSrcAddr, err := sdk.ValAddressFromBech32(red.ValidatorSrcAddress)
	if err != nil {
		panic(err)
	}

	valDstAddr, err := sdk.ValAddressFromBech32(red.ValidatorDstAddress)
	if err != nil {
		panic(err)
	}

	redKey := types.GetREDKey(delAddr, valSrcAddr, valDstAddr)
	store.Set(types.GetUnbondingIndexKey(id), redKey)

	// Set unbonding type so that we know how to deserialize it later
	k.SetUnbondingType(ctx, id, types.UnbondingType_Redelegation)
}
```
This function takes types.Redelegation as its argument, which have DelegatorAddress, ValidatorSrcAddress and ValidatorDstAddress as bech32 addresses
```go
// Redelegation contains the list of a particular delegator's redelegating bonds
// from a particular source validator to a particular destination validator.
type Redelegation struct {
	// delegator_address is the bech32-encoded address of the delegator.
	DelegatorAddress string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty"`
	// validator_src_address is the validator redelegation source operator address.
	ValidatorSrcAddress string `protobuf:"bytes,2,opt,name=validator_src_address,json=validatorSrcAddress,proto3" json:"validator_src_address,omitempty"`
	// validator_dst_address is the validator redelegation destination operator address.
	ValidatorDstAddress string `protobuf:"bytes,3,opt,name=validator_dst_address,json=validatorDstAddress,proto3" json:"validator_dst_address,omitempty"`
	// entries are the redelegation entries.
	Entries []RedelegationEntry `protobuf:"bytes,4,rep,name=entries,proto3" json:"entries"`
}
```
So the solution here is get all the data from types.GetUnbondingIndexKey(), change DelegatorAddress, ValidatorSrcAddress, ValidatorDstAddress into new prefix.

## SetValidatorByUnbondingID
This function sets an index to look up a Validator by the unbondingID corresponding to its current unbonding
```go
// SetValidatorByUnbondingID sets an index to look up a Validator by the unbondingID corresponding to its current unbonding
// Note, it does not set the validator itself, use SetValidator(ctx, val) for that
func (k Keeper) SetValidatorByUnbondingID(ctx sdk.Context, val types.Validator, id uint64) {
	store := ctx.KVStore(k.storeKey)

	valAddr, err := sdk.ValAddressFromBech32(val.OperatorAddress)
	if err != nil {
		panic(err)
	}

	valKey := types.GetValidatorKey(valAddr)
	store.Set(types.GetUnbondingIndexKey(id), valKey)

	// Set unbonding type so that we know how to deserialize it later
	k.SetUnbondingType(ctx, id, types.UnbondingType_ValidatorUnbonding)
}
```
This function takes types.Validator as its argument, which have OperatorAddress as bech32 address
```go
// Validator defines a validator, together with the total amount of the
// Validator's bond shares and their exchange rate to coins. Slashing results in
// a decrease in the exchange rate, allowing correct calculation of future
// undelegations without iterating over delegators. When coins are delegated to
// this validator, the validator is credited with a delegation whose number of
// bond shares is based on the amount of coins delegated divided by the current
// exchange rate. Voting power can be calculated as total bonded shares
// multiplied by exchange rate.
type Validator struct {
	// operator_address defines the address of the validator's operator; bech encoded in JSON.
	OperatorAddress string `protobuf:"bytes,1,opt,name=operator_address,json=operatorAddress,proto3" json:"operator_address,omitempty"`
	// consensus_pubkey is the consensus public key of the validator, as a Protobuf Any.
	ConsensusPubkey *types1.Any `protobuf:"bytes,2,opt,name=consensus_pubkey,json=consensusPubkey,proto3" json:"consensus_pubkey,omitempty"`
	// jailed defined whether the validator has been jailed from bonded status or not.
	Jailed bool `protobuf:"varint,3,opt,name=jailed,proto3" json:"jailed,omitempty"`
	// status is the validator status (bonded/unbonding/unbonded).
	Status BondStatus `protobuf:"varint,4,opt,name=status,proto3,enum=cosmos.staking.v1beta1.BondStatus" json:"status,omitempty"`
	// tokens define the delegated tokens (incl. self-delegation).
	Tokens github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,5,opt,name=tokens,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"tokens"`
	// delegator_shares defines total shares issued to a validator's delegators.
	DelegatorShares github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,6,opt,name=delegator_shares,json=delegatorShares,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"delegator_shares"`
	// description defines the description terms for the validator.
	Description Description `protobuf:"bytes,7,opt,name=description,proto3" json:"description"`
	// unbonding_height defines, if unbonding, the height at which this validator has begun unbonding.
	UnbondingHeight int64 `protobuf:"varint,8,opt,name=unbonding_height,json=unbondingHeight,proto3" json:"unbonding_height,omitempty"`
	// unbonding_time defines, if unbonding, the min time for the validator to complete unbonding.
	UnbondingTime time.Time `protobuf:"bytes,9,opt,name=unbonding_time,json=unbondingTime,proto3,stdtime" json:"unbonding_time"`
	// commission defines the commission parameters.
	Commission Commission `protobuf:"bytes,10,opt,name=commission,proto3" json:"commission"`
	// min_self_delegation is the validator's self declared minimum self delegation.
	//
	// Since: cosmos-sdk 0.46
	MinSelfDelegation github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,11,opt,name=min_self_delegation,json=minSelfDelegation,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"min_self_delegation"`
	// strictly positive if this validator's unbonding has been stopped by external modules
	UnbondingOnHoldRefCount int64 `protobuf:"varint,12,opt,name=unbonding_on_hold_ref_count,json=unbondingOnHoldRefCount,proto3" json:"unbonding_on_hold_ref_count,omitempty"`
	// list of unbonding ids, each uniquely identifing an unbonding of this validator
	UnbondingIds []uint64 `protobuf:"varint,13,rep,packed,name=unbonding_ids,json=unbondingIds,proto3" json:"unbonding_ids,omitempty"`
}
```
So the solution here is get all the data from types.GetUnbondingIndexKey(), change OperatorAddress into new prefix.

## SetValidator
This function sets an index to look up an Redelegation by the unbondingID of an RedelegationEntry that it contains
```go
// set the main record holding validator details
func (k Keeper) SetValidator(ctx sdk.Context, validator types.Validator) {
	store := ctx.KVStore(k.storeKey)
	bz := types.MustMarshalValidator(k.cdc, &validator)
	store.Set(types.GetValidatorKey(validator.GetOperator()), bz)
}
```
This function takes types.Validator (shown above) as its argument, which have OperatorAddress as bech32 addresses

So the solution here is get all the data from types.GetValidatorKey(), change OperatorAddress into new prefix.

## SetValidatorByConsAddr
```go
// validator index
func (k Keeper) SetValidatorByConsAddr(ctx sdk.Context, validator types.Validator) error {
	consPk, err := validator.GetConsAddr()
	if err != nil {
		return err
	}
	store := ctx.KVStore(k.storeKey)
	store.Set(types.GetValidatorByConsAddrKey(consPk), validator.GetOperator())
	return nil
}
```
This function takes types.Validator (shown above) as its argument, which have OperatorAddress as bech32 addresses

So the solution here is get all the data from types.GetValidatorByConsAddrKey(), change OperatorAddress into new prefix.

## SetValidatorByPowerIndex
```go
// validator index
func (k Keeper) SetValidatorByPowerIndex(ctx sdk.Context, validator types.Validator) {
	// jailed validators are not kept in the power index
	if validator.Jailed {
		return
	}

	store := ctx.KVStore(k.storeKey)
	store.Set(types.GetValidatorsByPowerIndexKey(validator, k.PowerReduction(ctx)), validator.GetOperator())
}
```
This function takes types.Validator (shown above) as its argument, which have OperatorAddress as bech32 addresses

So the solution here is get all the data from types.GetValidatorsByPowerIndexKey(), change OperatorAddress into new prefix.

## SetNewValidatorByPowerIndex
```go
// validator index
func (k Keeper) SetNewValidatorByPowerIndex(ctx sdk.Context, validator types.Validator) {
	store := ctx.KVStore(k.storeKey)
	store.Set(types.GetValidatorsByPowerIndexKey(validator, k.PowerReduction(ctx)), validator.GetOperator())
}
```
This function takes types.Validator (shown above) as its argument, which have OperatorAddress as bech32 addresses

So the solution here is get all the data from types.GetValidatorsByPowerIndexKey(), change OperatorAddress into new prefix.

## SetUnbondingValidatorsQueue
```go
// SetUnbondingValidatorsQueue sets a given slice of validator addresses into
// the unbonding validator queue by a given height and time.
func (k Keeper) SetUnbondingValidatorsQueue(ctx sdk.Context, endTime time.Time, endHeight int64, addrs []string) {
	store := ctx.KVStore(k.storeKey)
	bz := k.cdc.MustMarshal(&types.ValAddresses{Addresses: addrs})
	store.Set(types.GetValidatorQueueKey(endTime, endHeight), bz)
}
```
This function takes addrs as bech32 addresses

So the solution here is get all the data from types.GetValidatorQueueKey(), change addrs into new prefix.