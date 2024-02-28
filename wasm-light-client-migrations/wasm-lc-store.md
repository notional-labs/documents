## Chain
1. a new client is created here: https://github.com/cosmos/ibc-go/blob/main/modules/core/02-client/keeper/client.go#L19
2. `clientStore` is initialized here: https://github.com/cosmos/ibc-go/blob/main/modules/core/02-client/keeper/client.go#L35
3. `clientStore` is passed to wasm client through client Initialize() here: https://github.com/cosmos/ibc-go/blob/wasm-v8.0.0/modules/light-clients/08-wasm/types/client_state.go#L111
4. `clientStore` is passed into wasm VM for storing ClientState: https://github.com/cosmos/ibc-go/blob/main/modules/light-clients/08-wasm/types/vm.go#L115

## Contract
1. process_instantiate_msg() https://github.com/ComposableFi/composable-ibc/blob/master/light-clients/ics10-grandpa-cw/src/contract.rs#L132
2. store_client_state(): https://github.com/ComposableFi/composable-ibc/blob/master/light-clients/ics10-grandpa-cw/src/client.rs#L168
3. save ClientState: https://github.com/ComposableFi/composable-ibc/blob/master/light-clients/ics10-grandpa-cw/src/client.rs#L198