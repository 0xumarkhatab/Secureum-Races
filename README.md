# Secureum Races Journey

This repository will contain my results of different Secureum Races for the record
<br/> and write-ups for newbies ( thinking if something is unclear for me, might be for others too)

# Races with Write-ups



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
- 
## Race #5

## Scored 7.66 / 8 = 95%
## Found 19+ issues

![image](https://github.com/umaresso/Secureum-Races/assets/71306738/6255d5ec-2263-4162-982b-65f2f0a224bd)

### Write-up

Race Link : [Check here](https://ventral.digital/posts/2022/3/8/secureum-bootcamp-epoch-march-race-5)

#### Code Review

1- Import of introspection contract (ERC-165)  -According to Openzeppelin
 " This set of interfaces and contracts deal with type introspection of contracts, that is, examining which functions can be called on them. This is usually referred to as a contract’s interface."
 
2- `uri` method has to be external 

```solidity

function uri(uint256) public view virtual override returns (string memory) {
        return _uri;
    }
    
``` 


3- `balanceOfBatch` should be external

4- incorrect checks on array lengths are the same for both arrays `accounts` and `Ids`
```solidity

for (uint256 i = 0; i < accounts.length; ++i) {
            batchBalances[i] = balanceOf(accounts[i], ids[i]);
        }
        
```

5- `setApprovalForAll` has to be external

6- incorrect check for ownership in `safeTransferFrom`

```solidity
 require(
            from == _msgSender() || isApprovedForAll(from, _msgSender()),
            "ERC1155: caller is not owner nor approved"
        );

``` 

The `from` should not be the caller and the spender should be approved of all tokens strictly 
to execute the next part.
otherwise, he would be the owner of the tokes and he can transfer any



7- incorrect operator calculation. `_safeTransferFrom` is called by the contract itself so this line inside the contract will return contracts address as an operator instead of the EOA that called `safeTransferFrom`

```solidity

   address operator = _msgSender();
   
```
 
 
 
8-Underflow vulnerability inside `_safeTransferFrom`

```solidity

    unchecked {
            fromBalance = fromBalance - amount;
        }
        _b
```

and overflow can't occur with following line ```solidity _balances[id][to] += amount; ```
because we are using solidity compiler 0.8.0 which checks itself outside the unchecked block.


9- invalid `isContract` calculation. The code size of the contract is greater than zero not equal to zero.


```solidity

    function isContract(address account) internal view returns (bool) {
        return account.code.length == 0;
    }


```

10- `_doSafeTransferAcceptanceCheck` implements token rejection wrong.

it says 
```solidity

 if (response == IERC1155Receiver.onERC1155Received.selector) {

```

instead it should be like the response is not equal to the selector.


```solidity

 if (response != IERC1155Receiver.onERC1155Received.selector) {

```

for token rejection.


11- `catch` should check for a custom reason by checking if `reason.length==0` inside `_doSafeTransferAcceptanceCheck`


```
 try IERC1155Receiver(to).onERC1155Received(operator, from, id, amount, data) returns (bytes4 response) {
                if (response == IERC1155Receiver.onERC1155Received.selector) {
                    revert("ERC1155: ERC1155Receiver rejected tokens");
                }
            } catch Error(string memory reason) {
                revert(reason);
            } catch {
                revert("ERC1155: transfer to non ERC1155Receiver implementer");
            }

```


instead, it should be like openZeppelin's standard practice:


```solidity

catch (bytes memory reason) {
                if (reason.length == 0) {
                    // non-ERC1155Receiver implementer
                    revert ERC1155InvalidReceiver(to);
                } else {
                    /// @solidity memory-safe-assembly
                    assembly {
                        revert(add(32, reason), mload(reason))
                    }
                }
            }



```

12- `_safeTransferFrom` should be internal

13- `Un-equal arrays length` vulnerability in `safeBatchTransferFrom`


```solidity

  for (uint256 i = 0; i < ids.length; ++i) {
            uint256 id = ids[i];
            uint256 amount = amounts[i];

```


14-  Incorrect check for ownership in `safeTransferFrom`

The `from` should not be the caller and the spender should be approved of all tokens strictly 
to execute the next part.
otherwise, he would be the owner of the tokes and he can transfer any



```solidity

      from == _msgSender() || isApprovedForAll(from, _msgSender()),
            "ERC1155: transfer caller is not owner nor approved"
        );

```


15- `_safeBatchTransferFrom`same operator vulnerability as 7 

```solidity

address operator = _msgSender();

```


16-`_safeBatchTransferFrom` arrays length not equal vulnerability in a loop


17- incorrect require statement in `_mintBatch` 


```soldiity

 require(operator != address(0), "ERC1155: mint from the zero address");
      

```

as it is an internal function, the caller hence (operator=msgSender()) will always be the contract.


18- un-equal arrays length vulnerability in `_mintBatch`

```soliidty

  for (uint256 i = 0; i < ids.length; i++) {
            _balances[ids[i]][to] += amounts[i];
        }

```


#### Realized Later while solving questions

19- incorrect balance update in `_safeTransferBatch`


```solidity

 for (uint256 i = 0; i < ids.length; ++i) {
            uint256 id = ids[i];
            uint256 amount = amounts[i];
            uint256 fromBalance = _balances[id][from];
            fromBalance = fromBalance - amount;
            _balances[id][to] += amount;
        }

```
20 - minting to zero address essentially means burning the tokens as they come to birth. PATHETIC!  ( lost my 0.33 marks in race here )


## Sushi Bentobox Report
![Screenshot-14](https://github.com/umaresso/Secureum-Races/assets/71306738/e5a260e9-e8c5-4332-bf00-3e9f9722b224)



## Race # 11
Scored 5.7 .
Lost some marks to learn something new !

##
![Screenshot-50](https://github.com/umaresso/Secureum-Races/assets/71306738/7f8f5a27-3284-43a2-9146-2df927f290f6)

## Write-up



Race Link : https://ventral.digital/posts/2022/10/31/race-11-of-the-secureum-bootcamp-epoch

Code Review :

1. using solidity compiler 0.8.x so we don't have integer over.underflow by default unless it occurs in unchecked



2. So there seems an access controlled time lock deposit and withdrawl system by seeing these state variables 

```solidity

	bool internal _paused;
	address internal _operator;
	address internal _governance;
	address internal _token;
	uint256 internal _minDepositLockTime;
   
   ```
   
3. No zero address check and uint threshhold check in the constructor 

```solidity

   constructor(address operator, address governance, address token, uint256 minDepositLockTime) {
       _operator = operator;
       _governance = governance;
       _token = token;
       _minDepositLockTime = minDepositLockTime;
   }

```

4. So anyone can call `depositFor` to fund other people's account .


```soliidty

function depositFor(address user, uint256 amount) external 

```

5. There's an issue with the typecasting here .


```solidity

 IERC20(_token).safeTransferFrom(user, address(this), amount);
 
```

This is what i asked in Secureum Discord 


```
Hi @Rajeev | Secureum , the safeERC20 does not have a function safeTransferFrom which accepts following params user,receiver,amount rather it expects IERC20 token,user,receiver,amount.
In Race 11 , safeERC20 is used for IERC20 .

Then in depositFor , we have IERC20(_token).safeTransferFrom(user,receiver, amount).

First of all safeTransferFrom in safeERC20 is internal. secondly, it involves another parameter _token.
The expected parameters are 4 but 3 are given. 

Can you help me understand if this is a mistake at your end or my understanding ?

Thank you !

```

Till we get the answer , let's it over.


6. The event is misleading or unclear.

```soliidty

 emit Deposit(msg.sender, amount);

```

the event definition says :

```solidity


   event Deposit(
       address indexed user,
       uint256 amount
   );

```

who is the user ?
as anyone can deposit funds for anyone . so it is unclear who is the user here ?
The depositor (msg.sender ) ?
or the person who's account is funded (user) ?


### Withdraw Function


7. the contract should check for and maintain the last Withdraw time and min withdraw time limit instead of desposit.

```solidity

   function withdraw(uint256 amount) external {
       require(!_paused, 'paused');
       require(block.timestamp >= _userLastDeposit[msg.sender] + _minDepositLockTime, 'too early');
       

```
 
 because in this way , user can repeatedly withdraw if certain deposit time limit has been passed .
 
 
8. Same vulnerbaility as of point 5.

```solidity

IERC20(_token).safeTransferFrom(address(this), msg.sender, amount);


```

9. Re-entrancy : Although it is unclear for now but if `safeTransferFrom` calls some type of callback on the receiver of the tokens , and the receiver has some kind of malicious logic in the fallback function then the attacker contract can re-enter the contract because there are not Check-Effect-Interaction scheme implemented .

i.e check  		: determine if user has enough funds
    effect 		: increase or decrease user balance that prevents re-entrancy
    interaction		: send the call to the user/contract account
    
10. User can withdraw arbitrariy amount of tokens , greater than they might have deposited.


### Pause

11. Anyone who has a both operator and governance role , can pause the withdraw:

```solidity
function pause() external {
       // operator or gov
       require(msg.sender == _operator && msg.sender == _governance, 'unauthorized');

       _paused = true;
   }
   
```

12. The comment is misleading

```solidity
       // operator or gov

```


### Unpause

it is simple.

### Change Governance 

13. Damn , anyone can change the governance to their own address or to someone else's address.


```solidity

   function changeGovernance(address governance) external {
       _governance = governance;
   }

```
so any function that is ruled by governance like unpause , can be acccessed by anyone . so anyone can unpause the withdraws at anytime



### Realized later 

- The `_token` is choosen by the operator to be a trustworthy token do not allowing re-entrancy.




## Race #12

Learned Cleared my concerns about Re-Entrancy in ERC20 and ERC721  the hard way ( lost 1 marks : / )

But that's part of learning. <br/>

In this Race, I've Researched, cleared confusion asked questions to other auditors. <br/>
Waiting answers

We are Getting better and Closer : )

![Screenshot from 2023-06-25 12-17-22](https://github.com/umaresso/Secureum-Races/assets/71306738/d6d8c6b6-d802-4162-b7c7-3c07f647a2b0)


### Write-up 


Race Link : https://ventral.digital/posts/2022/12/6/race-12-of-the-secureum-bootcamp-epoch

Code Review : 

1. Solc 0.8.x , over/underflow is not possible unless it happens in unchecked

2. wrong import as of now `import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";
`

### Token V1 Fallback 

3. fallback gives the entire power to msg.sender to execute anything with a condition that call result should be true.

```solidity

fallback() external {
        if (hasRole(MIGRATOR_ROLE, msg.sender)) {
            (bool success, bytes memory data) = msg.sender.delegatecall(msg.data);
            require(success, "MIGRATION CALL FAILED");
            assembly {
                return(add(data, 32), mload(data))
            }
        }
    }

```

## Vault

### Constructor


4. ` address public UNDERLYING` can be immutable because it's value is set in the constructor only
 
5. sweep is really malicicious .
	- external visibility ( can be called by anyone)
	- anyone can pass the underlying token address , ( easy to get from block explorers for this contract Txs) and withdraw all the balance.
	
```solidity

   function sweep(address token) external {
        require(UNDERLYING != token, "can't sweep underlying");
        IEERC20(token).transfer(msg.sender, IEERC20(token).balanceOf(address(this)));
    }
    
```

## Token V2

### Fallback

6. Always Wrong Fallback condition `(address(this) != TOKEN_V1)` the new deployed address can never be the one deployed earlier unless the user determined it's address and fed into the constructor as TokenV1 , if that's so , then the entire logic is flawed.

### Realized Later

1. `safeTransferFrom` re-entrancy happens in just NFTs.
2. `safeTransferFrom` ERC20 implementation demands 4 parameters now ,
   ```solidity
   (address tokenAddress,address sender,address receiver,address reiceiver)
   ```



## Race #16
Race #16 ( FlashLoans, my First Encounter)

Scored 4/8.

Scores are decreasing but knowledge is uplifting.

Let's hope we'll get to it.

![Screenshot from 2023-06-26 06-49-19](https://github.com/umaresso/Secureum-Races/assets/71306738/d2e59ead-ef34-47ae-9a8f-27bf37444438)


### Write-up

Race Link:[Click here](https://ventral.digital/posts/2023/4/1/race-16-of-the-secureum-bootcamp-epoch-infinity)


#### Compiler Version

1. Solc 0.8.x ensures no over/under flow occurs unless something happens in unchecked.

#### Imports 

2. First time seeing this standard `IERC3156FlashLender`

### flashFee

3. The fee is static , what i would say is to have a dynamic percentage-like fee instead of simple uint value.

#### flashLoan

4. Things are working really fine unless we find this line 

```soliidty

 receiver.onFlashLoan(msg.sender, token, amount, fee, data)


```
This would probably allow re-entrancy and the hacker will make a big amount of approval to themselves. But the next lines mitigates the issue:


```soiidity


       if (IERC20(token).allowance(address(this), address(receiver)) > oldAllowance) {
           IERC20(token).approve(address(receiver), oldAllowance);
       }

```

This is great.


### Realized Later

5. ERC777 have sender-fallbacks that are dangerous for re-entrancy.




## Race 18

The Final Past Race - Race #18 

Scored 3/8 on the first attempt and then went back, re-calibrated, and solved again.

This race has taught me things like

1. Generalized Questions
2. try to quantify the losses
3. variable shadowing
4. wrong interface declaration

Done with past, amazing learning

Next Race - Live Race 19 Lesgo🚀🔥

Write up is below!

![image](https://github.com/umaresso/Secureum-Races/assets/71306738/99263761-9f42-4352-bd2b-cb6adfe4bef0)



### Write-up

Race Link : [Check here](https://ventral.digital/posts/2023/5/29/race-18-of-the-secureum-bootcamp-epoch-infinity)

#### Comments
comments are sometimes misleading, they are emphasizing that there are no Re-Entrancy bugs, does not mean there ain't ones.

#### IERC20 interface
1. The decimals function is non-standard

```solidity

function decimals() external view returns (uint256);

```
it should return uint8 instead.

#### modifier
2. The modifier is implemented incorrectly, it will allow the re-entrancy as `is_reenterant` variable will always be >=1

#### withdraw

3. Missing check on call result while sending funds

```solidity
_has_enough_balance(amount, balances[msg.sender]);
        minted[msg.sender] = amount * decimals_factor;
```


### Depoist ( Realized later )
4. It seems that anyone can deposit for anyone but seeing the `internal` visibility falsifies the assumption.

```solidity
  function deposit(uint amount) internal{
        balances[msg.sender] += amount;
    }

```

### _has_enough_balance
5. Precision loss will result in miscalculations.


```solidity

  uint256 required_balance = (desired_tokens / 10) * decimals_factor;
  
```


### Realized Later

6. Understand the question fully, it is a generalized or more specific
7. `No overflow can ever occur in a contract compiled with Solc version 0.8` should be stated with the exception of unchecked blocks.
8. return variables shadowing can return zero.





# Races Without Writeups 

## Race #1

![Screenshot-1](https://github.com/umaresso/Secureum-Races/assets/71306738/f4287a69-2c0e-4235-888c-5c9d3ddfc83c)





## Race #4

![Screenshot-12](https://github.com/umaresso/Secureum-Races/assets/71306738/d12a9f0e-63c2-4d53-9cf2-736a4c460722)

## Race #6

![Screenshot from 2023-06-23 16-18-14](https://github.com/umaresso/Secureum-Races/assets/71306738/1c2d4f03-5dd7-4c2a-a3ee-8034a1b493be)

## Race #7

![Screenshot-29](https://github.com/umaresso/Secureum-Races/assets/71306738/2806e4aa-049f-44ec-b842-ce8016cbddea)

## Race #8
![Screenshot-33](https://github.com/umaresso/Secureum-Races/assets/71306738/ed403eab-3383-452e-9a68-6f7eac7fa6a7)

## Race #9
So , in Race #9 , i came across Proxy Patterns.
This is something I knew I need to dig deeper .
Studied stuff , came back , solved some questions but I was not able to get a high score !

Score : 3.66 / 8

I need to focus on Proxy contracts : /
![Screenshot from 2023-06-24 07-17-00](https://github.com/umaresso/Secureum-Races/assets/71306738/28fae481-9065-4c6d-9cf3-7fb16628ebf3)


## Race #10
This time, I came across an abstract Contract (in the sense that its functionality was not clear beforehand). <br/>
I had to infer certain flow patterns and vulnerabilities that are not common in the space ( or new to me ).

Was able to score sufficiently but i need to level up my game !

![Screenshot-48](https://github.com/umaresso/Secureum-Races/assets/71306738/c2cdc36c-24f7-418b-9282-3841a6773e3a)

## Race #13

![image](https://github.com/umaresso/Secureum-Races/assets/71306738/e742d7ae-09d9-4740-8719-5117ccab5e5e)

### Write-up :

Race #13 was absolutely Assemblish ....
lost 3 marks due to it so had to learn assembly by an article and come back to understand my pitfalls.

Thank God I understood my mistakes
So that they can't re-enter in my mind in next contests.

No write-up for this race : )

## Race #14
So , this one was pure DeFi ..
A almost lost 60% of the questions.

Opened google, researched, solved again ..

Read the code solution again and then I was able to understand and score some 6/8 

This 6/8 is irrelevant as it was my second attempt but the stuff learned was great!
![Race#14](https://github.com/umaresso/Secureum-Races/assets/71306738/ccf1f67b-c317-4a98-9ac1-dc8e5da55f16)

## Race #15
It was a good race when I tried to solve the it in under 16 minutes.

Solved 7 questions in 16 minutes, and the last took 2 minutes more ...
( On Mobile , with absence of electricity)

Scored 5/8 .

I know I can do better but these races are making me learn !
![Race#15](https://github.com/umaresso/Secureum-Races/assets/71306738/5dd63329-6342-464c-8136-2f4808f1d148)

## Race #17
Here's how I've put my experience with the race in my tweet :

"
Race # 17

This one was a beast ( designed by Sir @HickupH  )

Introduced several things to me like event emission unpredictability, thinking outside of the box, deployed vs runtime costs, and much more.

Although I secured just 2/8 I really treasure this one.

Thank you Hickup

"
![image](https://github.com/umaresso/Secureum-Races/assets/71306738/0d8e5767-4ffd-460c-a3e2-aa2af4b08c55)


<div>
	<p align="center">---------------------------------------------- The Start -----------------------------------------------------------
</p>
</div>















