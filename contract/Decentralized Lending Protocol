// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title Decentralized Lending Protocol
 * @dev A simple decentralized lending platform allowing users to deposit, borrow, and repay loans
 */
contract DecentralizedLendingProtocol {
    
    // State variables
    mapping(address => uint256) public deposits;
    mapping(address => uint256) public loans;
    mapping(address => uint256) public lastBorrowTime;
    
    uint256 public totalDeposits;
    uint256 public totalLoans;
    uint256 public constant INTEREST_RATE = 5; // 5% annual interest rate
    uint256 public constant COLLATERAL_RATIO = 150; // 150% collateralization required
    uint256 public constant SECONDS_PER_YEAR = 365 * 24 * 60 * 60;
    
    // Events
    event Deposit(address indexed user, uint256 amount);
    event Withdraw(address indexed user, uint256 amount);
    event Borrow(address indexed user, uint256 amount);
    event Repay(address indexed user, uint256 amount);
    
    // Modifiers
    modifier hasDeposit() {
        require(deposits[msg.sender] > 0, "No deposits found");
        _;
    }
    
    modifier hasLoan() {
        require(loans[msg.sender] > 0, "No active loan");
        _;
    }
    
    /**
     * @dev Core Function 1: Deposit ETH to earn interest and use as collateral
     */
    function deposit() external payable {
        require(msg.value > 0, "Deposit amount must be greater than 0");
        
        deposits[msg.sender] += msg.value;
        totalDeposits += msg.value;
        
        emit Deposit(msg.sender, msg.value);
    }
    
    /**
     * @dev Core Function 2: Borrow against deposited collateral
     * @param amount The amount to borrow in wei
     */
    function borrow(uint256 amount) external hasDeposit {
        require(amount > 0, "Borrow amount must be greater than 0");
        
        // Calculate maximum borrowable amount (considering existing loans)
        uint256 maxBorrow = (deposits[msg.sender] * 100) / COLLATERAL_RATIO;
        uint256 currentDebt = getCurrentDebt(msg.sender);
        
        require(currentDebt + amount <= maxBorrow, "Insufficient collateral");
        require(address(this).balance >= amount, "Insufficient liquidity");
        
        // Update loan details
        if (loans[msg.sender] > 0) {
            loans[msg.sender] = currentDebt; // Update with accrued interest
        }
        
        loans[msg.sender] += amount;
        lastBorrowTime[msg.sender] = block.timestamp;
        totalLoans += amount;
        
        // Transfer borrowed amount to user
        payable(msg.sender).transfer(amount);
        
        emit Borrow(msg.sender, amount);
    }
    
    /**
     * @dev Core Function 3: Repay borrowed amount with interest
     */
    function repay() external payable hasLoan {
        require(msg.value > 0, "Repayment amount must be greater than 0");
        
        uint256 currentDebt = getCurrentDebt(msg.sender);
        require(msg.value <= currentDebt, "Repayment exceeds debt");
        
        loans[msg.sender] = currentDebt - msg.value;
        totalLoans -= msg.value;
        
        // If loan is fully repaid, reset timestamp
        if (loans[msg.sender] == 0) {
            lastBorrowTime[msg.sender] = 0;
        } else {
            lastBorrowTime[msg.sender] = block.timestamp;
        }
        
        emit Repay(msg.sender, msg.value);
    }
    
    /**
     * @dev Withdraw deposited funds (only if no active loans or sufficient collateral remains)
     * @param amount The amount to withdraw in wei
     */
    function withdraw(uint256 amount) external hasDeposit {
        require(amount > 0, "Withdrawal amount must be greater than 0");
        require(deposits[msg.sender] >= amount, "Insufficient deposit balance");
        
        // Check if withdrawal maintains required collateral ratio
        if (loans[msg.sender] > 0) {
            uint256 remainingDeposit = deposits[msg.sender] - amount;
            uint256 currentDebt = getCurrentDebt(msg.sender);
            uint256 requiredCollateral = (currentDebt * COLLATERAL_RATIO) / 100;
            
            require(remainingDeposit >= requiredCollateral, "Withdrawal would violate collateral ratio");
        }
        
        deposits[msg.sender] -= amount;
        totalDeposits -= amount;
        
        payable(msg.sender).transfer(amount);
        
        emit Withdraw(msg.sender, amount);
    }
    
    /**
     * @dev Calculate current debt including accrued interest
     * @param user The address of the borrower
     * @return The current debt amount including interest
     */
    function getCurrentDebt(address user) public view returns (uint256) {
        if (loans[user] == 0 || lastBorrowTime[user] == 0) {
            return loans[user];
        }
        
        uint256 timeElapsed = block.timestamp - lastBorrowTime[user];
        uint256 interest = (loans[user] * INTEREST_RATE * timeElapsed) / (100 * SECONDS_PER_YEAR);
        
        return loans[user] + interest;
    }
    
    /**
     * @dev Get user's borrowing capacity
     * @param user The address to check
     * @return The maximum amount the user can borrow
     */
    function getBorrowingCapacity(address user) external view returns (uint256) {
        if (deposits[user] == 0) return 0;
        
        uint256 maxBorrow = (deposits[user] * 100) / COLLATERAL_RATIO;
        uint256 currentDebt = getCurrentDebt(user);
        
        return maxBorrow > currentDebt ? maxBorrow - currentDebt : 0;
    }
    
    /**
     * @dev Get contract's total liquidity available for borrowing
     * @return Available ETH balance for lending
     */
    function getAvailableLiquidity() external view returns (uint256) {
        return address(this).balance;
    }
    
    /**
     * @dev Get user's account summary
     * @param user The address to query
     * @return depositAmount The user's total deposits
     * @return loanAmount The user's current debt including interest
     * @return borrowCapacity The additional amount user can borrow
     */
    function getAccountSummary(address user) external view returns (
        uint256 depositAmount,
        uint256 loanAmount,
        uint256 borrowCapacity
    ) {
        depositAmount = deposits[user];
        loanAmount = getCurrentDebt(user);
        
        if (depositAmount > 0) {
            uint256 maxBorrow = (depositAmount * 100) / COLLATERAL_RATIO;
            borrowCapacity = maxBorrow > loanAmount ? maxBorrow - loanAmount : 0;
        }
    }
    
    /**
     * @dev Emergency function to check contract health
     * @return Utilization ratio of the protocol
     */
    function getUtilizationRatio() external view returns (uint256) {
        if (totalDeposits == 0) return 0;
        return (totalLoans * 100) / totalDeposits;
    }
}
