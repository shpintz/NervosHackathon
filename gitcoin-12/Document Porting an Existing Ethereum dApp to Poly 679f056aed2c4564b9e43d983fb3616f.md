# Document Porting an Existing Ethereum dApp to Polyjuice

Created: August 20, 2021 9:44 PM

Alrighty, so what we are going to be doing is **porting an existing Ethereum application to run on Nervos' EVM, which is compatible with the layer 2 network of things in nervos.** 

We are going to be using Metamask where we will be able to create a way to turn a confirmation dialog to display the transition instead of the hex value when you send crypto around the blockchain... but first, we are going to need to be connected to the godwoken testnet which is what we will be connecting too.

Polyjuice is compatible with existing EVM smart contracts, but is still being developed. 

We are going to be using a little basic project for this tutorial you can **download it here** if you want, you want to make sure you have metamask installed and set up the godwoken testnet first.

You going to need to communicate with the layer 2 network on nervous to do that we need to configure the custom RPC.

You want to select the 3 dots and click on the button option (Custom RPC)**:** 

![step-1-metamask.png](Document%20Porting%20an%20Existing%20Ethereum%20dApp%20to%20Poly%20679f056aed2c4564b9e43d983fb3616f/step-1-metamask.png)

You then want to enter the following data in the information boxes that it asks for, you can leave the last 2 blank. 

```markdown
Network Name: Godwoken Testnet
RPC URL: https://godwoken-testnet-web3-rpc.ckbapp.dev
Chain ID: 71393
Currency Symbol: <Leave Empty>
Block Explorer URL: <Leave Empty>
```

![step-2-metamask.png](Document%20Porting%20an%20Existing%20Ethereum%20dApp%20to%20Poly%20679f056aed2c4564b9e43d983fb3616f/step-2-metamask.png)

