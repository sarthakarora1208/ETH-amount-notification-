ETH Amount notification

Video link:- [bit.ly/eth-amount-notification](https://bit.ly/eth-amount-notification)

System Architecture

![eth-alert drawio](https://user-images.githubusercontent.com/42542489/157976217-17db9e24-e3fd-4e74-868e-74d4aa8262e5.png)

Tools Used:

- [Bolt JS Slack Framework](https://slack.dev/bolt-js/concepts) - To make blocks in Slack
- [Ether.js](https://docs.ethers.io/v5/) - To get the account balance for wallet address. (Also to transact on the ropsten testnet)
- [AWS RDS (PostgreSQL)](https://aws.amazon.com/rds/) - To save Alert data
- [AWS EC2](https://aws.amazon.com/ec2/) - To run the Bolt App on AWS
- [Nodejs](https://nodejs.org/en/)
- [Etherscan](https://ropsten.etherscan.io/) - To check the status of our transactions and verify wallet balance
- [QuickNode](https://www.quicknode.com/)

### Steps

<ol>

<b>Step 1</b>

The /create-alert slash command upon invocation opens a new modal.

<img width="544" alt="Screenshot 2022-03-12 at 2 11 12 AM" src="https://user-images.githubusercontent.com/42542489/157977910-32cfa308-e16f-4497-a7f2-b97d1b1fbe60.png">

The /create-alert slash command upon invocation opens a modal.

The user is prompted to enter the following details:

- Alert Name
- Ethereum Address - (To be checked for insufficient funds)
- Threshold Ethereum Value - (Value below which the user will be notified)
- Notification Time - (specified time on which the cron job will run)

</br>

The current balance is fetched using the [getBalance()](https://docs.ethers.io/v4/cookbook-accounts.html) function in Ether.js

![Sample_Transaction](https://user-images.githubusercontent.com/42542489/157981095-94bf88ec-6577-4dc2-9504-73a850b21401.png)

```bash

import { ethers } from 'ethers';

(async () => {
  const connection = new ethers.providers.JsonRpcProvider(
    'https://bitter-dark-waterfall.ropsten.quiknode.pro/c034a2129f636607f6542a1f0fe93ff2acd9a4c0/'
  );

  const gasPrice = connection.getGasPrice();

  //const wallet = ethers.Wallet.createRandom();
  //console.log(wallet.address);
  const wallet = ethers.Wallet.fromMnemonic(
    'live elevator fault congress toward grace sure result place oppose subject speak'
  );
  //console.log(wallet.address);
  let balance = await connection.getBalance(wallet.address);
  console.log(
    'Address is ' +
      wallet.address +
      '\n Balance is ' +
      ethers.utils.formatEther(balance) +
      'ETH'
  );


})();

```

Output:

![Screenshot 2022-03-12 at 3 49 47 AM](https://user-images.githubusercontent.com/42542489/157980295-091a819f-7876-432a-884f-6199830dbc21.png)

<b>Step 2</b>

The data is saved in AWS RDS for future use.

![Alert](https://user-images.githubusercontent.com/42542489/157980836-80f16be3-4a7c-41f1-8988-685f0bce38dd.png)

<b>Step 3 </b>

![Block Addition](https://user-images.githubusercontent.com/42542489/157982173-f47cee8b-fdce-4a1e-9d40-dc4908a81e41.png)

In Ethers.js we can listen for [events](https://docs.ethers.io/v5/concepts/events/) whenever something happens on the blockchain. How do we ensure that the user is notified at the correct time? You can either receive ETH in your wallet or transfer them to another address. In both cases, we have to find a way to sync our Slack messages with our wallet balance. To accomplish this we can fire a webhook whenever a new verified block is added to the chain. The webhook will send a message to the relevant Slack channel if the currentBalance is less than the thresholdEthValue. (This check is done for all the alerts in the database for every block). This approach is slightly computationally expensive but it will prevent us from refactoring our existing code and can run as an independent microservice.

The alternative approach we can follow is that we can call the (chat.postMessage)[https://api.slack.com/methods/chat.postMessage] API every time you make a transaction. We can also use multiple bloom filters in ether.js to filter all the transactions that happen on the chain. We choose the first approach of firing the webhook on the 'block' event because it is the most scalable approach.

For setting up the time-based alerts we can set up a cron job using node-cron that works at the specified time each day. We'll then call the getBalance() function to get the balance for the ethereumAddress and then call the chat.postMessage Slack API to notify the user inside Slack.

</ol>

<img width="586" alt="Screenshot 2022-03-12 at 2 48 23 AM" src="https://user-images.githubusercontent.com/42542489/157985222-79969805-8ef0-447a-a83b-2797f45331a3.png">
