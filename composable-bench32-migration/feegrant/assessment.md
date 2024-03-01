# Assessment

The `feegrant` module `is not being affected` by the migration process.

This module have no setter funtion that update (store.Set()) any field related to bech32 address.

These functions take sdk.AccAddress as their inputs. So the migration does not affect this module.
```go
// GrantAllowance creates a new grant
func (k Keeper) GrantAllowance(ctx sdk.Context, granter, grantee sdk.AccAddress, feeAllowance feegrant.FeeAllowanceI) error {
	// create the account if it is not in account state
	granteeAcc := k.authKeeper.GetAccount(ctx, grantee)
	if granteeAcc == nil {
		granteeAcc = k.authKeeper.NewAccountWithAddress(ctx, grantee)
		k.authKeeper.SetAccount(ctx, granteeAcc)
	}

	store := ctx.KVStore(k.storeKey)
	key := feegrant.FeeAllowanceKey(granter, grantee)

	var oldExp *time.Time
	existingGrant, err := k.getGrant(ctx, granter, grantee)

	// If we didn't find any grant, we don't return any error.
	// All other kinds of errors are returned.
	if err != nil && !sdkerrors.IsOf(err, sdkerrors.ErrNotFound) {
		return err
	}

	if existingGrant != nil && existingGrant.GetAllowance() != nil {
		grantInfo, err := existingGrant.GetGrant()
		if err != nil {
			return err
		}

		oldExp, err = grantInfo.ExpiresAt()
		if err != nil {
			return err
		}
	}

	newExp, err := feeAllowance.ExpiresAt()
	switch {
	case err != nil:
		return err

	case newExp != nil && newExp.Before(ctx.BlockTime()):
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "expiration is before current block time")

	case oldExp == nil && newExp != nil:
		// when old oldExp is nil there won't be any key added before to queue.
		// add the new key to queue directly.
		k.addToFeeAllowanceQueue(ctx, key[1:], newExp)

	case oldExp != nil && newExp == nil:
		// when newExp is nil no need of adding the key to the pruning queue
		// remove the old key from queue.
		k.removeFromGrantQueue(ctx, oldExp, key[1:])

	case oldExp != nil && newExp != nil && !oldExp.Equal(*newExp):
		// `key` formed here with the prefix of `FeeAllowanceKeyPrefix` (which is `0x00`)
		// remove the 1st byte and reuse the remaining key as it is.

		// remove the old key from queue.
		k.removeFromGrantQueue(ctx, oldExp, key[1:])

		// add the new key to queue.
		k.addToFeeAllowanceQueue(ctx, key[1:], newExp)
	}

	grant, err := feegrant.NewGrant(granter, grantee, feeAllowance)
	if err != nil {
		return err
	}

	bz, err := k.cdc.Marshal(&grant)
	if err != nil {
		return err
	}

	store.Set(key, bz)
	ctx.EventManager().EmitEvent(
		sdk.NewEvent(
			feegrant.EventTypeSetFeeGrant,
			sdk.NewAttribute(feegrant.AttributeKeyGranter, grant.Granter),
			sdk.NewAttribute(feegrant.AttributeKeyGrantee, grant.Grantee),
		),
	)

	return nil
}

// UpdateAllowance updates the existing grant.
func (k Keeper) UpdateAllowance(ctx sdk.Context, granter, grantee sdk.AccAddress, feeAllowance feegrant.FeeAllowanceI) error {
	store := ctx.KVStore(k.storeKey)
	key := feegrant.FeeAllowanceKey(granter, grantee)

	_, err := k.getGrant(ctx, granter, grantee)
	if err != nil {
		return err
	}

	grant, err := feegrant.NewGrant(granter, grantee, feeAllowance)
	if err != nil {
		return err
	}

	bz, err := k.cdc.Marshal(&grant)
	if err != nil {
		return err
	}

	store.Set(key, bz)

	ctx.EventManager().EmitEvent(
		sdk.NewEvent(
			feegrant.EventTypeUpdateFeeGrant,
			sdk.NewAttribute(feegrant.AttributeKeyGranter, grant.Granter),
			sdk.NewAttribute(feegrant.AttributeKeyGrantee, grant.Grantee),
		),
	)

	return nil
}
```