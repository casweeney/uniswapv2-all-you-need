# Uniswap-ALL-IN-ONE

## Smart contract code breakdown

Uniswap is a Decentralized Exchange which uses the AMM (Automated Market Maker) mechanism
It uses the product constant formula k = x*y

### Contracts:
Uniswap is made of four (4) smart contracts which are organized into two sections:
- 1. The Core
- 2. The Periphery

- The UniswapV2 Core consists of 3 smart contracts
- i. UniswapV2ERC20.sol
- ii. UniswapV2Factory.sol
- iii. UniswapV2Pair.sol

- The UniswapV2 Periphery is consist of:
- i. UniswapV2Router.sol






### UniswapV2ERC20.sol
The UniswapV2ERC20.sol contract keeps track of pool ownership. This is the token that gets minted to liquidity providers when ever liquidity is provided. It serves as a certificate(tokens) that an address (liquidity provider) provided liquidity in a particular liquidity pool and such address will get rewards which is gotten from the fees traders pay when they swap tokens.


The fees can only be received when a liquidity provider decides to remove liquidity, which will burn the UniswapV2ERC20 token they have and they will get their funds back plus the accumulated rewards.

############################## UniswapV2ERC20 Code Breakdown #########################
- The UniswapV2ERC20 contract code is a standard ERC20 token which means it implements all the functions in a the ERC20 standard and it token name and symbol are set at compilation time unlike Openzeppelin that allows you set token name and symbol at contract creation time.

- Uniswap also implement a meta transaction in their UniswapV2ERC20.sol using the ERC20 permit.
- This is done using the permit() function in the Uniswap ERC20 smart contract






### UniswapV2Factory.sol
The UniswapV2Factory.sol creates the UniswapV2Pair.sol contract and keeps track of all the pair that has been created.
Note: Every token pair traded on Uniswap is a separate UniswapV2Pair.sol contract.

The UniswapV2Factory.sol passes the addresses of the two tokens(token0 and token1) to an initialize function in the UniswapV2Pair.sol contract which is called immediately after the UniswapV2Pair.sol contract is created by the UniswapV2Factory.sol contract.

