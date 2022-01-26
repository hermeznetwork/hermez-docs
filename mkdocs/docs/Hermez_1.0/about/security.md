# Security

Hermez relies on certain assumptions that guarantee users can always recover their assets deposited on
the network. These assumptions are based on several design and architectural decisions that we will review here.

Hermez is a Layer2 solution running on top of Ethereum 1.0. This means that the security of Hermez depends
 on the security assumptions and guarantees provided by Ethereum.
 In addition, Hermez is a ZK-Rollup protocol:
 on top of Ethereum blockchain, Hermez adds another layer of security borrowed from Zcash. Following their work, Hermez integrates a ZK-SNARK prover/verifier module to validate in constant time the execution of a series of transactions.

 These ZK-SNARKs make use of certain cryptographic primitives such as hashes and signatures that make further security assumptions as it will be reviewed later.  Finally, Hermez embeds operating rules in different smart contracts to guarantee that user's transactions cannot be blocked by operators and can withdraw their assets at all times.

As a summary, Hermez makes the following security assumptions:
1. Security assumptions of Ethereum.
2. Groth16 assumptions (knowledge of exponent assumption).
3. Certain cryptographic assumptions from  primitives such as signatures and hashes
4. Software security assumptions that rely on correct design and implementation.

## Ethereum
Hermez runs on top of Ethereum. All Hermez data is available on Ethereum and borrows layer 1 security too.

## ZK-Proofs
User transactions are always verified by an Ethereum smart contract by verifying the ZK-Proof supplied by the coordinator.
The specific ZK-SNARK that is used in these ZK-Proofs is [`Groth16`](https://eprint.iacr.org/2016/260.pdf).
This protocol has been widely used and tested by the Zcash team of researchers and it is currently
considered mature enough to be used in production.

At this time, Ethereum precompiled smart contracts only support BN254 elliptic curve operations for zk-SNARK proofs validation. For this reason, Hermez uses this curve for generating and validating proofs and Baby Jubjub [`here`](https://iden3-docs.readthedocs.io/en/latest/_downloads/33717d75ab84e11313cc0d8a090b636f/Baby-Jubjub.pdf) and [`here`](https://github.com/ethereum/EIPs/pull/2494) for implementing elliptic curve cryptography inside circuits.

In place of BN254, which offers 100 bits of security, Zcash uses [`BLS12-381`](https://tools.ietf.org/id/draft-yonezawa-pairing-friendly-curves-00.html#rfc.section.4.3), with 128 bits of security [`see here`](https://tools.ietf.org/id/draft-yonezawa-pairing-friendly-curves-00.html#rfc.section.4.3).
Hermez will likely migrate to [`BLS12-381`] curve as soon as it is available for Ethereum.

Among other benefits, BLS12-381 provides [`128 bits`](https://electriccoin.co/blog/new-snark-curve/) of security instead of the 100 bits provided by BN256.  The [`EIP`](https://github.com/ethereum/EIPs/pull/2537) that implements BLS12-381 curve was already approved and the migration is very likely to happen by the next planned Berlin Hard Fork. This change will improve the security level.
At this point Baby Jubjub will be substituted by [`Jubjub curve`](https://z.cash/technology/jubjub/).

Baby Jubjub curve satisfies security standards as shown [`here`](https://safecurves.cr.yp.to/) and [`here`](https://github.com/barryWhiteHat/baby_jubjub).

### Multi-party Computation for the Trusted Setup
The proving and verification keys of the ZK-SNARK protocol require the generation
of some random values that need to be eliminated. This elimination process is a
crucial step: if these values are ever exposed, the security of the whole scheme is
compromised.
To construct the setting, Hermez uses a [`Multi-party computation (MPC)`](https://en.wikipedia.org/wiki/Secure_multi-party_computation)
 ceremony that allows multiple independent parties to collaboratively construct the parameters or
trusted setup. With MPC, it is enough that one single participant deletes its secret counterpart of the
contribution in order to keep the whole scheme secure.

The construction of the trusted setup has two phases:
1. General MPC ceremony that is valid for any circuit (also known as Powers of Tau ceremony)
2. Phase 2 that is constructed for each specific circuit.

Anyone can contribute with their randomness to the MPC ceremonies and typically, before getting the final
parameters, a random beacon is applied.

To contribute to the robustness of the setup, Hermez implemented an independent
 [`snarkjs`](https://github.com/iden3/snarkjs) module for computing and validating the MPC ceremonies.
 The software is compatible with current [`Powers of Tau`](https://github.com/kobigurk/phase2-bn254), and it
allows one to see the list of contributions of a given setup, to import a response, export a challenge of the
ceremony, or to verify if the whole process has been correctly computed. Hermez’ contribution can be
 found [`here`](https://github.com/weijiekoh/perpetualpowersoftau/blob/master/0049_jordi_response/README.md)).

## Cryptography
Hermez makes use of two main cryptographic primitives inside circuits: a signature and a hash
function.
1. The signature schema is the [`Edwards Digital Signature Algorithm (EdDSA)`](https://tools.ietf.org/html/rfc8419)
 on Baby Jubjub (after the migration, it will use EdDSA on Jubjub). This protocol was implemented making use of
 the circuit language [`circom`](https://docs.circom.io) and following the circuit design of Zcash.
2. The hash function used is [`Poseidon`](https://eprint.iacr.org/2019/458.pdf),
a similar hash to [`MiMC`](https://eprint.iacr.org/2016/492.pdf) but with a mixing layer. These hash functions
 have been used in projects such as [`TornadoCash`](https://tornado.cash/) (MiMC) and
[`Semaphore`](https://docs.zkproof.org/pages/standards/accepted-workshop3/proposal-semaphore.pdf) (Poseidon)

Assumptions made on Poseidon hash function include that it is collision and preimage resistant.

## Design
Hermez attempts to decentralize the role of coordinators while simultaneously enforcing some rules or
guidelines on the coordinators to ensure user transactions cannot be blocked. Some of these features are:

1. Coordinators are required to process L1 user transactions periodically as established in the smart contract.
Since L1 transactions are concatenated together, a coordinator must process all pending L1 transactions, thus preventing
 it from blocking specific users or L1 operations. Note that withdrawal of funds is a L1 transaction so it cannot be
blocked by a coordinator.
2. If a coordinator doesn't process (or forge) transactions during its allotted time, any online coordinator can forge
 transactions. This mechanism is known as Coordinator override.
3. HermezDAO foundation controls a last resort coordinator called Boot coordinator. If there are no coordinators
available to forge any batches, the boot coordinator will forge transactions. The HermezDAO foundation is a
non-profit organization created for the maintenance and operation of the Hermez Network, registered under
BVI 2043757 in Wickhams Cay II, Road Town, Tortola, VG1110, British Virgin Islands.
4. Bidding process format allows the governance to set different minimum bidding prices to different sots to
 increase the chances that transactions are forged by Boot coordinator.

### Security Audits
Smart contracts and circuits designed for ZK-proof system are being audited by different entities. The results will be published here as
soon as they are available.

Results from the first audit performed by [`Solidified`](https://solidified.io/) can be found
[`here`](https://github.com/hermeznetwork/contracts/blob/master/audits/Solidified%20-%20Audit%20Report%20-%20Hermez%20%5B31.10.2020%5D.pdf)

Results from the second audit performed by [`Trail of Bits`](https://trailofbits.com/) can be found
[`here`](https://github.com/hermeznetwork/contracts/blob/master/audits/Trail%20of%20Bits%20-%20Audit%20Report%20-%20Hermez%20%5B23.12.2020%5D.pdf)

