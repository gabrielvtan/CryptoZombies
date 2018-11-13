# ERC721 & Crypto-Collectibles
## Chapter 1: Tokens on Ethereum
- A token on Ethereum is basically just a smart contract that follows some common rules — namely it implements a standard set of functions that all other token contracts share, such as transfer(address _to, uint256 _value) and balanceOf(address _owner).
- Internally the smart contract usually has a mapping, mapping(address => uint256) balances, that keeps track of how much balance each address has.
- So basically a token is just a contract that keeps track of who owns how much of that token, and some functions so those users can transfer their tokens to other addresses.
- Since all ERC20 tokens share the same set of functions with the same names, they can all be interacted with in the same ways.
- This means if you build an application that is capable of interacting with one ERC20 token, it's also capable of interacting with any ERC20 token. 
- When an exchange adds a new ERC20 token, really it just needs to add another smart contract it talks to. Users can tell that contract to send tokens to the exchange's wallet address, and the exchange can tell the contract to send the tokens back out to users when they request a withdraw.
- There's another token standard that's a much better fit for crypto-collectibles like CryptoZombies — and they're called ERC721 tokens.
- ERC721 tokens are not interchangeable since each one is assumed to be unique, and are not divisible. You can only trade them in whole units, and each one has a unique ID. So these are a perfect fit for making our zombies tradeable.
- Note that using a standard like ERC721 has the benefit that we don't have to implement the auction or escrow logic within our contract that determines how players can trade / sell our zombies. If we conform to the spec, someone else could build an exchange platform for crypto-tradable ERC721 assets, and our ERC721 zombies would be usable on that platform. So there are clear benefits to using a token standard instead of rolling your own trading logic.
- We're going to store all the ERC721 logic in a contract called ZombieOwnership.

## Chapter 2: ERC721 Standard, Multiple Inheritance
- ERC721 Standard:
```solidity
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);

  function balanceOf(address _owner) public view returns (uint256 _balance);
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
  function transfer(address _to, uint256 _tokenId) public;
  function approve(address _to, uint256 _tokenId) public;
  function takeOwnership(uint256 _tokenId) public;
}
```
- The ERC721 standard is currently a draft, and there is no officially agreed-upon implementation yet. For this tutorial we're using the current version from OpenZeppelin's library, but it is possible it will change in the future before its official release. So consider this one possible implementation, but don't take it as the official standard for ERC721 tokens.

## Chapter 3: balanceOf & ownerOf
### balanceOf
```solidity
  function balanceOf(address _owner) public view returns (uint256 _balance);
```
- This function simply takes an address, and retuns how many tokens that address owns
- In our case, our "tokens" are Zombies.

### ownerOf
```solidity
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
```
- This function takes a token ID (in our case, a Zombie ID), and returns the address of the person who owns it.
- Again, this is very straightforward for us to implement, since we already have a mapping in our DApp that stores this information. We can implement this function in one line, just a return statement.

## Chapter 4: Refactoring
- Remember, we're using the ERC721 token standard, which means other contracts will expect our contract to have functions with these exact names defined. That's what makes these standards useful — if another contract knows our contract is ERC721-compliant, it can simply talk to us without needing to know anything about our internal implementation decisions.

## Chapter 5: ERC721: Transfer Logic 
- Note that the ERC721 spec has 2 different ways to transfer tokens:
```solidity
function transfer(address _to, uint256 _tokenId) public;
function approve(address _to, uint256 _tokenId) public;
function takeOwnership(uint256 _tokenId) public;
```
1. The first way is the token's owner calls transfer with the address he wants to transfer to, and the _tokenId of the token he wants to transfer.
2. The second way is the token's owner first calls approve, and sends it the same info as above. The contract then stores who is approved to take a token, usually in a mapping (uint256 => address). Then when someone calls takeOwnership, the contract checks if that msg.sender is approved by the owner to take the token, and if so it transfers the token to him.
- If you'll notice, both transfer and takeOwnership will contain the same transfer logic, just in reverse order. (In one case the sender of the token calls the function; in the other the receiver of the token calls it).
- So it makes sense for us to abstract this logic into its own private function, _transfer, which is then called by both functions. That way we don't repeat the same code twice.

## Chapter 6: ERC721: Transfer Cont'd 
- That was the difficult part — now implementing the public transfer function will be easy, since our _transfer function already does almost all the heavy lifting.

## Chapter 7: ERC721: Approve 
- Remember, with approve / takeOwnership, the transfer happens in 2 steps:
1. You, the owner, call approve and give it the address of the new owner, and the _tokenId you want him to take
2. The new owner calls takeOwnership with the _tokenId, the contract checks to make sure he's already been approved, and then transfers him the token.
- Because this happens in 2 function calls, we need a data structure to store who's been approved for what in between function calls.

## Chapter 8: ERC321: takeOwnership
- The final function, takeOwnership, should simply check to make sure the msg.sender has been approved to take this token / zombie, and call _transfer if so.

## Chapter 9: Preventing Overflows
- There are extra features we may want to add to our implementation, such as some extra checks to make sure users don't accidentally transfer their zombies to address 0 (which is called "burning" a token — basically it's sent to an address that no one has the private key of, essentially making it unrecoverable)
- Or to put some basic auction logic in the DApp itself. 

### Using SafeMath
- To prevent stack overflows or underflows, OpenZeppelin has created a library called SafeMath that prevents these issues by default.

## Chapter 10: SafeMath Part 2
- First we have the library keyword — libraries are similar to contracts but with a few differences. For our purposes, libraries allow us to use the using keyword, which automatically tacks on all of the library's methods to another data type
- all uint256s are now SafeMath uint256s
- assert is similar to require, where it will throw an error if false. The difference between assert and require is that require will refund the user the rest of their gas when a function fails, whereas assert will not. So most of the time you want to use require in your code; assert is typically used when something has gone horribly wrong with the code

## Chapter 11 - 12: SafeMath Part 3 - 4
- However we have a slight problem — winCount and lossCount are uint16s, and level is a uint32. So if we use SafeMath's add method with these as arguments, it won't actually protect us from overflow since it will convert these types to uint256:
- This means we're going to need to implement 2 more libraries to prevent overflow/underflows with our uint16s and uint32s. We can call them SafeMath16 and SafeMath32.

## Chapter 13: Comments
- In the next lessons, we'll look at how to deploy the code to Ethereum, and how to interact with it with Web3.js.
- The standard in the Solidity community is to use a format called natspec, which looks like this:
```solidity
/// @title A contract for basic math operations
/// @author H4XF13LD MORRIS 💯💯😎💯💯
/// @notice For now, this contract just adds a multiply function
contract Math {
  /// @notice Multiplies 2 numbers together
  /// @param x the first uint.
  /// @param y the second uint.
  /// @return z the product of (x * y)
  /// @dev This function does not currently check for overflows
  function multiply(uint x, uint y) returns (uint z) {
    // This is just a normal comment, and won't get picked up by natspec
    z = x * y;
  }
}
```
- @title and @author are straightforward.
- @notice explains to a user what the contract / function does. @dev is for explaining extra details to developers.
- @param and @return are for describing what each parameter and return value of a function are for.

## Recap
In this lesson we learned about:
- Tokens, the ERC721 standard, and tradable assets/zombies
- Libraries and how to use them
- How to prevent overflows and underflows using the SafeMath library
- Commenting your code and the natspec standard
This lesson concludes our game's Solidity code! (For now — we may add even more lessons in the future).