# ECDSA adaptor signatures

DLCs work by conditioning a transaction on the revelation of an oracle's signature .
Concretely the approach is to encrypt a transaction signature such that the oracle's released signature is the decryption key.
Once the oracle has revealed a signature, either party may use it to decrypt the corresponding transaction signature and broadcast the transaction.
By this method an oracle's view of a real world event enforces a particular distribution of funds between two parties according to their private bet.

## Adaptor Signatures

*Adaptor signatures* are an efficient form of verifiable signature encryption that can achieve our required structure.
Given an anticipated signature `Y` we may encrypt a signature (for the purposes of this document, an ECDSA signature) under `Y` such that once a party learns the discrete logarithm of `Y` they can decrypt it and get a valid ECDSA signature.

An interesting aspect of adaptor signatures is that the decryption key (the discrete logarithm of the encryption key `Y`) can be derived if both the plaintext (the valid ECDSA signature) and the ciphertext (the adaptor signature) are known.
In our case the leaked decryption key is the scalar *s* value of a [[BIP340]] signature which turns out to be quite useful.
The ability to recover the *s* value from the on-chain transaction actually improves the security of the protocol compared to the original DLC description in [[Dry]].
To see why, imagine an oracle who engages in bets based on their own signatures.
In order to maintain their credibility they must publish the correct signature scalar `s_valid` publicly, but what is to stop them from privately generating a favorable and wrong `s_cheat` to settle their own bet?
In the original DLC protocol this was possible and difficult to catch.
Using adaptor signatures the wronged party can extract the malicious `s_cheat` value from the on-chain signature and use it along with `s_valid` to produce a proof of fraud.

An appropriate ECDSA adaptor signature scheme that can be used in Bitcoin prior to the Taproot soft-fork was originally posted to the Bitcoin mailing list by [[Mor]] and then further refined by [[Fou],[Aum]].
The algorithms presented here resemble the scheme from [[Fou]] but the security proof of the scheme in [[Aum]] can be applied to it which requires a proof of knowledge of the discrete logarithm to be attached to each encryption key.
Although we do not explicitly attach proofs of knowledge, in the DLC application the encryption key is the anticipated signature of an oracle which by definition the oracle must "know" and therefore we can safely omit the proof of knowledge.

## Notation and Conventions

- secp256k1 operations: We refer to *points* and *scalars* of the secp256k1 group and the operations between them.
  - *n* denotes the curve order of secp256k1 `0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141`
  - Scalars are integers between *0..(n-1)* inclusive.
  - Scalars have three infix operations: multiplication (`*`), addition (`+`) and subtraction (`-`) which are done modulo *n*.
  - Scalars have a postfix operation (`⁻¹`) which denotes multiplicative inversion modulo *n*.
  - Points are members of the secp256k1 group.
  - Points have two binary operations between them: addition (`+`) and subtraction (`-`) which represent the group operation and the inverse group operation respectively.
  - Points and scalars have a infix multiplication (`*`) operation which for `s * P` means adding the point `P` to itself `s` times.
  - *G* is the standard generator point of secp256k1 with coordinates:
    - x: `0x79BE667EF9DCBBAC55A06295CE870B07029BFCDB2DCE28D959F2815B16F81798`
    - y: `0x483ADA7726A3C4655DA4FBFC0E1108A8FD17B448A68554199C47D08FFB10D4B8`
  - When non-zero points are encoded as bytes they are encoded using the 33-byte compact [[SEC1]] encoding (implementations should fail if they try and encode the point at infinity).
  - When scalars are encoded as bytes they are encoded as 32-byte big endian integers.
- other operations:
  - `||` is an infix operation referring to byte concatenation. Its two arguments are implicitly converted to bytes.

### Secure Nonce Generation

