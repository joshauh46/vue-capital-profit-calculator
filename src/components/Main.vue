<template>
  <div class='container'>
    <h1 class='introheading'>
      Capital Calculator
    </h1>

    <div class='col-lg-6 offset-md-3'>
      <div class='row'>
        <div class='col'>
          <label for='Base Currency'>Base Currency</label>

          <select
            class='form-control'
            v-model='state.baseCurrency'
            @change='calculateCurrencyConversion("target")'
          >
            <option v-for='(value, key) in state.currencies' :key='value'>
              {{ key.toUpperCase() }}
            </option>
          </select>

          <br />

          <div class='form-group'>
            <input
              type='number'
              inputmode='decimal'
              class='form-control'
              v-model='state.baseCurrencyInput'
              @input='calculateCurrencyConversion("base")'
            />
          </div>
        </div>

        <div class='col'>
          <label for='Target Currency'>Target Currency</label>
          <select
            class='form-control'
            v-model='state.targetCurrency'
            @change='calculateCurrencyConversion("base")'
          >
            <option v-for='(value, key) in state.currencies' :key='value'>
              {{ key.toUpperCase() }}
            </option>
          </select>

          <br />

          <div class='form-group'>
            <input
              type='number'
              inputmode='decimal'
              class='form-control'
              v-model='state.targetCurrencyInput'
              @input='calculateCurrencyConversion("target")'
            />
          </div>
        </div>
      </div>
    </div>

    <hr />
    
    <form>
      <div class='col-lg-6 offset-md-3'>
        <div class='row'>
          <div class='col'>
            <label for='Base Currency'>Satoshi</label> 
            
            <br />

            <div class='form-group'>
              <input
                v-model='state.Satoshi'
                class='form-control'
                inputmode='decimal'
                type='text'
                placeholder='Satoshi'
                @input='calculateBtcUsdConversion'
              />
            </div>
          </div>

          <div class='col'>
            <label for='Base Currency'>USD</label> 
            
            <br />

            <div class='form-group'>
              <input
                v-model='state.USD'
                class='form-control'
                inputmode='decimal'
                type='text'
                placeholder='USD'
                @input='calculateUsdBtcConversion'
              />
            </div>
          </div>
        </div>
      </div>
    </form>

    <hr />

    <div class='col-lg-6 offset-md-3'>
      <fieldset class='values'>
        <div class='row'>
          <div class='col'>
            <label for='From'>From</label>
            <div class='from-group'>
              <input
                type='text'
                inputmode='decimal'
                class='form-control'
                placeholder='Enter From Value'
                v-model='state.PercentInputOne'
              />
            </div>
          </div>
          <div class='col'>
            <label for='To'>To</label>
            <div class='from-group'>
              <input
                type='text'
                inputmode='decimal'
                class='form-control'
                placeholder='Enter To Value'
                v-model='state.PercentInputTwo'
              />
            </div>
          </div>
        </div>
      </fieldset>

      <fieldset class='results' data-children-count='1'>
        <br class='remove' />
        <div class='input-group mb-3'>
          <div class='input-group-prepend'>
            <span class='input-group-text'>%</span>
          </div>
          <input
            type='text'
            v-model='state.PercentResult'
            class='form-control custom-input'
            readonly='readonly'
          />
        </div>
        <div class='row'>
          <div class='col'>
            <input
              type='button'
              value='Calculate'
              class='btn btn-primary btn-block'
              @click='getPercent()'
            />
          </div>
          <div class='col'>
            <input
              type='button'
              value=' Copy %'
              class='btn btn-primary btn-block'
              @click='copyPercent()'
            />
          </div>
        </div>
      </fieldset>
    </div>

    <br />

    <div class='col-lg-6 offset-md-3'>
      <div class='row'>
        <div class='col'>
          <label for='Capital'>Capital</label> 
          
          <br />

          <div class='form-group'>
            <input
              class='form-control'
              inputmode='decimal'
              type='number'
              min='0'
              v-model='state.capitalInputOne'
              placeholder='Enter Capital'
            />
          </div>
        </div>
        <div class='col'>
          <label for='Percentage'>Percentage</label> 
          
          <br />
          
          <div class='form-group'>
            <input
              class='form-control'
              inputmode='decimal'
              type='number'
              min='0'
              v-model='state.capitalInputTwo'
              placeholder='Enter Percentage'
            />
          </div>
        </div>
      </div>
      <div class='input-group mb-3'>
        <div class='input-group-prepend'>
          <span class='input-group-text'>$</span>
        </div>
        <input
          type='text'
          size='6'
          class='form-control custom-input'
          readonly='readonly'
          v-model='capitalGainResult'
        />
      </div>
    </div>
  </div>
