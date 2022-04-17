<template>
  <div>
    <div class="container" style="max-width:800px">
     <div class="mb-5" style="margin-top:100px">
       <h1>JPYC Faucet for Tip bot</h1>
     </div>
     <div class="input-group mb-5">
       <input class="form-control" v-model="faucetValue" v-bind:class="{ 'is-invalid' : isInvalidValue }" type="text" placeholder="Social Network URL containing your withdrawal tweet">
       <div class="invalid-tooltip">
         {{errorMsg}}  
       </div>

       <button class="btn btn-secondary" v-on:click="submit()">Give Me Gas!</button>
     </div>

     <div class="text-start">
     <h2>How does this work?</h2>
     <p>This faucet allows users who perform certain actions to obtain gas tokens for the blockchain.</p>
     
     <p>It is currently available to users of the JPYC Tip Bot.</p>
     <p>To request funds via Twitter, make a tweet for a JPYC Tip Bot withdrawal request.(Only tweets of approved requests are valid.)
     Copy-paste the tweets URL into the above input box and fire away!The faucet is running invisible reCaptcha protection against bots.</p>
     <h3>Donate and Contribute</h3>

     <p>This faucet can be contributed not only through direct donations, but also through projects registered with Public Property DAO.</p>

     <p>To donate directly, please send Matic tokens to the following address (currently Polygon network only)</p>
     <div class="col-8 col-offset-2" style="border-style:solid">
       <p>{{ faucetAddress }}<img v-on:click="addressCopy()" src="@/assets/clipboard_icon.png" > </p>
     </div>


     <p>Click here for the project registered with Public Property DAO.</p>
    </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import axios from 'axios'

export default defineComponent({
  name: 'Home',
  data() {
     return {
       debug:false,
       faucetBalance:0,
       faucetValue:'',
       faucetAddress:  '',
       errorMsg:'Invalid Request!',
       isInvalidValue : false,
     }
  },
  methods: {
    async submit() {
      try {
        if(this.faucetValue == '' || this.faucetValue.match(/(http[s]?):\/\/[^/.]+?\..+\w$/i) == null ) {
          this.isInvalidValue = true;
          this.errorMsg = 'Input URL!';
          return;
        }
        this.isInvalidValue = false;
        const result = await axios.post(process.env.VUE_APP_DOMAIN, 
          {
            tweet : this.faucetValue as string
          }
        );
        const data = JSON.parse(result.data);
        if(data.status == "error" ) {
          this.isInvalidValue = true;
          this.errorMsg = data.message;
          return;
        }
        
      } catch (err: any){
          this.errorMsg = 'Invalid Request!';
          this.isInvalidValue = true;
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
    this.faucetAddress = process.env.VUE_APP_DEPOSIT_ADDRESS   
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