Nonce generation is the most difficult aspect of cryptographic protocols to specify and the most important to get right.
We abandon some of the responsibility for this in this document for brevity and flexibility.
In the specification we use a function `sample_nonce` that takes a byte string and reliably returns a uniformly random scalar.
Implementations may decide precisely how to instantiate this function.
If `sample_nonce` is instantiated with a secure hash function then the resulting scheme is secure if we model it as a random oracle as shown by [[Bel]].
However, we recommend adding system randomness into the process as well.
This can be done by hashing 32 random bytes along side the input string or applying more sophisticated approach as in [[BIP340]].

## Proof of Discrete Logarithm Equality

As part of the ECDSA adaptor signature a proof of discrete logarithm equality must be provided.
In our case this is a proof that the discrete logarithm of some `X` to the standard base `G` is the same as the discrete logarithm of some `Z` to the base `Y`.
This proof can be constructed by using *equality composition* on two Sigma protocols proving knowledge of the discrete logarithm between both pairs of points.
In other words the prover proves knowledge of `a` such that `X = a * G` and `b` such that `Z = b * Y` and that `a = b`.
We make the resulting Sigma protocol non-interactive by applying the Fiat-Shamir transformation with SHA256 as the challenge hash.
For background on Sigma protocols and making them non-interactive see [[Sch]].

To disambiguate the proof we first *tag* a SHA256 as specified in [[BIP340]] with a string that identifies the kind of statement we are proving: `tag_string = "eq(DLG(secp256k1),DL(secp256k1))"`.
Specifically we set `tag` to `SHA256(tag_string) || SHA256(tag_string)` and use it to prefix hash operations.
This allows the proof system here to be used in other applications while also domain separating the proof enough so that no part of it could be used by a malicious actor in some unintended context.
The challenge hash `H` below is then defined as `H(x) = scalar(SHA256(tag || x))`.

### Proving

The `DLEQ_prove(x, (X, Y, Z))` algorithm takes the following inputs:

- `x`: A **non-zero** scalar representing the witness for the proof.
- `(X, Y, Z)`: Three **non-zero** secp256k1 points which define the statement to be verified.

and is defined as:

- Set `a` to `sample_nonce(tag || X || Y || Z || x)`
- Set `A_G` to `a * G`
- Set `A_Y` to `a * Y`
- Set `b` to  `H(X || Y || Z || A_G || A_Y)`
- Set `c` to `a + b * x`
- Set `proof` to `b || c`
- Return `proof`

### Verifying

The `DLEQ_verify((X,Y,Z), proof)` algorithm takes as input:

- `(X,Y,Z)`: Three **non-zero** secp256k1 points which define the statement we are verifying.
             Specifically, there exists `x` such that `X = x * G` and `Z = x * Y`.
- `proof`: the proof for the statement.

and is defined as:

- Parse `proof` as two scalars `(b,c)`
- Set `A_G` to `c * G - b * X`
- Set `A_Y` to `c * Y - b * Z`
- Set `implied_b` to `H(X || Y || Z || A_G || A_Y)`
- Check `implied_b == b`

## Adaptor Signature

### Encrypted Signing

The encrypted signing algorithm `ecdsa_adaptor_encrypt(x, Y, message_hash)` takes as input:

- `x`: The signing *secret key*.
- `Y`: The encryption *public key* (in our case the *anticipated signature* of an oracle associated with a particular outcome).
- `message_hash`: A 32-byte array which **must** be the hash of the message (in our case a transaction digest).

and is defined as:

- Set `k` to `sample_nonce(Y || message_hash || x)`
- Set `m` to `scalar(message_hash)`
- Set `R_a` to `k * G`
- Set `R` to `k * Y`
- Set `proof` to `DLEQ_prove(k, (R_a, Y, R))`
- Set `r` to the x-coordinate of `R` modulo `n` (i.e. convert it to a scalar)
- Set `s_a` to `k⁻¹ (m + r * x)`
- Set `a` to `R || R_a || s_a || proof`
- Return `a`

The return value is an adaptor signature.

### Encryption Verification

The verification algorithm `ecdsa_adaptor_verify(X, Y, message_hash, a)` takes as input:

