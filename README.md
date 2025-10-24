[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/1neRm4kC)
# assignment-4
Bitcoin Scripting


Assignment A

Given this script:

OP_DUP OP_HASH160 <PubKeyHash> OP_EQUALVERIFY OP_CHECKSIG

* This is a standard Pay-to-PublicKey-Hash locking script used in bitcoin transactions.

Tasks:

Break down each opcode's purpose

| Opcode           | Meaning                                | Purpose                                                                                       |
| ---------------- | -------------------------------------- | --------------------------------------------------------------------------------------------- |
| `OP_DUP`         | Duplicates the top stack item          | Copies the public key so one copy can be hashed and the other kept for signature verification |
| `OP_HASH160`     | Applies `SHA-256` then `RIPEMD-160`    | Produces a 20-byte hash of the public key                                                     |
| `<PubKeyHash>`   | Constant pushed in script              | The expected public key hash (recipient’s address)                                            |
| `OP_EQUALVERIFY` | Checks equality and verifies           | Ensures the hash of the provided pubkey equals the expected one; if not, script fails         |
| `OP_CHECKSIG`    | Verifies signature with the public key | Confirms the transaction is signed by the owner of the private key                            |


Create a diagram showing data flow

Below are the execution steps

    (Therefore in the Pay-to-PubkeyHash transaction,first we prove that the public key that the redeemer states is the same as we had in the InputTX ,then we verify if the redeemer has the right secret key by verifying the signature of the transaction.)

Identify what happens if signature verification fails

* If `OP_CHECKSIG` returns `FALSE`
  * The final stack is `FALSE`
  * The script fails and transaction is invalid
  * Bitcoin nodes reject it and coins remain unspent

Explain the security benefits of hash verification

* Hashing the public key before placing it in the output script:
  * Reduces key exposure: The public key isnt revealed until the coins are spent.
  * Improves privacy: The blockchain doesnt show public key directly
  * Protects against quantum preimage attack: It future proofs, if public keys were visible they could be targeted once  quantum attacks become viable
  * Reduces transaction size: RIPEMD 160 hash is smaller(20bytes) than full public key(33/65 bytes.)


Assignment B

Implement a Hashed Time-Lock Contract for atomic swap between Alice and Bob:

Alice can claim with secret preimage within 21 minutes

Bob gets refund after 21 minutes

Tasks:

Complete the HTLC script

* Conditions
  * Alice can claim funds if she reveals a secret preimage `S` that hashes to `H` within 21 minutes.
  * Bob can refund after 21 minutes if Alice doesnt claim

```shell

OP_IF
    # Alice claims with preimage
    OP_HASH160 <H> OP_EQUALVERIFY
    <AlicePubKey> OP_CHECKSIG
OP_ELSE
    # Bob refund fund
    <21 * 60> OP_CHECKLOCKTIMEVERIFY OP_DROP
    <BobPubKey> OP_CHECKSIG
OP_ENDIF

```

Create claiming transaction script

* When Alice claims with her secret `S` and signature(Alice's Redeem Script):

Unlocking Script(ScriptSig):

```shell

<sigAlice> <S> OP_TRUE

```
   * `OP_TRUE`: -> selects the first branch(`OP_IF`)
   * `S`: -> Used for hash check
   * `sigAlice`: -> Used for signature verification


| Step                            | Action                               |
| ------------------------------- | ------------------------------------ |
| Push `<sigAlice> <S> OP_TRUE`   | Initial stack                        |
| Enters `OP_IF` branch           | Claim path active                    |
| `OP_HASH160 <H> OP_EQUALVERIFY` | Verifies `RIPEMD160(SHA256(S)) == H` |
| `<AlicePubKey> OP_CHECKSIG`     | Verifies Alice’s signature           |

* Funds go to Alice if valid within 21 minutes

Create refund transaction script

* If timeout 21 minutes passes and Alice doesnt claim

Unlocking Script (ScriptSig)

```shell

<sigBob> OP_FALSE

```

   * `OP_FALSE`: -> selects the else branch(`OP_ELSE`)
   * `OP_CHECKLOCKTIMEVERIFY`: -> enforces the time lock(must be >= current_block_time)

* Bob gets refund after timeout

Test with sample hash and timeout

| Parameter           | Example                                                                     |
| ------------------- | --------------------------------------------------------------------------- |
| Secret preimage `S` | `"secret123"`                                                               |
| Hash `H`            | `RIPEMD160(SHA256("secret123")) = a8f6e2b76a4e34c3b0dfc17d5a66be4f1db2d32c` |
| Timeout             | `21 * 60 = 1260` seconds                                                    |
