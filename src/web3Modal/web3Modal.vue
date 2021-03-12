<template>
  <div>
    <p>
      <router-link to="/home">Go to Home</router-link>

    </p>
    <button @click="setup">Show web3 modal popup</button>
    <br/>
    <br/>
    <button @click="clearCachedProvider">Clear Cached Provider</button>
    <br/>
    <p>
      <a href="https://github.com/MyEtherWallet/MEWconnect-web-client/blob/mewconnect-demo/src/web3Modal/web3Modal.vue" target="_blank">Code for this example</a>
    </p>
    <ul v-show="userAddress !== ''">
      <li>
        <button @click="disconnect">Disconnect</button>
      </li>
      <li>
        <hr/>
        <h2>Send</h2>
        <label for="toAmount">
          to amount
          <input
              id="toAmount"
              v-model="toAmount"
              placeholder="amount"
          /> </label
        ><br />
        <button v-show="userAddress !== ''" @click="sendTx">send</button>
        <h6>Sends to the connected wallet address</h6>
        <h3>Tx Hash:</h3>
        {{ txHash }}
      </li>
      <li>
        <hr/>
        <h2>Send With Data</h2>
        <label for="toAmountData">
          to amount
          <input
              id="toAmountData"
              v-model="toAmount"
              placeholder="amount"
          /> </label
        ><br />
        <label for="toData">
          to data
          <input
              id="toData"
              v-model="toData"
              placeholder="0x"
          /> </label
        ><br />
        <label for="toAddressData">
          to data
          <input
              id="toAddressData"
              v-model="toAddressData"
              placeholder="0x"
          /> </label
        ><br />
        <button v-show="userAddress !== ''" @click="sendTxData">send</button>
        <h6>Sends to the connected wallet address</h6>
        <h3>Tx Hash:</h3>
        {{ txHash }}
      </li>
      <li>
        <hr/>
        <h2>Send Token</h2>
        <label for="tokenAddress">
          token address
          <input
              id="tokenAddress"
              v-model="tokenAddress"
              placeholder="token address"
          /> </label
        ><br />
        <label for="tokenDecimals">
          token decimals
          <input
              id="tokenDecimals"
              v-model="tokenDecimals"
              placeholder="token decimals"
          /> </label
        ><br />
        <label for="tokenAmount">
          token amount
          <input
              id="tokenAmount"
              v-model="tokenAmount"
              placeholder="amount"
          /> </label
        ><br />
        <button v-show="tokenAddress !== ''" @click="approveToken(tokenAmount)">
          approve
        </button>
        <br/>
        <button v-show="tokenAddress !== ''" @click="sendToken(tokenAmount)">
          send
        </button>
        <h3>Tx Hash:</h3>
        {{ tokenTxHash }}
      </li>
      <li>
        <hr/>
        <h2>Other Actions</h2>
        <button @click="getAccount">get account</button>
        <h3>{{ account }}</h3>
      </li>
      <li>
        <input v-model="messageToSign" />
        <button @click="signMessage">sign message</button><br/>
        <textarea v-if="signature !== ''" v-model="signature" disabled style="margin: 0px; height: 169px; width: 454px;"></textarea>
        <br/>
      </li>
      <li>
        <button @click="getBalance">balance</button>
        <h3>{{ balance }}</h3>
      </li>
      <li>
        <button @click="getCoinBase">getCoinBase</button>
        <h3>{{ coinBase }}</h3>
      </li>
      <li>
        <button @click="makeCall">makeCall</button>
        <h3>{{ callRes }}</h3>
      </li>
      <li>
        <button @click="getChainId">getChainId</button>
        <h3>{{ chainId }}</h3>
      </li>
      <li>
        <button @click="createSubscription">createSubscription</button>
      </li>
    </ul>
  </div>
</template>

<script>
/* eslint-disable */


import Web3 from 'web3';
import Web3Modal from 'web3modal';
// @ts-ignore
import MewConnect from '@myetherwallet/mewconnect-web-client';
import BigNumber from 'bignumber.js';

