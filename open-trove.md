# borrowerOperations.sol

0x2377…596c (ETH branch)

# `openTrove()` interface

Path `contracts/src/Interfaces/IBorrowerOperations.sol`

Etherscan https://sepolia.etherscan.io/address/0x2377b5a07bdfa02812203bab749e7bd43e4c596c#code#F3#L19

GitHub https://github.com/liquity/bold/blob/main/contracts/src/Interfaces/IBorrowerOperations.sol

```solidity
function openTrove(
        address _owner,
        uint256 _ownerIndex,
        uint256 _ETHAmount,
        uint256 _boldAmount,
        uint256 _upperHint,
        uint256 _lowerHint,
        uint256 _annualInterestRate,
        uint256 _maxUpfrontFee,
        address _addManager,
        address _removeManager,
        address _receiver
    ) external returns (uint256);
```

---

# `openTrove()` implementation

Path `contracts/src/BorrowerOperations.sol`

Etherscan https://sepolia.etherscan.io/address/0x2377b5a07bdfa02812203bab749e7bd43e4c596c#code#F1#L1

GitHub https://github.com/liquity/bold/blob/main/contracts/src/BorrowerOperations.sol

```solidity
function openTrove(
    address _owner,
    uint256 _ownerIndex,
    uint256 _collAmount,
    uint256 _boldAmount,
    uint256 _upperHint,
    uint256 _lowerHint,
    uint256 _annualInterestRate,
    uint256 _maxUpfrontFee,
    address _addManager,
    address _removeManager,
    address _receiver
) external override returns (uint256) {
    _requireValidAnnualInterestRate(_annualInterestRate);

    OpenTroveVars memory vars;

    vars.troveId = _openTrove(
        _owner,
        _ownerIndex,
        _collAmount,
        _boldAmount,
        _annualInterestRate,
        address(0),
        0,
        0,
        _maxUpfrontFee,
        _addManager,
        _removeManager,
        _receiver,
        vars.change
    );

    // Set the stored Trove properties and mint the NFT
    troveManager.onOpenTrove(_owner, vars.troveId, vars.change, _annualInterestRate);

    sortedTroves.insert(vars.troveId, _annualInterestRate, _upperHint, _lowerHint);

    return vars.troveId;
}
```

# Description

- `owner`: Address that will own the new Trove.
- `ownerIndex`: Index for the owner's Trove, each trove is represented as an NFT. So if an owner has 2 troves already, and if you call the function `balanceOf(owner)` in the TroveNFT contract https://sepolia.etherscan.io/address/0xe9b841c5d2a6a1cc927ee081f1e3bd976416f387#code and returns 2, you'd use `ownerIndex = 2` to open their 3rd trove.
- `ETHAmount`: Amount of ETH/collateral to deposit.
- `boldAmount`: Amount of BOLD to borrow.
- `upperHint`: Address hint for efficient insertion in sorted list (gas optimization) - address of Trove with higher collateral ratio than yours.
- `lowerHint`: Address hint for efficient insertion in sorted list (gas optimization) - address of Trove with lower collateral ratio than yours.
- `annualInterestRate`: Annual interest rate for the loan (user-set in v2). 
- `maxUpfrontFee`: Maximum upfront fee willing to pay.
- `addManager`: Address granted permissions for operations that add money to the trove (add collateral, pay debt).
- `removeManager`: Address granted permissions for operations that withdraw money from the trove (withdraw collateral, borrow).
- `receiver`: Address to receive the borrowed BOLD tokens, (If no receiver is set, funds go to the `owner`).

The `uint256` returned by `openTrove()` is the Trove ID.

The `owner` always retains full control and can withdraw. Setting managers with `addManager` and `removeManager` only grants additional permissions to others.

Only the `owner` or designated `removeManager` can perform **withdrawal operations**.