# Composable wasm client migration

* Architecture: https://github.com/cosmos/ibc-go/tree/wasm-v8.0.0/docs/docs/03-light-clients/04-wasm
* Goal: determine differences in syntax and behaviour

1. compare tags
* Notional: https://github.com/notional-labs/ibc-go/tree/wasm-client-centauri
* Cosmos: https://github.com/cosmos/ibc-go/tree/wasm-v8.0.0

## Structure difference

#### Proxy light client (light client state will be persisted. As such, should try to avoid change structure)
1. ClientState: different in syntax, same in behavior
* Notional: use Notional: CodeId in place of Cosmos: CheckSum, deprecate Notional: XInner
```go
type ClientState struct {
	Data         []byte       `protobuf:"bytes,1,opt,name=data,proto3" json:"data,omitempty"`
    // CodeId is used to determine wasm code to be called
	CodeId       []byte       `protobuf:"bytes,2,opt,name=code_id,json=codeId,proto3" json:"code_id,omitempty"`
	LatestHeight types.Height `protobuf:"bytes,3,opt,name=latest_height,json=latestHeight,proto3" json:"latest_height" yaml:"latest_height"`
	// Types that are valid to be assigned to XInner:
	//	*ClientState_Inner
    // XInner is used to hold received bytes data for ClientState, which will be later converted to ClientState
    // this feature has been incorporated into Cosmos:08-wasm/types/wasm.pb.go
	XInner isClientState_XInner `protobuf_oneof:"_inner" json:",omitempty"`
}
```

* Cosmos:
```go
type ClientState struct {
	// bytes encoding the client state of the underlying light client
	// implemented as a Wasm contract.
	Data         []byte       `protobuf:"bytes,1,opt,name=data,proto3" json:"data,omitempty"`
    // Checksum is used to determint wasm code to be called
	Checksum     []byte       `protobuf:"bytes,2,opt,name=checksum,proto3" json:"checksum,omitempty"`
	LatestHeight types.Height `protobuf:"bytes,3,opt,name=latest_height,json=latestHeight,proto3" json:"latest_height"`
}
```

2. ConsensusState

3. ClientMessage

#### Messages (Message will not be persisted in store, as such can change name)
1. MsgStoreCode: different in syntax, same in behavior
* Notional: change Notional: MsgPushNewWasmCode to Cosmos: MsgStoreCode, change Notional: Code to Cosmos: WasmByteCode
```go
// Message type to push new wasm code
type MsgPushNewWasmCode struct {
	Signer string `protobuf:"bytes,1,opt,name=signer,proto3" json:"signer,omitempty"`
    // Code stores wasm byte code
	Code   []byte `protobuf:"bytes,2,opt,name=code,proto3" json:"code,omitempty"`
}
```

* Cosmos:
```go
// MsgStoreCode defines the request type for the StoreCode rpc.
type MsgStoreCode struct {
	// signer address
	Signer string `protobuf:"bytes,1,opt,name=signer,proto3" json:"signer,omitempty"`
	// wasm byte code of light client contract. It can be raw or gzip compressed
	WasmByteCode []byte `protobuf:"bytes,2,opt,name=wasm_byte_code,json=wasmByteCode,proto3" json:"wasm_byte_code,omitempty"`
}
```

2. MsgMigrateContract

3. MsgRemoveChecksum

