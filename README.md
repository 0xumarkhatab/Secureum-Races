# Secureum-Races
This repository will contain my results of different Secureum Races for the record and write-ups for new-bies ( thinking if something is unclear for me , might be for others too)
# Race #1

![Screenshot-1](https://github.com/umaresso/Secureum-Races/assets/71306738/f4287a69-2c0e-4235-888c-5c9d3ddfc83c)
# Race #2

![Screenshot-3](https://github.com/umaresso/Secureum-Races/assets/71306738/147e25a5-3514-4af7-9cf4-3f5412b9ef6a)

## Write up

Race Link : https://ventral.digital/posts/2021/11/28/secureum-bootcamp-audit-findings-101-quiz


Code Review :

- Anyone can see the mapping - Not really a vulnerbility

- onlyWhenOpen modifier checks for balance of the contract - vulnerable because anyone can send Force Ether.

- Constructor ensures that admin is zero address `require(_admin == address(0));` . This is Funny : / - Vulberable ofcourse because no one will ever be able to deploy contract without zero admin and then no one call call the admin access control functions because admin is zero address and that is not controlled by any 

- Non re-entrancy guard should come before any other modiifer 
` function join() external payable onlyWhenOpen nonReentrant `
 
 The value should not be stricyly equal , this can cause discrepencies :

`require(msg.value == membershipFee, 'InSecureumDAO: Incorrect ETH amount');`

rather consider doing something like >= so that people who accidently sent one more wei in the  Tx can still become member and not face the overhead of sending the tx again cause it will fail in the current scenario.


- `whenNotPaused` modifier is not decalared
- Remove all members function does not necessarily remove all the members



Questions:

For most of the questions , the explanation was more than enough !

But here are few i've struggled with:

2.-

 what is "Guarded launch via circuit breakers"? - it is a mechanism in which system can be paused in the extreme situation of crisis.

5.- 

Why not Option A?

`A. join() to not disclose msg.sender while joining the DAO`

We do not nessecarily have to hide membership like membership in a club . Should be public .


Why Not Option B?

`B. createVote() to not disclose the possible outcomes during creation`


Any member can create the vote and it does not violate and privacy rules of the individual.
Rather votes should be seen by public for transparency and deciding where to cast there vote ?


6.-


Why not Option C ?


`getWinningOutcome() for existing _voteId`

In solidity , when we try to access a variable at a position in mapping , that is not initialized before , it returns the default value of the value type :

Here in the mapping:


`mapping (uint256 => uint8) public winningOutcome;`


Key type is   - uint256
Value type is - uint8

default values for uint8 is 0 

So if we try to access a vote Id's winning outcome that does not exist in the mapping , we will still return the zero outcome which is not any outcome.

That's why we don't need to check if vote id exists in this function.

And It's a wrap !







