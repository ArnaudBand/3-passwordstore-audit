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

### [S-#] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

**Description:** 

The `PasswordStore::setStore` function is set to be an an `external` function, however, the natspec of the function and overall purpose of the smart contract is that `This function allows only the owner to set a new password.`

```typescript

    function setPassword(string memory newPassword) external {
 @>     // @audit any user can set a password: Missing access control
        s_password = newPassword;
        emit SetNetPassword();
    }

```

**Impact:** 

Anyone can set/change the password of the contract, severly breaking the contract intended functionality.

**Proof of Concept:**

Add the following to the `PasswordStore.t.sol` test file.


<details>

<summary>Code</summary>

```javascript
    function test_anyone_can_set_password(address randomUser) public {
        vm.assume(randomUser != owner);
        vm.prank(randomUser);
        string memory expectedPassword = "myNewPassword";
        passwordStore.setPassword(expectedPassword);
        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }
```

</details>

**Recommended Mitigation:** 

Add an access control conditional to the `setPassword` function.

```javascript

```