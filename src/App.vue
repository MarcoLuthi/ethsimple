<template>
<div id="app">
    <div class="max-w-7xl mx-auto sm:px-6 lg:px-8 sm:py-6 lg:py-8">
        <div class="md:flex md:items-center md:justify-between mb-16">
            <div class="flex-1 min-w-0">
                <h2 class=" text-xl font-bold leading-7 text-gray-900 sm:text-xl sm:truncate">
                    Please set metamask to Ropsten
                </h2>
            </div>
            <div class="mt-4 flex md:mt-0 md:ml-4">
                <button
                    v-on:click="connect()"
                    v-if="account === null"
                    type="button"
                    class="ml-3 inline-flex items-center px-4 py-2 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
                    Connect wallet
                </button>
                <div v-else>
                    <span href="#" class="bg-gray-200  text-gray-700 px-3 py-2 font-medium text-sm rounded-l-md">
                        {{limitETH}}
                        ETH

                    </span>
                    <span class="bg-gray-100 truncate  w-36 text-gray-700 px-3 py-2 font-medium text-sm rounded-r-md">
                        {{account}}
                    </span>

                </div>
            </div>
        </div>
        <ul class="grid grid-cols-1 gap-6 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
            <Token v-for="data in info.data" :key="data.id" :state="status[data.id]" :data="data" :connected='connected' @check="check(data.id)" @buy="buy(data.id)"></Token>

        </ul>
    </div>



</div>
</template>

<script>
import Token from './components/Token.vue'
import { ethers } from "ethers";
import axios from 'axios'

export default {
  name: 'App',
  components: {
    Token
  },
  mounted () {
    axios
      .get('https://api.pattern.fyi/tokensLMBO/')
      .then(response => (this.info = response)).then(() =>{
        if(!window.ethereum){
          this.signer = new ethers.providers.InfuraProvider('ropsten')
          this.contractAddress ='0x8120d2990D068376ac349373A7d4dDa7901F2c6a'
          this.initApp()
          setTimeout(function () {
            this.connected = true;
          }.bind(this), 2000);
        }else{
          this.ethers = new ethers.providers.Web3Provider(window.ethereum)
          this.contractAddress ='0x43CF70f925B3e92beB68824311e6CF75a31eef74'
          this.signer =  this.ethers.getSigner()

          this.initApp()
          setTimeout(function () {
            this.connected = true;
          }.bind(this), 2000);
        }
      })



  },
  data() {
        return {
          info: [],
          checkSold: 0,
          connected: false,
          ethers: null,
          signer: null,
          balance: null,
          ethBalance: '0',
          accounts: [],
          status: [],
          account: null,
          contract: null,
          contractAddress: "0x43CF70f925B3e92beB68824311e6CF75a31eef74",
          contractAbi: require('@/NFT.json'),
          txPending: false,
          txComplete: false,
          tokenSettings: {
            tokenNameInput: null,
            tokenSymbolInput: null,
            tokenThresholdInput: null,
          },
      }
  },

methods: {
        async connect() {

          if (window.ethereum) {
            await window.ethereum.enable();
            this.signer =  this.ethers.getSigner()
            this.account =  await this.signer.getAddress()
            this.balance = await this.signer.getBalance()
            this.ethBalance =  await ethers.utils.formatEther(this.balance)

            //window.ethereum.request('eth_requestAccounts');

            this.connected = true;

            //this.initApp()
          } else {
            alert(
              "To use this app you'll need metamask or another web3 provider.'"
            );
          }
        },
        async initApp() {
          this.contract = new ethers.Contract(
            this.contractAddress,
            this.contractAbi.abi,
            this.signer);
        },
        async sold(item){
          if(this.connected === true){
          this.contract.exists(item).then(transaction => {
          this.status[item].owned = transaction
          this.status[item].loading = false
          return transaction
        }).then(owner => {
          if(this.status[item].owned === true){
          this.status[item].owner = this.owner(item)
          }
        });
          }
        },
        async owner(item){
          if(this.connected === true){
          this.contract.ownerOf(item).then(transaction => {
          this.status[item].owner = transaction
          this.status[item].pendingTx = false
          });
          }
        },
        async getPrice(item){
          this.contract.getPrice(item).then(transaction => {
          this.status[item].price = transaction,
          this.status[item].ethPrice = ethers.utils.formatEther(transaction)

          });

        },
        check(item){
          setTimeout(function () {
            this.sold(item)
            this.getPrice(item)
            console.log('checking')
          }.bind(this), 2000);
        },
        transactionCheck(item){
          var checker = setInterval(function () {
            this.sold(item)
            console.log('Still Checking')
            if(this.status[item].pendingTx === false)
            clearInterval(checker)
          }.bind(this), 1000);
        },
        async buy(item){
           const parms ={
            from: this.account,
            value: this.status[item].price
           }

           console.log(parms)

          var sendBuy = this.contract.buy(item, parms);

          sendBuy.then(transaction =>{
          this.status[item].pendingTx = true;
        }).then(result => {
          this.transactionCheck(item)
          console.log('checking')
        });
        },
  },
  computed: {
    limitETH: function () {
      if(this.ethBalance !== null){
      var decimal = Number(this.ethBalance).toFixed(2)
      return decimal
     }
   },

 },
 watch: {
   connected: function(val){
     if(val === true){
       for (let i = 0; i < this.info.data.length; i++) {
        // Runs 5 times, with values of step 0 through 4.
        this.status.push({
          owned: false,
          owner: '',
          loading: true,
          pendingTx: false,
          price: '',
          ethPrice: ''
        })
        }
     }
   }
 }

}
</script>

<style>
body {
 font-size: 10px !important;
}
</style>
