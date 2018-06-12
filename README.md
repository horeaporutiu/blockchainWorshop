# Blockchain Workshop
Learn how to build a blockchain network with Hyperledger Composer

# Overview
This workshop is inspired by a recent project I have been working on. I was given the task to create a generic blockchain network to model the path that a batch of coffee goes through as it moves through the supply chain. We will start with the coffee grower who has a batch of coffee beans that is ready to be sold. That batch will go from the grower, to the importer who will buy the coffee from the grower. Then a regulator such as the international coffee association will have to check the coffee to ensure its quality. Finally, the coffee will be moved to the retailer, who will sell the coffee. 

## Viewpoint
We will take the view of the regulator, who wants to see all transactions throughout the blockchain, to ensure that the coffee meets international standards.

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

After clicking `connect now`, you should be taken to your editor. On the left-hand side, you will see a `model file`. The first thing we need to do is change our namespace. Let's type in 
