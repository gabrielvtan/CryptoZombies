# Zombie Battlegroup
## Chapter 1: Payable
1. We have visibility modifiers that control when and where the function can be called from: private means it's only callable from other functions inside the contract; internal is like private but can also be called by contracts that inherit from this one; external can only be called outside the contract; and finally public can be called anywhere, both internally and externally.
2.  We also have state modifiers, which tell us how the function interacts with the BlockChain: view tells us that by running the function, no data will be saved/changed. pure tells us that not only does the function not save any data to the blockchain, but it also doesn't read any data from the blockchain. Both of these don't cost any gas to call if they're called externally from outside the contract (but they do cost gas if called internally by another function).
3. Then we have custom modifiers, which we learned about in Lesson 3: onlyOwner and aboveLevel, for example. For these we can define custom logic to determine how they affect a function.

### The payable Modifier
- payable functions are part of what makes Solidity and Ethereum so cool — they are a special type of function that can receive Ether.
- But in Ethereum, because both the money (Ether), the data (transaction payload), and the contract code itself all live on Ethereum, it's possible for you to call a function and pay money to the contract at the same time.
- This allows for some really interesting logic, like requiring a certain payment to the contract in order to execute a function.
```solidity
contract OnlineStore {
  function buySomething() external payable {
    // Check to make sure 0.001 ether was sent to the function call:
    require(msg.value == 0.001 ether);
    // If so, some logic to transfer the digital item to the caller of the function:
    transferThing(msg.sender);
  }
}
```
- Here, msg.value is a way to see how much Ether was sent to the contract, and ether is a built-in unit.
- What happens here is that someone would call the function from web3.js (from the DApp's JavaScript front-end) as follows:
```solidity
// Assuming `OnlineStore` points to your contract on Ethereum:
OnlineStore.buySomething({from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001)})
```
- If a function is not marked payable and you try to send Ether to it as above, the function will reject your transaction.

## Chapter 2: Withdraws
- After you send Ether to a contract, it gets stored in the contract's Ethereum account, and it will be trapped there — unless you add a function to withdraw the Ether from the contract.
- You can write a function to withdraw Ether from the contract as follows:
```solidity
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    address _owner =  owner();
    _owner.transfer(address(this).balance);
  }
}
```
- Note that we're using owner() and onlyOwner from the Ownable contract, assuming that was imported.
- You can transfer Ether to an address using the transfer function, and address(this).balance will return the total balance stored on the contract. So if 100 users had paid 1 Ether to our contract, address(this).balance would equal 100 Ether.
- You can use transfer to send funds to any Ethereum address. For example, you could have a function that transfers Ether back to the msg.sender if they overpaid for an item:
```solidity
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
```
- Or in a contract with a buyer and a seller, you could save the seller's address in storage, then when someone purchases his item, transfer him the fee paid by the buyer
```solidity
seller.transfer(msg.value)
```

## Chapter 3: Zombie Battles 
- Create new .sol file for Zombie Battles

## Chapter 4: Random Numbers
- In solidity, you can't create random numbers safely 

### Random Number generation via keccak256
- The best source of randomness we have in Solidity is the keccak256 has function
- We could do the following to create a random number:
```solidity
// Generate a random number between 1 and 100:
uint randNonce = 0;
uint random = uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % 100;
randNonce++;
uint random2 = uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % 100;
```
- What this would do is take the timestamp of now, the msg.sender, and an incrementing nonce (a number that is only ever used once, so we don't run the same hash function with the same input parameters twice).
- It would then "pack" the inputs and use keccak to convert them to a random hash. Next, it would convert that hash to a uint, and then use % 100 to take only the last 2 digits. This will give us a totally random number between 0 and 99.

## This method is vulnerable to attack by a dishonest node
- Once a node has solved the PoW, the other nodes stop trying to solve the PoW, verify that the other node's list of transactions are valid, and then accept the block and move on to trying to solve the next block.
- **This makes our random number function exploitable.**
- If I were running a node, I could publish a transaction only to my own node and not share it. I could then run the coin flip function to see if I won — and if I lost, choose not to include that transaction in the next block I'm solving. I could keep doing this indefinitely until I finally won the coin flip and solved the next block, and profit.

### So how do we generate random numbers safely in Ethereum
- Because the entire contents of the blockchain are visible to all participants, this is a hard problem, and its solution is beyond the scope of this tutorial. You can read this StackOverflow thread for some ideas.
- One idea would be to use an oracle to access a random number function from outside of the Ethereum blockchain.
- So while this random number generation is NOT secure on Ethereum, in practice unless our random function has a lot of money on the line, the users of your game likely won't have enough resources to attack it.
- Because we're just building a simple game for demo purposes in this tutorial and there's no real money on the line, we're going t
- to accept the tradeoffs of using a random number generator that is simple to implement, knowing that it isn't totally secure.

## Chapter 5: Zombie Fightin' 
- Now that we have a source of some randomness in our contract, we can use it in our zombie battles to calculate the outcome

Our zombie battles will work as follows:
- You choose one of your zombies, and choose an opponent's zombie to attack.
- If you're the attacking zombie, you will have a 70% chance of winning. The defending zombie will have a 30% chance of winning.
- All zombies (attacking and defending) will have a winCount and a lossCount that will increment depending on the outcome of the battle.
- If the attacking zombie wins, it levels up and spawns a new zombie.
- If it loses, nothing happens (except its lossCount incrementing).
- Whether it wins or loses, the attacking zombie's cooldown time will be triggered.

## Chapter 6-7: Refactoring Common Logic 
- Whoever calls our attack function — we want to make sure the user actually owns the zombie they're attacking with.
- We've done this check multiple times now in previous lessons. In changeName(), changeDna(), and feedAndMultiply(), we used the following check:
```solidity
require(msg.sender == zombieToOwner[_zombieId]);
```
- This is the same logic we'll need for our attack function. Since we're using the same logic multiple times, let's move this into its own modifier to clean up our code and avoid repeating ourselves.

## Chapter 8: Back to Attack
- We're going to continue defining our attack function, now that we have the ownerOf modifier to use.
- The first thing our function should do is get a storage pointer to both zombies so that we can more easily interact with them

## Chapter 9: Zombie Wins and Losses
- For our zombie game, we're going to want to keep track of how many battles our zombies have won and lost. That way we can maintain a "zombie leaderboard" in our game state.
- We could store this data in a number of ways in our DApp — as individual mappings, as leaderboard Struct, or in the Zombie struct itself
- Each has its own benefits and tradeoffs depending on how we intend on interacting with the data. In this tutorial, we're going to store the stats on our Zombie struct for simplicity, and call them winCount and lossCount.

## Chapter 10: Zombie Victory
- Now that we have a winCount and lossCount, we can update them depending on which zombie wins the fight.
- In chapter 6 we calculated a random number from 0 to 100. Now let's use that number to determine who wins the fight, and update our stats accordingly.

## Chapter 11: Zombie Loss
- Now that we've coded what happens when your zombie wins, let's figure out what happens when it loses.
- In our game, when zombies lose, they don't level down — they simply add a loss to their lossCount, and their cooldown is triggered so they have to wait a day before attacking again.
- To implement this logic, we'll need an else statement.
```solidity
if (zombieCoins[msg.sender] > 100000000) {
  // You rich!!!
} else {
  // We require more ZombieCoins...
}
```
