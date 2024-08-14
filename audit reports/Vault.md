# **Security Audit Report**

---

**Project Name:** Vault Smart Contract  
**Audit Date:** August 14, 2024  
**Auditor:** Shruti

---

## **Executive Summary**

The `Vault` smart contract is designed to store a boolean lock state (`s_locked`) and a private password (`s_password`) that can unlock the vault if correctly provided. However, the contract is vulnerable to an **on-chain data exposure** issue. Despite marking the `s_password` variable as `private`, it can still be accessed by reading the blockchain directly. This allows an attacker to retrieve the password and unlock the vault, compromising its security.

## **Scope**

The audit covers the following aspects of the `Vault` smart contract:
1. **On-Chain Data Exposure**
2. **Access Control**
3. **Input Validation**
4. **Best Practices and Code Quality**

## **Findings**

### **1. On-Chain Data Exposure**
   - **Severity:** Critical
   - **Description:** The `s_password` variable is stored on-chain as a `private` state variable. However, marking a variable as `private` in Solidity does not mean it is hidden from everyone; it only restricts direct access via the contract's interface. The actual value is stored on the blockchain and can be accessed by anyone with knowledge of blockchain exploration tools (e.g., by using `eth_getStorageAt`).
   - **Impact:** An attacker can easily retrieve the password from the blockchain and use it to call the `unlock()` function, bypassing any intended security and unlocking the vault. This completely compromises the contract's purpose.
   - **Recommendation:** Do not store sensitive information on-chain if it must remain secret. Instead, consider off-chain storage of sensitive data and use cryptographic methods to validate access (e.g., using hash commitments where the secret itself is not revealed on-chain).

### **2. Access Control**
   - **Severity:** Medium
   - **Description:** The contract allows anyone who knows the password to call the `unlock()` function. There is no additional layer of access control beyond the password itself. This could be problematic if the contract is part of a larger system where only specific users should have the authority to unlock it.
   - **Impact:** In systems where access control is critical, allowing anyone to unlock the vault if they know the password could lead to unauthorized access.
   - **Recommendation:** Consider implementing additional access control mechanisms. For example, require that the caller be a specific authorized address or that the password is combined with some user-specific data (e.g., msg.sender) to make it unique per user.

### **3. Input Validation**
   - **Severity:** Low
   - **Description:** The contract checks that the provided password matches the stored password but does not validate the format or length of the input password. While this isn't a direct vulnerability, it could lead to unexpected behavior if non-standard inputs are used.
   - **Impact:** Invalid or unexpected input formats could lead to potential errors or unexpected behavior, especially if the contract is modified in the future.
   - **Recommendation:** Ensure that the password input adheres to expected formats. Consider adding validation checks if future modifications introduce complexity or reliance on specific password characteristics.

### **4. Best Practices and Code Quality**
   - **Severity:** Low
   - **Description:** The contract lacks detailed comments and documentation, which could lead to misunderstandings or errors during future maintenance. Additionally, no event is emitted when the vault is unlocked, which could make tracking and auditing actions difficult.
   - **Impact:** Lack of documentation and event emissions can make the contract difficult to understand, audit, and maintain. It also reduces transparency for users interacting with the contract.
   - **Recommendation:** Add detailed comments and documentation to improve code readability and maintainability. Implement event emissions when the vault is unlocked to enhance transparency:
     ```solidity
     event Unlocked(address indexed user);

     function unlock(bytes32 password) external {
         if (s_password == password) {
             s_locked = false;
             emit Unlocked(msg.sender);
         }
     }
     ```

## **Exploit Scenario**

### **On-Chain Data Exposure**
1. **Step 1:** An attacker scans the blockchain for the contract's deployment address.
2. **Step 2:** The attacker uses a blockchain explorer or tool like `eth_getStorageAt` to retrieve the value of the `s_password` variable from storage.
3. **Step 3:** The attacker calls the `unlock()` function with the retrieved password.
4. **Step 4:** The vault is unlocked, and the attacker gains unauthorized access.

## **Recommendations**

1. **Avoid On-Chain Storage of Sensitive Data:**
   - Do not store sensitive information such as passwords directly on the blockchain. Consider off-chain storage and use hash commitments or other cryptographic techniques for on-chain verification.

2. **Implement Additional Access Controls:**
   - Add access control mechanisms to ensure that only authorized users can unlock the vault, even if they know the password.

3. **Improve Input Validation:**
   - Validate the format and length of input passwords to prevent unexpected behavior, especially in future contract versions.

4. **Enhance Documentation and Transparency:**
   - Add detailed comments and documentation to improve code readability. Emit events for critical actions like unlocking the vault to increase transparency and traceability.

## **Conclusion**

The `Vault` contract is critically vulnerable due to the on-chain exposure of the `s_password` variable. This vulnerability allows an attacker to retrieve the password from the blockchain and unlock the vault, rendering the contract ineffective for secure storage. To mitigate this, it is recommended not to store sensitive data on-chain and to consider more secure alternatives for access control and validation. Additionally, enhancing input validation, documentation, and event emissions will improve the contract's security and maintainability.

---

**End of Report**