############################## UniswapV2Factory Code Breakdown #########################
The UniswapV2Factory contract has the following:
- 1. feeTo : This variable is a type of address, and it is the address that receives fees if turned on. If this address is not set(it's value is address(0)), this means fee is off, otherwise fee is on.

- 2. feeToSetter: This variable is also a type of address and it holds the address of the user who has the permission to set or change the value of the feeToo variable. This value of the feeToSetter is set in the construction at contract creation time.

- 3. getPair: this is a 2D mapping that keeps track of all the pairs that has been created. It tracks the pair created by the UniswapV2Factory contract using the tokens(token0 and token1) of the pair.
#### Example: ####
getPair[token0Address][token1Address] = pairContractAddress;
getPair[token1Address][token0Address] = pairContractAddress

- 4. allPairs: This is an array of type address that holds the address of all the pair (UniswapV2Pair.sol) contracts that the factory (UniswapV2Factory) contract has created.

- 5. The createPair() function creates the pair contract and takes in the tokens for the pair to be created as arguments
After creating the Pair contract, it calls the initialize() function of the pair it just created which sets the tokens of the pair immediately the pair contract's state variable.

- 6. setFeeTo() is the function that is called to set the "feeTo" variable to a value which is an address that will receive Uniswap fees

- 7. setFeeToSetter(): this function can only be called by the previous "feeToSetter" which was set during contract creation. The function is used to change the previous "feeToSetter" address to a new address.




%%%%%%%%%%%%%%%%%%%%%%%%%%%%%% UniswapV2Pair Starts Here %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

### UniswapV2Pair.sol
The UniswapV2Pair.sol contract handles the heavy lifting of the entire UniswapV2 platform.
- It contains the mint() function which handles liquidity provision
- It has the burn() function which handles liquidity removal.
- It has the swap() function which controls swapping from one token to another.

#### The UniswapV2Pair contract functionalities are:
- i. Managing funds
- ii. Handling functions for liquidity providers
- iii. Handling functions for traders - swapping
- iv. Managing pool ownership tokens
- v. Handling protocol fee

####### Code Breakdown UniswapV2Pair Contract ########

-------- Managing Funds ------------
A uniswap pair is an exchange between a pair of token for example: Dogecoin and ShibaInu

These tokens are represented by token0 (contract address of token0) and token1(contract address of token1) in the pair contract. 

- reserve variables store how much of the token the pair contract has
- reserve0 : amount of token0 in the pair contract
- reserve1 : amount of token1 in the pair contract

- token0 : contract address of token0 (one of the token pair)
- token1 : contract address of token1 (the second token of pair)

- blockTimestampLast : this keeps track of the last time the reserve was updated

- price0CumulativeLast : This keeps track of the cumulative price of token0 each time there is a change in reserve0

- price1CumulativeLast : This keeps track of the cumulative price of token1 each time there is a change in reserve1

- kLast : This is the product constant of reserve0 and reserve1 (reserve0 * reserve1)

Note: The pair contract only keep track of reserves of the two tokens that makes up the pair (It doesn't store the actual token)

- The pair contract uses the balanceOf(address(this)) and the transfer function to manage tokens
- It uses a low level call to perform its transfer function: It gets the function selector of the transfer() function
- Then it implements the transfer function inside the _safeTransfer() function using low level call.
- The _safeTransfer() function takes the address of the token to be transferred, address of the receiver and amount of token to be transferred.



%%%%%%% Update Function %%%%%%%%
- _update() function : the _update() function is called whenever there are new funds deposited or withdrawn by the liquidity providers or tokens are swapped by traders.
- balance0 and balance1 are the balances of the ERC20 tokens in the Pair contract. They are the return values of balanceOf(address(this)) function.
- _reserve0 and _reserve1 are Uniswap's previously known balances (last time balanceOf() was checked)
- All the update function does is to check for overflow, update price oracle, update reserves and update a Sync event

------ How Uniswap price oracle works inside the update function -------
- asumme blockTimestampLast = 1672082889
- blockTimeStamp = uint32(block.timestamp % 2^32)
- blockTimestamp = uint32(1672083251 % 4294967296)
- blockTimeStamp = uint32(1672083251)
- blockTimeStamp = 1672083251

- timeElapsed = blockTimestamp - blockTimestampLast
- timeElapsed = 1672083251 - 1672082889
- timeElapsed = 362

```````````````````` THE CODE BELOW
    if (timeElapsed > 0 && _reserve0 != 0 && _reserve1 != 0) {
        // * never overflows, and + overflow is desired
        price0CumulativeLast += uint(UQ112x112.encode(_reserve1).uqdiv(_reserve0)) * timeElapsed;
        price1CumulativeLast += uint(UQ112x112.encode(_reserve0).uqdiv(_reserve1)) * timeElapsed;
    }
```````````````````
Asumming:
price0CumulativeLast = 0
price1CumulativeLast = 0

reserve0 = 1000
reserve1 = 500

price0CumulativeLast += ( (500 * 2^112) / 1000 ) * 362
price0CumulativeLast += 2596148429267413814265248164610048 * 362
price0CumulativeLast += 939805731394803800764019835588837376
price0CumulativeLast = 939805731394803800764019835588837376

price1CumulativeLast += ( (1000 * 2^112) / 500 ) * 362
price1CumulativeLast += ( 5192296858534827628530496329220096000 / 500) * 362
price1CumulativeLast += 10384593717069655257060992658440192 * 362
price1CumulativeLast += 3759222925579215203056079342355349504
price1CumulativeLast = 3759222925579215203056079342355349504


%%%%%%%% Mint Fee Function %%%%%%%%
- _mintFee() function
- The _mintFee() function is used to mint 1/6 of the fee to be shared by liquidity providers to uniswap. This is the fee uniswap get for maintaining the platform and it is optional (which means it can be turned off or on). This _mintFee() mints fee if fee is on. Fee is on if the feeTo address is not address(0).

Asumming:
reserve0 = 1000
reserve1 = 500
klast = 1000 * 500
klast = 500000

new liquidity:
token0 100
token1 50

currentReserve0 = 1000 + 100 = 1100
currentReserve1 = 500 + 50 = 550

rootK = Math.sqrt(currentReserve0 * currentReserve1)
rootK = Math.sqrt(605000)
rootK = 777.8174593052 (round down)
rootK = 777

rootKLast = Math.sqrt(_kLast);
rootKLast = Math.sqrt(500000);
rootKLast = 707.1067811865
rootKLast = 707

kDifference = rootK - rootKLast
kDifference = 777.8174593052 - 707.1067811865
kDifference = 70.7106781187
kDifference = 70

using totalSupply = 499000

numerator = totalSupply * kDifference
numerator = 499000 * 70
numerator = 34930000

denominator = (rootK * 5) + rootKLast
denominator = (777 * 5) + 707
denominator = 3885 + 707
denominator = 4592

liquidity = numerator / denominator
liquidity = 34930000 / 4592
liquidity = 7606.7073170732 (round down)
liquidity = 7606 (This liquidity goes to Uniswap: feeTo address)

/////// Now let's calculate what goes to the liquidity provider ////////

Using: liquidity = Math.min(amount0.mul(_totalSupply) / _reserve0, amount1.mul(_totalSupply) / _reserve1);
amount0 = 100
amount1 = 50
reserve0 = 1000
reserve1 = 500
totalSupply = 499000 // previous total supply
// after minting fee to feeTo, totalSupply will be increased
totalSupply = 499000 + 7606
totalSupply = 506606

liquidity = Math.min( (100 * 506606) / 1000, (50 * 506606) / 500)
liquidity = Math.min( 50660600 / 1000, 25330300 / 500 )
liquidity = Math.min(50660.600, 50660.600)
liquidity = 50660


----------- Minting and Burning ---------------
Minting and Burning are also very important functionalities Uniswap handles. 

- Minting is when a liquidity provider adds funds (provide liquidity) to the pool which leads to minting of new ownership token (LP tokens - UniswapV2ERC20)
- Burning is the opposite of minting : this is when a liquidity provider withdraws funds (and the accumulated rewards) and his pool ownership token (LP tokens) are burned (destroyed)


- Mint Function: mint()
***********************


- In the mint() function, at first Uniswap used a gas saving mechanism to save gas by transferring reserve0, reserve1 and totalSupply from storage to memory and hold them at _reserve0, _reserve1, and _totalSupply respectively.
- The totalSupply is the total UniswapV2ERC20 that has been minted. If the value is zero(0), this means liquidity has not be provided before. The pair contract extends the UniswapV2ERC20, this is why it has access to the totalSupply variable
- amount0 and amount1 holds the amount that was deposited.
-  This is calculated by subtracting the reserves from the the current balance.
- amount0 = balance0 - reserve0 (current balance - last reserve balance)
- amount1 = balance1 - reserve1 (current balance - last reserve balance)

- If totalSupply is zero (0), it means the pool is a brand new one and liquidity has not been provided before: A brand new pool needs to lock in MINIMUM_LIQUIDITY amount of pool ownership (LP) token to avoid division by zero in the liquidity calculations.
-  This is locked by sending the Mininum Liquidity to address(0) : Once this is done, the total supply can never be reduced to zero again even if the liquidity providers remove their liquidity from the pool.

----
- If the pool is a brand new pool,
- Liquidity is calculated using:
- liquidity = Math.sqrt( (amount0 * amount1) - MINIMUM_LIQUIDITY )

reserve0 = 0
reserve1 = 0
balance0 = 1000 // amount of token0 sent (Shiba)
balance1 = 500 // amount of token1 sent (Doge)

amount0 = 1000 - 0
amount1 = 500 - 0

amount0 = 1000
amount1 = 500

_totalSupply = 0
MINIMUM_LIQUIDITY 10 ^ 3

liquidity = Math.sqrt( (1000 * 500) - 1000)
liquidity = Math.sqrt(499000)
liquidity = 706.399320498
liquidity = 706

totalSupply = 706 + 1000
totalSupply = 1706

After this, reserves are updated and kLast is updated too
reserve0 = 1000
reserve1 = 500
kLast = 500000

- If the pool is not brand new
- Liquidity is calculated using:
- liquidity = Math.min( (amount0 * _totalSupply) / _reserve0, (amount1 * _totalSupply) / _reserve1 

----

- Once the liquidity value is derived, then it mints the liquidity to the address of the liquidity provider
- Which means it mint the Ownership (LP) token to the liquidity provider using calculated value from liquidity as the amount to token it will mint.
-  This is done by calling the _mint() function that takes in the address of the receiver and the amount of token to be minted.
- The address of the liquidity provider that receives the LP token is provided by the Router Contract.

- After minting, then the _update() function is called which takes in the current balance of the tokens of the pair contract (balance0 and balance1), takes in the previous reserves of the tokens of the pair(reserve0 and reserve1), the _update() function then updates the reserves(reserve0 and reserve1) with the new balance of the tokens(balance0 and balance1).

--------- Note This ---------
The way adding funds works is: they are just deposited to the Pair contract (by calling transfer(from: liquidity provider’s address, to: Pair contract’s address, amount) for each token). Then the Pair contract will read the balances  and compare them to the last known balances. This is how the Pair contract can deduce the amounts deposited.
-----------------------------


- Burn Function: mint()
***********************
- The burn() function is the exact opposite of the mint() function
- The same gas saving mechanism is used just like in the mint() function
- balance0 and balance1 are total balances of the pair tokens in this pool

- balance0 = IERC20(_token0).balanceOf(address(this))
- balance1 = IERC20(_token1).balanceOf(address(this))
- liquidity = balanceOf[address(this)];

- liquidity is the amount of pool ownership tokens that the liquidity provider (who wishes to cash out) has. 

--- QUESTION ---
- But Why do we access the liquidity as the balance of address(this)? 

--- ANSWER ---
- Because the liquidity was transferred to the Pair contract by the Periphery (Router) contract before calling the burn function.

- The amount of token to withdraw to the liquidity provider is proportional to the amount of Pool Ownership tokens (LP tokens) he has and this is calculated by:
- amount0 = (liquidity * balance0) / totalSupply
- amount1 = (liquidity * balance1) / totalSupply

- After the amount is calculated, the burn function is called, which burns the LP token of the liquidity provider that was transferred into the contract from the contract.
- Then it transfers the pair token the the liquidity provider used in providing liquidity. (This transfer also includes the accumulated rewards from traders fees over time)
- The _update() function is called again which updated the reserves(reserve0 and reserve1) with the new balance (balance0 and balance1) after the pair tokens (token0 and token1) + the rewards have been transferred to the liquidity provider that removed their liquidity.

reserve0 = 1000
reserve1 = 500

balance0 = 1000
balance1 = 500

liquidity = 706
totalSupply = 1706

amount0 = liquidity.mul(balance0) / _totalSupply;

amount0 = (706 * 1000) / 1706
amount0 = 706000 / 1706
amount0 = 413.8335
amount0 = 413

amount1 = liquidity.mul(balance1) / _totalSupply;
amount1 = (706 * 500) / 1706
amount1 = 353000 / 1706
amount1 = 206.91676
amount1 = 206

balance0 = 1000 - 413
balance0 = 587

balance1 = 500 - 206
balance1 = 294

reserve0 = 587
reserve1 = 294

kLast = 172578




- Swap Function: swap()
***********************
- The swap() function is used by traders to swap tokens
- The swap function ensures that the amount of token you are swapping to is greater than zero (It can be any of the tokens in the pair contract) : Only one of the tokens can have value greater than zero at a time.
- It also ensures that the amount of token you are swapping from and the amount of token you are swapping to is less than the available reserve: hence it will throw an INSUFFICIENT LIQUIDITY error.

- If this checks are passed, it checks which of the amount out (amount0Out or amount1Out) that is greater than zero, then it transfers the amount out to the trader optimistically.

- Note: (without making sure that the trader has already transferred corresponding tokens into our balance. We can optimistically transfer tokens out because the swap function have assertions later in the function to check if we received corresponding tokens (the Periphery contract should send in the tokens to the pair contract before calling it for the swap). If the pair contract have not received any tokens, assertions will fail and Solidity will revert the entire function.

- The code: if (data.length > 0) IUniswapV2Callee(to).uniswapV2Call(msg.sender, amount0Out, amount1Out, data);
will inform the receiver about the swap if requested

- Then it will check how many tokens was received by the pair contract.

- amount0In = balance0 > _reserve0 - amount0Out ? balance0 - (_reserve0 - amount0Out) : 0
- amount1In = balance1 > _reserve1 - amount1Out ? balance1 - (_reserve1 - amount1Out) : 0

Let's assume:
- reserve0 = 1000
- reserve1 = 500

A user is swapping 100 token0
This means the user will get 50 token1

token0 incoming = 100
token1 incoming = 0

token0 outgoing = 0
token1 outgoing = 50

At this point: 
balance0 = 1100
balance1 = 450

Using this formula:
- amount0In = balance0 > _reserve0 - amount0Out ? balance0 - (_reserve0 - amount0Out) : 0
- amount1In = balance1 > _reserve1 - amount1Out ? balance1 - (_reserve1 - amount1Out) : 0

amount0In = 1100 > 1000 - 0 ? 1100 - (1000 - 0) : 0

amount0In = 100

amount1In = 450 > 500 - 50 ? 450 - (500 - 50) : 0
amount1In = 0


- If the amountIn of either tokens of the pair contract is less than zero, it will revert with: INSUFFICIENT_INPUT_AMOUNT and the entire function will revert and nothing will have taken place.

- If the checks passes, then it proceeds to where the 0.3% fee paid by traders is calculated. It does this calculation to check and ensure that the fee was paid. Note the fee is also transferred to the pair contract and this is made possible by the Router contract.

- This is calculated by:
- balance0Adjusted = (balance0 * 1000) - (amount0In * 3)
- balance1Adjusted = (balance1 * 1000) - (amount1In * 3)

Remember:
balance0 = 1100
balance1 = 450

- balance0Adjusted = (1100 * 1000) - (100 * 3)
- balance0Adjusted = 1100000 - 300
- balance0Adjusted = 1099700

- balance1Adjusted = (450 * 1000) - (0 * 3)
- balance1Adjusted = 450000 - 0
- balance1Adjusted = 450000

- Then it checks if the k value (x*y=k) has decreased after the trade. The k value can never decrease because otherwise, Uniswap would lose from the swap.
**
require(balance0Adjusted.mul(balance1Adjusted) >= uint(_reserve0).mul(_reserve1).mul(1000**2), 'UniswapV2: K');
**

Remember k = x*y

adjustedK = balance0Adjusted * balance1Adjusted
previousK = (_reserve0 * _reserve1) * 1000^2 

adjustedK = 1099700 * 450000
adjustedK = 494865000000

previousK = (1000 * 500) * 1000^2
previousK = 500000000000

(The 1000^2 is because we multiplied balance0 and balance1 by 1000 when calculating the balance0Adjusted and balance1Adjusted)

- Finally, the _update() function is called to update the known reserves with the new balances and emit a Swap event.

- Lock modifier:
- lock is for guarding against reentrancy abuse. Essentially this function modifier prevents 2 different parts of this contract to be executed simultaneously. It kinda makes the contract execute with a single thread.


==============
skim and sync are needed when balances on the ERC20 contracts of the exchange tokens, fall out of sync with the reserve variables in the Pair contract. This can happen for example when someone just transfers some Dogecoin to Pair contract’s account for no reason. There are 2 solutions to keep reserve variables in sync with the actual balances on ERC20 contracts:
==============


- Skim Function: skim():
- It allows someone to withdraw the extra funds from the Pair contract. Anyone can call this function!
- The skim() function forces the balances to match reserves
- it transfers any extra token to the address that called the function
- _safeTransfer(_token0, to, IERC20(_token0).balanceOf(address(this)) - reserve0)
- _safeTransfer(_token1, to, IERC20(_token1).balanceOf(address(this)) - reserve1)

- Sync Function: sync():
- The sync() function forces reserves to match balances
- It calls the _update() function

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%% UniswapV2Pair Ends Here %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%


### UniswapV2Router.sol
The UniswapV2Router.sol contract is for interacting with the core. It is quite complicated and dangerous to interact with the core directly, this may lead to loss of funds if not careful. The UniswapV2Router.sol implements useful
security checks before calling functions in the core contracts.

The core have checks to make sure that they aren't cheated, but they don't check for any other person, hence the need for the router contract.

######## UniswapV2Router Contract Breakdown ###########

The Router contract contains self explanatory contracts for:
- i. adding liquidity
- ii. removing liquidity
- iii. swapping tokens

