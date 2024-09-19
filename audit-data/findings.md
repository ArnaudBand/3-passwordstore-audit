### [S-#] Storing the password on-chain makes it visible to anyone, and no longer private.

**Description:** All data stored on-chain is visible to anyone, and can be read directly from the blockchain. The `__PasswordStore::s_password__ `variable is intended to be a private variable and only accessed through the `__PasswordStore::getPassword function__`, which is intended tobe only called by the owner of the contract.
We show one such method of reading any data off chain below.

**Impact:** Anyone can read the private password, severly breaking the functionality of the protocol.

**Proof of Concept:**

**Recommended Mitigation:** 