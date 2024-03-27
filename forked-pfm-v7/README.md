# Assessment

This document specifies the different between forked pfm v7 and pfm v8

## Keeper.go
### NewKeeper
- Add paramSpace paramtypes.Subspace
- Add transferMiddlewareKeeper types.TransferMiddlewareKeeper,
- Remove authority
- Check if paramSpace has key table

### WriteAcknowledgementForForwardedPacket
- Check if GetParachainTokenInfoByNativeDenom
    - True: Burn nativeToken and ibcToken
    - False: transfer funds from escrow account for forwarded packet to escrow account going back for refund
- Check If SenderChainIsSource is false
    - Check if GetParachainTokenInfoByAssetID
        - True: 
            - send native token to native escrow address
            - send ibc token to ibc escrow address
        - False: Mint funds to refundEscrowAddress

### Add GetParachainTokenInfoByAssetID func
### Add GetParachainTokenInfoByNativeDenom func

## Params.go
### GetFeePercentage
- Get from ParamSpace
### GetParams
- Return GetFeePercentage
### SetParams
- Set into ParamSpace

## ibc_middleware.go
### OnRecvPacket
- Check if the packet was sent from Picasso
    - disableDenomComposition = true
    - denomOnThisChain = paraChainIBCTokenInfo.NativeDenom
