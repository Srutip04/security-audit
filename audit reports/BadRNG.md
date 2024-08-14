# **Security Audit Report**

---

**Project Name:** BadRNG Smart Contract  
**Audit Date:** August 14, 2024  
**Auditor:** Shruti

---

## **Executive Summary**

The smart contract `BadRNG` implements a raffle system where users can enter by sending Ether, and a random winner is selected using a simple random number generator. However, the randomness mechanism used in the contract is highly insecure, making it susceptible to manipulation by malicious actors. Additionally, there are no access controls on the function that picks the winner, further increasing the risk of exploitation. This audit highlights the critical vulnerabilities present in the contract and provides recommendations to mitigate these risks.

## **Scope**

The audit covers the following aspects of the `BadRNG` smart contract:
1. **Randomness Generation**
2. **Access Control**
3. **Reentrancy**
4. **Best Practices and Code Quality**

## **Findings**

### **1. Predictable Randomness**
   - **Severity:** Critical
   - **Description:** The `pickWinner` function uses a predictable method to generate a random winner index:
     ```solidity
     uint256 randomWinnerIndex = uint256(keccak256(abi.encodePacked(block.difficulty, msg.sender)));
     ```
     This method is insecure because both `block.difficulty` and `msg.sender` are values that can be known or controlled by an attacker. As a result, an attacker can predict the output of the `keccak256` hash and manipulate the outcome of the winner selection.
   - **Impact:** An attacker can repeatedly call the `pickWinner` function with different addresses to ensure they are selected as the winner. This could lead to the complete loss of funds from the contract.
   - **Recommendation:** Replace the current randomness mechanism with a secure and verifiable source of randomness, such as Chainlink VRF. Chainlink VRF provides tamper-proof and provable randomness that cannot be predicted or influenced by any party, including the contract owner.

### **2. Lack of Access Control on `pickWinner`**
   - **Severity:** High
   - **Description:** The `pickWinner` function is publicly accessible and can be called by anyone:
     ```solidity
     function pickWinner() external {
     ```
     This allows any user, including potential attackers, to call the function at any time and manipulate the winner selection process.
   - **Impact:** An attacker could call the `pickWinner` function as soon as they have ensured they will win, effectively stealing all the funds stored in the contract.
   - **Recommendation:** Implement access control mechanisms to restrict who can call the `pickWinner` function. This could involve using OpenZeppelin's `AccessControl` or implementing a role-based system where only authorized accounts can trigger the winner selection.

### **3. Reentrancy Risk**
   - **Severity:** Medium
   - **Description:** The contract uses the low-level `call` function to transfer the balance to the winner:
     ```solidity
     (bool success, ) = winner.call{value: address(this).balance}("");
     ```
     While the current contract implementation is not directly vulnerable to reentrancy attacks, future modifications could introduce this risk if additional state changes are made after the Ether transfer.
   - **Impact:** If a reentrancy vulnerability were introduced, an attacker could drain the contract's funds by repeatedly calling the `pickWinner` function within the context of a malicious contract.
   - **Recommendation:** Implement reentrancy guards using OpenZeppelin's `ReentrancyGuard` or by following the checks-effects-interactions pattern, which ensures that state changes are made before any external calls.

## **Recommendations**

1. **Implement Secure Randomness:**
   - Replace the insecure randomness mechanism with Chainlink VRF or another verifiable random number generator to ensure that the outcome of the raffle cannot be predicted or manipulated.

2. **Add Access Control:**
   - Restrict access to the `pickWinner` function to trusted entities only, such as the contract owner or an authorized role. Consider using OpenZeppelin's `AccessControl` or `Ownable` to simplify role management.

3. **Reentrancy Protection:**
   - Although the current contract does not include reentrancy vulnerabilities, future modifications could introduce this risk. Consider adding reentrancy protection mechanisms to safeguard against potential future issues.

4. **Code Quality and Best Practices:**
   - Consider adding comments and documentation to the code for better readability and maintainability.
   - Implement error handling for the `enterRaffle` function to ensure that users cannot accidentally send more Ether than required.

## **Conclusion**

The `BadRNG` contract contains critical vulnerabilities that make it unsafe for use in production. The primary issue lies in the insecure randomness generation, which is easily predictable and can be manipulated by an attacker. Additionally, the lack of access control on the `pickWinner` function further exacerbates the risk of exploitation. To mitigate these risks, it is crucial to implement a secure source of randomness, restrict access to critical functions, and ensure that reentrancy protections are in place. Following the recommendations provided in this audit will significantly improve the security of the contract.

---

**End of Report**
