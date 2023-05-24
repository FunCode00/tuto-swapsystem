#Swap System Walkthrough Tutorial
This walkthrough tutorial explains the inner workings of a swap system similar to Uniswap V1.
---
###Overview
The swap system is designed to facilitate decentralized token swaps between different tokens. It uses liquidity pools, which are reserves of tokens that users contribute to. The prices of tokens are determined by the ratio of reserves in the pools. When users want to swap tokens, the swap system calculates the appropriate exchange rate based on the pool's reserves.

The key components of the swap system are tokens, liquidity pools, and the functions for adding liquidity and swapping tokens.

###Tokens
Tokens represent the different types of assets that can be swapped in the system. Each token has a name and a balance associated with it. The balances reflect the amount of tokens held by different accounts in the system.
```typescript
class Token {
  public balance: u64;

  constructor (public name: string) {
    this.balance = 0;
  }
}
```
###Liquidity Pools
Liquidity pools are created for each token pair that can be swapped. Each pool consists of two tokens and their corresponding reserves. The reserves represent the amount of tokens available in the pool for each token type. The ratio of reserves determines the exchange rate between the tokens.
```typescript
class LiquidityPool {
  constructor(public tokenA: Token, public tokenB: Token, public reserveA: u64, public reserveB: u64) {}
  ...
}
```

###Adding Liquidity
To add liquidity to the swap system, users deposit an equal value of both tokens in a liquidity pool. The tokens are added to the reserves, increasing the liquidity of the pool. This process allows users to contribute to the liquidity available for token swaps.
```typescript
  /**
   * Adds liquidity to the pool by depositing tokens.
   * @param amountA - The amount of Token A to deposit.
   * @param amountB - The amount of Token B to deposit.
   */
  addLiquidity(amountA: u64, amountB: u64): void {
    this.tokenA.balance += amountA;
    this.tokenB.balance += amountB;
    this.reserveA += amountA;
    this.reserveB += amountB;
  }
```
###Swapping Tokens
Token swaps occur when a user wants to exchange one token for another. The swap system calculates the amount of tokens the user will receive based on the exchange rate determined by the liquidity pool's reserves. The reserves are adjusted accordingly, reflecting the tokens swapped by the user.
```typescript
/**
 * Swaps tokens between Token A and Token B.
 * @param fromToken - The token to swap from.
 * @param toToken - The token to swap to.
 * @param amount - The amount of tokens to swap.
 */
/**
 * Represent Swap System
 */
class SwapToken {
  public tokens: Map<string, Token>;
  public liquidityPools: Map<string, LiquidityPool>;

  constructor() {
    this.tokens = new Map<string, Token>();
    this.liquidityPools = new Map<string, LiquidityPool>();
  }

  /**
   * Add Token to swap system
   * @param name - name of the token
   */
  addToken(name: string): void {
    const token = new Token(name);
    this.tokens.set(name, token);
  }

  /**
   * Adds a liquidity pool to the swap system.
   * @param tokenAName - The name of Token A in the pool.
   * @param tokenBName - The name of Token B in the pool.
   * @param reserveA - The initial reserve of Token A.
   * @param reserveB - The initial reserve of Token B.
   */
  addLiquidityPool(tokenAName: string, tokenBName: string, reserveA: u64, reserveB: u64): void {
    const tokenA = this.tokens.get(tokenAName);
    const tokenB = this.tokens.get(tokenBName);

    if (!tokenA || !tokenB) {
      return;
    }

    const liquidityPool = new LiquidityPool(tokenA, tokenB, reserveA, reserveB);
    this.liquidityPools.set(`${tokenAName}-${tokenBName}`, liquidityPool);
  }

  /**
   * Checks the balance of a token.
   * @param tokenName - The name of the token.
   * @returns The balance of the token.
   */
  getTokenBalance(tokenName: string): u64 {
    const token = this.tokens.get(tokenName);
    return token ? token.balance : 0;
  }

  /**
   * Calculates the price of Token A in terms of Token B in a liquidity pool.
   * @param tokenAName - The name of Token A in the pool.
   * @param tokenBName - The name of Token B in the pool.
   * @returns The price of Token A in terms of Token B.
   */
  calculatePrice(tokenAName: string, tokenBName: string): u64 {
    const liquidityPool = this.liquidityPools.get(`${tokenAName}-${tokenBName}`);
    return liquidityPool ? liquidityPool.calculatePrice() : 0;
  }

  /**
   * Adds liquidity to a liquidity pool.
   * @param tokenAName - The name of Token A in the pool.
   * @param tokenBName - The name of Token B in the pool.
   * @param amountA - The amount of Token A to deposit.
   * @param amountB - The amount of Token B to deposit.
   */
  addLiquidity(tokenAName: string, tokenBName: string, amountA: u64, amountB: u64): void {
    const liquidityPool = this.liquidityPools.get(`${tokenAName}-${tokenBName}`);
    if (!liquidityPool) {
      return;
    }
    liquidityPool.addLiquidity(amountA, amountB);
  }

  /**
   * Swaps tokens between two accounts.
   * @param from - The account to swap tokens from.
   * @param to - The account to swap tokens to.
   * @param tokenName - The name of the token to swap.
   * @param amount - The amount of tokens to swap.
   */
  swapTheToken(tokenAName: string, tokenBName: string, fromTokenName: string, toTokenName: string, amount:u64): void {
    const liquidityPool = this.liquidityPools.get(`${tokenAName}-${tokenBName}`);
    const fromToken = this.tokens.get(fromTokenName);
    const toToken = this.tokens.get(toTokenName);

    if (!liquidityPool || !fromToken || !toToken) { return }

    liquidityPool.swapTokens(fromToken, toToken, amount);

  }
}
```

