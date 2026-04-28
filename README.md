# Vesting-Wallet-with-Cliff-Linear-Release
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract VestingWallet {
    address public beneficiary;
    uint256 public startTime;
    uint256 public cliffDuration;
    uint256 public vestingDuration;
    uint256 public totalAllocation;
    uint256 public released;

    error CliffNotPassed();
    error NoTokensToRelease();

    event TokensReleased(uint256 amount);

    constructor(
        address _beneficiary,
        uint256 _cliffDays,
        uint256 _vestingDays,
        uint256 _totalAllocation
    ) payable {
        beneficiary = _beneficiary;
        startTime = block.timestamp;
        cliffDuration = _cliffDays * 1 days;
        vestingDuration = _vestingDays * 1 days;
        totalAllocation = _totalAllocation;
    }

    function release() public {
        if (msg.sender != beneficiary) revert("Not beneficiary");
        if (block.timestamp < startTime + cliffDuration) revert CliffNotPassed();

        uint256 vested = vestedAmount();
        uint256 unreleased = vested - released;

        if (unreleased == 0) revert NoTokensToRelease();

        released += unreleased;
        payable(beneficiary).transfer(unreleased);
        emit TokensReleased(unreleased);
    }

    function vestedAmount() public view returns (uint256) {
        if (block.timestamp < startTime + cliffDuration) return 0;
        if (block.timestamp >= startTime + cliffDuration + vestingDuration) return totalAllocation;

        uint256 timePassed = block.timestamp - (startTime + cliffDuration);
        return (totalAllocation * timePassed) / vestingDuration;
    }
}
