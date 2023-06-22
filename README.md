# Secureum Races Journey

This repository will contain my results of different Secureum Races for the record
<br/> and write-ups for newbies ( thinking if something is unclear for me, might be for others too)

## Race #1

![Screenshot-1](https://github.com/umaresso/Secureum-Races/assets/71306738/f4287a69-2c0e-4235-888c-5c9d3ddfc83c)
## Race #2

![Screenshot-3](https://github.com/umaresso/Secureum-Races/assets/71306738/147e25a5-3514-4af7-9cf4-3f5412b9ef6a)

### Write up

Race Link : [Check here](https://ventral.digital/posts/2021/11/28/secureum-bootcamp-audit-findings-101-quiz)


Code Review :

- Anyone can see the mapping - Not really a vulnerability

- onlyWhenOpen modifier checks for the balance of the contract - vulnerable because anyone can send Force Ether.

- Constructor ensures that admin is zero address ```solidity require(_admin == address(0));``` . This is Funny: / - Vulnerable ofcourse because no one will ever be able to deploy a contract without zero admin and then no one call the admin access control functions because admin is zero address and that is not controlled by any 

- Non-re-entrancy guard should come before any other modifier 
` function join() external payable onlyWhenOpen nonReentrant `
 
 The value should not be strictly equal, this can cause discrepancies:

```solidity require(msg.value == membershipFee, 'InSecureumDAO: Incorrect ETH amount');```

rather consider doing something like >= so that people who accidentally sent one more wei
in the  Tx can still become member and not face the overhead of sending the Tx again cause it will fail in the current scenario.


- `whenNotPaused` modifier is not declared
- Remove all members function does not necessarily remove all the members



Questions:

For most of the questions, the explanation was more than enough!

But here are a few i've struggled with:

2.-

 what is "Guarded launch via circuit breakers"? - it is a mechanism in which the system can be paused in an extreme situation of crisis.

5.- 

Why not Option A?

`A. join() to not disclose msg.sender while joining the DAO`

We do not necessarily have to hide membership like membership in a club. Should be public.


Why Not Option B?

`B. createVote() to not disclose the possible outcomes during creation`


Any member can create the vote and it does not violate and privacy rules of the individual.
Rather votes should be seen by public for transparency and deciding where to cast their vote.


6.-


Why not Option C?


`getWinningOutcome() for existing _voteId`

In solidity, when we try to access a variable at a position in mapping, that is not initialized before, it returns the default value of the value type :

Here in the mapping:


```solidity mapping (uint256 => uint8) public winningOutcome;```


The key type is   - uint256
The value type is - uint8

default values for uint8 are 0 

So if we try to access a vote Id's winning outcome that does not exist in the mapping, we will still return the zero outcome which is not any outcome.

That's why we don't need to check if the vote id exists in this function.

And It's a wrap!

## Race # 3
![image](https://github.com/umaresso/Secureum-Races/assets/71306738/2454a61a-c473-48a8-a94b-faeb6281210a)

### Write up

Race Link : [Check here](https://ventral.digital/posts/2021/11/28/secureum-bootcamp-audit-findings-201-quiz)

Code Review:

- Only test variables are given, the production variables are commented
- No zero address check for _beneficiary in the constructor 
```solidity

 constructor(address payable _benificiary) {
        deployer = payable(msg.sender);
        beneficiary = _benificiary;
    }
    
    ```
    
- incorrect check in `startSale` , deployer or anyone with zero price as tx value can start sale

```solidity

require(msg.sender == deployer || _price != 0, "Only deployer and price cannot be zero");
 
 ```

- Unreliable source of randomness in `randomIndex` function

```solidity
  uint index = uint(keccak256(abi.encodePacked(nonce, msg.sender, block.difficulty, block.timestamp))) % totalSize;
      

```

- Sale price decreases after every second + after the sale, the price is zero and it does not check if sale has eventually ended.

```solidity

  if (elapsed > saleDuration) {
            return 0;
        } else {
            return ((saleDuration - elapsed) * price) / saleDuration;
        }
        
```

- in mint, when the duration of the sale has reached to end, the price will be 0. <br/> These lines will let the user mint upto TokensLimit tokens because 

contract balance = 0
price =0

the condition satisfies.
It transfers 0 funds and then mints.


```solidity

  require((address(this)).balance >= salePrice, "Insufficient funds to purchase.");
        if ((address(this)).balance >= salePrice) {
            payable(msg.sender).transfer((address(this)).balance - salePrice);
        }
        return _mint(msg.sender);

```

### Recommendation:
	- It should check msg.value instead of contract balance
	- PublicSale variable should be modified once saleDuration finishing is encountered
	- Function should not allow the minting at price=0
	

- Force Sending some Ether to the contract can do the exact same thing, <br/> actually stealing all the funds in the contract because the contract only checks for the balance of the contract rather msg.value.

## Race #4


![Screenshot-12](https://github.com/umaresso/Secureum-Races/assets/71306738/d12a9f0e-63c2-4d53-9cf2-736a4c460722)