export default {
  name: 'web3Modal',
  title: 'web3 Modal Example',
  data() {
    return {
      connect: {},
      altPopup: {},
      userAddress: '',
      ethereum: {},
      balance: 0,
      web3: {},
      coinBase: 0,
      txHash: '',
      callRes: '',
      chainId: 0,
      account: '',
      tokenAddress: '',
      tokenDecimals: 18,
      tokenTxHash: '',
      tokenAmount: 0,
      toAmount: 0,
      signature: '',
      toData: '0x',
      toAddressData: '',
      messageToSign: 'sign this'
    };
  },
  mounted () {
  },
  methods: {
    async setup () {
      const providerOptions = {
        mewconnect: {
          package: MewConnect,
          options: {
            infuraId: process.env.REACT_APP_INFURA_ID || '859569f6decc4446a5da1bb680e7e9cf'
          }
        },
      };

      const web3Modal = new Web3Modal({
        network: 'mainnet', // optional
        cacheProvider: true, // optional
        providerOptions // required
      });

      const provider = await web3Modal.connect();

      provider.on("close", () => {
        console.log('closed'); // todo remove dev item
        this.userAddress = ''
      });

      provider.on("accountsChanged", async (accounts) => {
        this.userAddress = accounts[0];
      });

      this.web3 = new Web3(provider);

      const accounts = await this.web3.eth.getAccounts();

      this.userAddress = accounts[0];
      // provider.on("chainChanged", async (chainId) => {
      //   const { web3 } = this.state;
      //   const networkId = await web3.eth.net.getId();
      //   await this.getAccountAssets();
      // });
      //
      // provider.on("networkChanged", async (networkId) => {
      //   const { web3 } = this.state;
      //   const chainId = await web3.eth.chainId();
      //   await this.getAccountAssets();
      // });
    },
    clearCachedProvider(){
      window.localStorage.removeItem('WEB3_CONNECT_CACHED_PROVIDER')
    },
    async disconnect(){
      if (this.web3 && this.web3.currentProvider && this.web3.currentProvider.close) {
        await this.web3.currentProvider.close();
      }
    },
    partialReset(){
      this.toAddressData = '';
      this.toData = '0x';
      this.toAmount = 0;
    },
    getAccount() {
      this.web3.currentProvider.send('eth_requestAccounts').then(accounts => {
        console.log(`User's address is ${accounts[0]}`);
      });
    },
    getBalance() {
      this.web3.eth
          .getBalance(this.userAddress)
          .then(bal => (this.balance = bal));
    },
    sendTx() {
      this.web3.eth.getBalance(this.userAddress).then(bal => this.balance);
      this.web3.eth.getGasPrice().then(gasPrice => {
        this.web3.eth.getTransactionCount(this.userAddress).then(nonce => {
          this.web3.eth
              .sendTransaction({
                from: this.userAddress,
                to: this.userAddress,
                nonce,
                value: new BigNumber(this.toAmount)
                    .times(new BigNumber(10).pow(18))
                    .toFixed(),
                gasPrice: gasPrice,
                gas: 21000
              })
              .once('transactionHash', hash => {
                console.log(['Hash', hash]);
                this.tokenTxHash = hash;
                this.partialReset();
              })
              .once('receipt', res => {
                console.log(['Receipt', res]);
              })
              .on('error', err => {
                console.log(['Error', err]);
              })
              .then(txhash => console.log('THEN: ', txhash));
        });
      });
    },
    sendTxData() {
      this.web3.eth.getBalance(this.userAddress).then(bal => this.balance);
      this.web3.eth.getGasPrice().then(gasPrice => {
        this.web3.eth.getTransactionCount(this.userAddress).then(nonce => {
          const rawTx = {
            from: this.userAddress,
            to: this.toAddressData === '' ? this.userAddress : this.toAddressData,
            nonce,
            value: new BigNumber(this.toAmount)
                .times(new BigNumber(10).pow(18))
                .toFixed(),
            gasPrice: gasPrice,
            data: this.toData
          }
          this.web3.eth
              .estimateGas(rawTx).then(gasLimit => {
            rawTx.gas = gasLimit;
            this.web3.eth
                .sendTransaction(rawTx)
                .once('transactionHash', hash => {
                  console.log(['Hash', hash]);
                  this.tokenTxHash = hash;
                  this.partialReset();
                })
                .once('receipt', res => {
                  console.log(['Receipt', res]);
                })
                .on('error', err => {
                  console.log(['Error', err]);
                })
                .then(txhash => console.log('THEN: ', txhash));
          })

        });
      });
    },
    signMessage() {
      this.web3.eth
          .sign(this.messageToSign, this.userAddress)
          .then(_signedMessage => {
            this.signature = JSON.stringify(
                {
                  address: this.userAddress,
                  msg: this.messageToSign,
                  sig: _signedMessage,
                  version: '3',
                  signer: 'MEWconnect'
                },
                null,
                2
            );
          })
          .catch(e => {
            console.log(e);
          });
    },
    sendToken(amount, decimals = this.tokenDecimals) {
      const jsonInterface = [
        {
          constant: false,
          inputs: [
            { name: '_to', type: 'address' },
            { name: '_amount', type: 'uint256' }
          ],
          name: 'transfer',
          outputs: [{ name: '', type: 'bool' }],
          payable: false,
          stateMutability: 'nonpayable',
          type: 'function'
        }
      ];
      const contract = new this.web3.eth.Contract(jsonInterface);
      const data = contract.methods
          .transfer(
              this.tokenAddress.toLowerCase(),
              new BigNumber(amount).times(new BigNumber(10).pow(decimals)).toFixed()
          )
          .encodeABI();

      let gasLimit = 100000;
      this.web3.eth
          .estimateGas({
            from: this.userAddress,
            to: this.tokenAddress,
            value: 0,
            data
          })
          .then(gas => {
            console.log(gas);
            gasLimit = gas;
          })
          .catch(console.error);

      this.web3.eth.getGasPrice().then(gasPrice => {
        this.web3.eth.getTransactionCount(this.userAddress).then(nonce => {
          this.web3.eth
              .sendTransaction({
                from: this.userAddress,
                to: this.tokenAddress,
                nonce,
                value: 0,
                gasPrice: gasPrice,
                gas: gasLimit,
                data
              })
              .once('transactionHash', hash => {
                console.log(['Hash', hash]);
                this.txHash = hash;
              })
              .once('receipt', res => {
                console.log(['Receipt', res]);
              })
              .on('error', err => {
                console.log(['Error', err]);
              })
              .then(txhash => console.log('THEN: ', txhash));
        });
      });
    },
    approveToken(amount, decimals = this.tokenDecimals) {
      const jsonInterface = [
        {
          "constant": false,
          "inputs": [
            {
              "internalType": "address",
              "name": "usr",
              "type": "address"
            },
            {
              "internalType": "uint256",
              "name": "wad",
              "type": "uint256"
            }
          ],
          "name": "approve",
          "outputs": [
            {
              "internalType": "bool",
              "name": "",
              "type": "bool"
            }
          ],
          "payable": false,
          "stateMutability": "nonpayable",
          "type": "function"
        }
      ];
      const contract = new this.web3.eth.Contract(jsonInterface);
      const data = contract.methods
          .approve(
              this.tokenAddress.toLowerCase(),
              new BigNumber(amount).times(new BigNumber(10).pow(decimals)).toFixed()
          )
          .encodeABI();

      let gasLimit = 100000;
      this.web3.eth
          .estimateGas({
            from: this.userAddress,
            to: this.tokenAddress,
            value: 0,
            data
          })
          .then(gas => {
            console.log(gas);
            gasLimit = gas;
          })
          .catch(console.error);

      this.web3.eth.getGasPrice().then(gasPrice => {
        this.web3.eth.getTransactionCount(this.userAddress).then(nonce => {
          this.web3.eth
              .sendTransaction({
                from: this.userAddress,
                to: this.tokenAddress,
                nonce,
                value: 0,
                gasPrice: gasPrice,
                gas: gasLimit,
                data
              })
              .once('transactionHash', hash => {
                console.log(['Hash', hash]);
                this.txHash = hash;
              })
              .once('receipt', res => {
                console.log(['Receipt', res]);
              })
              .on('error', err => {
                console.log(['Error', err]);
              })
              .then(txhash => console.log('THEN: ', txhash));
        });
      });
    },
    getCoinBase() {
      this.web3.eth.getCoinbase().then(cb => (this.coinBase = cb));
    },
    makeCall() {
      this.web3.eth
          .call({
            to: '0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe', // contract address
            data:
                '0xc6888fa10000000000000000000000000000000000000000000000000000000000000003'
          })
          .then(res => (this.callRes = res));
    },
    getChainId() {
      this.web3.eth.getChainId().then(res => (this.chainId = res));
    },
    createSubscription() {
      let subscription = this.web3.eth
          .subscribe('newBlockHeaders', function(error, result) {
            if (!error) {
              console.log(result);
            } else {
              console.log(error);
            }
          })
          .on('data', function(transaction) {
            console.log(transaction);
          })
          .on('error', function(transaction) {
            console.log(transaction);
          })
          .on('connected', function(transaction) {
            console.log(transaction);
          });
    }
  }
};
</script>

<style lang="scss">
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }

  ul {
    list-style-type: none;

    li {
      text-align: center;
    }
  }
</style>
