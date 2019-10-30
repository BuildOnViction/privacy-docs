## TomoZ protocol

TomoZ is a protocol to issue tokens based on the TRC21 standard. TRC21 tokens solve a critical user adoption issue found in nearly every major smart contract platform, including Ethereum: Individual users must hold the native token (Ether, for example) in their wallets to pay for transactions, even when immersed in a dApp’s ecosystem. That’s real friction.

TomoChain’s TRC21 allows users to pay the transaction fee with the same TRC21 token they are already using. That’s a frictionless experience so now users can focus on the experience you provide as a TRC21 token issuer.

On privacy, TomoZ protocol helps to sign the transaction so it's easy to hide the sender on private send and withdraw

## Elliptic-curve cryptography (ECC)
## Keys (spend key, view key)
Spend key and view key of an account calculated by following:

- `private_spend_key* = *private_key` - 32 bytes or 64 in hex form
- `private_view_key* = *keccka(private_key)` - 32 bytes or 64 in hex form
- `public_spend_key* = *private_spend_key * sec256k1.G` - 33 bytes or 66 in hex form
- `public_view_key* = *private_view_key * sec256k1.G` - 33 bytes or 66 in hex form

In general, you need private spend key to send money and private view key to know how much you got in your balance.

The sender reads the public spend key and public view key from your privacy address, and uses them to construct a special one-time "container" which will hold the funds intended for you.

On the receiving end you check each TX against your private view key (technically, you also need the public spend key) and when there's a match, your wallet takes note of that, adds it to the balance and saves the details into wallet cache.

When you want to spend, your wallet uses both the private view key and private spend key to reconstruct a key to the particular "container", and a key image of the container. With that you can spend, or just check if it was spent.

## Privacy address
Privacy address contain the public view key and public spend key of an user.

`privacy address = base58_encode(public_spend_key + public_view_key + checksum)` - length 95


## Stealth Address (One Time Address)
A stealth address is a privacy-enhancing technique for protecting the privacy of receivers of cryptocurrencies. Stealth addresses require the sender to create random, one-time address for every transaction on behalf of the recipient so that different payments made to the same payee are unlinkable.
Relies on the Elliptic Curve Diffie-Hellman (ECDH) protocol and works as follows:

- The receiver has a private/public key pair (b, B), where B = b·G and G is the base point of an elliptic curve group.
- The sender generates an ephemeral key pair (r, R), where R = r·G and transmits it with the transaction.
- Both the sender and receiver can compute a shared secret c using the ECDH: c = H(r·b·G) = H(r·B) = H(b·R), where H(·) is a cryptographic hash function.
- The sender uses c·G + B as the ephemeral destination address for sending the payment.
   The receiver actively monitors the blockchain and check whether some transaction has been sent to the purported destination address c·G + B. If it is, the payment can be spent using the corresponding private key c + b. Note that the ephemeral private key c + b can only be computed by the receiver.

## Pedersen Commitment
Pederson commitment is cryptographic commitment scheme equivalent to secretly writing a secret message m in a sealed, tamper-evident, individually numbered (or/and countersigned) envelope kept by who wrote the message. The envelope's content can't be changed by any means, and the message does not leak any information.

In blockchain, a Pederson commitment C to a number f is defined as follows:
C = b*G + f*H
Where b is 256-bit number called blinding factor/mask, H is another generator point. 

If we consider f as an account’s balance on TomoChain, the commitment C can be used for hiding the transaction amount of such account. In other words, C is the Pederson encrypted amount of such account.

Pederson commitment is a form of additively homomorphic encryption, which has the following nice property: The sum of two Pederson commitments to f1 and f2 is equal to the Pederson commitment of f1 + f2.
            C(b1, f1) + C(b2, f2) = C(b1 + b2, f1 + f2)
This is the key application of Pederson commitment to privacy-focused blockchain.


## UTXO
An unspent transaction output is the output of a transaction that a user receives and is able to spend in the future. This is true because, as the name suggests, it is the unspent output of a transaction. It's a way of organizing a blockchain's ledger such that no funds are spent twice, avoiding the double spending problem.

Each and every UTXO is like a single fiat coin or a single fiat bill. If you have $45 in cash, you must have more than one bill because there’s no such thing as a forty-five dollar bill. So while you have $45 dollars in your wallet, you may have any number of combination of bills— UTXO— sitting in your wallet.

In this simple example, you could have any of the following combinations of fiat bills:

    1. Forty-five $1 bills
    2. Nine $5 bills
    3. Four $10 bills and one $5 bill
    4. Two $20 bills and five $1 bills

And so on. There are many more combinations of bills that add up to $45 but you get the idea. In each case, you have exactly $45 despite the fact that you have a different number of bills in each scenario.

The same is true of UTXO. Although you see a single balance when you log in to your crypto wallet, you may have one or many UTXO sitting in your wallet. These UTXO vary in size but when added together, the sum is equal to the total balance of your wallet.

## MLSAG (Multilayered Linkable Spontaneous Ad-Hoc GroupSignatures)
In order to describe how a Tomochain transaction hides both the spending UTXOs and the amount of the transaction, we introduce 2 additional concepts:

1. A generalization of the ring signature to allow each member of the ring to have a key-pair vector [(pk_1,sk_1),.,(pk_n,sk_n)] instead of only one pair (pk,sk).
2. A particular map known as the (Pedersen Commitment) that will be used to hide transaction amounts while allowing the network to check that input and output amounts always balance out.

## RingCT

## Range-proof (Bullet Proof)
Range proofs is a type of zero-knowledge proof used for proving that a secret is within a value range without revealing the precise value of the secret. Bulletproofs is a new non-interactive zero-knowledge proof protocol with very short proofs and without a trusted setup; the proof size is only logarithmic in the witness size. Bulletproofs are especially well suited for efficient range proofs on committed values: they enable proving that a committed value is in a range using only (2 logn + 9) group and field elements, where n is the bit length of the range. Proof generation and verification times are linear in n.

Bulletproofs will be used as follows:

    1. User A creates transaction sending 10 TOMO to User B
    2. User A sets transaction amount as 10 TOMO in the transaction
    3. User A generates blinding factor and then Pederson commitment to 10 TOMO
    4. User A generates Bulletproofs for 10 TOMO and embed the proof in the transaction
    5. User A sets transaction amount as 0 to hide it from any one, except the recipient of the transaction.

The last step reminds us a question: how does the recipient know the transaction amount if the transaction amount is packed in the commitment, given that reversing transaction amount from the commitment is impossible without knowing the blinding factor? This will be solved by using a Symmetric Encryption scheme with a secret encrypting key shared only between the sender and recipient of the transaction. 

Using Bulletproofs also allows fullnodes to verify that the sender really has enough balance to V units of cryptocurrency.

