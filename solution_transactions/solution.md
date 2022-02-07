# Solution transactions and their generation

Contracts were deployed to Rinkeby testnet using brownie cli and Alchemy nodes.

## Tokens used in Meta-pool deployment

-> go to tokens dir

`yarn hardhat run scripts/deploy.js --network rinkebyalchemy`

```
Deploying contracts with the account: 0x1ff6aa202687612137c88D47369671A7654c0c56
DAI   contract address: 0xF86176aF4687a9E65177913Ebe0A333D79E19fF4, supply: 100000000000000000000000
USDC  contract address: 0x28D0C916Df5bDBB6636b90E25A97363d009eF7e0, supply: 100000000000
USDT  contract address: 0x9BcB3F98236eE1eFB9455637Fa69E2BE27963725, supply: 100000000000
sbUSD contract address: 0xFe8cD5c88B3e0135b83d914C75dA211eCA13B1E5, supply: 100000000000000000000000
```
DAI and sbUSD have 18 decimals. USDC and USDT have 6 decimals.

## Curve dao contracts deployment

-> go to curve dao dir

`brownie run deployment/deploy_dao development --network rinkebyalchemy`

```
Running 'scripts/deployment/deploy_dao.py::development'...
  (...)
  ERC20CRV deployed at: 0xD87FAaC4eE2954a1900A0EEE3846640257C5D3aB
  VotingEscrow deployed at: 0x5af132d964D78b205904234564EB5E4a83F96272
  GaugeController deployed at: 0x6BaA06a8b785C868f45eDc435D405C1EFe0eaaCb
  PoolProxy deployed at: 0x5640568e9c925d71E62d95A6B69a5bf48f2B0D4A
  Minter deployed at: 0x209d2b2e6F5EA6f9A6392E9e84C3bDde56E4D803
  FeeDistributor deployed at: 0xB681Cb29c440Ac3E92169F00Aac93c868CBbe812
  LiquidityGauge deployed at: 0xB10e4Ec05fDcAd9700d28B0063a69634802124f6
  (...)
Deployment complete! Total gas used: 13114289  
```

## Curve dao base pool deployment and config

-> go to 3pool dir

`brownie run deploy --network rinkebyalchemy -I`

```
Running 'scripts/deploy.py::main'...
  (...)
  CurveTokenV2 deployed at: 0x9B19C4CA339DA7fd81FCb2F6eA55062107b8966b
  StableSwap3Pool deployed at: 0xAa8684e82B496423559587e39986E0f85D988952
  LiquidityGaugeV3 deployed at: 0xc864afaC0c7882Bd8Db55385FAC5bFBe82bf4E8e
  (...)
Gas used in deployment: 0.0271 ETH  
```
Configuration and checks:
```
-> Approve base pool to use tokens
>>> dai = ERC20Mock.at('0xF86176aF4687a9E65177913Ebe0A333D79E19fF4')
>>> dai.approve('0xAa8684e82B496423559587e39986E0f85D988952', 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> usdc = ERC20Mock.at('0x28D0C916Df5bDBB6636b90E25A97363d009eF7e0')
>>> usdc.approve('0xAa8684e82B496423559587e39986E0f85D988952', 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> usdt = ERC20Mock.at('0x9BcB3F98236eE1eFB9455637Fa69E2BE27963725')
>>> usdt.approve('0xAa8684e82B496423559587e39986E0f85D988952', 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> crv3 = CurveTokenV2.at('0x9B19C4CA339DA7fd81FCb2F6eA55062107b8966b')
>>> crv3.approve('0xAa8684e82B496423559587e39986E0f85D988952', 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})

#---> init pool with creating initial pool balances (otherwise you will get in D0 = 0 and division by 0 error)
#-> allow my self to send tokens
>>> dai.approve(ADMIN, 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> usdc.approve(ADMIN, 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> usdt.approve(ADMIN, 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> sbUsd.approve(ADMIN, 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> crv3.approve(ADMIN, 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})

#-> send DAI, USDCm, USDT to admin account (which is basally the 3Pool contract account)  -- here we send 10 000 of each coin
>>> dai.transferFrom(ADMIN, '0xAa8684e82B496423559587e39986E0f85D988952', 10000*10**18, {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> usdc.transferFrom(ADMIN, '0xAa8684e82B496423559587e39986E0f85D988952', 10000*10**6, {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> usdt.transferFrom(ADMIN, '0xAa8684e82B496423559587e39986E0f85D988952', 10000*10**6, {'from': ADMIN, 'priority_fee': '3 gwei'})

#-> assign admin base tokens (DAI, USDCm, UDST) to be the pool tokens (there is no funds transfer just assign that ERC20 pool(admin) tokens are also internal 3pool contract tokens)
# it must be done otherwise there will be a divins by zero (and you can not even add liquidity)
>>> pool = StableSwap3Pool.at('0xAa8684e82B496423559587e39986E0f85D988952')
>>> pool.donate_admin_fees()

#--> add liquidity function test
#-> add 3 dai, 3 usdc and 3 usdt to pool form transaction sender account
>>> pool.add_liquidity([3000000000000000000, 3000000, 3000000], 0, {'from': ADMIN, 'priority_fee': '3 gwei'})
{
  Transaction sent: 0x0dc7f88fe01f588ae499b4d2593da0c6d3500bc2b5329c836508e1d902d6ab42
  StableSwap3Pool.add_liquidity confirmed   Block: 10122972   Gas used: 179373 (80.94%)   Gas price: 3.00000001 gwei
}

#--> make exchange test on 3Pool
#-> perform exchange 1 usdc to 1 usdt
>>> expected = pool.get_dy(1, 2, 1*10**6) * 0.99 # = 989604.0
>>> pool.exchange(1, 2, 1*10**6, expected, {'from': ADMIN, 'priority_fee': '3 gwei'})
{
  Transaction sent: 0x397c91eabc25f8b2a9313b77db254027deece43a423c61baaff2588c8a9a4f7e
  StableSwap3Pool.exchange confirmed   Block: 10122990   Gas used: 108439 (75.52%)   Gas price: 3.00000001 gwei
}

```

