# Foundry DAO

A complete Decentralized Autonomous Organization (DAO) implementation built with Foundry and OpenZeppelin's governance contracts. This project demonstrates a fully functional on-chain governance system with proposal creation, voting, and timelock execution.

## Overview

This DAO allows token holders to propose and vote on changes to smart contracts. The implementation includes:

- **Governance Token (GovToken)**: ERC20 token with voting capabilities
- **Governor Contract (MyGovernor)**: Manages proposals, voting, and execution
- **Timelock Controller (TimeLock)**: Enforces a delay before executing approved proposals
- **Box Contract**: Example target contract controlled by the DAO

## Contracts

### GovToken
- ERC20 token with voting and delegation features
- Implements `ERC20Votes` for on-chain governance
- Supports gas-efficient off-chain signatures via `ERC20Permit`
- Token holders must delegate voting power (to themselves or others) to participate

### MyGovernor
- Voting delay: **7200 blocks** (~1 day)
- Voting period: **50400 blocks** (~1 week)
- Quorum: **4%** of total token supply
- Proposal threshold: **0 tokens** (anyone can propose)

### TimeLock
- Minimum delay: **3600 seconds** (1 hour)
- Adds a security buffer between proposal approval and execution
- Allows time for stakeholders to react to approved proposals

### Box
- Simple storage contract used for testing governance
- Ownership transferred to the TimeLock contract
- Can only be modified through successful governance proposals

## Governance Flow

1. **Propose**: A token holder creates a proposal with target contract(s), value(s), and calldata
2. **Delay**: Voting begins after the voting delay period (7200 blocks)
3. **Vote**: Token holders vote FOR, AGAINST, or ABSTAIN during the voting period (50400 blocks)
4. **Queue**: If quorum is reached and the proposal passes, it's queued in the timelock
5. **Execute**: After the timelock delay (3600 seconds), anyone can execute the proposal

## Installation

```bash
# Clone the repository
git clone <repository-url>
cd foundry-dao

# Install dependencies
forge install
```

## Dependencies

- [Foundry](https://book.getfoundry.sh/)
- [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts)
- [Forge Standard Library](https://github.com/foundry-rs/forge-std)

## Testing

Run all tests:
```bash
forge test
```

Run tests with detailed traces:
```bash
forge test -vvvv
```

Run specific test:
```bash
forge test --match-test testGovernanceUpdatesBox
```

## Test Coverage

The test suite includes:

- ✅ **testCantUpdateBoxWithoutGovernance**: Ensures Box can only be modified through governance
- ✅ **testGovernanceUpdatesBox**: Full governance lifecycle test (propose → vote → queue → execute)

## Usage Example

The test suite demonstrates a complete governance workflow:

```solidity
// 1. Create a proposal to update the Box contract
bytes memory encodedFunctionCall = abi.encodeWithSignature("store(uint256)", 888);
uint256 proposalId = governor.propose(
    targets,      // [address(box)]
    values,       // [0]
    calldatas,    // [encodedFunctionCall]
    description   // "store 888 in box"
);

// 2. Wait for voting delay to pass
vm.warp(block.timestamp + VOTING_DELAY + 1);
vm.roll(block.number + VOTING_DELAY + 1);

// 3. Cast vote
governor.castVoteWithReason(proposalId, 1, "I approve this proposal");

// 4. Wait for voting period to end
vm.warp(block.timestamp + VOTING_PERIOD + 1);
vm.roll(block.number + VOTING_PERIOD + 1);

// 5. Queue the proposal
bytes32 descriptionHash = keccak256(abi.encodePacked(description));
governor.queue(targets, values, calldatas, descriptionHash);

// 6. Wait for timelock delay
vm.warp(block.timestamp + MIN_DELAY + 1);
vm.roll(block.number + MIN_DELAY + 1);

// 7. Execute the proposal
governor.execute(targets, values, calldatas, descriptionHash);
```

## Deployment

1. Deploy the GovToken contract
2. Mint tokens and delegate voting power
3. Deploy the TimeLock contract
4. Deploy the MyGovernor contract
5. Grant roles to the Governor and configure TimeLock
6. Deploy target contracts (e.g., Box) and transfer ownership to TimeLock

## Security Considerations

- Voting power must be delegated before it can be used
- Proposals have a delay before voting starts to prevent flash loan attacks
- Timelock delay provides a safety window after proposal approval
- The admin role should be revoked from the deployer after setup
- Anyone can execute queued proposals after the delay period

## Built With

- [Solidity](https://soliditylang.org/) - Smart contract language
- [Foundry](https://book.getfoundry.sh/) - Development framework
- [OpenZeppelin](https://www.openzeppelin.com/contracts) - Secure contract libraries

## License

MIT
