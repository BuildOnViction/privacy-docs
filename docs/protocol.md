## General flow

*** [Deposit](#deposit) ***

To use privacy transaction, you need to deposit to your privacy account first - this process is visible to everyone, recorded on chain as
In transactions to smart-contract. The output of this progress is an UTXO
```mermaid
graph LR;
    A[Normal Account] -->|Deposit| B("Privacy Account (thru SC)") 
    B -- "Event(UTXO)" --> A
```
*** Generate Transaction Proof in Deposit ***
```mermaid
graph TD;
    A[Normal Account]
    A --> E(Gen 1 UTXO output)
    A --> F(Gen Mask)
    E --> G(Gen Commitment)
    F --> G(Gen Commitment)
    H(plain amount) --> G(Gen Commitment)
    A --> H(Set amount)
    G --> TxProof
    
```

*** Private Send ***

```mermaid
graph LR;
    Sender[Sender] -->|TxProof| B(Privacy SC - TRC21) 
    B -- "Event(2 UTXOs)" --> Sender
```
How the TxProof generated is explained in detail below.

*** Generate Transaction Proof in Private Send ***

```mermaid
graph TD;
    Sender[Sender] --> B(Decode Receiver Privacy Address)
    B --> C(Select UTXOs to spend)
    C --> D{Balance enough} 
    D -->|False| Sender
    D -->|True| K(Gen Proof)
    K --> E(Gen UTXOs output)
    K --> F(Gen RingCT)
    K --> G(Gen RangeProof)
    E --> E1(Gen receiver UTXO)
    E --> E2("Gen remain UTXO - even == 0")
    F --> F2(Select Decoys)
    F2 --> F21(Gen MLSAG)
    F2 --> F23(Gen Confidential Transaction ring)
    F21 --> F3(RingCT)
    F23 --> F3(RingCT)
    G --> |Merge|TxProof
    F3 --> |Merge|TxProof
    E1 --> |Merge|TxProof
    E2 --> |Merge|TxProof
```

## Deposit


## UTXO data structure