ADMIN is deployer and owner account

## Meta-pool deployment

-> go to metapool dir

`brownie run deploy --network rinkebyalchemy -I`

```
Running 'scripts/deploy.py::main'...
  Factory deployed at: 0x2c3CfDed0C08F399b6C51910f38AadF0A47191cf         #Gas used: 2861190
  MetaUSDBalances deployed at: 0xf3646529c010b8B6f364D7AB661647ff600FB288 #Gas used: 5339051
  OwnerProxy deployed at: 0xF5067Fde1878eEb578388B9dA52f0d67C4A40e62      #Gas used: 1022108
  DepositZapUSD deployed at: 0x8780f4341e0e4f869Ab246a0A894d712be2BCB5F   #Gas used: 1355519
```
Configuration and checks:
```
#--> approve spending by Uniswap router (on rinkeby) as fallback on high slippage :
>>> dai.approve('0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D', 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> usdc.approve('0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D', 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> usdt.approve('0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D', 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> sbUsd.approve('0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D', 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})

#-> create Meta-pool
>>> base3PoolAddr = '0xAa8684e82B496423559587e39986E0f85D988952'
>>> sbUsdAddr = '0xFe8cD5c88B3e0135b83d914C75dA211eCA13B1E5'
>>> factory.deploy_metapool(base3PoolAddr, "sbUsd Metapool", "sbUsd", sbUsdAddr, 10, 4000000, {'from': ADMIN, 'priority_fee': '3 gwei'})
{
  Transaction sent: 0xbfa7ffca0f44ff5c6173eab5be60c21c87bae6ed739d0cd4880af6ed24a40207
  Factory.deploy_metapool confirmed   Block: 10128257   Gas used: 855145 (90.91%)   Gas price: 3.00000001 gwei
}
#-> get the addres of deplyed metapool
>>> factory.pool_list(0)
 0xf3646529c010b8B6f364D7AB661647ff600FB288

#------> check Metapool
>>> meta = MetaUSDBalances.at('0xf3646529c010b8B6f364D7AB661647ff600FB288')

#-> allow transfers for metaPool
>>> sbUsd.approve('0xf3646529c010b8B6f364D7AB661647ff600FB288', 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> crv3.approve('0xf3646529c010b8B6f364D7AB661647ff600FB288', 20000000000000000000000000,  {'from': ADMIN, 'priority_fee': '3 gwei'})

#-----> init initial balances for metapool for sbUSD and crv3 (they can not be 0) - otherwise it will be division by 0
>>> sbUsd.transferFrom(ADMIN, '0xf3646529c010b8B6f364D7AB661647ff600FB288', 10000*10**18, {'from': ADMIN, 'priority_fee': '3 gwei'})
>>> crv3.transferFrom(ADMIN, '0xf3646529c010b8B6f364D7AB661647ff600FB288', 10000*10**18, {'from': ADMIN, 'priority_fee': '3 gwei'})

#-> allow transfers for metaPool when it will fallback to using uniswap
(...)

#-> test add liquidity  (add 10 sbUsd and 10 LP to the pool)
>>> meta.add_liquidity([10*10**18, 10*10**18], 0, {'from': ADMIN, 'priority_fee': '3 gwei'})
{
  Transaction sent: 0x3e469e95cafbced3a0c58095745a201c90afbc818acadfb1ffb6e39f2f53ef89
  MetaUSDBalances.add_liquidity confirmed   Block: 10128378   Gas used: 176731 (79.71%)   Gas price: 3.000000015 gwei
}

```
## Meta-pool transactions

