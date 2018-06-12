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