After that let us open our **[project files](https://github.com/shpintz/nervos-example)** that if want to [download](https://github.com/shpintz/nervos-example) above and setup the environment

```markdown
cd ~/projects
git clone https://github.com/shpintz/nervos-example.git
cd nervos-example
```

From there start run the dependencies and start ganache to run a local etherum development chain

```markdown
cd ~/projects/nervos-example
yarn
yarn build
yarn start:ganache
```

When the ganache server is up inside your metamask you want to switch your network to your **[localhost](http://localhost) 8545** and start a second terminal and start your UI server.c

![step-4-zombieGallary.png](Document%20Porting%20an%20Existing%20Ethereum%20dApp%20to%20Poly%20679f056aed2c4564b9e43d983fb3616f/step-4-zombieGallary.png)

```markdown
cd ~/projects/nervos-example
yarn ui
```

If you want to join on the contract here is the address: 

ETH: 0x5a980781512aA617b482a0f434B2C04e86D9e54b

Polyjuice address: 0x7c6560761c4d068FcF1bbdD4Bd0011b1d516AA3e

Now to set begin porting to the Nervos' Layer 2 network. Before we begin we are going to need to download 2 dependence to work with Godwoken and PolyJuice.

```markdown
cd ~/projects/nervos-example
yarn add @polyjuice-provider/web3@0.0.1-rc7 nervos-godwoken-integration@0.0.6
```

- `@polyjuice-provider/web3` is a custom Polyjuice web3 provider. It is required for interaction with Nervos' Layer 2 smart contracts.
- `nervos-godwoken-integration` is a tool that can generate Polyjuice address based on your Ethereum address. You might be required to use Polyjuice address if you store values mapped to addresses in your contracts.

Once that is installed we are going to go into our  **/src** folder and create a **config.ts** file and instead of using the **local host** connect we are going to change it for the **Polyjuice Web3 Provider.** 

```tsx
export const CONFIG = {
    WEB3_PROVIDER_URL: 'https://godwoken-testnet-web3-rpc.ckbapp.dev',
    ROLLUP_TYPE_HASH: '0x4cc2e6526204ae6a2e8fcf12f7ad472f41a1606d5b9624beebd215d780809f6a',
    ETH_ACCOUNT_LOCK_CODE_HASH: '0xdeec13a7b8e100579541384ccaf4b5223733e4a5483c3aec95ddc4c1d5ea5b22'
};
```

You'll notice we are also adding two other keys that are needed for the Polyjuice Web3Provider; ROLLUP_TYPE_HASH and  ETH_ACCOUNT_LOCK_CODE_HASH.  These values are constants needed for the Godwoken Testnet. 

With that taken care of, we are going to hook up the file into our UI(**~/projects/nervos-example/src/ui/app.tsx**) at the same time adding in the main dependency we installed earlier.

We will import the following dependcey:

```tsx
import { PolyjuiceHttpProvider } from '@polyjuice-provider/web3';
import { CONFIG } from '../config';
```

This imports the Polyjuice Web3 Provider, which we will use in a moment, and the config file that we just created. Basically, we then want to locate our Ethereum instance of web3 and replace it with the Polyjuice web3. 

```tsx
const godwokenRpcUrl = CONFIG.WEB3_PROVIDER_URL;
const providerConfig = {
    rollupTypeHash: CONFIG.ROLLUP_TYPE_HASH,
    ethAccountLockCodeHash: CONFIG.ETH_ACCOUNT_LOCK_CODE_HASH,
    web3Url: godwokenRpcUrl
};
const provider = new PolyjuiceHttpProvider(godwokenRpcUrl, providerConfig);
const web3 = new Web3(provider);
```

We will use this to create a web3 instance using a polyjuice web3 provider instead of the etherium web3 instance.

```tsx
async function createWeb3() {
    // Modern dapp browsers...
    if ((window as any).ethereum) {
        const godwokenRpcUrl = CONFIG.WEB3_PROVIDER_URL;
        const providerConfig = {
            rollupTypeHash: CONFIG.ROLLUP_TYPE_HASH,
            ethAccountLockCodeHash: CONFIG.ETH_ACCOUNT_LOCK_CODE_HASH,
            web3Url: godwokenRpcUrl
        };
        const provider = new PolyjuiceHttpProvider(godwokenRpcUrl, providerConfig);
        const web3 = new Web3(provider || Web3.givenProvider);

        try {
            // Request account access if needed
            await (window as any).ethereum.enable();
        } catch (error) {
            // User denied account access...
        }

        return web3;
    }

    console.log('Non-Ethereum browser detected. You should consider trying MetaMask!');
    return null;
}
```

Godwoken requires the gas limit to be set when sending transactions. This may not always be the case in the future, but it is a requirement for the current version on the Testnet.

To implement this, we can make a simple change to default the gas limit to 6000000 for the user when they make transactions. In our project, this is all handled in the file ZombieFactoryWrapper.ts

```tsx
const DEFAULT_SEND_OPTIONS = {
    gas: 6000000
};
```

We simply define a simple object that will contain the gas property used by Metamask. We will be adding it into the send() object as the default value inside our createRandomZombie function

```tsx
async createRandomZombie(name: string, imgURL: string, fromAddress: string) {

        const tx = await this.contract.methods.createRandomZombie(name, imgURL).send({
            ...DEFAULT_SEND_OPTIONS,
            from: fromAddress,
            // value
        });
        
        return tx;
    }
```

We also are going to need to edit out the deploy function to be able to deploy contracts as well. 

```tsx
async deploy(fromAddress: string) {
        const deployTx = await (this.contract
            .deploy({
                data: ZombieFactoryJSON.bytecode,
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
```

you need to add in the deploy gas object within the send method, it took a little bit of configuration but this seems to be able to create a contract that will create a contract under your address.

We will also need to go back into our **app.tx** file and configure our **deployContract** to be able to grab the transaction hash so we will be able to present it to the user

```tsx
async function deployContract() {
        const _contract = new ZombieFactoryWrapper(web3);

        try {
            setDeployTxHash(undefined);
            setTransactionInProgress(true);

            const transactionHash = await _contract.deploy(account);

            setDeployTxHash(transactionHash);
            setExistingContractAddress(_contract.address);
            toast(
                'Successfully deployed a smart-contract. You can now proceed to get or set the value in a smart contract.',
                { type: 'success' }
            );
        } catch (error) {
            console.error(error);
            toast('There was an error sending your transaction. Please check developer console.');
        } finally {
            setTransactionInProgress(false);
        }
    }
```

We also want to create a **useState()** variables such as,  to be able to set the transaction hash and display it into our render, I will let you display it yourself.

```tsx
**const [deployTxHash,setDeployTxHash] = useState<string | undefined>();
const [polyjuiceAddress, setPolyjuiceAddress] = useState<string | undefined>();**
```

Now we want to be able to display the polyjuice address in our application, every etherum address can be translated into a polyjuice address on Nervos layer 2, this can be done with the addressTranslater class from the other package we had downloaded, let us import it into our app.tsx file.

```tsx
import { AddressTranslator } from 'nervos-godwoken-integration';
```

After that you want to integrate it into our app.tsx file where we will then be able to capture the address and present it to the user, for i had created a useEffect() where we would be able to translate the address once an address is loaded

```tsx
useEffect(() => {
        if (accounts?.[0]) {
            const addressTranslator = new AddressTranslator();
            setPolyjuiceAddress(addressTranslator.ethAddressToGodwokenShortAddress(accounts?.[0]));
        } else {
            setPolyjuiceAddress(undefined);
        }
    }, [accounts?.[0]]);
```

Once we have that setup we can run **yarn build** to rebuild the application and launch the **ui** and within our metamask make sure we are connected to the **GodWoken testnet!**

![step-5-connect-contract-with-metamask.png](Document%20Porting%20an%20Existing%20Ethereum%20dApp%20to%20Poly%20679f056aed2c4564b9e43d983fb3616f/step-5-connect-contract-with-metamask.png)

Once connected correctly you should be able to get your polyjuice address

Deployed contract address: **0x56B5aB1dE889Ef6f4009b0AC1581374fA3E9A527**

Deploy transaction hash: **0xc22a7d50c8e16c45d0e12f95860f6a818e944b4a9bec007fccb20073f70a1a1d**

Github: [https://github.com/shpintz/nervos-example](https://github.com/shpintz/nervos-example)

[GitHub - shpintz/nervos-example](https://github.com/shpintz/nervos-example)
