# Front-End & Web3.js
## Chapter 1: Intro to Web3.js
- Remember, the Ethereum network is made up of nodes, with each containing a copy of the blockchain. When you want to call a function on a smart contract, you need to query one of these nodes and tell it
1. The address of the smart contract
2. The function you want to call, and
3. The variables you want to pass to that function.
- Ethereum nodes only speak a language called JSON-RPC, which isn't very human-readable. 
- Instead of needing to construct the above query, calling a function in your code will look something like this:
```solidity
CryptoZombies.methods.createRandomZombie("Vitalik Nakamoto 🤔")
  .send({ from: "0xb60e8dd61c5d32be8058bb8eb970870f07233155", gas: "3000000" })
```

## Chapter 2: Web3 Providers
- The first thing we need is a Web3 Provider.
- Setting a Web3 Provider in Web3.js tells our code which node we should be talking to handle our reads and writes. It's kind of like setting the URL of the remote web server for your API calls in a traditional web app.

## Infura
- Infura is a service that maintains a set of Ethereum nodes with a caching layer for fast reads, which you can access for free through their API. 
- Using Infura as a provider, you can reliably send and receive messages to/from the Ethereum blockchain without needing to set up and maintain your own node.
- You can set up Web3 to use Infura as your web3 provider as follows:
```javascript
var web3 = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));
```
- However, since our DApp is going to be used by many users — and these users are going to WRITE to the blockchain and not just read from it — we'll need a way for these users to sign transactions with their private key.

## MetaMask
- Metamask is a browser extension for Chrome and Firefox that lets users securely manage their Ethereum accounts and private keys, and use these accounts to interact with websites that are using Web3.js.
- Metamask uses Infura's servers under the hood as a web3 provider, just like we did above — but it also gives the user the option to choose their own web3 provider. So by using Metamask's web3 provider, you're giving the user a choice, and it's one less thing you have to worry about in your app.

## Using Metamask's web3 provider
- Metamask injects their web3 provider into the browser in the global JavaScript object web3. So your app can check to see if web3 exists, and if it does use web3.currentProvider as its provider.
- Here's some template code provided by Metamask for how we can detect to see if the user has Metamask installed, and if not tell them they'll need to install it to use our app:
```javascript
window.addEventListener('load', function() {

  // Checking if Web3 has been injected by the browser (Mist/MetaMask)
  if (typeof web3 !== 'undefined') {
    // Use Mist/MetaMask's provider
    web3js = new Web3(web3.currentProvider);
  } else {
    // Handle the case where the user doesn't have web3. Probably 
    // show them a message telling them to install Metamask in 
    // order to use our app.
  }

  // Now you can start your app & access web3js freely:
  startApp()

})
```
- You can use this boilerplate code in all the apps you create in order to require users to have Metamask to use your DApp.

## Chapter 3: Talking to Contracts
- Now that we've initialized Web3.js with MetaMask's Web3 provider, let's set it up to talk to our smart contract.
- Web3.js will need 2 things to talk to your contract: its address and its ABI.

### Contract Address
- After you deploy your contract, it gets a fixed address on Ethereum where it will live forever.
- You'll need to copy this address after deploying in order to talk to your smart contract.