**Do exchange using DAI and sbUSD (we can see that 0.999 DAI was bought for 1 sbUSD so we have the peg):**
```
#----> do exchange on underlying assets (buy DAI with one sbUSD)
>>> meta.get_dy_underlying(0, 1, 10**18) 
   #--- result = 999390675280087667 ~= 0.999 DAI for one sbUSD
>>> expected = meta.get_dy_underlying(0, 1, 10**18) * 0.9
>>> dai.balanceOf(ADMIN)
  89101930192066263375045
>>> sbUsd.balanceOf(ADMIN)
  88489000000000000000000
>>> meta.exchange_underlying(0, 1, 10**18, expected, {'from': ADMIN, 'priority_fee': '3 gwei'})
{
  Transaction sent: 0x7432ec1b3e285c8f4837a8382a0c546547a613dd673e3460eaf72b8ef348fd83
  MetaUSDBalances.exchange_underlying confirmed   Block: 10128452   Gas used: 231171 (71.15%)   Gas price: 3.000000012 gwei
}
>>> dai.balanceOf(ADMIN)
  89102929563925958886444
>>> sbUsd.balanceOf(ADMIN)
  88488000000000000000000
```
**Do exchange using Uniswap when slippage is too high:** 
```
#----> use uniswap fallback
>>> expected = 1.1 # (to simulate slippage too high)
>>> meta.exchange_underlying(0, 1, 10**18, expected, {'from': ADMIN, 'priority_fee': '3 gwei'})
{
  Transaction sent: 0x3196a9185d6d7c096547f2346aa4ee321b4ee10d7c9807d0ee78395f7102d17c
  MetaUSDBalances.exchange_underlying confirmed   Block: 10128466   Gas used: 233710 (71.16%)   Gas price: 3.000000012 gwei
}
>>> dai.balanceOf(ADMIN)
  89103928916970489340209
>>> sbUsd.balanceOf(ADMIN)
  88487000000000000000000
```

## Discussion on custom solution for slippage too high

Uniswap was used insted of 1Inch cos 1Inch v2-protocol is already deprecated and new AggregationProtocol is recommended to be used with their web API rather than smart contract calls
(also it would be quite hard to do it).

On Rinkeby testnet there no contracts useful help with high slippage so in that situation so none was implemented.

But if we would be on mainnet following could be used:
* use [Mooniswap](https://mooniswap.exchange/#/swap) which solves (not 100% but a lot) the [front running and slippage issues](https://blog.1inch.io/1inch-revolutionizes-automated-market-maker-amm-segment-with-mooniswap-e068c20d94c)
* use [Sorbet finance](https://www.sorbet.finance/limit-order?ref=gelato.network#/limit-order) and their limit orders based on Gelato and Uniswap
* use [Gelato](https://app.gelato.network/) and implement delayed swap execution in hope the slippage will be smaller.
* use Flahbots bundle or [MistX](https://mistx.io/swap) (latter is also using Flashbots) to avoid front running and send transaction directly to miners, but this had to be submitted off-chain not from smart contracts





