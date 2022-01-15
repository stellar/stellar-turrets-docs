# Anatomy of a Turret

Before you can host a turret, it is probably good to have an understanding of
what you're actually hosting. Here's a quick description of the important pieces
at work.

> If you prefer a more technical explanation, or just want to follow along with
> these descriptions with real-world pieces, you can check out the [reference_implementation]

## An API Application

> If you're following along with the [reference_implementation], this is what is
> set up as the "cloudflare worker."

The first thing you'll need is a place where `txFunctions` can be uploaded, managed, paid for, called for execution, and respond to requests. Additionally, this is where `txFunctions` are assigned a signing key and transactions can actually be signed by the turret.

The bulk of the Stellar Turrets codebase is dedicated to this piece of the turret. This part receives and responds to API calls,

When a user, application, or anybody wants to actually use an authored `txFunction`, this API application will take the following actions:

1. Receives the request to run a specified `txFunction`, ensuring it includes a valid txFunction hash, has paid the turret fee for running, and is properly authenticated.

2. Sends the `txFunction` code to the JavaScript environment (see below) to be executed in a safe, isolated sandbox, and receives back the generated XDR from the `txFunction`.

3. Signs the transaction XDR from the previous step with the Keypair it has assigned for that particular `txFunction`.

4. Returns to the initial requester the transaction XDR, a base64-encoded signature that can be added to the transaction, and the public key of the signer.

## A JavaScript Environment

> If you're following along with the [reference_implementation], this is the AWS
> lambda function that is setup through serverless.com.
> So, now we have a way to upload, interact with, and call our turret contracts.
> The next thing we'll need is a place to actually execute that contract. (If
> you're familiar with Ethereum smart contracts, this piece is what's known as the
> Ethereum Virtual Machine (EVM): A runtime environment where the code of the
> smart contract can run.)

This environment is where the `txFunctions` will actually run and execute. It must
be able to run arbitrary JavaScript code (whatever a `txFunction` author has
written) in a safe way, and output a predictable result.

This environment must also provide a consistent, reliable environment for
`txFunction` authors. The execution will take place in a `nodejs14.x`
environment, with the following default `node_module` packages installed:

- [bignumber.js]
- [node-fetch]
- [stellar-sdk]
- [lodash]

[reference_implementation]: https://github.com/tyvdh/stellar-turrets/blob/master/README.md
[bignumber.js]: https://www.npmjs.com/package/bignumber.js
[node-fetch]: https://www.npmjs.com/package/node-fetch
[stellar-sdk]: https://www.npmjs.com/package/stellar-sdk
[lodash]: https://www.npmjs.com/package/lodash
