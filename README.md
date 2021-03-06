# SB home task solution 

## Curve coding task

### Solution discussion, transaction logs and contract address

[go here](/solution_transactions/solution.md) 

### Links to forked and modified Curve.fi repos and tokens generation code.  
It is important to emphasize that current master branches in Curve.fi repos are outdated and do not correctly build so it was necessary to debug and edit existing Curve repos for full meta pool deployment.

Also, brownie used had to be updated to 1.17.2

**->ERC-20 token mocks for DAI, USDC, USDT, sbUSD**

[Token generation contract (mocks for DAI, USDC, USDT, sbUSD tokens used by MetaPool)](https://github.com/gwalen/erc20_token_mock)

**->curve-dao**

[Curve-dao contracts forked and modified repo](https://github.com/gwalen/curve-dao-contracts/tree/sb-task-changes)

[PR to highlight changes](https://github.com/gwalen/curve-dao-contracts/pull/1)

**->curve-contracts (3Pool)**

[Curve base pool (3Pool) contracts forked and modified repo](https://github.com/gwalen/curve-contract/pull/1)

[PR to highlight changes](https://github.com/gwalen/curve-contract/tree/sb-task-changes)

**->curve-factory (meta-pool)**

[Curve meta pool factory contracts forked and modified repo](https://github.com/gwalen/curve-factory/tree/sb-task-changes)

[PR to highlight changes](https://github.com/gwalen/curve-factory/pull/1)


## Open questions

#### 1) If a trade is rerouted to 1inch, how would it affect the liquidity providers in the sbUSD metapool?
If trade is rerouted the liquidity providers on Curve would not be affected. They would not get any LP token rewards as Curve pools would not be used - this is under assumption
that 1Inch would not reroute back to Curve pool. If Uniswap is used it is the same : no rewards for Curve liquidity providers, as Uniswap pools are used
and theirs liquidity providers are affected and rewarded.

#### 2) A common attack pattern in recent exploits has been re-entrancy. Briefly describe how this works
When in a contract function the balance state change happens after the ETH transfer and there is no limit on gas then attacker (contract receiving the ETH) can make recursive call to that function from
his payable(in DAO case it was fallback function) function which receives the ETH and drain the base contract account.

#### 3) As of Solidity 0.6, plain ether transfers to a smart contract will invoke the receive() function. If this is not defined it will look for the fallback() function. If this is also not defined (or not marked as payable), an exception will be raised. Can you think of a way to forcibly send ether to a smart contract even if the two functions are not defined?

````
function sendETHtoContract() public payable {
    //msg.value is the amount of wei that the msg.sender sent with this transaction. 
    //If the transaction doesn't fail, then the contract now has this ETH.
}
````

#### 4) What is the CHSB and what are its key features?
CHSB is a SwissBorg utility token. It is an ERC-20 token. It has a maximum total supply (no new token can be minted) of 1,000,000,000 created during TGE (token generation event).
Utilities to incentivise holders: users can stake tokens and participate in yield program, users get incentivised for engaging with the ecosystem (take part in the referendum),
Buy Back & Burn used to buy back tokens from the market and destroy them which removes tokens from circulating supply (increases the likelihood of the CHSB token price increasing), 
The Community Index. 

