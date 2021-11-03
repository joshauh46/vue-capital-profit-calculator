<template>
  <div class="container">
    <h1 style="text-align: center margin-top: 40px margin-bottom: 40px">
      Capital Calculator
    </h1>

    <div class="col-lg-6 offset-md-3">
      <div class="row">
        <div class="col">
          <label for="Base Currency">Base Currency</label>

          <select
            class="form-control"
            v-model="baseCurrency"
            @change="calculateCurrencyConversion('target')"
          >
            <option v-for="(value, key) in currencies" :key="value">
              {{ key.toUpperCase() }}
            </option>
          </select>

          <br />

          <div class="form-group">
            <input
              type="number"
              inputmode="decimal"
              class="form-control"
              v-model="baseCurrencyInput"
              @input="calculateCurrencyConversion('base')"
            />
          </div>
        </div>

        <div class="col">
          <label for="Target Currency">Target Currency</label>
          <select
            class="form-control"
            v-model="targetCurrency"
            @change="calculateCurrencyConversion('base')"
          >
            <option v-for="(value, key) in currencies" :key="value">
              {{ key.toUpperCase() }}
            </option>
          </select>

          <br />

          <div class="form-group">
            <input
              type="number"
              inputmode="decimal"
              class="form-control"
              v-model="targetCurrencyInput"
              @input="calculateCurrencyConversion('target')"
            />
          </div>
        </div>
      </div>
    </div>

    <hr />
    <form>
      <div class="col-lg-6 offset-md-3">
        <div class="row">
          <div class="col">
            <label for="Base Currency">Satoshi</label> <br />
            <div class="form-group">
              <input
                v-model="Satoshi"
                class="form-control"
                inputmode="decimal"
                type="text"
                placeholder="Satoshi"
                @input="calculateBtcUsdConversion"
              />
            </div>
          </div>

          <div class="col">
            <label for="Base Currency">USD</label> <br />
            <div class="form-group">
              <input
                v-model="USD"
                class="form-control"
                inputmode="decimal"
                type="text"
                placeholder="USD"
                @input="calculateUsdBtcConversion"
              />
            </div>
          </div>
        </div>
      </div>
    </form>

    <hr />

    <div class="col-lg-6 offset-md-3">
      <fieldset class="values">
        <div class="row">
          <div class="col">
            <label for="From">From</label>
            <div class="from-group">
              <input
                type="text"
                inputmode="decimal"
                class="form-control"
                placeholder="Enter From Value"
                v-model="PercentInputOne"
              />
            </div>
          </div>
          <div class="col">
            <label for="To">To</label>
            <div class="from-group">
              <input
                type="text"
                inputmode="decimal"
                class="form-control"
                placeholder="Enter To Value"
                v-model="PercentInputTwo"
              />
            </div>
          </div>
        </div>
      </fieldset>

      <fieldset class="results" data-children-count="1">
        <br class="remove" />
        <div class="input-group mb-3">
          <div class="input-group-prepend">
            <span class="input-group-text" id="basic-addon1">%</span>
          </div>
          <input
            type="text"
            v-model="PercentResult"
            class="form-control"
            readonly="readonly"
            name="answer"
            style="
              border-top-left-radius: 0px !important
              border-bottom-left-radius: 0px !important
            "
          />
        </div>
        <div class="row">
          <div class="col">
            <input
              type="button"
              value="Calculate"
              class="btn btn-primary btn-block"
              @click="getPercent()"
            />
          </div>
          <div class="col">
            <input
              type="button"
              value=" Copy %"
              class="btn btn-primary btn-block"
              @click="copyPercent()"
            />
          </div>
        </div>
      </fieldset>
    </div>

    <br />

    <div class="col-lg-6 offset-md-3">
      <div class="row">
        <div class="col">
          <label for="Capital">Capital</label> <br />
          <div class="form-group">
            <input
              class="form-control"
              inputmode="decimal"
              type="number"
              min="0"
              v-model="capitalInputOne"
              placeholder="Enter Capital"
            />
          </div>
        </div>
        <div class="col">
          <label for="Percentage">Percentage</label> <br />
          <div class="form-group">
            <input
              class="form-control"
              inputmode="decimal"
              type="number"
              min="0"
              v-model="capitalInputTwo"
              placeholder="Enter Percentage"
            />
          </div>
        </div>
      </div>
      <div class="input-group mb-3">
        <div class="input-group-prepend">
          <span class="input-group-text" id="basic-addon1">$</span>
        </div>
        <input
          type="text"
          size="6"
          class="form-control"
          readonly="readonly"
          v-model="capitalGainResult"
          style="
            border-top-left-radius: 0px !important
            border-bottom-left-radius: 0px !important
          "
        />
      </div>
    </div>
  </div>
</template>

<script>
import copy from "copy-to-clipboard"
import axios from "axios"


export default {
  data() {
    return {
      currencies: [],
      baseCurrency: null,
      targetCurrency: null,
      baseCurrencyInput: null,
      targetCurrencyInput: null,
      PercentInputOne: null,
      PercentInputTwo: null,
      PercentResult: null,
      BTCPrice: null,
      capitalInputOne: null,
      capitalInputTwo: null,
      Satoshi: null,
      USD: null
    }
  },

  created() {
    this.getCurrenciesList()
    this.getBTCPrice()
  },

  methods: {
    async getCurrenciesList() {
      try {
        const response = await axios.get(
          "https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies.json"
        )
        this.currencies = response.data
        this.baseCurrency = "USD"
        this.targetCurrency = "EUR"
        this.baseCurrencyInput = "1"
        this.calculateCurrencyConversion("base")
      } catch (error) {
        console.log(error)
      }
    },

    async getCurrencyRate(baseCurrency, targetCurrency) {
      try {
        const response = await axios.get(
          `https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies/${baseCurrency}.json`
        )
        let data = response.data
        return data[baseCurrency][targetCurrency]
      } catch (error) {
        console.log(error)
      }
    },

    getPercent() {
      let Result = ((this.PercentInputTwo - this.PercentInputOne) / this.PercentInputOne) * 100
      this.PercentResult = Math.round(Result * Math.pow(10, 2)) / Math.pow(10, 2)
    },

    copyPercent() {
      copy(this.PercentResult)
    },

    async getBTCPrice() {
      try {
        const response = await axios.get(
          "https://api.coindesk.com/v1/bpi/currentprice.json"
        )
        this.BTCPrice = response.data.bpi.USD.rate.replace(/,/g, null)
      } catch (error) {
        console.log(error)
      }
    },

    calculateBtcUsdConversion() {
      this.USD = (1e-8 * this.Satoshi * this.BTCPrice).toFixed(8)
    },

    calculateUsdBtcConversion() {
      this.Satoshi = ((1e8 * this.USD) / this.BTCPrice).toFixed(2)
    },

    async calculateCurrencyConversion(type) {
      const baseCurrency = this.baseCurrency.toLowerCase()
      const targetCurrency = this.targetCurrency.toLowerCase()

      if (type === "base") {
        const result = await this.getCurrencyRate(baseCurrency, targetCurrency)
        this.targetCurrencyInput = (this.baseCurrencyInput * result).toFixed(2)
      } else {
        const result = await this.getCurrencyRate(targetCurrency, baseCurrency)
        this.baseCurrencyInput = (this.targetCurrencyInput * result).toFixed(2)
      }
      
    },
  },

  computed: {
    capitalGainResult() {
      return (
        this.capitalInputOne + (this.capitalInputOne * this.capitalInputTwo) / 100
      )
    },
  },
}
</script>
