### [S-#] Storing the password on-chain makes it visible to anyone, and no longer private.

**Description:** 

All data stored on-chain is visible to anyone, and can be read directly from the blockchain. The `PasswordStore::s_password `variable is intended to be a private variable and only accessed through the `PasswordStore::getPassword function`, which is intended tobe only called by the owner of the contract.
We show one such method of reading any data off chain below.

**Impact:** 

Anyone can read the private password, severly breaking the functionality of the protocol.

**Proof of Concept:**

The below test case shows how anyone can read the password directly from the blockchain.

1. Create a locally running chain.

```bash
make anvil
```

2. Deploy the contract to the chain.

```bash
make deploy
```

3. Run the storage tool

We use `1` because that's the storage slot of `s_password` in the contract.

```bash
cast storage <address of the contract> <storage slot> --rpc-url http://127.0.0.1:8545
```
You'll get an output that looks like this:

`0x6d7950617373776f726400000000000000000000000000000000000000000014`
4. Convert the bytes32 into a string 

```bash 
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

and the output will be: `myPassword`

**Recommended Mitigation:** 

As a result, the contract's overall architecture needs to be reconsidered. One approach is to encrypt the password off-chain and store only the encrypted version on-chain. This would mean the user must remember a separate off-chain password to decrypt it. Additionally, it’s advisable to remove the `view` function, as you wouldn't want users to accidentally expose the decryption password in a transaction.