### Contract ABI
- ABI stands for Application Binary Interface. Basically it's a representation of your contracts' methods in JSON format that tells Web3.js how to format function calls in a way your contract will understand.
- When you compile your contract to deploy to Ethereum (which we'll cover in Lesson 7), the Solidity compiler will give you the ABI, so you'll need to copy and save this in addition to the contract address.

### Instantiating a Web3.js Contract
- Once you have your contract's address and ABI, you can instantiate it in Web3 as follows:
```javascript
// Instantiate myContract
var myContract = new web3js.eth.Contract(myABI, myContractAddress);
```

## Chapter 4: Calling Contract Functions 
- Web3.js has two methods we will use to call functions on our contract: call and send
### Call
- call is used for view and pure functions. It only runs on the local node, and won't create a transaction on the blockchain.
- Using Web3.js, you would call a function named myMethod with the parameter 123 as follows:
```javascript
myContract.methods.myMethod(123).call()
```

### Send 
- send will create a transaction and change data on the blockchain. You'll need to use send for any functions that aren't view or pure.
- Using Web3.js, you would send a transaction calling a function named myMethod with the parameter 123 as follows:
```javascript
myContract.methods.myMethod(123).send()
```
- In Solidity, when you declare a variable public, it automatically creates a public "getter" function with the same name. So if you wanted to look up the zombie with id 15, you would call it as if it were a function: zombies(15).
- Here's how we would write a JavaScript function in our front-end that would take a zombie id, query our contract for that zombie, and return the result:
```javascript
function getZombieDetails(id) {
  return cryptoZombies.methods.zombies(id).call()
}

// Call the function and do something with the result:
getZombieDetails(15)
.then(function(result) {
  console.log("Zombie 15: " + JSON.stringify(result));
});
```
- cryptoZombies.methods.zombies(id).call() will communicate with the Web3 provider node and tell it to return the zombie with index id from Zombie[] public zombies on our contract.
- Note that this is asynchronous, like an API call to an external server. So Web3 returns a promise here. 
- Once the promise resolves (which means we got an answer back from the web3 provider), our example code continues with the then statement, which logs result to the console.
- result will be a javascript object that looks like this:
```javascript
{
  "name": "H4XF13LD MORRIS'S COOLER OLDER BROTHER",
  "dna": "1337133713371337",
  "level": "9999",
  "readyTime": "1522498671",
  "winCount": "999999999",
  "lossCount": "0" // Obviously.
}
```

## Chapter 5: MetaMask & Accounts
### Getting the user's account in MetaMask
- We can see which account is currently active on the injected web3 variable via:
```javascript
var userAccount = web3.eth.accounts[0]
```
- Because the user can switch the active account at any time in MetaMask, our app needs to monitor this variable to see if it has changed and update the UI accordingly.
- We can do that with a setInterval loop as follows:
```javascript
var accountInterval = setInterval(function() {
  // Check if account has changed
  if (web3.eth.accounts[0] !== userAccount) {
    userAccount = web3.eth.accounts[0];
    // Call some function to update the UI with the new account
    updateInterface();
  }
}, 100);
```
- What this does is check every 100 milliseconds to see if userAccount is still equal web3.eth.accounts[0] (i.e. does the user still have that account active). If not, it reassigns userAccount to the currently active account, and calls a function to update the display.

## Chapter 6: Displaying our Zombie Army
- However, realistically, you'll want to use a front-end framework like React or Vue.js in your app, since they make your life a lot easier as a front-end developer. But covering React or Vue.js is way outside the scope of this tutorial — that would be an entire tutorial of multiple lessons in itself.
### Displaying zombie data - a rough example
- Thus we'll want our displayZombies function to:
1. First clear the contents of the #zombies div, if there's anything already inside it. (This way if the user changes their active MetaMask account, it will clear their old zombie army before loading the new one).
2. Loop through each id, and for each one call getZombieDetails(id) to look up all the information for that zombie from our smart contract, then
3. Put the information about that zombie into an HTML template to format it for display, and append that template to the #zombies div.

## Chapter 7: Sending Transactions
- Now let's look at using send functions to change data on our smart contract.
- There are a few major differences from call functions:
1. sending a transaction requires a from address of who's calling the function (which becomes msg.sender in your Solidity code). We'll want this to be the user of our DApp, so MetaMask will pop up to prompt them to sign the transaction.
2. sending a transaction costs gas
3. There will be a significant delay from when the user sends a transaction and when that transaction actually takes effect on the blockchain. This is because we have to wait for the transaction to be included in a block, and the block time for Ethereum is on average 15 seconds. If there are a lot of pending transactions on Ethereum or if the user sends too low of a gas price, our transaction may have to wait several blocks to get included, and this could take minutes.
Thus we'll need logic in our app to handle the asynchronous nature of this code.

### Creating Zombies
Solidity Contract
```solidity
function createRandomZombie(string _name) public {
  require(ownerZombieCount[msg.sender] == 0);
  uint randDna = _generateRandomDna(_name);
  randDna = randDna - randDna % 100;
  _createZombie(_name, randDna);
}
```

Web3.js call using MetaMask
```javascript
function createRandomZombie(name) {
  // This is going to take a while, so update the UI to let the user know
  // the transaction has been sent
  $("#txStatus").text("Creating new zombie on the blockchain. This may take a while...");
  // Send the tx to our contract:
  return cryptoZombies.methods.createRandomZombie(name)
  .send({ from: userAccount })
  .on("receipt", function(receipt) {
    $("#txStatus").text("Successfully created " + name + "!");
    // Transaction was accepted into the blockchain, let's redraw the UI
    getZombiesByOwner(userAccount).then(displayZombies);
  })
  .on("error", function(error) {
    // Do something to alert the user their transaction has failed
    $("#txStatus").text(error);
  });
}
```
- Our function sends a transaction to our Web3 provider, and chains some event listeners
- receipt will fire when the transaction is included into a block on Ethereum, which means our zombie has been created and saved on our contract
- error will fire if there's an issue that prevented the transaction from being included in a block, such as the user not sending enough gas. We'll want to inform the user in our UI that the transaction didn't go through so they can try again.
- You can optionally specify gas and gasPrice when you call send, e.g. .send({ from: userAccount, gas: 3000000 }). If you don't specify this, MetaMask will let the user choose these values.

## Chapter 8: Calling Payable Functions
- The logic for attack, changeName, and changeDna will be extremely similar, so they're trivial to implement and we won't spend time coding them in this lesson.
- In fact, there's already a lot of repetitive logic in each of these function calls, so it would probably make sense to refactor and put the common code in its own function. (And use a templating system for the txStatus messages

### Level Up!
- The way to send Ether along with a function is simple, with one caveat: we need to specify how much to send in wei, not Ether.

### Wei
- A wei is the smallest sub-unit of Ether — there are 10^18 wei in one ether.
- That's a lot of zeroes to count — but luckily Web3.js has a conversion utility that does this for us.
```javascript
// This will convert 1 ETH to Wei
web3js.utils.toWei("1");
```
In our DApp, we set levelUpFee = 0.001 ether, so when we call our levelUp function, we can make the user send 0.001 Ether along with it using the following code:
```javascript
cryptoZombies.methods.levelUp(zombieId)
.send({ from: userAccount, value: web3js.utils.toWei("0.001", "ether") })
```

## Chapter 9: Subscribing to Events
- As you can see, interacting with your contract via Web3.js is pretty straightforward — once you have your environment set up, calling functions and sending transactions is not all that different from a normal web API.
- There's one more aspect we want to cover — subscribing to events from your contract.

### Listening for New Zombies
- In Web3.js, you can subscribe to an event so your web3 provider triggers some logic in your code every time it fires:
```javascript
cryptoZombies.events.NewZombie()
.on("data", function(event) {
  let zombie = event.returnValues;
  // We can access this event's 3 return values on the `event.returnValues` object:
  console.log("A new zombie was born!", zombie.zombieId, zombie.name, zombie.dna);
}).on("error", console.error);
```

### Using Indexed
- In order to filter events and only listen for changes related to the current user, our Solidity contract would have to use the indexed keyword, like we did in the Transfer event of our ERC721 implementation:
- In this case, because _from and _to are indexed, that means we can filter for them in our event listener in our front end:
```javascript
// Use `filter` to only fire this code when `_to` equals `userAccount`
cryptoZombies.events.Transfer({ filter: { _to: userAccount } })
.on("data", function(event) {
  let data = event.returnValues;
  // The current user just received a zombie!
  // Do something here to update the UI to show it
}).on("error", console.error);
```

### Querying past events
- We can even query past events using getPastEvents, and use the filters fromBlock and toBlock to give Solidity a time range for the event logs ("block" in this case referring to the Ethereum block number):
```javascript
cryptoZombies.getPastEvents("NewZombie", { fromBlock: 0, toBlock: "latest" })
.then(function(events) {
  // `events` is an array of `event` objects that we can iterate, like we did above
  // This code will get us a list of every zombie that was ever created
});
```
- Because you can use this method to query the event logs since the beginning of time, this presents an interesting use case: Using events as a cheaper form of storage.
- The tradeoff here is that events are not readable from inside the smart contract itself. But it's an important use-case to keep in mind if you have some data you want to be historically recorded on the blockchain so you can read it from your app's front-end.
- 