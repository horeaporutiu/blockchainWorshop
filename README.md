# Blockchain Workshop
Learn how to build a blockchain network with Hyperledger Composer. The intro slides to this workshop are found here: https://www.slideshare.net/HoreaPorutiu/blockchain-workshop-ibm-code-day-montevideo

This repo includes a walkthrough of how to build a basic blockchain network, and export it as a .bna. All files included are the result of using this README as a walkthrough. 

# Overview
This workshop is inspired by a recent project I have been working on. I was given the task to create a generic blockchain network to model the path that a batch of coffee goes through as it moves through the supply chain. We will start with the coffee grower who has a batch of coffee beans that is ready to be sold. That batch will go from the grower, to the importer who will buy the coffee from the grower. Then a regulator such as the international coffee association will have to check the coffee to ensure its quality. Finally, the coffee will be moved to the retailer, who will sell the coffee. 

## Viewpoint
We will take the view of the regulator, who wants to see all transactions throughout the blockchain, to ensure that the coffee meets international standards. The end goal of the regulator is to have the ability to do a simple query on a particular batch of coffee and see all participants and transactions that have worked on this particular batch. Nice. Let's build this!

# Steps
1. [Learn the modeling language](#1-learn-the-modeling-language) 
2. [Learn about transaction Processor Functions](#2-learn-about-transaction-processor-functions)
3. [Learn about Access Control language](#3-learn-about-access-control-language)
4. [Learn about query language](#4-learn-about-query-language) 
5. [Test the network](#5-test-the-network) 
6. [Export the business network](#6-export-the-business-network)
7. [Deploy the network to the Starter Plan](#7-deploy-the-network-to-the-starter-plan)

## Prerequisite
Go to https://composer-playground.mybluemix.net in your browser. 
1. Click on `Deploy a new business network` 
2. For business network name, type in `coffeetracker`
3. For network admin card, type in `admin@coffeetracker`
4. Under template, select `empty-business-network`
5. On the right-hand-side, click `deploy`
6. This should take you to a page showing you your business networks. Under the coffeetracker network, click `connect now`.

## 1. Learn the modeling language 
The [**Hyperledger Composer Modeling Language**](https://hyperledger.github.io/composer/latest/reference/cto_language.html) is an object-oriented language that is used to define the model of a business network. The .cto file (or the model file) is composed of the following elements:
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
  o RETAILER
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

## 2. Learn about transaction Processor Functions
Now, it's time to write the heart of our application. The .js file. Let's start with the addCofee transaction. 

We will need to create two transactions. To do this, we first need to create our .js file. Click on `add a file` on the bottom left corner of the page. Then click on `Script file .js`, and then `add`. You should now see a new .js file in the left side of the screen. To learn more about transaction processor functions, go here: https://hyperledger.github.io/composer/latest/reference/js_scripts

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

## 3. Learn about Access Control language 
Since we are building a very basic network, we will just keep the default file that comes out of the box with our network. No need to change anything here. To learn more about the access control language, go here: https://hyperledger.github.io/composer/latest/reference/acl_language

## 4. Learn about query language 
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

## 5. Test the network
To test the network, let's click on the `test` tab, at the top of the page. First, we have to create new participants. Click on the `Grower` tab on the left side of the page. Click on `Create new participant` on the top-right corner of the page. Add in the following json: 

```
{
  "$class": "org.ibm.coffee.Grower",
  "isFairTrade": false,
  "growerId": "growerA",
  "organization": "",
  "address": {
    "$class": "org.ibm.coffee.Address",
    "country": ""
  }
}
```

Now, click on `importer` and follow the same process. Add in the following json:

```
{
  "$class": "org.ibm.coffee.Importer",
  "importerId": "importerA",
  "organization": "",
  "address": {
    "$class": "org.ibm.coffee.Address",
    "country": ""
  }
}
```

Now, let's do the regulator:

```
{
  "$class": "org.ibm.coffee.Regulator",
  "regulatorId": "regulatorA",
  "organization": "",
  "address": {
    "$class": "org.ibm.coffee.Address",
    "country": ""
  }
}
```

Lastly, the Retailer:

```
{
  "$class": "org.ibm.coffee.Retailer",
  "retailerId": "retailerA",
  "organization": "",
  "address": {
    "$class": "org.ibm.coffee.Address",
    "country": ""
  }
}
```

### Submit addCoffee transaction

Now, click on the `submit transaction` button at the bottom-left corner of the page. Select `addCoffee` from the drop down. Fill in the following json:

```
{
  "$class": "org.ibm.coffee.addCoffee",
  "size": "SMALL",
  "roast": "LIGHT",
  "batchState": "READY_FOR_DISTRIBUTION",
  "grower": "resource:org.ibm.coffee.Grower#growerA"
}
```

Next, click `submit`.

Great! Click on the coffee tab on the left, under 'assets' to ensure that your batch is created. Note the batchId. 

### Submit transferCoffee transaction

Now, click on the `submit transaction` button at the bottom-left corner of the page. Select `transferCoffee` from the drop down. ⚠️⚠️⚠️⚠️⚠️⚠️⚠️USE YOUR batchId from above⚠️⚠️⚠️⚠️⚠️⚠️⚠️, instead of mine. Fill in the following json:

```
{
  "$class": "org.ibm.coffee.transferCoffee",
  "newOwner": "resource:org.ibm.coffee.Importer#importerA",
  "oldOwner": "resource:org.ibm.coffee.Grower#growerA",
  "batchId": "1234567",
  "newOwnerType": "IMPORTER"
}
```

Next, let's transfer the coffee from the importer to the regulator. Using the same steps from above, fill in the following json:

```
{
  "$class": "org.ibm.coffee.transferCoffee",
  "newOwner": "resource:org.ibm.coffee.Regulator#regulatorA",
  "oldOwner": "resource:org.ibm.coffee.Importer#importerA",
  "batchId": "1234567",
  "newOwnerType": "REGULATOR"
}
```

Lastly, let's transfer the coffee from the regulator to the retailer. This is the last step. Using the same steps as above to submit the `transferCoffee` transaction, fill in the following json:

```
{
  "$class": "org.ibm.coffee.transferCoffee",
  "newOwner": "resource:org.ibm.coffee.Retailer#retailerA",
  "oldOwner": "resource:org.ibm.coffee.Regulator#regulatorA",
  "batchId": "1234567",
  "newOwnerType": "RETAILER"
}
```

Let's check the asset by clicking on `asset` in the left hand side of the page. You should see the state as `READY_FOR_SALE` and the owner should now be retailerA. Nice job! You have successfully transferred the coffee through the blockchain! 

## 6. Export the business network 
Click on `define` in the top of the page. Then `export` at the bottom of the page. This will automatically download your .bna file. 

## 7. Deploy the network to the Starter Plan  
For this step, you will need to create an [IBM Cloud](https://ibm.biz/BdjLxy) account if you do not have one.  

Use this guide to deploy your .bna file to the Starter Kit, and then use the rest-server to submit transactions and log them into to starter kit running on the cloud: https://hackernoon.com/deploy-a-business-network-on-free-ibm-blockchain-starter-plan-93fafb3dd997

## Demo of live network running on IBM Blockchain starter plan
This API is currently live, and I will show you the end result of deploying the rest-server to the cloud. The API can be found here: https://composer-rest-server-coffeetrackr.mybluemix.net/. After submitting transactions there, all transactions are logged on the blockchain starter plan.

## Next steps
Deploy your business network to the cloud using this guide: https://github.com/sstone1/blockchain-starter-kit
