// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract FlashUSDT is ERC20 {

    address public owner;

    constructor() ERC20("Mock Flash USDT", "fUSDT") {
        owner = msg.sender;
    }

    // Mint function - only owner can mint (for testing)
    function mint(address to, uint256 amount) external {
        require(msg.sender == owner, "Not authorized");
        _mint(to, amount);
    }

    // Burn function - optional
    function burn(uint256 amount) external {
        _burn(msg.sender, amount);
    }

    // Flash Mint function (very basic)
    function flashMint(uint256 amount) external {
        uint256 balanceBefore = balanceOf(address(this));
        _mint(msg.sender, amount);

        // Call borrower logic (must repay in same tx)
        IFlashBorrower(msg.sender).executeOnFlashLoan(amount);

        require(
            balanceOf(address(this)) >= balanceBefore,
            "Flash loan not repaid"
        );
    }
}

interface IFlashBorrower {
    function executeOnFlashLoan(uint256 amount) external;
}

