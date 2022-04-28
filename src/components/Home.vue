<template>
  <div>
    <div class="container" style="max-width:800px">
     <div class="mb-5" style="margin-top:100px">
       <h1>JPYC Faucet for Tip bot</h1>
     </div>
     <div class="input-group mb-2">
       <input class="form-control" v-model="faucetValue" v-bind:class="{ 'is-invalid' : isInvalidValue }" type="text" placeholder="Social Network URL containing your withdrawal tweet">

       <button v-if="!isProcessing" class="btn btn-primary" v-on:click="submit()" v-bind:disabled="recaptchaToken == ''">Give Me Gas!</button>
       <button v-if="isProcessing" class="btn btn-primary" type="button" disabled>
         <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
         Sending...
       </button>
     </div>
     <p>現在のFaucet残高 : {{faucetBalance}} Matic <small v-if="isBalanceShort" class="text-danger">残高が少ないです、、、</small></p>

     <div v-if="isSuccess" class="alert alert-success  p-1" role="alert">
       Success!!  <a v-bind:href="explorerUrl" style="max-width:200px"  target="_blank" rel="noopener noreferrer">{{txHash}}</a>
     </div>
      <div v-if="isInvalidValue" class="alert alert-danger  p-1" role="alert">
       {{errorMsg}} 
     </div>

     
     <hr> 
     <div class="text-start mb-5">
       <h2>How does this work?</h2>
       <p>このFaucetは、ユーザーが特定のアクションを行うことで、ブロックチェーン用のガストークンを入手することができます。</p>
       <p>現在、JPYC Tip Botの利用者が利用可能です。</p>
       <p>JPYC Tip Botの出金依頼をツイートしてください（承認された依頼のツイートのみ有効です）。下記画像のようなツイートのURLを上記の入力ボックスにコピーペーストして送信してください！</p>

       <div class="mb-2 text-center">
         <img src="@/assets/tipbot_tweet.png"  style="max-width:400px">
       </div>
       <p>このFaucetはボットに対する見えないreCaptcha保護機能を実行しています。</p>
       <p>[注意]</p>
       <p><small class="text-danger">すでにMaticトークンを持っているアドレスは付与対象外になります</small></p>
       <p><small class="text-danger">同じTwitterアカウントで複数回利用はできません(現時点)</small></p>
       <p><small class="text-danger">30日以内のツイートが対象です</small></p>
       <h3>Donate and Contribute</h3>
       <p>To donate directly, please send Matic tokens to the following address (currently Polygon network only)</p>
       <div class="col-8 col-offset-2 p-2 text-center" style="border-style:solid;">
         <span>{{ faucetAddress }}<img v-on:click="addressCopy()" src="@/assets/clipboard_icon.png" > </span>
       </div>

     </div>
    </div>

  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import axios from 'axios'
import Web3 from 'web3'
const blockExplorers = "https://polygonscan.com/tx/";

let web3: Web3;
export default defineComponent({
  name: 'Home',
  data() {
     return {
       debug:false,
       faucetBalance: 0,
       recaptchaToken:'',
       faucetValue:'',
       faucetAddress:  '',
       errorMsg:'Invalid Request!',
       isInvalidValue : false,
       explorerUrl : '',
       txHash : '',
       isSuccess : false,
       isBalanceShort : false,
       isProcessing : false,
     }
  },
  methods: {
    async submit() {
      try {
        if(this.faucetValue == '' || this.faucetValue.match(/(http[s]?):\/\/[^/.]+?\..+\w$/i) == null ) {
          this.isInvalidValue = true;
          this.errorMsg = 'ツイートのURLを貼り付けてください!';
          return;
        }

        if(this.isBalanceShort) {
          this.isInvalidValue = true;
          this.errorMsg = '現在残高が十分にないため、機能停止中です'
          return;
        }

        this.isProcessing = true;
        this.isInvalidValue = false;
        const result = await axios.post(process.env.VUE_APP_DOMAIN as string, 
          {
            token : this.recaptchaToken as string,
            tweet : this.faucetValue as string
          }
        );
        const data = JSON.parse(result.data);
        if(data.status == "error" ) {
          this.isInvalidValue = true;
          this.errorMsg = data.message;
          this.isProcessing = false;
          return;
        }

        this.explorerUrl = blockExplorers + data.txId;
        this.txHash = data.txId;
        this.isSuccess = true;
        this.isProcessing = false;
        
      } catch (err: any){
        console.log(err);
        this.errorMsg = '不正なリクエストです!';
        this.isInvalidValue = true;
        this.isProcessing = false;
        /* no execution*/
      }
    },
    addressCopy() {
      navigator.clipboard.writeText(this.faucetAddress).catch((e) => {
        console.error(e)
      })
    }
  },
  async created() {
    web3 = new Web3(process.env.VUE_APP_PROVIDER as string);
    this.faucetAddress = process.env.VUE_APP_DEPOSIT_ADDRESS as string
    let tmpBalance = web3.utils.fromWei(await web3.eth.getBalance(this.faucetAddress));
    this.faucetBalance = Math.floor(parseFloat(tmpBalance) * 100) / 100;
    if(this.faucetBalance < 0.1) {
      this.isBalanceShort = true;
    }
  },
  mounted() {
    let externalScript = document.createElement('script')
    externalScript.async = true
    externalScript.onload = () => {
      (window as any).grecaptcha.ready(() => {
        (window as any).grecaptcha.execute(process.env.VUE_APP_SITEKEY, {action: 'submit'}).then( async (token: any) =>
          {
            this.recaptchaToken = token;
          });
        });
    }
    externalScript.setAttribute('src', "https://www.google.com/recaptcha/api.js?render=" + process.env.VUE_APP_SITEKEY); 
    document.head.appendChild(externalScript);
  
  }

});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
