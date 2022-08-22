<template>
  <div class="app">
    <div class="header-container">
      <h1 id="page-title" class="text-center text-light py-4">Welcome to the foreign exchange API controller</h1>
    </div>
    <div class="container form-container">
      <h3 class="text-light">Currency converter</h3>
      <p class="disclaimer">Remember that you're working based on the {{baseCurrency}} currency</p>

      <div v-if="error" class="error-msg-container">
        <ErrorMessage />
      </div>

      <div class="form">
        <div class="mb-3">
          <label for="exampleFormControlInput1" class="form-label">Amount</label>
          <input type="number" class="form-control" id="exampleFormControlInput1" placeholder="Value to convert" v-model="userAmount" required="true">
        </div>
        <div class="mb-3">
          <label for="exampleFormControlInput1" class="form-label">Origin currency</label>
          <select id="selectOrigin" class="form-select" aria-label="Default select example" @change="onChangeOriginCurrency($event)">
            <option selected>Select your origin currency</option>
            <option v-for="(currency, index) in currencyNames" :key="index" :value="getObjectKey(currencyNames, currency)">{{`${currency} (${getObjectKey(currencyNames, currency)})`}}</option>
          </select>
        </div>
        <div class="mb-3">
          <label for="exampleFormControlInput1" class="form-label">Destination currency</label>
          <select id="selectDestination" class="form-select" aria-label="Default select example"  @change="onChangeDestinationCurrency($event)">
            <option selected>Select your destination currency</option>
            <option v-for="(currency, index) in currencyNames" :key="index" :value="getObjectKey(currencyNames, currency)">{{`${currency} (${getObjectKey(currencyNames, currency)})`}}</option>
          </select>
        </div>
        <button class="btn btn-success" @click="handleClick">Convert</button>
      </div>
      <div class="convert-response" v-if="mountConverted !== ''">
        <h1 :class="`text-dark ${userAmount === '' ? 'hidden':''}`">{{userAmount}} {{originCurrency}} = ${{mountConverted}} {{destinationCurrency.toLocaleString()}}</h1>
        <button class="clear-form-btn btn btn-warning" @click="resetForm">Reset form</button>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios';
  import ErrorMessage from '../components/ErrorMessage.vue';
  export default {
    name: "IndexPage",
    fetchDelay: 1000,
    data() {
        return {
            baseCurrency: "",
            currencyPrices: [],
            currencyNames: [],
            userAmount: "",
            originCurrency: "",
            destinationCurrency: "",
            mountConverted: "",
            error: false
        };
    },
    async fetch() {
        await this.getCurrencyPrices();
        await this.getCurrencyNames();
    },
    methods: {
        async getCurrencyPrices() {
            const response = await axios.get("https://openexchangerates.org/api/latest.json?app_id=9cc7bbd8398146fcb8272e273677dc43");
            this.currencyPrices = response.data.rates;
            //console.log(this.currencyPrices);
            this.baseCurrency = response.data.base;
        },
        async getCurrencyNames() {
            const response = await axios.get("https://openexchangerates.org/api/currencies.json?app_id=9cc7bbd8398146fcb8272e273677dc43");
            this.currencyNames = response.data;
            //console.log(this.currencyNames);
            this.currencyObjectives = Object.keys(response.data);
        },
        onChangeOriginCurrency(e) {
            this.originCurrency = e.target.value;
        },
        onChangeDestinationCurrency(e) {
            this.destinationCurrency = e.target.value;
        },
        getObjectKey(obj, value) {
            return Object.keys(obj).find(key => obj[key] === value);
        },
        handleClick() {
            let originCurrencyPrice = this.currencyPrices[`${this.originCurrency}`];
            let destinationCurrencyPrice = this.currencyPrices[`${this.destinationCurrency}`];
            if (this.userAmount === 0 || this.userAmount === "" || this.originCurrency === "" || this.destinationCurrency === "") {
                this.error = true;
            }
            else {
              this.error = false;
              let dollarUSLocale = Intl.NumberFormat("en-US");
              this.mountConverted = this.userAmount / originCurrencyPrice * destinationCurrencyPrice;
              this.mountConverted = dollarUSLocale.format(this.mountConverted);
            }
        },
        resetForm(){
          this.userAmount = '';
          this.originCurrency = '';
          document.getElementById("selectOrigin").selectedIndex = 0;
          this.destinationCurrency = '';
          document.getElementById("selectDestination").selectedIndex = 0;
          this.mountConverted = '';
        }
    },
    components: { ErrorMessage }
}
</script>

<style scoped>

  *{
    font-family: 'Bebas Neue', cursive;
  }

  #page-title{
    font-size: 3rem;
  }

  .form-container{
    background-color: #2B4865;
    padding: 15px;
    border-radius: 20px;
    border: solid;
  }

  .form{
    margin: 20px 0;
  }

  .form-label{
    color: white;
  }

  .disclaimer{
    color: #FCF8E8;
  }

  .convert-response{
    border: solid;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 5px;
    border-radius: 10px;
    background-color: #F1F1F1;
  }

</style>
