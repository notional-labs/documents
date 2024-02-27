# Composable wasm client migration

* Architecture: https://github.com/cosmos/ibc-go/tree/wasm-v8.0.0/docs/docs/03-light-clients/04-wasm
* Goal: determine differences in syntax and behaviour

1. compare tags
* A: https://github.com/notional-labs/ibc-go/tree/wasm-client-centauri
* B: https://github.com/cosmos/ibc-go/tree/wasm-v8.0.0

## Structure difference

#### Proxy light client
1. ClientState: different in syntax, same in behavior
* A: use A: CodeId in place of B: CheckSum, remove A: XInner
```go
type ClientState struct {
	Data         []byte       `protobuf:"bytes,1,opt,name=data,proto3" json:"data,omitempty"`
    // CodeId is used to determine wasm code to be called
	CodeId       []byte       `protobuf:"bytes,2,opt,name=code_id,json=codeId,proto3" json:"code_id,omitempty"`
	LatestHeight types.Height `protobuf:"bytes,3,opt,name=latest_height,json=latestHeight,proto3" json:"latest_height" yaml:"latest_height"`
	// Types that are valid to be assigned to XInner:
	//	*ClientState_Inner
    // XInner is used to hold received bytes data for ClientState, which will be later converted to ClientState
    // this feature has been incorporated into B:08-wasm/types/wasm.pb.go
	XInner isClientState_XInner `protobuf_oneof:"_inner" json:",omitempty"`
}
```

* B:
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

1. ConsensusState



1. ClientMessage

#### Messages
1. MsgStoreCode
* a: 


2. MsgMigrateContract

3. MsgRemoveCodeHash

