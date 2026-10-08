# ABI handoff

The adjacent `LaunchToken.json`, `SwarmHarvester.json` and `SwarmSeller.json` are plain compiler-generated ABI arrays, suitable for ethers or viem. They include tuple components, errors and events. They contain no deployment addresses; obtain those from the deployment handoff.

Regenerate after source changes with Solidity 0.8.26:

```sh
forge build
forge inspect LaunchToken abi --json > docs/abi/LaunchToken.json
forge inspect SwarmHarvester abi --json > docs/abi/SwarmHarvester.json
forge inspect SwarmSeller abi --json > docs/abi/SwarmSeller.json
```

`Sale.minOut` is gross IMD. `sellMany.minImdOut` and its return value are net IMD after the single aggregate fee. All amounts use the corresponding token's minor units. `deadline` is a Unix timestamp in seconds. `executeSale` and `unlockCallback` are protocol plumbing and must not be exposed as wallet actions. The only user write actions are token `approve`/ERC20 transfers, harvester `claimMany`, and seller `sellMany`.
