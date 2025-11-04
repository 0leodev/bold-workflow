# Interface

### IZapper.sol

Path `contracts/src/Zappers/Interfaces/IZapper.sol`

GitHub https://github.com/liquity/bold/blob/main/contracts/src/Zappers/Interfaces/IZapper.sol

VSCODE https://vscode.blockscan.com/sepolia/0x7e4a8e4691585c17341c31986cafcf86f0d1ded3

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "./IFlashLoanProvider.sol";
import "./IExchange.sol";

interface IZapper {
    struct OpenTroveParams {
        address owner;
        uint256 ownerIndex;
        uint256 collAmount;
        uint256 boldAmount;
        uint256 upperHint;
        uint256 lowerHint;
        uint256 annualInterestRate;
        address batchManager;
        uint256 maxUpfrontFee;
        address addManager;
        address removeManager;
        address receiver;
    }

    struct CloseTroveParams {
        uint256 troveId;
        uint256 flashLoanAmount;
        uint256 minExpectedCollateral;
        address receiver;
    }

    function flashLoanProvider() external view returns (IFlashLoanProvider);

    function exchange() external view returns (IExchange);

    function openTroveWithRawETH(OpenTroveParams calldata _params) external payable returns (uint256);

    function closeTroveFromCollateral(uint256 _troveId, uint256 _flashLoanAmount, uint256 _minExpectedCollateral)
        external;
}

```

---

# Implementation

### WETHZapper.sol

### Method: openTroveWithRawETH (0xf926c2d2)

Etherscan https://sepolia.etherscan.io/address/0x7e4a8e4691585c17341c31986cafcf86f0d1ded3#code

GitHub https://github.com/liquity/bold/blob/main/contracts/src/Zappers/WETHZapper.sol

VSCODE https://vscode.blockscan.com/sepolia/0x7e4a8e4691585c17341c31986cafcf86f0d1ded3

```solidity
    function openTroveWithRawETH(OpenTroveParams calldata _params) external payable returns (uint256) {
        require(msg.value > ETH_GAS_COMPENSATION, "WZ: Insufficient ETH");
        require(
            _params.batchManager == address(0) || _params.annualInterestRate == 0,
            "WZ: Cannot choose interest if joining a batch"
        );
```

---

# Description

- `owner`: Address that will own the new Trove.
- `ownerIndex`: Index for the owner's Trove, each trove is represented as an NFT. So if an owner has 2 troves already, and if you call the function `balanceOf(owner)` in the TroveNFT contract https://sepolia.etherscan.io/address/0xe9b841c5d2a6a1cc927ee081f1e3bd976416f387#code and returns 2, you'd use `ownerIndex = 2` to open their 3rd trove.
- `collAmount`: Amount of wstETH/rETH to deposit.
- `boldAmount`: Amount of BOLD to borrow.
- `upperHint`: Address hint for efficient insertion in sorted list (gas optimization) - address of Trove with higher collateral ratio than yours.
- `lowerHint`: Address hint for efficient insertion in sorted list (gas optimization) - address of Trove with lower collateral ratio than yours.
- `annualInterestRate`: Annual interest rate for the loan (user-set in v2). 
- `maxUpfrontFee`: When first opening the loan you pay an Upfront Borrowing Fee. This fee is calculated as 7 days’ worth of average interest on the respective collateral branch. By applying this fee, borrowers are discouraged from continually closing and reopening Troves to evade redemptions, as it increases the cost of frequent adjustments.
- `addManager`: The optional _addManager permits that address to improve the collateral ratio of the Trove - i.e. to add collateral or to repay debt
- `removeManager`: The optional _removeManager is permitted to reduce the collateral ratio of the Trove, that is, to remove collateral or draw new debt, and also to close it.
- `receiver`: The _receiver is the address the _removeManager can send funds to.

The `uint256` returned by `openTrove()` is the Trove ID.

The `owner` always retains full control and can withdraw. Setting managers with `addManager` and `removeManager` only grants additional permissions to others.

Only the `owner` or designated `removeManager` can perform **withdrawal operations**.
