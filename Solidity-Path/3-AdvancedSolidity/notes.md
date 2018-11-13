# Advanced Solidity

##Chapter 1: Immutability of Contracts
- After you deploy a contract to Ethereum, it is immutable
- No one can later change that contract and give you unexpected results

### External Dependencies
- It often makes sense to have functions that will allow you to update key portion of the Dapp

##Chapter 2: Ownable Contracts
- When functions are set to external, that means anyone who called the function could chnage the address of the CryptoKitties contract, and break our app for all its users
- To handle cases like this, one cmmon practice is to make contracts 'Ownable' - meaning they have an owner(you) who has special privileges
- Constructors: function Ownable() is a constructor, which is an optional special function that has the same name as the contract. It will get executed only one time, when the contract is first created.
- Function Modifiers: modifier onlyOwner(). Modifiers are kind of half-functions that are used to modify other functions, usually to check some requirements prior to execution. In this case, onlyOwner can be used to limit access so only the owner of the contract can run this function. We'll talk more about function modifiers in the next chapter, and what that weird _; does.

So the Ownable contract basically does the following:
1. When a contract is created, its constructor sets the owner to msg.sender (the person who deployed it)
2. It adds an onlyOwner modifier, which can restrict access to certain functions to only the owner
3. It allows you to transfer the contract to a new owner

- onlyOwner is such a common requirement for contracts that most Solidity DApps start with a copy/paste of this Ownable contract, and then their first contract inherits from it.

## Chapter 3: onlyOwner Function Modifier
- A function modifier looks just like a function, but uses the keyword modifier instead of the keyword function. And it can't be called directly like a function can — instead we can attach the modifier's name at the end of a function definition to change that function's behavior.
- In the case of onlyOwner, adding this modifier to a function makes it so only the owner of the contract (you, if you deployed it) can call that function.

## Chapter 4: Gas
### Struct packing to save gas
- If you have multiple uints inside a struct, using a smaller-sized uint when possible will allow Solidity to pack these variables together to take up less storage. For example:
```solidity
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` will cost less gas than `normal` because of struct packing
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```
- For this reason, inside a struct you'll want to use the smallest integer sub-types you can get away with.

##Chapter 5: Time Units
### Time Units
- The variable now will return the current unix timestamp of the latest block 
Here's an example of how these time units can be useful:
```solidity
uint lastUpdated;

// Set `lastUpdated` to `now`
function updateTimestamp() public {
  lastUpdated = now;
}

// Will return `true` if 5 minutes have passed since `updateTimestamp` was 
// called, `false` if 5 minutes have not passed
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
```

## Chapter 6: Zombie Cooldowns
We are going to modify our feedAndMultiply such that:
1. Feeding triggers a zombie's cooldown
2. Zombies can't feed on kitties until their cooldown period has passed

### Passing Structs as arguments
- You can pass a storage pointer to a struct as an argument to a private or internal function. This is useful, for example, for passing around our Zombie structs between functions.
```solidity
function _doStuff(Zombie storage _zombie) internal {
  // do stuff with _zombie
}
```
- This way we can pass a reference to our zombie into a function instead of passing in a zombie ID and looking it up.

## Chapter 7: Public Function and Security
- An important security practice is to examine all your public and external functions, and try to think of ways users might abuse them. Remember — unless these functions have a modifier like onlyOwner, any user can call them and pass them any data they want to.

## Chapter 8: More on Function Modifiers
- Previously we looked at the simple example of onlyOwner. But function modifiers can also take arguments. For example:
```solidity
// A mapping to store a user's age:
mapping (uint => uint) public age;

// Modifier that requires this user to be older than a certain age:
modifier olderThan(uint _age, uint _userId) {
  require(age[_userId] >= _age);
  _;
}

// Must be older than 16 to drive a car (in the US, at least).
// We can call the `olderThan` modifier with arguments like so:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // Some function logic
}
```

## Chapter 9: Zombie Modifiers
- This is how we are able to change the ids of a given zombie above a given level with the aboveLevel() modifier

## Chapter 10: Saving Gas With 'View' Functions
- This function will only need to read data from the blockchain, so we can make it a view function. Which brings us to an important topic when talking about gas optimization

### View functions don't cost gas
- view functions don't cost any gas when they're called externally by a user.
- This is because view functions don't actually change anything on the blockchain – they only read the data.
- So marking a function with view tells web3.js that it only needs to query your local Ethereum node to run the function, and it doesn't actually have to create a transaction on the blockchain (which would need to be run on every single node, and cost gas).
- If a view function is called internally from another function in the same contract that is not a view function, it will still cost gas. This is because the other function creates a transaction on Ethereum, and will still need to be verified from every node. So view functions are only free when they're called externally.

## Chapter 11: Storage is Expensive
- One of the more expensive operations in Solidity is using storage

### Declaring arrays in memory
- In order to keep costs down, you want to avoid writing data to storage except when absolutely necessary. Sometimes this involves seemingly inefficient programming logic — like rebuilding an array in memory every time a function is called instead of simply saving that array in a variable for quick lookups.
- You can use the memory keyword with arrays to create a new array inside a function without needing to write anything to storage.
- The array will only exist until the end of the function call, and this is a lot cheaper gas-wise than updating an array in storage — free if it's a view function called externally.
```solidity
function getArray() external pure returns(uint[]) {
  // Instantiate a new array in memory with a length of 3
  uint[] memory values = new uint[](3);
  // Add some values to it
  values.push(1);
  values.push(2);
  values.push(3);
  // Return the array
  return values;
}
```
- Memory arrays must be created with a length argument (in this example, 3). They currently cannot be resized like storage arrays can with array.push(), although this may be changed in a future version of Solidity

## Chapter 12: For Loops
- Since view functions don't cost gas when called externally, we can simply use a for-loop in getZombiesByOwner to iterate the entire zombies array and build an array of the zombies that belong to this specific owner.
- Then our transfer function will be much cheaper, since we don't need to reorder any arrays in storage, and somewhat counter-intuitively this approach is cheaper overall.
### Using for loops
The syntax of for loops in Solidity is similar to JavaScript
```solidity
function getEvens() pure external returns(uint[]) {
  uint[] memory evens = new uint[](5);
  // Keep track of the index in the new array:
  uint counter = 0;
  // Iterate 1 through 10 with a for loop:
  for (uint i = 1; i <= 10; i++) {
    // If `i` is even...
    if (i % 2 == 0) {
      // Add it to our array
      evens[counter] = i;
      // Increment counter to the next empty index in `evens`:
      counter++;
    }
  }
  return evens;
}
```
## RECAP
Let's recap:
- We've added a way to update our CryptoKitties contracts
- We've learned to protect core functions with onlyOwner
- We've learned about gas and gas optimization
- We added levels and cooldowns to our zombies
- We now have functions to update a zombie's name and DNA once the zombie gets above a certain level
- And finally, we now have a function to return a user's zombie army