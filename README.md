# NidhiDAO Smart Contracts


##  🔧 Setting up Local Development
Required: 
- [Node v14](https://nodejs.org/download/release/latest-v14.x/)  
- [Git](https://git-scm.com/downloads)


Local Setup Steps:
1. git clone https://github.com/NidhiDAO/NidhiDAO-contracts.git
1. Install dependencies: `npm install` 
    - Installs [Hardhat](https://hardhat.org/getting-started/) & [OpenZepplin](https://docs.openzeppelin.com/contracts/4.x/) dependencies
1. Compile Solidity: `npm run compile`
1. **_TODO_**: How to do local deployments of the contracts.


## 🤨 How it all works
##![High Level Contract Interactions](./docs/box-diagram.png)

## Mainnet Contracts & Addresses

|Contract       | Addresss                                                                                                            | Notes   |
|:-------------:|:-------------------------------------------------------------------------------------------------------------------:|-------|
|GURU            |[<guruAddress>](https://polygonscan.io/address/<guruAddress>)| Main Token Contract|
|sGURU           |[<sGuruAddress>](https://polygonscan.io/address/<sGuruAddress>)| Staked GURU|
|Treasury       |[<treasuryAddress>](https://polygonscan.io/address/<treasuryAddress>)| Nidhi Treasury holds all the assets        |
|NidhiStaking |[<nidhiStaking>](https://polygonscan.io/address/<nidhiStaking>)| Main Staking contract responsible for calling rebases every 2200 blocks|
|StakingHelper  |[<stakingHelper>](https://polygonscan.io/address/<stakingHelper>)| Helper Contract to Stake with 0 warmup |
|DAO            |[<daoAddress>](https://polygonscan.io/address/<daoAddress>)|Storage Wallet for DAO under MS |
|Staking Warm Up|[<stakingWarmup>](https://polygonscan.io/address/<stakingWarmup>)| Instructs the Staking contract when a user can claim sGURU |


**Bonds**
- **_TODO_**: What are the requirements for creating a Bond Contract?
All LP bonds use the Bonding Calculator contract which is used to compute RFV. 

|Contract       | Addresss                                                                                                            | Notes   |
|:-------------:|:-------------------------------------------------------------------------------------------------------------------:|-------|
|Bond Calculator|[<bondCalculator>](https://polygonscan.io/address/<bondCalculator>)| |
|DAI bond|[<daiBond>](https://polygonscan.io/address/<daiBond>)| Main bond managing serve mechanics for GURU/DAI|
|DAI/GURU SLP Bond|[<daiGuruSLPBond>](https://polygonscan.io/address/<daiGuruSLPBond>)| Manages mechanism for the protocol to buy back its own liquidity from the pair. |


### Testnet Addresses

Network: `Mumbai` (4)
- GURU: `tbd`
- DAI: `tbd` 
- Treasury: `tbd`
- GURU/DAI Pair: `tbd`
- Calc: `tbd` 
- Staking: `tbd` 
- sGURU: `tbd` 
- Distributor `tbd` 
- Staking Warmup `tbd` 
- Staking Helper `tbd`


**Managing**:
The first step is withdraw funds from the treasury via the "manage" function. "Manage" allows an approved address to withdraw excess reserves from the treasury.

*Note*: This contract must have the "reserve manager" permission, and that withdrawn reserves decrease the treasury's ability to mint new GURU (since backing has been removed).

Pass in the token address and the amount to manage. The token will be sent to the contract calling the function.

```
function manage( address _token, uint _amount ) external;
```

Managing treasury assets should look something like this:
```
treasury.manage( DAI, amountToManage );
```

**Returning**:
The second step is to return funds after the strategy has been closed.
We utilize the `deposit` function to do this. Deposit allows an approved contract to deposit reserve assets into the treasury, and mint GURU against them. In this case however, we will NOT mint any GURU. This will be explained shortly.

*Note* The contract must have the "reserve depositor" permission, and that deposited reserves increase the treasury's ability to mint new GURU (since backing has been added).


Pass in the address sending the funds (most likely the allocator contract), the amount to deposit, and the address of the token. The final parameter, profit, dictates how much GURU to send. send_, the amount of GURU to send, equals the value of amount minus profit.
```
function deposit( address _from, uint _amount, address _token, uint _profit ) external returns ( uint send_ );
```

To ensure no GURU is minted, we first get the value of the asset, and pass that in as profit.
Pass in the token address and amount to get the treasury value.
```
function valueOf( address _token, uint _amount ) public view returns ( uint value_ );
```

All together, returning funds should look something like this:
```
treasury.deposit( address(this), amountToReturn, DAI, treasury.valueOf( DAI, amountToReturn ) );
```
