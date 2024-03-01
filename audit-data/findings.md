### [H-01] Data stored on-chain is visible for anyone, and therefore breaks the confidentiality of the contract

**Description:** All data stored on-chain is visible for anyone and therefore can not be secret. The variable `PasswordStore::s_password` was designed to be private and only accessible through the function `PasswordStore::getPassword`. 

**Impact:** Anyone can read the private passwords and break the confidentiality of the contract.

**Proof of Concept:** (Proof of Code)
1. Run a local chain: 
```bash
anvil
```

2. Deploy the contract to the local chain using:
```bash
make deploy
```

3. Run the storage tool (`cast storage`) on the address of the deployed contract
```bash
cast storage $CONTRACT_ADDRESS 1 --rpc-url http://127.0.0.1:8545
```

4. Convert the received hex storage variable to a string using `cast parse-bytes32-string` command.
```bash
cast parse-bytes32-string $OUTPUT_OF_THE_LAST_COMMAND
```
**Recommended Mitigation:** Rethinking of the whole architecture is advised. One of the approaches could be to store the off-chain encrypted password on-chain. 