- `X`: The signing *public key* the adaptor signature was created under.
- `Y`: The encryption *public key* the adaptor signature was encrypted under.
- `message_hash`: The message hash the adaptor signature was created for (in our case a transaction digest).
- `a`: The adaptor signature

and is defined as:

- Parse `a` as `(R, R_a, s_a, proof)`
- Check `DLEQ_verify((R_a, Y, R), proof)` or fail
- Set `m` to `scalar(message_hash)`
- Set `u_1` to `s_a⁻¹ * m`
- Set `u_2` to `s_a⁻¹ * r`
- Check `u_1 * G + u2 * X == R_a`

### Decryption

The `ecdsa_adaptor_decrypt(a, y)` algorithm takes as input:

- `a`: The adaptor signature.
- `y`: The decryption *secret key*.

and is defined as:

- Parse `a` as `(R, R_a, s_a, proof)`
- Set `s` to `s_a * y⁻¹`
- Negate `s` if it is high as specified in [[BIP62]]
- Set `r` to the x-coordinate of `R` modulo `n`
- Return `r || s`

The returned value is a ECDSA signature.
The signature will be valid if `a` passed `ecdsa_adaptor_verify` and the decryption key is `y` is such that the original encryption key `Y` is equal to `y*G`.

### Key Recovery

The decryption key recovery algorithm `ecdsa_adaptor_recover(Y, a, sig)` takes the following input:

- `Y`: The encryption *public key* the adaptor signature was encrypted under.
- `a`: The ECDSA adaptor signature
- `sig`: The ECDSA signature decrypted from `a`

and is defined as follows:

- Parse `a` as `(R, R_a, s_a, proof)`
- Parse `sig` as `(r,s)`
- Set `r_implied` to the x-coordinate of `R` modulo `n`
- Check `r == r_implied` or fail
- Set `y` to `s⁻¹ * s_a`
- Set `Y_implied` to `y * G`
- If `Y_implied == Y` then return `y`
- Otherwise, if `Y_implied == -Y` then return `-y`
- Otherwise fail

The returned value is the decryption key corresponding to the original encryption key `Y` used to create `a`.
In our context this is an oracle signature *s* value.

Note that if this function returns successfully and `a` has been verified with `ecdsa_adaptor_verify` for some public key and message hash then this implies that `sig` is a valid ECDSA signature on the message hash
i.e. there is no strict need to verify `sig` after (or before) successfully calling `ecdsa_adaptor_recover` if you have already ensured that `ecdsa_adaptor_verify` passes on the original adaptor signature.
However, successfully recovering the signature does not ensure that it follows the `low_s` rule described in [[BIP62]] necessary to be able to broadcast the signature on a transaction in Bitcoin.
This point is not relevant to on-chain DLCs since the ECDSA signatures always come from the blockchain where they have hopefully already been verified.


## Specification tests

Several verification test vectors are in [./ecdsa_adaptor.json].
These only check that verification is done correctly.
Implementations should use the vectors in the follwowing way:

- First check `adaptor_sig` passes `ecdsa_adaptor_verify`.
- Check that `ecdsa_adaptor_decrypt` recovers `signature`.
- Check that  `ecdsa_adaptor_recover` recovers `decryption_key`.

These should fail only when `error` is set in the test vector.

[Dry]: https://adiabat.github.io/dlc.pdf
[Mor]: https://lists.linuxfoundation.org/pipermail/lightning-dev/attachments/20180426/fe978423/attachment-0001.pdf
[Fou]: https://github.com/LLFourn/one-time-VES/blob/master/main.pdf
[Aum]: https://eprint.iacr.org/2020/476.pdf
[SEC1]: https://www.secg.org/sec1-v2.pdf
[Bel]: https://eprint.iacr.org/2015/1157.pdf
[Sch]: https://www.win.tue.nl/~berry/CryptographicProtocols/LectureNotes.pdf
[BIP62]: https://github.com/bitcoin/bips/blob/master/bip-0062.mediawiki#low-s-values-in-signatures
[BIP340]: https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki
