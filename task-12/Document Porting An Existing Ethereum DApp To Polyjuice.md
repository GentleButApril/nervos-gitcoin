

Step 1: Setup the Godwoken Testnet Network in MetaMask

    Your MetaMask wallet will need to be configured to communicate with the Godwoken Layer 2 network. To do this, you will need to configure a new custom RPC. From the network selection dropdown, select “Custom RPC”.
    Now, you will be presented with a form to specify the network settings.
    Enter the following details in the given form.


```
Network Name: Godwoken Testnet
RPC URL: https://godwoken-testnet-web3-rpc.ckbapp.dev
Chain ID: 71393
Currency Symbol: <Leave Empty>
Block Explorer URL: <Leave Empty>
```

Step 2: Fund your Ethereum account on Nervos Layer 2

To deploy a contract on Godwoken Testnet your Ethereum account needs to be funded on Nervos Layer 2. This is a two step process.

1. Create and fund an account with CKBytes on Layer 1:

In this step you need to create an account on the Testnet Nervos CKB Layer 1 blockchain, fund it with CKBytes, and then export the private key for the account so it can be used in step 2. This can be done by following the instructions given here.

2. Deposit CKBytes on Layer 2:

In this step you need to make a deposit of CKBytes from Layer 1 to the Layer 2 which is provided by Godwoken. This step is necessary for Godwoken to create the user’s Layer 2 account. This can be done by following the instructions given here.


Step 3: Clone your existing dApp


```
git clone git@github.com:GentleButApril/nervos-create-nft.git
cd nervos-create-nft
yarn
yarn build
yarn start:ganache
```



 Open the brower http://localhost:3000/ and make sure everything is working correctly.

Now we will begin porting our Ethereum dApp to Nervos’Layer2.

Step 4: Configure your dApp for Polyjuice

1. Install Polyjuice Dependencies:

We will install the required dependencies for working with Godwoken and Polyjuice.

npm install @polyjuice-provider/web3@0.0.1-rc7 nervos-godwoken-integration@0.0.6

    @polyjuice-provider/web3 is a custom Polyjuice web3 provider. It is required for interaction with Nervos' Layer 2 smart contracts.
    nervos-godwoken-integration is a tool that can generate Polyjuice address based on your Ethereum address.

    Note: You might be required to use Polyjuice address if you store values mapped to addresses in your contracts.

2. Configure the Web3 Provider for the Polyjuice Web3 Provider:

Polyjuice Web3 Provider replaces the normal Web3 Provider that we currently use for Ethereum with one for the Godwoken Testnet.

Under your src folder create a new file config.js and add the code given below.


```
export const CONFIG = {
    WEB3_PROVIDER_URL: 'https://godwoken-testnet-web3-rpc.ckbapp.dev',
    ROLLUP_TYPE_HASH: '0x4cc2e6526204ae6a2e8fcf12f7ad472f41a1606d5b9624beebd215d780809f6a',
    ETH_ACCOUNT_LOCK_CODE_HASH: '0xdeec13a7b8e100579541384ccaf4b5223733e4a5483c3aec95ddc4c1d5ea5b22'
};
```


Next, we need to import Polyjuice Web3 Provider and the config file that we just created.

Open src/ui/app.tsx and add the following lines in the main dependency importation section of the file.


```
import { PolyjuiceHttpProvider } from '@polyjuice-provider/web3';
import { CONFIG } from '../config';
```


Now we will create the Polyjuice Provider, and use the Polyjuice Provider with a Web3 instance. This will replace our existing Ethereum Web3 instance.


```
const godwokenRpcUrl = CONFIG.WEB3_PROVIDER_URL;
const providerConfig = {
    rollupTypeHash: CONFIG.ROLLUP_TYPE_HASH,
    ethAccountLockCodeHash: CONFIG.ETH_ACCOUNT_LOCK_CODE_HASH,
    web3Url: godwokenRpcUrl
};
const provider = new PolyjuiceHttpProvider(godwokenRpcUrl, providerConfig);
const web3 = new Web3(provider);
```


In app.tsx, locate the existing Ethereum Web3 instance, which should match the line below. Delete this line, and replace it with the Polyjuice Web3 code from above.

const web3 = new Web3(window.ethereum);

Our application is now setup to communicate with Polyjuice using Web3

3. Set High Gas Limit:

As of now, Godwoken requires the gas limit to be set when sending transactions. To accommodate for this, we default the gas limit to 6000000 for the user when they make transactions.

To do this locate all calls to your contract using send() and pass the object below as a parameter.


```
const DEFAULT_SEND_OPTIONS = {
gas: 6000000
};
```


In my dApp, inside src/lib/contracts/SimpleNFTWrapper.ts locate
        
        async createNFT(imgUrl: string, fromAddress: string) {
        const tx = await this.contract.methods.awardItem(fromAddress, imgUrl).send({
            ...DEFAULT_SEND_OPTIONS,
            from: fromAddress
        });
        return tx;
    }
     async deploy(fromAddress: string) {
        const deployTx = await (this.contract
            .deploy({
                data: SimpleNFTJson.bytecode,
                arguments: []
            })
            .send({
                ...DEFAULT_SEND_OPTIONS,
                from: fromAddress,
                to: '0x0000000000000000000000000000000000000000'
            } as any) as any);

        this.useDeployed(deployTx.contractAddress);

        return deployTx.transactionHash;
    }

4. Translate Ethereum Address to PolyJuice Address (Optional):

    Every Ethereum address can be translated into a Polyjuice address on Nervos’ Layer 2. This can be done using the AddressTranslator class.

To translate an ethereum address to polyjuice address we need to import nervos-godwoken-integration in our app.

- [x] import { AddressTranslator } from 'nervos-godwoken-integration';

We can then use the following code to find the Polyjuice address.


```
const addressTranslator = new AddressTranslator();
const polyjuiceAddress = addressTranslator.ethAddressToGodwokenShortAddress(<ETH_ADDRESS>);
```

Step 5: View your Polyjuice dApp

In terminal enter the following command.


```
cd nervos-create-nft
yarn
yarn build
yarn ui
```


Now open the browser with MetaMask extension and go to http://localhost:3000/ to open your dApp.

Congratulations! You have successfully ported your Ethereum dApp to Polyjuice 