###Walkthrough
Let's walk through the steps involved in using the swap system:

1. __Initialize the Swap System__: The swap system is initialized by deploying the smart contract. This sets up the initial state of the system, including the tokens and liquidity pools.
```typescript
const swapSystem = new SwapToken();
```

2. __Add Liquidity__: To contribute liquidity to the swap system, select a token pair that you want to provide liquidity for. Make sure you have an equal value of both tokens. Call the addLiquidity function, specifying the tokens and the amounts you want to contribute. The function will update the liquidity pool's reserves and your token balances.
```typescript
/**
 * Adds liquidity to a liquidity pool.
 *
 * @param binaryArgs - Arguments serialized with Args
 * @returns An empty array.
 */
export function addLiquidity(binaryArgs: StaticArray<u8>): StaticArray<u8> {
  if (!callerHasWriteAccess()) {
    return [];
  }

  const argsDeser = new Args(binaryArgs);
  const tokenAName = argsDeser.nextString().expect('Token A name argument is missing or invalid');
  const tokenBName = argsDeser.nextString().expect('Token B name argument is missing or invalid');
  const amountA = argsDeser.nextU64().expect('Amount A argument is missing or invalid');
  const amountB = argsDeser.nextU64().expect('Amount B argument is missing or invalid');

  swapSystem.addLiquidity(tokenAName, tokenBName, amountA, amountB);

  return [];
}
```
```typescript
// Example of adding liquidity
addLiquidity('TokenA', 'TokenB', amountA, amountB);
```

3. __Check Token Balances__: You can check the balance of a specific token by calling the getTokenBalance function and providing the token's name. The function will return the balance of the token for your account.
```typescript
/**
 * Checks the balance of a token.
 *
 * @param binaryArgs - Arguments serialized with Args
 * @returns The balance of the token serialized in bytes.
 */
export function getTokenBalance(binaryArgs: StaticArray<u8>): StaticArray<u8> {
  const argsDeser = new Args(binaryArgs);
  const tokenName = argsDeser.nextString().expect('Token name argument is missing or invalid');

  const balance = swapSystem.getTokenBalance(tokenName);

  return stringToBytes(balance.toString());
}
```
```typescript
// Example of checking token balance
const balance = getTokenBalance('TokenA');
console.log(`TokenA Balance: ${balance}`);
```

4. __Calculate Token Prices__: To calculate the price of one token in terms of another, call the calculatePrice function, providing the names of the two tokens. The function will determine the exchange rate between the tokens based on the liquidity pool's reserves.
```typescript
/**
 * Calculates the price of Token A in terms of Token B in a liquidity pool.
 *
 * @param binaryArgs - Arguments serialized with Args
 * @returns The price of Token A in terms of Token B serialized in bytes.
 */
export function calculatePrice(binaryArgs: StaticArray<u8>): StaticArray<u8> {
  const argsDeser = new Args(binaryArgs);
  const tokenAName = argsDeser.nextString().expect('Token A name argument is missing or invalid');
  const tokenBName = argsDeser.nextString().expect('Token B name argument is missing or invalid');

  const price = swapSystem.calculatePrice(tokenAName, tokenBName);

  return stringToBytes(price.toString());
}
```
```typescript
// Example of calculating token price
const price = calculatePrice('TokenA', 'TokenB');
console.log(`TokenA Price in TokenB: ${price}`);
```

5. __Swap Tokens__: To swap tokens, call the swapTokens function, specifying the tokens you want to swap, the amount you want to exchange, and the desired output token. The function will calculate the amount of tokens you will receive based on the exchange rate and update the token balances and liquidity pool reserves accordingly.
```typescript
/**
 * Swaps tokens between two accounts.
 *
 * @param binaryArgs - Arguments serialized with Args
 * @returns the emitted event serialized in bytes
 */
export function swapTokens(binaryArgs: StaticArray<u8>): StaticArray<u8> {
  if (!callerHasWriteAccess()) {
    return [];
  }

  const argsDeser = new Args(binaryArgs);
  const fromTokenName = argsDeser.nextString().expect('From argument is missing or invalid');
  const toTokenName = argsDeser.nextString().expect('To argument is missing or invalid');
  const tokenAName = argsDeser.nextString().expect('Token name argument is missing or invalid');
  const tokenBName = argsDeser.nextString().expect('Token B name argument is missing or invalid');
  const amount = argsDeser.nextU64().expect('Amount argument is missing or invalid');

  swapSystem.swapTheToken(tokenAName, tokenBName, fromTokenName, toTokenName, amount);

  return [];
}
```
```typescript
// Example of swapping tokens
swapTokens('TokenA', 'TokenB', 'TokenA', 'TokenB', amount);
```

By following these steps, you can interact with the swap system, provide liquidity, check token balances, calculate prices, and perform token swaps.
---
Congratulations! You've completed the walkthrough tutorial for the swap system. You now have a better understanding of how a decentralized token swap system, similar to Uniswap V1, works.

Feel free to explore and enhance the functionality of the swap system by adding additional features and optimizations.

If you wanna have more information about __The Uniswap V1 Smart Contracts__, check this link : [Click Here](https://docs.uniswap.org/contracts/v1/overview)

Happy swapping!