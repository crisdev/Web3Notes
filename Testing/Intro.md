There are a couple of different testing methods supported by Forge:

- Fuzz Testing
- Invariant Testing
- Differential Testing

To be added: (as of 28/10/2023)
- Symbolic Execution
	Dicen que no tienen esto pero Patrick lo usa supuestamente en su video [7:41:00](https://www.youtube.com/watch?v=wUjYK5gwNZs&t=2s&ab_channel=PatrickCollins&t=27700) y lo que implementa esta documentado aca: [Symbolic Execution ~? Model Checker](https://book.getfoundry.sh/reference/config/solidity-compiler#model-checker). Entonces se confundio, es model checker
- Mutation Testing

## Tools

- Static Analyzer
	- [Slither](https://github.com/crytic/slither)
	- Solc
- Contract Fuzzer y unas boludeces mas
	Tiene un workshop ahi dentro
	https://github.com/crytic/echidna
- [Solidity Metrics](https://github.com/Consensys/solidity-metrics)
	Metricas de codigo
- Unix command line: cloc for outputting files with their lines # 

## Fuzz, Invariant and Diferrential Tests

All three of these are supported in Forge

### Articles

[Foundry - Advanced Testing](https://book.getfoundry.sh/forge/advanced-testing)

[Patrick's Medium](https://patrickalphac.medium.com/fuzz-invariant-tests-the-new-bare-minimum-for-smart-contract-security-87ebe150e88c)

[Mirror.xyz - Invariant Testing with Foundry](https://mirror.xyz/horsefacts.eth/Jex2YVaO65dda6zEyfM_-DXlXhOWCAoSpOx5PLocYgw)

[Fuzz & Invariant Tests | The secret to finding CRITICAL vulnerabilities faster](https://www.youtube.com/watch?v=juyY-CTolac&ab_channel=PatrickCollins)

Base Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MyContract {  
	uint256 public shouldAlwaysBeZero = 0;
	uint256 private hiddenValue = 0;
	
	function doStuff(uint256 data) public {
		if (data == 2) {
			shouldAlwaysBeZero = 1;
		}
		if (hiddenValue == 7) {
			shouldAlwaysBeZero = 1;
		}
		hiddenValue = data;
	}
}
```

Stateless Fuzzing
```solidity
function testIsAlwaysZeroFuzz(uint256 randomData) public {
	exampleContract.doStuff(randomData);
	assert(exampleContract.shouldAlwaysBeZero() == 0);
}
```


Stateful Fuzzing / Invariant 
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
  
import { MyContract } from "../src/MyContract.sol";
import { Test } from "forge-std/Test.sol";
import { StdInvariant } from "forge-std/StdInvariant.sol";
  
contract MyContractTest is StdInvariant, Test {
	MyContract exampleContract;
	  
	function setUp() public {
		exampleContract = new MyContract();
		targetContract(address(exampleContract));
	}
	
	
	function invariant_testAlwaysReturnsZero() public {
		assert(exampleContract.shouldAlwaysBeZero() == 0);
	}
 }
```


"Now, important aside on how Foundry uses the term **invariant**. As we’ve described, an invariant is a property of the system that must always hold, but Foundry uses the term to mean “stateful fuzzing.” Just keep this in mind.

- Foundry Invariant Tests == Stateful Fuzzing
- Foundry Fuzz Tests == Stateless Fuzzing
- Invariants == Property of the system that must always hold

So in an actual smart contract, your invariant won’t be that a balloon shouldn’t pop or some function should always be zero; it’ll be something like:

- New tokens minted < inflation rate
- There should only be 1 winner of a random lottery
- Someone shouldn’t be able to take more money out of the protocol than they put in"

## Formal Verification 
- Symbolic Execution
	[Symbolic Execution kind of explainer](https://twitter.com/palinatolmach/status/1653030270684270592)
- Abstract Interpretation
- Model Checking

Manticore

hevm
Solc
```solidity
// SPDX-License-identifier: MIT

pragma solidity ^0.8.16;

contract SmallSol {
	function f(uint256 a) public returns (uint256) {
		a = a + 1;
		return a;
	}
}
```

```bash
solc --model-checker-engine chc --model-checker-targets overflow \\ SmallSol.sol
```
