# StoreWasmCode

> `08-wasm/keeper/keeper.go`

1. Validate wasm code
2. create checksum
3. create the code in the vm
4. store the check sum

```go
err = ibcwasm.Checksums.Set(ctx, checksum)
```

## CreateChecksum

#TODO check how `CodeID` on notional is different from

```go
func CreateChecksum(wasm []byte) (Checksum, error) {
	if len(wasm) == 0 {
	return Checksum{}, fmt.Errorf("Wasm bytes nil or empty")
	}

	if len(wasm) < 4 {
		return Checksum{}, fmt.Errorf("Wasm bytes shorter than 4 bytes")
	}

	// magic number for Wasm is "\0asm"
	// See https://webassembly.github.io/spec/core/binary/modules.html#binary-module
	if !bytes.Equal(wasm[:4], []byte("\x00\x61\x73\x6D")) {
		return Checksum{}, fmt.Errorf("Wasm bytes do not not start with Wasm magic number")
	}

	hash := sha256.Sum256(wasm)
	return Checksum(hash[:]), nil
}
```

# `CreateClient`

Params: clientState, consensusState

> `modules/core/02-client/keeper/client.go

1. Generate client ID
2. Get client store: `clients/clientID`
3. Initialize client state
4. Check if status is active

## clientState.initialize

> `08-wasm/types/client-state.go/initialize`

Params: clientStore, consensusState

1. check checksum has been stored. #TODO
2. Create payload msg

```go
payload := InstantiateMessage{
	ClientState: cs.Data,
	ConsensusState: consensusState.Data,
	Checksum: cs.Checksum,
}
```

3. `wasmInstantiate`
   Using `storeAdapter`, in `types/store.go`, serves as a bridge between the [[Cosmos SDK]] store impl and the WasmVM store interface

```go
response, gasUsed, err := ibcwasm.GetVM().Instantiate(checksum, env, msgInfo, msg, newStoreAdapter(clientStore), wasmvmAPI, newQueryHandler(ctx, clientID), multipliedGasMeter, gasLimit, costJSONDeserialization)
```

The `storeApdater`, which create an instance of state before initialization:
https://github.com/CosmWasm/wasmvm/blob/main/spec/Specification.md#state-access

## Granpa instantiate #Wasm

> `composable-ibc/light-clients/ics10-grandpa-cw/src/contracts.rs`

```rust
pub struct InstantiateMessage {
	pub client_state: Bytes,
	pub consensus_state: Bytes,
	pub checksum: Bytes,
}
```

1. Get the `client_id`
   ```rust
   let client_id: ClientId = ClientId::from_str(env.contract.address.as_str()).expect("client id is valid");
   ```

> `client_id` in the client initialization is the `contract address` 2. Store `client_state` on the contract

    1. Path: `clients/{client_id}/clientState`

3. store `consensus_state` on the contract
   1. Path: `clients/{client_id}/consensusStates/{height}`

Ref: `ics10-granpa-cw/src/ics23/client_state.rs`

# UpdateClient

> `modules/core/02-client/keeper/client.go`

Params: clientID, clientMsg

1. Get client state `clients/{client_id}`
2. Get client store
3. client_state.verify_client_msg()
4. check misbehavior
5. client_state.update_state()

## UpdateState #Wasm

1. get_client_state (from storage)

```rust
fn client_state(&self, client_id: &ClientId) -> Result<ClientState<H>, Error> {
	let client_states: ReadonlyClientStates<'_> = ReadonlyClientStates::new(self.storage());

	let data = client_states.get().ok_or_else(|| Error::client_not_found(client_id.clone()))?;

	let state = Self::decode_client_state(&data)?;
	Ok(state)
}
```

2.
