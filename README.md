# Blockchain Workshop
Learn how to build a blockchain network with Hyperledger Composer. The intro slides to this workshop are found here: https://www.slideshare.net/HoreaPorutiu/blockchain-workshop-ibm-code-day-montevideo

# Overview
This workshop is inspired by a recent project I have been working on. I was given the task to create a generic blockchain network to model the path that a batch of coffee goes through as it moves through the supply chain. We will start with the coffee grower who has a batch of coffee beans that is ready to be sold. That batch will go from the grower, to the importer who will buy the coffee from the grower. Then a regulator such as the international coffee association will have to check the coffee to ensure its quality. Finally, the coffee will be moved to the retailer, who will sell the coffee. 

## Viewpoint
We will take the view of the regulator, who wants to see all transactions throughout the blockchain, to ensure that the coffee meets international standards. The end goal of the regulator is to have the ability to do a simple query on a particular batch of coffee and see all participants and transactions that have worked on this particular batch. Nice. Let's build this!

## Steps
1. Learn the [**Hyperledger Composer Modeling Language**](https://hyperledger.github.io/composer/latest/reference/cto_language.html) - effectively the data schema of the blockchain. 
2. Create the .cto file.
3. Learn how to create [**Transaction Processor Functions**](https://hyperledger.github.io/composer/latest/reference/js_scripts) - the way to transfer assets between participants on the blockchain.
4. Create .js file.
5. Learn about [**Access Control Language**](https://hyperledger.github.io/composer/latest/reference/acl_language) -  determine which users/roles are permitted to create, read, update or delete elements in a business network.
5. Create .acl file.
6. Learn about [**Queries**](https://hyperledger.github.io/composer/latest/reference/query-language) -  Quickly retrieve data from the blockchain. 
7. Create .qry file
8. Test the buisness network.
9. Export the business network as a .bna (business network archive) file.
10. Deploy .bna file to IBM Blockchain Starter Plan.

## Prerequisite
Go to https://composer-playground.mybluemix.net in your browser. 
1. Click on `Deploy a new business network` 
2. For business network name, type in `coffeetracker`
3. For network admin card, type in `admin@coffeetracker`
4. Under template, select `empty-business-network`
5. On the right-hand-side, click `deploy`
6. This should take you to a page showing you your business networks. Under the coffeetracker network, click `connect now`.

## Modeling language - .cto file
The Hyperledger Composer modeling language is an object-oriented language that is used to define the model of a business network.  The .cto file (or the model file) is composed of the following elements:
1. A single namespace
2. Resource definitions - assets, transactions, participants, and events.

After clicking `connect now`, you should be taken to your editor. On the left-hand side, you will see a `model file`. The first thing we need to do is change our namespace. Let's type in `org.ibm.coffee` for the namespace. This is really important, as everything you create in your model file will fall under this namespace.

First let's create some participants in the network. They will be businesses. And the language supports object-oriented programming, let's make full use of this! Let's create an abstract class first, and then create classes that will inhert from this one.

Under the namespace, type in: 
```
abstract participant Business {
  o String organization
  o Address address
}
```
We have given our abstract class two required fields - a string which is the name of the organization, and an address concept. Let's define the address as below:

```
concept Address {
  o String city optional
  o String country
  o String street optional
  o String zip optional
}
```

### Create participants
Great, now that the abstract class is taken care of, let's create classes that inherit from this. For our network, we will have four participants: a grower, an importer, a regulator, and a retailer. Let's create these as below:

```
participant Grower identified by growerId extends Business {
  o Boolean isFairTrade
  o String growerId
}

participant Importer identified by importerId extends Business {
     o String importerId
}

participant Regulator identified by regulatorId extends Business{
  o String regulatorId
}

participant Retailer identified by retailerId extends Business{
  o String retailerId
}
```
That's it for our participants. That wasn't so bad, was it?

### Create asset

Now, let's create our first asset. That is - a coffee asset! Simple enough. We need to have a unique identifier by which to query our particular batch of coffee with, and that is really imporant. That's our goal as the regulator - remember?! Great, let's create the coffee asset as below, and have the batchId as the unique identifier for our asset.

```
asset Coffee identified by batchId {
  o String batchId
  o Size size
  o CoffeeRoast roast
  o State batchState
  --> Business owner
}
```

### Create enumerated types
Enumerated types are used to specify when a type has 1 to N possible values. These are useful when you know what possible values a field can take. So, for our coffee example, we will have 4 enumerated types:

1. Type of roast
2. Size of batch
3. State of batch 
4. Type of owner

Let's define the types as below:

```
enum CoffeeRoast {
  o LIGHT
  o MEDIUM
  o DARK
}

enum Size {
  o SMALL
  o MEDIUM
  o LARGE
}

enum State {
  o READY_FOR_DISTRIBUTION
  o IMPORTED
  o REGULATION_TEST_PASSED
  o READY_FOR_SALE
}

enum ownerType {
  o GROWER
  o IMPORTER
  o REGULATOR
  o REATAILER
}
```

## Create transactions
Let's add the data schema for our transactions. Let's model them as below:
```
transaction transferCoffee {
  --> Business newOwner
  --> Business oldOwner
  o String batchId
  o ownerType newOwnerType
}

transaction addCoffee{
  o Size size
  o CoffeeRoast roast
  o State batchState
  --> Grower grower
}
```
The transferCoffee transaction takes two types of Businesses, one which is the newOwner, and the other which is the oldOwner. The newOwnerType will be the expected type of owner that will receive the batch of coffee.

Nice! That's it for the model.cto file. Let's move on to the logic of our application, the script file.

Nice. If everything is good, you should see a ✅at the bottom of the page. If so, click on `deploy` in the bottom left corner. Great job!

## Transaction Processor Functions - .js file
Now, it's time to write the heart of our application. The .js file. Let's start with the addCofee transaction. 

We will need to create two transactions. To do this, we first need to create our .js file. Click on `add a file` on the bottom left corner of the page. Then click on `Script file .js`, and then `add`. You should now see a new .js file in the left side of the screen. 

### Create addCoffee transaction
This transaction will take in a size, a type of roast, a batchState, and a grower participant. Let's define the function as below: 

```
/**
 * Sample transaction processor function.
 * @param {org.ibm.coffee.addCoffee} tx The send message instance.
 * @transaction
 */
async function addCoffee(newCoffee) {
  
  const participantRegistry = await getParticipantRegistry('org.ibm.coffee.Grower');
  var NS = 'org.ibm.coffee';
  var coffee = getFactory().newResource(NS, 'Coffee', Math.random().toString(36).substring(3));
  coffee.size = newCoffee.size;
  coffee.roast = newCoffee.roast;
  coffee.owner = newCoffee.grower;
  coffee.batchState = newCoffee.batchState;
  
  const assetRegistry = await getAssetRegistry('org.ibm.coffee.Coffee');
  await assetRegistry.add(coffee);
  await participantRegistry.update(newCoffee.grower);
}
```

### Create transferCoffee transaction

Next, let's write the transferCoffee transaction. This transaction will take in two businesses, a newOwner, and oldOwner, a batchId, and then an emum, which is the newOwnerType. Let's define this transaction as below: 

```
/**
 * Sample transaction processor function.
 * @param {org.ibm.coffee.transferCoffee} tx The send message instance.
 * @transaction
 */
async function transferCoffee(coffeeBatch) {
  
  if (coffeeBatch.batchId.length <= 0) {
    throw new Error('Please enter the batchId');
  }
  
  if (coffeeBatch.newOwner.length <= 0) {
    throw new Error('Please enter the new owner');
  }
  
  const assetRegistry = await getAssetRegistry('org.ibm.coffee.Coffee');
  
  const exists = await assetRegistry.exists(coffeeBatch.batchId);
  
  if (exists) {
  	const coffee = await assetRegistry.get(coffeeBatch.batchId);
    
    coffeeBatch.oldOwner = coffee.owner;
    coffee.owner = coffeeBatch.newOwner;
   
    if (coffeeBatch.newOwnerType.toLowerCase() == 'importer') {

      const participantRegistry = await getParticipantRegistry('org.ibm.coffee.Importer');
      await participantRegistry.update(coffeeBatch.newOwner);
      coffee.batchState = "IMPORTED";
      
    } else if (coffeeBatch.newOwnerType.toLowerCase() == 'regulator') {
      const participantRegistry = await getParticipantRegistry('org.ibm.coffee.Regulator');
	  await participantRegistry.update(coffeeBatch.newOwner);
      coffee.batchState = "REGULATION_TEST_PASSED";
    } else {
      const participantRegistry = await getParticipantRegistry('org.ibm.coffee.Retailer');
      await participantRegistry.update(coffeeBatch.newOwner);
      coffee.batchState = "READY_FOR_SALE";
    }
    
    await assetRegistry.update(coffee);
    
    
  } else {
  	throw new Error('the batch you specified does not exist!');
  }
  
  
}
```

Nice! We are done with the .js file. If you see the ✅ in the bottom of the screen, you are good to go! Click on `deploy` in the bottom-left corner.

## Access Control language - .acl file
Since we are building a very basic network, we will just keep the default file that comes out of the box with our network. No need to change anything here. 

## Query language - .qry file
Ok, we are almost done. Let's build a query to check all the different times a particular batch has gone through the 'transferCoffee' transaction. This will help us, as a regulator, gain insight into where this batch has been, and which participants have worked with this batch. 

Click on `add a file` in the bottom left corner. Click on `query.qry` file. Click `add`. Let's define the queries as below:

```
query getBatchHistory { 
  description: "see all of the participants that have worked with a particular batch" 
  statement: 
  		SELECT org.ibm.coffee.transferCoffee
  			WHERE (batchId == _$batchId ) 
}
```
What we have done here, is used a `SELECT` statement, which selects all of the transferCoffee transactions in the history of the blockchain, and queries them for the particular batchId that we enter.

Since when we first add coffee to the blockchain, we use the addCoffee function, we will have to create a seperate query for that function. Let's define it as below: 

```
query getBatchHistory { 
  description: "see all of the participants that have worked with a particular batch" 
  statement: 
  		SELECT org.ibm.coffee.transferCoffee
  			WHERE (batchId == _$batchId ) 
}
```

Cool. If you see the ✅ you are good to go! Nice! Click `deploy` in the bottom-left corner.

Nice!! We are ready to test the network!

