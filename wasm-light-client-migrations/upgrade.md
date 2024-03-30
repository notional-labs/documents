## Update flake 
Migration to v8 involves using [`MsgStoreCode`](./differences.md#messages), as detailed in the [composable-cosmos.nix](https://github.com/ComposableFi/composable/blob/main/flake/cosmos/composable-cosmos.nix#L24-L34) configuration. The transition requires updating the message format from `MsgPushNewWasmCode` to `MsgStoreCode`. Below is an example of how the message format should be updated:

     ibc-lightclients-wasm-v1-msg-push-new-wasm-code = code: {
        "messages" = [{
          "@type" = "/ibc.lightclients.wasm.v1.MsgPushNewWasmCode";
          "signer" = "${gov.account}";
          "code" = code;
        }];
        "deposit" = "5000000000000000ppica";
        "metadata" = "none";
        "title" = "none";
        "summary" = "none";
      };


## [Update lighclient 08-wasm](https://github.com/ComposableFi/composable-ibc/blob/master/light-clients/ics08-wasm/src/msg.rs#L32)

In version 7, we used the `MsgPushNewWasmCode` message. Starting with version 8, we must transition to using `MsgStoreCode` according to the new specifications. This change aligns with the updated requirements for light client implementations.


## [Update Hyperspace](https://github.com/ComposableFi/composable-ibc/blob/46588f5ba06a5e8256946533268c847ceffa66d4/hyperspace/cosmos/src/provider.rs#L61)

Hyperspace is currently using `MsgPushNewWasmCode`. To comply with the new specifications, we must transition to using `MsgStoreCode`. This update ensures compatibility with the latest protocol standards and enhances the system's overall efficiency and security.