# Composable wasm client migration

- Architecture: https://github.com/cosmos/ibc-go/tree/wasm-v8.0.0/docs/docs/03-light-clients/04-wasm
- Goal: determine differences in syntax and behaviour

1. Compare tags

- Notional: https://github.com/notional-labs/ibc-go/tree/wasm-client-centauri
- Cosmos: https://github.com/cosmos/ibc-go/tree/wasm-v8.0.0

2. Index

- [Composable wasm client migration](#composable-wasm-client-migration)
	- [Structure difference](#structure-difference)
		- [Proxy light client state](#proxy-light-client-state)
			- [1. ClientState: different in syntax, same in behavior](#1-clientstate-different-in-syntax-same-in-behavior)
			- [2. ConsensusState](#2-consensusstate)
		- [Proxy light client message](#proxy-light-client-message)
			- [1. ClientMessage](#1-clientmessage)
		- [Messages](#messages)
			- [1. MsgStoreCode: different in syntax, same in behavior](#1-msgstorecode-different-in-syntax-same-in-behavior)
			- [2. MsgMigrateContract](#2-msgmigratecontract)
			- [3. MsgRemoveChecksum](#3-msgremovechecksum)
		- [Wasm entrypoint messages](#wasm-entrypoint-messages)
			- [1. InstantiateMessage: migration of contract is needed](#1-instantiatemessage-migration-of-contract-is-needed)
			- [2. QueryMessage: migration of contract is needed](#2-querymessage-migration-of-contract-is-needed)
			- [3. SudoMessage: migration of contract is needed](#3-sudomessage-migration-of-contract-is-needed)

## Structure difference

### Proxy light client state

light client state will be persisted. As such, should try to avoid change structure

#### 1. ClientState: different in syntax, same in behavior

- Notional: use Notional: CodeId in place of Cosmos: CheckSum, deprecate Notional: XInner

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

- Cosmos:

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

#### 2. ConsensusState

### Proxy light client message

Message will not be persisted in store, as such can change name

#### 1. ClientMessage
* Notional: change the handling of Notional: Header, Notional: Misbehaviour to Cosmos: ClientMessage
```go
type Header struct {
	Data   []byte       `protobuf:"bytes,1,opt,name=data,proto3" json:"data,omitempty"`
	Height types.Height `protobuf:"bytes,2,opt,name=height,proto3" json:"height" yaml:"height"`
	// Types that are valid to be assigned to XInner:
	//	*Header_Inner
	XInner isHeader_XInner `protobuf_oneof:"_inner" json:",omitempty"`
}
```

```go
// Wasm light client Misbehaviour
type Misbehaviour struct {
	Data []byte `protobuf:"bytes,1,opt,name=data,proto3" json:"data,omitempty"`
}
```

* Cosmos:
```go
// Wasm light client message (either header(s) or misbehaviour)
type ClientMessage struct {
	Data []byte `protobuf:"bytes,1,opt,name=data,proto3" json:"data,omitempty"`
}
```

### Messages

Message will not be persisted in store, as such can change name

#### 1. MsgStoreCode: different in syntax, same in behavior

- Notional: change Notional: MsgPushNewWasmCode to Cosmos: MsgStoreCode, change Notional: Code to Cosmos: WasmByteCode

```go
// Message type to push new wasm code
type MsgPushNewWasmCode struct {
	Signer string `protobuf:"bytes,1,opt,name=signer,proto3" json:"signer,omitempty"`
    // Code stores wasm byte code
	Code   []byte `protobuf:"bytes,2,opt,name=code,proto3" json:"code,omitempty"`
}
```

- Cosmos:

```go
// MsgStoreCode defines the request type for the StoreCode rpc.
type MsgStoreCode struct {
	// signer address
	Signer string `protobuf:"bytes,1,opt,name=signer,proto3" json:"signer,omitempty"`
	// wasm byte code of light client contract. It can be raw or gzip compressed
	WasmByteCode []byte `protobuf:"bytes,2,opt,name=wasm_byte_code,json=wasmByteCode,proto3" json:"wasm_byte_code,omitempty"`
}
```

#### 2. MsgMigrateContract

#### 3. MsgRemoveChecksum

### Wasm entrypoint messages

Different messages will require deploying new contract. As such, should try to avoid change structure

#### 1. InstantiateMessage: migration of contract is needed

- Notional: []byte("{}")

- Cosmos:

```go
// InstantiateMessage is the message that is sent to the contract's instantiate entry point.
type InstantiateMessage struct {
	ClientState    []byte `json:"client_state"`
	ConsensusState []byte `json:"consensus_state"`
	Checksum       []byte `json:"checksum"`
}
```

#### 2. QueryMessage: migration of contract is needed

- Notional: lacks TimestampAtHeight, VerifyClientMessage, CheckForMisbehaviour

```go
var (
statusPayloadInner struct{}
statusPayload      struct {
    Status statusPayloadInner `json:"status"`
}
)
```

```go
type ExportMetadataPayload struct {
	ExportMetadata ExportMetadataInnerPayload `json:"export_metadata"`
}

type ExportMetadataInnerPayload struct{}
```

- Cosmos:

```go
// QueryMsg is used to encode messages that are sent to the contract's query entry point.
// The json omitempty tag is mandatory since it omits any empty (default initialized) fields from the encoded JSON,
// this is required in order to be compatible with Rust's enum matching as used in the contract.
// Only one field should be set at a time.
type QueryMsg struct {
	Status               *StatusMsg               `json:"status,omitempty"`
	ExportMetadata       *ExportMetadataMsg       `json:"export_metadata,omitempty"`
	TimestampAtHeight    *TimestampAtHeightMsg    `json:"timestamp_at_height,omitempty"`
	VerifyClientMessage  *VerifyClientMessageMsg  `json:"verify_client_message,omitempty"`
	CheckForMisbehaviour *CheckForMisbehaviourMsg `json:"check_for_misbehaviour,omitempty"`
}

// StatusMsg is a queryMsg sent to the contract to query the status of the wasm client.
type StatusMsg struct{}

// ExportMetadataMsg is a queryMsg sent to the contract to query the exported metadata of the wasm client.
type ExportMetadataMsg struct{}

// TimestampAtHeightMsg is a queryMsg sent to the contract to query the timestamp at a given height.
type TimestampAtHeightMsg struct {
	Height clienttypes.Height `json:"height"`
}

// VerifyClientMessageMsg is a queryMsg sent to the contract to verify a client message.
type VerifyClientMessageMsg struct {
	ClientMessage []byte `json:"client_message"`
}

// CheckForMisbehaviourMsg is a queryMsg sent to the contract to check for misbehaviour.
type CheckForMisbehaviourMsg struct {
	ClientMessage []byte `json:"client_message"`
}
```

#### 3. SudoMessage: migration of contract is needed

- Notional: use execute instead of sudo

```go
type (
	verifyMembershipPayloadInner struct {
		Height           exported.Height `json:"height"`
		DelayTimePeriod  uint64          `json:"delay_time_period"`
		DelayBlockPeriod uint64          `json:"delay_block_period"`
		Proof            []byte          `json:"proof"`
		Path             exported.Path   `json:"path"`
		Value            []byte          `json:"value"`
	}
	verifyMembershipPayload struct {
		VerifyMembershipPayloadInner verifyMembershipPayloadInner `json:"verify_membership"`
	}
)
```

```go
type (
	verifyNonMembershipPayloadInner struct {
		Height           exported.Height `json:"height"`
		DelayTimePeriod  uint64          `json:"delay_time_period"`
		DelayBlockPeriod uint64          `json:"delay_block_period"`
		Proof            []byte          `json:"proof"`
		Path             exported.Path   `json:"path"`
	}
	verifyNonMembershipPayload struct {
		VerifyNonMembershipPayloadInner verifyNonMembershipPayloadInner `json:"verify_non_membership"`
	}
)
```

```go
type checkForMisbehaviourPayload struct {
	CheckForMisbehaviour checkForMisbehaviourInnerPayload `json:"check_for_misbehaviour"`
}
type checkForMisbehaviourInnerPayload struct {
	ClientMessage clientMessageConcretePayloadClientMessage `json:"client_message"`
}
```

```go
type checkSubstituteAndUpdateStatePayload struct {
	CheckSubstituteAndUpdateState CheckSubstituteAndUpdateStatePayload `json:"check_substitute_and_update_state"`
}

type CheckSubstituteAndUpdateStatePayload struct{}
```

```go
type verifyClientMessagePayload struct {
	VerifyClientMessage verifyClientMessageInnerPayload `json:"verify_client_message"`
}

type clientMessageConcretePayloadClientMessage struct {
	Header       *Header       `json:"header,omitempty"`
	Misbehaviour *Misbehaviour `json:"misbehaviour,omitempty"`
}
type verifyClientMessageInnerPayload struct {
	ClientMessage clientMessageConcretePayloadClientMessage `json:"client_message"`
}
```

```go
type updateStatePayload struct {
	UpdateState updateStateInnerPayload `json:"update_state"`
}
type updateStateInnerPayload struct {
	ClientMessage clientMessageConcretePayload `json:"client_message"`
}

type clientMessageConcretePayload struct {
	Header       *Header       `json:"header,omitempty"`
	Misbehaviour *Misbehaviour `json:"misbehaviour,omitempty"`
}
```

```go
type updateStateOnMisbehaviourPayload struct {
	UpdateStateOnMisbehaviour updateStateOnMisbehaviourInnerPayload `json:"update_state_on_misbehaviour"`
}
type updateStateOnMisbehaviourInnerPayload struct {
	ClientMessage clientMessageConcretePayloadClientMessage `json:"client_message"`
}
```

```go
type verifyUpgradeAndUpdateStatePayload struct {
	VerifyUpgradeAndUpdateStateMsg verifyUpgradeAndUpdateStateMsgPayload `json:"verify_upgrade_and_update_state"`
}

type verifyUpgradeAndUpdateStateMsgPayload struct {
	UpgradeClientState         exported.ClientState    `json:"upgrade_client_state"`
	UpgradeConsensusState      exported.ConsensusState `json:"upgrade_consensus_state"`
	ProofUpgradeClient         []byte                  `json:"proof_upgrade_client"`
	ProofUpgradeConsensusState []byte                  `json:"proof_upgrade_consensus_state"`
}
```

- Cosmos:

```go
// SudoMsg is used to encode messages that are sent to the contract's sudo entry point.
// The json omitempty tag is mandatory since it omits any empty (default initialized) fields from the encoded JSON,
// this is required in order to be compatible with Rust's enum matching as used in the contract.
// Only one field should be set at a time.
type SudoMsg struct {
	UpdateState                 *UpdateStateMsg                 `json:"update_state,omitempty"`
	UpdateStateOnMisbehaviour   *UpdateStateOnMisbehaviourMsg   `json:"update_state_on_misbehaviour,omitempty"`
	VerifyUpgradeAndUpdateState *VerifyUpgradeAndUpdateStateMsg `json:"verify_upgrade_and_update_state,omitempty"`
	VerifyMembership            *VerifyMembershipMsg            `json:"verify_membership,omitempty"`
	VerifyNonMembership         *VerifyNonMembershipMsg         `json:"verify_non_membership,omitempty"`
	MigrateClientStore          *MigrateClientStoreMsg          `json:"migrate_client_store,omitempty"`
}

// UpdateStateMsg is a sudoMsg sent to the contract to update the client state.
type UpdateStateMsg struct {
	ClientMessage []byte `json:"client_message"`
}

// UpdateStateOnMisbehaviourMsg is a sudoMsg sent to the contract to update its state on misbehaviour.
type UpdateStateOnMisbehaviourMsg struct {
	ClientMessage []byte `json:"client_message"`
}

// VerifyMembershipMsg is a sudoMsg sent to the contract to verify a membership proof.
type VerifyMembershipMsg struct {
	Height           clienttypes.Height         `json:"height"`
	DelayTimePeriod  uint64                     `json:"delay_time_period"`
	DelayBlockPeriod uint64                     `json:"delay_block_period"`
	Proof            []byte                     `json:"proof"`
	Path             commitmenttypes.MerklePath `json:"path"`
	Value            []byte                     `json:"value"`
}

// VerifyNonMembershipMsg is a sudoMsg sent to the contract to verify a non-membership proof.
type VerifyNonMembershipMsg struct {
	Height           clienttypes.Height         `json:"height"`
	DelayTimePeriod  uint64                     `json:"delay_time_period"`
	DelayBlockPeriod uint64                     `json:"delay_block_period"`
	Proof            []byte                     `json:"proof"`
	Path             commitmenttypes.MerklePath `json:"path"`
}

// VerifyUpgradeAndUpdateStateMsg is a sudoMsg sent to the contract to verify an upgrade and update its state.
type VerifyUpgradeAndUpdateStateMsg struct {
	UpgradeClientState         []byte `json:"upgrade_client_state"`
	UpgradeConsensusState      []byte `json:"upgrade_consensus_state"`
	ProofUpgradeClient         []byte `json:"proof_upgrade_client"`
	ProofUpgradeConsensusState []byte `json:"proof_upgrade_consensus_state"`
}

// MigrateClientStore is a sudoMsg sent to the contract to verify a given substitute client and update to its state.
type MigrateClientStoreMsg struct{}
```
