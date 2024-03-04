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

### [S-02] `PasswordStore::setPassword` function has no access controls, meaning anyone could call the function and change the password

**Description:** The `PasswordStore::setPassword` function does not check the msg.sender and therefore allows anyone to make use of the functionality (i.e. change the password). The documentation of the project provided states that the password change functionality must only be called by the user who stored the password. Others according to the documentation should not have the ability to access it.

```solidity
    function setPassword(string memory newPassword) external {
        // @audit Missing access controls
        s_password = newPassword;
        emit SetNetPassword();
    }
```


**Impact:** Anyone can change the password stored in the contract, therefore breaking the main idea of the project.

**Proof of Concept:**
Add the following to the test suite:
```solidity
    function test_anyone_can_set_password(address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory expectedPassword = "myNewPassword";
        passwordStore.setPassword(expectedPassword);
        
        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(expectedPassword, actualPassword);
    }
```

**Recommended Mitigation:** 
Add access control to the function `PasswordStore::setPassword` like this:

```solidity
    require(msg.sender == owner);
```


### [I-03] NatSpec for the function `PasswordStore::getPassword` includes @param which it actually isn' needed

**Description:** The documentation for the function `PasswordStore::getPassword` includes the @param, which shouldnt be there as the function doesnt take any inputs.

**Impact:** Wrong documentation could mislead any users or developers working with this contract.


```solidity
/*
     * @notice This allows only the owner to retrieve the password.
     @info There is no input parameter in this function 
     * @param newPassword The new password to set.
     */
    function getPassword() external view returns (string memory) {
        if (msg.sender != s_owner) {
            revert PasswordStore__NotOwner();
        }
        return s_password;
    }
```

**Recommended Mitigation:** Remove the incorrect NatSpec line.

```diff
-   * @param newPassword The new password to set.
```