</template>

<script >
import { reactive, computed } from 'vue'
import copy from 'copy-to-clipboard'
import axios from 'axios'


export default {
  setup() {

    const state = reactive({
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
    })

    const getCurrenciesList = async () => {
      try {
        const response = await axios.get(
          'https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies.json'
        )
        state.currencies = response.data
        state.baseCurrency = 'USD'
        state.targetCurrency = 'EUR'
        state.baseCurrencyInput = '1'
        calculateCurrencyConversion('base')
      } catch (error) {
        console.log(error)
      }
    }

    const getCurrencyRate = async (baseCurrency, targetCurrency) => {
      try {
        const response = await axios.get(
          `https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies/${baseCurrency}.json`
        )
        const data = response.data
        return data[baseCurrency][targetCurrency]
      } catch (error) {
        console.log(error)
      }
    }

    const getPercent = () => {
      const Result = ((state.PercentInputTwo - state.PercentInputOne) / state.PercentInputOne) * 100
      state.PercentResult = Math.round(Result * Math.pow(10, 2)) / Math.pow(10, 2)
    }

    const copyPercent = () => {
      copy(state.PercentResult)
    }

    const getBTCPrice = async () => {
      try {
        const response = await axios.get(
          'https://api.coindesk.com/v1/bpi/currentprice.json'
        )
        state.BTCPrice = response.data.bpi.USD.rate.replace(/,/g, '')
      } catch (error) {
        console.log(error)
      }
    }

    const calculateBtcUsdConversion = () => {
      state.USD = (1e-8 * state.Satoshi * state.BTCPrice).toFixed(8)
    }

    const calculateUsdBtcConversion = () => {
      state.Satoshi = ((1e8 * state.USD) / state.BTCPrice).toFixed(2)
    }

    const calculateCurrencyConversion = async (type) => {
      const baseCurrency = state.baseCurrency.toLowerCase()
      const targetCurrency = state.targetCurrency.toLowerCase()

      if (type === 'base') {
        const result = await getCurrencyRate(baseCurrency, targetCurrency)
        state.targetCurrencyInput = (state.baseCurrencyInput * result).toFixed(2)
      } else {
        const result = await getCurrencyRate(targetCurrency, baseCurrency)
        state.baseCurrencyInput = (state.targetCurrencyInput * result).toFixed(2)
      }
    }


    const capitalGainResult = computed(() => {
      return state.capitalInputOne + (state.capitalInputOne * state.capitalInputTwo) / 100
    })


    getCurrenciesList()
    getBTCPrice()


    return {
      state,
      getCurrenciesList,
      getCurrencyRate,
      getBTCPrice,
      getPercent,
      copyPercent,
      capitalGainResult,
      calculateBtcUsdConversion,
      calculateUsdBtcConversion,
      calculateCurrencyConversion
    }

  }
}
</script>

<style scoped>
.introheading {
  text-align: center; 
  margin-top: 40px; 
  margin-bottom: 40px;
}
.custom-input {
 border-top-left-radius: 0px !important;
 border-bottom-left-radius: 0px !important;
}
</style>