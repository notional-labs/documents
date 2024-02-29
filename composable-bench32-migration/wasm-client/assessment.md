# Assessment

The `wasm-client` module `is not being affected` by the migration process.

## Client store path

- `clients/{clientID}`
- `clients/{clientID}/consensusState/{height}`
- `clients/{clientID}/clientState`

`client_id` is not being affected during if the `bench32` prefix is changed.

```go
// modules/core/02-client/types/keys.go
func FormatClientIdentifier(clientType string, sequence uint64) string {
	return fmt.Sprintf("%s-%d", clientType, sequence)
}
```

> These path is not affected by the migration process.

## States

### Client States

```go
type ClientState struct {
	Data         []byte       `protobuf:"bytes,1,opt,name=data,proto3" json:"data,omitempty"`
	CodeId       []byte       `protobuf:"bytes,2,opt,name=code_id,json=codeId,proto3" json:"code_id,omitempty"`
	LatestHeight types.Height `protobuf:"bytes,3,opt,name=latest_height,json=latestHeight,proto3" json:"latest_height" yaml:"latest_height"`
	// Types that are valid to be assigned to XInner:
	//	*ClientState_Inner
	XInner isClientState_XInner `protobuf_oneof:"_inner" json:",omitempty"`
}
```

- ics10_grandpa

```rust
pub struct ClientState<H> {
	/// Relay chain
	pub relay_chain: RelayChain,
	// Latest relay chain height
	pub latest_relay_height: u32,
	/// Latest relay chain block hash
	pub latest_relay_hash: H256,
	/// Block height when the client was frozen due to a misbehaviour
	pub frozen_height: Option<Height>,
	/// latest parachain height
	pub latest_para_height: u32,
	/// ParaId of associated parachain
	pub para_id: u32,
	/// Id of the current authority set.
	pub current_set_id: u64,
	/// authorities for the current round
	pub current_authorities: AuthorityList,
	/// phantom type.
	pub _phantom: PhantomData<H>,
}
```

### ConsensusState

```go
type ConsensusState struct {
	Data []byte `protobuf:"bytes,1,opt,name=data,proto3" json:"data,omitempty"`
	// timestamp that corresponds to the block height in which the ConsensusState
	// was stored.
	Timestamp uint64 `protobuf:"varint,2,opt,name=timestamp,proto3" json:"timestamp,omitempty"`
	// Types that are valid to be assigned to XInner:
	//	*ConsensusState_Inner
	XInner isConsensusState_XInner `protobuf_oneof:"_inner" json:",omitempty"`
}
```

- ics10_grandpa

```rust
pub struct ConsensusState {
	pub timestamp: Time,
	pub root: CommitmentRoot,
}
```

The client state, and consensus state, which does not using the `Addr` as a key / value, will not be affected by the migration process.

## References

- [notional-labs/ibc-go](https://github.com/notional-labs/ibc-go/tree/6cf43006971fe5862ca420c278a408d189918b98)
