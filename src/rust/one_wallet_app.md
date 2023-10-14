## Signing transactions with another secure app

[Link](https://substrate.stackexchange.com/questions/10156/signing-transactions-with-another-secure-app)

If we make an Android dapp, can't we use Android [intents](https://developer.android.com/training/basics/intents/sending) to sign a transaction in another Android wallet app that supports receiving data through intents? We can pass message and public key in the intent (e.g. using [subxt](https://github.com/paritytech/subxt/blob/d7124b56f7e7ed4cba1f05cdb38a8fe48eebf898/examples/wasm-example/src/routes/signing.rs)) , and wallet makes transaction. Will there be any security issues? Though we can't connect the wallet to get the public key, we can just copy the public key from the wallet and paste it on the dapp, to get the dapp UI for the public key.

Another approach could involve creating a trusted [server apps](https://github.com/tauri-apps/tauri/discussions/2751) responsible for handling transactions and securely storing private keys or seeds. Authentication mechanisms can be implemented to ensure that only trusted applications on the computer can access the server API. This approach effectively separates the authentication logic from the dapp itself, similar to browser extensions wallet, and can also be utilized on desktop systems where "intents" are not available.

Here, two passwords can be utilized: one for communication between the Dapp and the server, which the Dapp can interact with, here [one time password](https://docs.rs/libotp/latest/libotp/) can be used for more security, also you don't need to memorize the password, and another password for signing transactions, similar to how MetaMask or the Polkadot.js extension wallet operates, where only the server wallet app can interact with this password.

For dapps that you trust or low stake transaction (e.g. your wallet just have a very small amount of money), you can allow automatic approval of signing transaction through API using ui of dapps, whereas for an untrusted dapp you need to approve the transaction in the wallet. This will improve user experience. You can set the desired security in wallet app. 

You can also use the wallet for transacting [web2 apps](https://www.toptal.com/ethereum/one-click-login-flows-a-metamask-tutorial). 

> One wallet for all your signing needs. 

## Leptos
You can use [leptos](https://github.com/leptos-rs/leptos) for dapp with tauri.

Isomorphic: Leptos provides primitives to write isomorphic server functions, i.e., functions that can be called with the “same shape” on the client or server, but only run on the server. This means you can write your server-only logic (database requests, authentication etc.) alongside the client-side components that will consume it, and call server functions as if they were running in the browser, without needing to create and maintain a separate REST or other API.

[Actix in leptos](https://docs.rs/leptos_actix/latest/leptos_actix/)

## One time otp
```
 https://docs.rs/libotp/latest/libotp/
```

```rust
use libotp::{totp, validate_totp};

const TOTP_STEP: u64 = 30;
const OTP_DIGITS: u32 = 8;

fn check_user_otp(user: User, guess: u32) -> Option<bool> {
   // get the shared secret from some database.
   let secret = user.get_totp_secret();

   validate_totp(guess, 1, secret, OTP_DIGITS, TOTP_STEP, 0)
}

fn get_user_otp(user: User) -> Option<u32> {
   // get shared secret
   let secret = user.get_totp_secret();

   totp(secret, OTP_DIGITS, TOTP_STEP, 0)
}
```

## Signing remotely

[Discussion](https://github.com/paritytech/subxt/issues/1196)


Yes, you can use [signer_payload](https://docs.rs/subxt/latest/subxt/tx/struct.PartialExtrinsic.html#method.signer_payload) but you need construct a PartialExtrinsic

Example:

```rust
    let tx = node_runtime::tx().preimage().note_preimage(vec![0u8]);
    let partial_extrinsic = api
        .tx()
        .create_partial_signed(&tx, &alice.public_key().into(), Default::default())
        .await
        .unwrap();

    // Get the signer payload.
    let signer_payload = partial_extrinsic.signer_payload();
    // Sign it (possibly externally).
    let signature = alice.sign(&signer_payload);
    // Use this to build a signed extrinsic.
    let extrinsic = partial_extrinsic
        .sign_with_address_and_signature(&alice.public_key().into(), &signature.into());

    // And now submit it.
    extrinsic
        .submit_and_watch()
        .await
        .unwrap()
        .wait_for_finalized_success()
        .await
        .unwrap();
```

## Building a cryptowallet in rust
[Website](https://tms-dev-blog.com/build-a-crypto-wallet-using-rust/)

[Github](https://github.com/tmsdev82/rust-eth-crypto-wallet-tutorial)





