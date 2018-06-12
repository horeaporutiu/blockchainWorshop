# blockchainWorshop
Learn how to build a blockchain network with Hyperledger Composer

# Overview
This workshop is inspired by a recent project I have been working on. I was given the task to create a generic blockchain network to model the path that a batch of coffee goes through as it moves through the supply chain. We will start with the coffee grower who has a batch of coffee beans that is ready to be sold. That batch will go from the grower, to the importer who will buy the coffee from the grower. Then a regulator such as the international coffee association will have to check the coffee to ensure its quality. Finally, the coffee will be moved to the retailer, who will sell the coffee. 

## Viewpoint
We will take the view of the regulator, who wants to see all transactions throughout the blockchain, to ensure that the coffee meets international standards.

## Steps
1. Learn the [**Hyperledger Composer Modeling Language**](https://hyperledger.github.io/composer/latest/reference/cto_language.html)
2. Located and click on your newly created application 
3. Select 'Runtime' in the left menu
4. Select the 'Environment Variables' tab in the middle of the page
5. Scroll down to the User defined variables section
6. Click on ``add``. 
7. THIS IS EXTREMELY IMPORTANT. Make sure to write the name of the env variable EXACTLY as shown, otherwise, the app wont work. Scroll up until you see `VCAP_SERVICES`. You will then see `cloudantNoSQLDB` and under that `url`.  Under 'Name', type in `CLOUDANT_URL`, and under 'Value', paste the `url` value from the `cloudantNoSQLDB` section of `VCAP_SERVICES`.
