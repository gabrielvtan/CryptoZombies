# Solidity Basics

## Structs 
- Similar to classes - this is how you create complex data types
- Structs are constructed within a contract

## Arrays
- Fixed arrays & dynamic arrays exist in solidity
- You can also create an array of structs. Creating a dynamic array of stucts can be useful for storing structured data in your contract, kind of like a database

## Function Declaration
- It's convention (but not required) to start function parameter variable names with an underscore (_) in order to differentiate them from global variables. 

## Private/Public Functions
- Thus it's good practice to mark your functions as private by default, and then only make public the functions you want to expose to the world.
- And as with function parameters, it's convention to start private function names with an underscore (_).

Example of Declaring a Private Function
```solidity
uint[] numbers;

function _addToArray(uint _number) private {
  numbers.push(_number);
}
```

## Return Values
- To return a value from a function, the declaration looks like this:
``` solidity
string greeting = "What's up dog";

function sayHello() public returns (string) {
  return greeting;
}
```
- In Solidity, the function declaration contains the type of the return value (in this case string).
- The above function doesn't actually change state in Solidity — e.g. it doesn't change any values or write anything.
- So in this case we could declare it as a view function, meaning it's only viewing the data but not modifying it
- Solidity also contains pure functions, which means you're not even accessing any data in the app

## Events
- Events are a way for your contract to communicate that something happened on the blockchain to your app front-end, which can be 'listening' for certain events and take action when they happen

Example
``` solidity
// declare the event
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public {
  uint result = _x + _y;
  // fire an event to let the app know the function was called:
  emit IntegersAdded(_x, _y, result);
  return result;
}
```






