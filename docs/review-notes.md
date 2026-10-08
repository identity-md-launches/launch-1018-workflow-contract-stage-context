# Implementation check evidence

This is the implementer's report, not the stage's independent source/manifest review.

## Previous rejection addressed

The rejected design exposed a payer argument on `executeSale` and pulled from it. This implementation's only allowance pull encodes `transferFrom(msg.sender, address(this), amount)` in the batch entrypoint's internal helper. The self-only swap helper has no payer parameter and cannot spend allowances. Tests demonstrate that another wallet cannot consume a victim's approval, and that direct helper and idle/forged/repeated callback calls fail.

The seller uses the PoolManager's signed swap deltas and verified settlement payment, not externally observable token balances, to account for each sale. Every hop must spend exactly its specified input; native intermediate credit cancels inside the manager. This removes the balance-snapshot pattern from the previous reentrancy findings. The batch is guarded across all external interactions. Tests attempt reentry during input pull, settlement, swap hooks, IMD take, fee transfer and payout.

Batch totals and claim flags are explicitly initialized. External calls in loops are intentional for batching; claim, pull and swap calls have fixed gas budgets, and refunds use atomic transaction rollback if they cannot complete. Timestamp comparison enforces the caller's deadline; it is not a randomness source.

## Checks performed

- `forge build`, compiler 0.8.26, metadata hash disabled.
- `forge test`: 38 passing tests, including real local v4 core integration, fuzz tests and 4,096 invariant calls with no invariant-handler reverts. The full suite also passed with an empty environment, offline mode and four test threads.
- `forge fmt --check`.
- `forge lint --severity high -- src/LaunchToken.sol src/SwarmHarvester.sol src/SwarmSeller.sol`: no high-severity diagnostics.
- Runtime scan stepping over PUSH operands: no DELEGATECALL, CALLCODE or SELFDESTRUCT in any deployed application/token bytecode; all three below 24,576 bytes. At this build: LaunchToken 1,722 bytes, SwarmHarvester 963 bytes, SwarmSeller 5,955 bytes.
- Optional real-mainnet distributor fork replay: passed against block 26,145,141 through `https://rpc.mevblocker.io`. The first attempted public RPC lacked historical state; the committed test uses the endpoint that completed the check.

Foundry's broader lint still emits generic warnings about batching external calls, deadline timestamps, checked signed conversions, temporary callback authorization state and guard/event ordering around external calls. The explicit call graph and adversarial tests above address the intended behavior; no detector suppression was added. Slither, Mythril and an independent adversarial audit were not run in this assignment.

The fork proof was copied from successful mainnet transaction `0x088f898e9e2789544793bec9e4e69da3ec58b88d004bed1c39664e2f4796ea96` at block 26,145,142, with account `0xc60c81b48bdf107e1651e8ebee971e84885febda`. It is a historical public allocation, not a production configuration value or a stand-in wallet. It is embedded in Solidity so the default tests need neither a proof API nor filesystem reads.

## Independent review and service handoff

Review should verify the exact mainnet seller constructor addresses, absence of wallet spending paths other than the caller pull, callback lifecycle, atomic refunds, gross-versus-net minima, integer fee rounding, full-fill requirement, hook assumptions, unsupported token behavior and inability to recover donations. The raw pull call copies at most one return word; a revert is safely skipped, while a successful false/malformed return aborts the batch because it may already have moved tokens.

Use the canonical service linkage model for policy and signed artifacts. Publishing, attestation, admission, deployment, source verification and frontend hosting are later service operations. This contribution supplies no `launch.json`, policy signature, deployment transaction or frontend deployment address.

Primary protocol references: [Uniswap v4.0.0 IPoolManager](https://github.com/Uniswap/v4-core/blob/v4.0.0/src/interfaces/IPoolManager.sol), [v4.0.0 PoolManager](https://github.com/Uniswap/v4-core/blob/v4.0.0/src/PoolManager.sol), and the licenses preserved with each vendored dependency.
