---
NEP: XXXX
Title: Adding extra hashing functions and trie data structure required for IBC compatibility
Author: Blas Rodriguez Irizar <rodrigblas@gmail.com>
DiscussionsTo: https://github.com/nearprotocol/neps/pull/0000
Status: Draft
Type: Runtime Spec
Category: Contract
Created: 15-Jul-2022
---

## Summary

Following NEP-364 additional elements are introduced to make NEAR IBC compatible.
In particular, this NEP introduces a set of missing hashing functions and an interface to
efficiently retrieve data from the storage.
// TODO: improve this paragraph once the proposal is fleshed out

## Motivation

Trustless bridges require a protocol to adequately sync states between different chains.
The Cosmos ecosystem has been using [IBC](https://github.com/cosmos/ibc)
(inter-blockchain communication protocol) successfully for some time. Projects aiming to
interoperate with NEAR building trustless bridges using a standard protocol
might opt for IBC.

This NEP introduces the changes that - combined with NEP-364 - are missing to provide developers
with the tools to use IBC on NEAR.

## Rationale and alternatives

There are trade-offs between running code that is compiled natively and exposed through a host function or
having the smart contract developer compile it using wasm. Host functions provide users with faster primitives,
while they introduce both breaking changes into the protocol and potential risks for vulnerabilities in the system.
Wasm-compiled code on the other hand is safer, given that's isolated into the smart contract runtime, at the cost
of throughput and or latency.

This NEP introduces very CPU-intensive operations (such as hashing) that shine when running on native code.
And introducing these functions as native functions will allow developers using IBC on NEAR to process
more packages in one transaction at a lower gas cost.

The impact of not doing this change is either making some interoperability unfeasible or unnecessarily expensive.

## Specification

```rust
    /// Hashes the given value using sha512 algorithm and returns it into `register_id`.
    ///
    /// # Errors
    ///
    /// If `value_len + value_ptr` points outside the memory or the registers use more memory than
    /// the limit with `MemoryAccessViolation`.
    ///
    /// # Cost
    ///
    /// `base + write_register_base + write_register_byte * num_bytes + sha512_base + sha512_byte * num_bytes`
    pub fn sha512(&mut self, value_len: u64, value_ptr: u64, register_id: u64) -> Result<()>;

    /// Hashes the given value using sha3 512 algorithm and returns it into `register_id`.
    ///
    /// # Errors
    ///
    /// If `value_len + value_ptr` points outside the memory or the registers use more memory than
    /// the limit with `MemoryAccessViolation`.
    ///
    /// # Cost
    ///
    /// `base + write_register_base + write_register_byte * num_bytes + sha3512_base + sha3512_byte * num_bytes`
    pub fn sha3_512(&mut self, value_len: u64, value_ptr: u64, register_id: u64) -> Result<()>;

    /// Hashes the given value using blake2 256 algorithm and returns it into `register_id`.
    ///
    /// # Errors
    ///
    /// If `value_len + value_ptr` points outside the memory or the registers use more memory than
    /// the limit with `MemoryAccessViolation`.
    ///
    /// # Cost
    ///
    /// `base + write_register_base + write_register_byte * num_bytes + blake2_256_base + blake2_256_byte * num_bytes`
    pub fn blake2_256(&mut self, value_len: u64, value_ptr: u64, register_id: u64) -> Result<()>

    // TODO: what about trie + state management? (discuss w/ SEUN)
```

3 hashing functions are included: `{sha512, sha3_512, blake2_256}`.
TODO: understand the specific use cases of each (ask seun)

## Future possibilities

The need behind introducing these precompiled functions is driven by performance which
ultimately translates into less gas cost. Having a more generic tool, such as a better
compiler or even something like EBPF (used in Solana) could be an alternative to provide developers
with a sandboxed environment that's more performant than current WASM-compiled code.

The aforementioned feature is a substantial change, but one that can potentially save NEAR
from having to introduce large amounts of functions in the future.

## Copyright

[copyright]: #copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
