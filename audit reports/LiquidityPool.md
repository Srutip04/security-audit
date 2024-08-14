
# **Security Audit Report**

---

**Project Name:** LiquidityPoolAsOracle Smart Contract  
**Audit Date:** August 14, 2024  
**Auditor:** Shruti

---

## **Executive Summary**

The `LiquidityPoolAsOracle` smart contract allows users to swap between two ERC20 tokens using a liquidity pool as a price oracle. However, using a liquidity pool as an oracle introduces significant risks, particularly the vulnerability to flash loan attacks. This audit identifies critical security vulnerabilities related to price manipulation and offers recommendations to mitigate these risks.

## **Scope**

The audit covers the following aspects of the `LiquidityPoolAsOracle` smart contract:
1. **Price Oracle Vulnerability**
2. **Flash Loan Attack Surface**
3. **Access Control**
4. **Reentrancy**
5. **Best Practices and Code Quality**

## **Findings**

### **1. Price Oracle Vulnerability**
   - **Severity:** Critical
   - **Description:** The contract relies on the liquidity pool's balances to calculate the price for token swaps:
     ```solidity
     function getSwapPrice(
         address from,
         address to,
         uint256 amount
     ) public view returns (uint256) {
         return ((amount * IERC20(to).balanceOf(address(this))) /
             IERC20(from).balanceOf(address(this)));
     }
     ```
     This pricing mechanism is based solely on the token balances held by the contract. An attacker can manipulate the balances within a single transaction, causing the price calculation to become inaccurate and allowing them to profit from the manipulation.
   - **Impact:** An attacker could use a flash loan to borrow a large amount of one token, swap it for the other token at a favorable rate (caused by the temporary imbalance), and then repay the loan, profiting from the difference. This could result in significant losses for the liquidity pool.
   - **Recommendation:** Do not use liquidity pool balances as a price oracle. Instead, integrate a decentralized oracle network like Chainlink Data Feeds, which provides tamper-resistant and accurate price data.

### **2. Flash Loan Attack Surface**
   - **Severity:** Critical
   - **Description:** The contract is vulnerable to flash loan attacks due to its reliance on current token balances to determine the swap price. A flash loan allows an attacker to borrow large amounts of tokens without collateral, manipulate the contract’s pricing logic, and return the loan in the same transaction, all while profiting from the temporary price discrepancy.
   - **Impact:** An attacker could drain the liquidity pool by repeatedly exploiting the price discrepancies caused by flash loans, leading to significant financial loss for the contract’s liquidity providers.
   - **Recommendation:** Implement mechanisms to prevent flash loan attacks, such as using time-weighted average prices (TWAP) or integrating a decentralized oracle that cannot be manipulated within a single transaction.

### **3. Lack of Access Control on `addLiquidity`**
   - **Severity:** Medium
   - **Description:** The `addLiquidity` function allows any user to add liquidity to the pool:
     ```solidity
     function addLiquidity(address tokenAddress, uint256 amount) external {
         bool success = IERC20(tokenAddress).transferFrom(msg.sender, address(this), amount);
         require(success, "Failed to add liquidity");
     }
     ```
     While this is typical behavior for liquidity pools, without proper safeguards, it could lead to potential manipulation or abuse.
   - **Impact:** Malicious users might add small amounts of liquidity to manipulate token balances or to prepare for a flash loan attack.
   - **Recommendation:** Implement access controls or sanity checks to ensure that only legitimate liquidity providers can add liquidity. For example, setting a minimum liquidity amount or verifying the source of the liquidity.

### **4. Reentrancy Risk**
   - **Severity:** Low
   - **Description:** The contract performs external token transfers, which can introduce reentrancy vulnerabilities:
     ```solidity
     bool txFromSuccess = IERC20(from).transferFrom(msg.sender, address(this), amount);
     require(txFromSuccess, "Failed to transfer from");
     bool txToSuccess = IERC20(to).transfer(msg.sender, swap_amount);
     require(txToSuccess, "Failed to transfer to");
     ```
     While the current implementation does not have any state changes after these transfers, future modifications could introduce reentrancy vulnerabilities if the contract’s logic changes.
   - **Impact:** Potential future reentrancy vulnerabilities could allow attackers to manipulate the contract’s state or drain funds.
   - **Recommendation:** Consider adding reentrancy guards using OpenZeppelin’s `ReentrancyGuard` or following the checks-effects-interactions pattern to protect against future vulnerabilities.

### **5. Code Quality and Best Practices**
   - **Severity:** Low
   - **Description:** The contract lacks detailed comments and documentation, which could lead to misunderstandings or errors during future maintenance.
   - **Impact:** Poor code documentation can lead to bugs and make it difficult for developers to understand and maintain the contract.
   - **Recommendation:** Add comments and documentation to explain the logic and purpose of each function. Ensure that variable names are descriptive and follow best practices for code readability.

## **Exploit Scenario**

### **Flash Loan Attack Example**
1. **Step 1:** The attacker takes out a flash loan to borrow a large amount of `Token1`.
2. **Step 2:** The attacker swaps the borrowed `Token1` for `Token2` using the vulnerable contract, manipulating the contract’s token balances.
3. **Step 3:** The contract calculates the swap price based on the temporarily inflated `Token1` balance and deflated `Token2` balance, resulting in an artificially favorable rate for the attacker.
4. **Step 4:** The attacker swaps back a portion of `Token2` for `Token1` at a now advantageous rate, drains the liquidity pool, and repays the flash loan.
5. **Step 5:** The attacker profits from the difference, while the liquidity pool suffers a loss.

## **Recommendations**

1. **Integrate Secure Price Oracles:**
   - Replace the current price calculation mechanism with a decentralized oracle network like Chainlink Data Feeds to ensure accurate and tamper-resistant pricing data.

2. **Mitigate Flash Loan Attacks:**
   - Implement time-weighted average prices (TWAP) or similar mechanisms to prevent price manipulation within a single transaction.
   - Consider adding a delay or requiring multiple transactions to finalize a swap, making flash loan attacks less feasible.

3. **Add Access Controls:**
   - Implement access control mechanisms for adding liquidity or set minimum liquidity thresholds to prevent abuse.

4. **Enhance Reentrancy Protection:**
   - Use OpenZeppelin’s `ReentrancyGuard` or the checks-effects-interactions pattern to protect against potential future reentrancy vulnerabilities.

5. **Improve Code Documentation:**
   - Add comprehensive comments and documentation to improve code readability and maintainability.

## **Conclusion**

The `LiquidityPoolAsOracle` contract is vulnerable to critical security issues, particularly related to price manipulation and flash loan attacks. These vulnerabilities could lead to significant financial losses for users and liquidity providers. It is essential to replace the current oracle mechanism with a secure, decentralized price feed and implement safeguards against flash loan attacks. Additionally, improvements in access control, reentrancy protection, and code documentation are recommended to enhance the contract’s overall security and maintainability.

---

**End of Report**
```
