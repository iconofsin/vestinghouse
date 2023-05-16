- Removed receive() and fallback(); not much point in having them if the currency cannot be extracted.
- Removed releasing of vested tokens on vesting schedule revocation (clawback)
- Added an interface (events split into a separate ancestor interface for easier testing w Foundry)
- Replaced onlyOwner and onlyIfVestingScheduleNotRevoked with internal functions
- Added events
- Some stylistic changes
- Shortened and edited revert messages for clarity/conciseness
- Refactored schedule creation for a more explicit construction 
- Added proper documentation
- Using uint64 instead of uint256 for time-related variables, saving some gas on SSTOREs (around 60k per createVestingSchedule call)

Note, `extend` only works on revocable, non-expired schedules.

`. ./cover.sh` for coverage. 

```
Overall coverage rate:
  lines......: 100.0% (78 of 78 lines)
  functions..: 100.0% (24 of 24 functions)
  branches...: 97.2% (35 of 36 branches)
```

Also see Slither report below.

Note that multiplication on the results of a division is intentional in the original code, it serves to calculate the  number of full vesting periods elapsed.  As for timestamp comparisons, we cannot avoid those without complicating the architecture with commitment schemes which would not be wise under the current schedule.

```
⸻ Starting analysis ⸻
❌ TokenVesting._computeReleasableAmount(VestingSchedule) (contracts/TokenVesting.sol:211-238) performs a multiplication on the result of a division:
	• vestedSlicePeriods = timeFromStart / secondsPerSlice (contracts/TokenVesting.sol#230)
	• vestedSeconds = vestedSlicePeriods * secondsPerSlice (contracts/TokenVesting.sol#231)


❌ TokenVesting._computeReleasableAmount(VestingSchedule) (contracts/TokenVesting.sol:211-238) uses timestamp for comparisons
	• (currentTime < vestingSchedule.cliff) || vestingSchedule.revoked (contracts/TokenVesting.sol#217)
	• currentTime >= vestingSchedule.start + vestingSchedule.duration (contracts/TokenVesting.sol#222)


❌ TokenVesting._vestingScheduleNotExpired(bytes32) (contracts/TokenVesting.sol:265-272) uses timestamp for comparisons
	• require(bool,string)(vestingSchedules[vestingScheduleId].start + vestingSchedules[vestingScheduleId].duration > block.timestamp,Vesting Schedule Expired) (contracts/TokenVesting.sol#266-271)


❌ TokenVesting.release(bytes32,uint256) (contracts/TokenVesting.sol:97-109) uses timestamp for comparisons
	• require(bool,string)(vestedAmount >= amount,Insufficient Vested Balance) (contracts/TokenVesting.sol#103)


❌ Parameter TokenVesting.getVestingSchedulesCountByBeneficiary(address)._beneficiary (contracts/TokenVesting.sol:115) is not in mixedCase


❌ Variable TokenVesting.getVestingSchedulesCountByBeneficiary(address)._beneficiary (contracts/TokenVesting.sol:115) is too similar to ITokenVesting.createVestingSchedule(address,uint64,uint64,uint64,uint64,uint256,bool).beneficiary_ (contracts/ITokenVesting.sol#33)


❌ Variable TokenVesting.getVestingSchedulesCountByBeneficiary(address)._beneficiary (contracts/TokenVesting.sol:115) is too similar to TokenVesting.createVestingSchedule(address,uint64,uint64,uint64,uint64,uint256,bool).beneficiary_ (contracts/TokenVesting.sol#40)


❌ Variable ITokenVesting.getVestingSchedulesCountByBeneficiary(address)._beneficiary (contracts/ITokenVesting.sol:94) is too similar to ITokenVesting.createVestingSchedule(address,uint64,uint64,uint64,uint64,uint256,bool).beneficiary_ (contracts/ITokenVesting.sol#33)


❌ Variable ITokenVesting.getVestingSchedulesCountByBeneficiary(address)._beneficiary (contracts/ITokenVesting.sol:94) is too similar to TokenVesting.createVestingSchedule(address,uint64,uint64,uint64,uint64,uint256,bool).beneficiary_ (contracts/TokenVesting.sol#40)
```

