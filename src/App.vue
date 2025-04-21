<script setup>
import { ref, watch, onMounted } from 'vue'
import axios from 'axios'

/*Supported tickers on free tier:
Symbol Limited to 
AAPL, TSLA, AMZN, MSFT, NVDA, GOOGL, META, NFLX, JPM, V, BAC, AMD, PYPL, 
DIS, T, PFE, COST, INTC, KO, TGT, NKE, SPY, BA, BABA, XOM, WMT, GE, CSCO, 
VZ, JNJ, CVX, PLTR, SQ, SHOP, SBUX, SOFI, HOOD, RBLX, SNAP, AMD, UBER, 
FDX, ABBV, ETSY, MRNA, LMT, GM, F, RIVN, LCID, CCL, DAL, UAL, AAL, TSM, 
SONY, ET, NOK, MRO, COIN, RIVN, SIRI, SOFI, RIOT, CPRX, PYPL, TGT, VWO, 
SPYG, NOK, ROKU, HOOD, VIAC, ATVI, BIDU, DOCU, ZM, PINS, TLRY, WBA, VIAC, 
MGM, NFLX, NIO, C, GS, WFC, ADBE, PEP, UNH, CARR, FUBO, HCA, TWTR, BILI, 
SIRI, VIAC, FUBO, RKT */

// State to store the ticker and the stock data
const ticker = ref('')
const stockData = ref([])

// Store the API key in a reactive variable
const apiKey = ref(import.meta.env.VITE_API_KEY || '')
const showApiKey = ref(false)

const toggleApiKeyVisibility = () => {
  showApiKey.value = !showApiKey.value
}

const copyApiKey = async () => {
  try {
    await navigator.clipboard.writeText(apiKey.value)
    alert('API key copied!')
  } catch (err) {
    alert('Failed to copy API key.')
  }

}

const lookupStock = async () => {
  const tickerSymbol = ticker.value.toUpperCase().trim()
  
  if (tickerSymbol === '') return

  try {
    const url = `https://financialmodelingprep.com/stable/dividends?symbol=${tickerSymbol}&apikey=${apiKey.value}`
    // Fetch stock data from the API
    const response = await axios.get(url)
    

    // If the stock is found, update stockData
    if (response.data && response.data.length > 0) {
      const latest = response.data[0]
      // Calculate ex-dividend date = recordDate - 1 day (simplified)
      const recordDate = new Date(latest.recordDate)
      const exDivDate = new Date(recordDate)
      exDivDate.setDate(recordDate.getDate() - 1)
      
      const stock = {
          ticker: latest.symbol,
          dividendYield: `${latest.yield.toFixed(2)}%`,
          dividendAmount: `$${latest.dividend.toFixed(2)}`,
          computedExDivDate: exDivDate.toISOString().split('T')[0], // format as YYYY-MM-DD // calculated from recordDate - 1 day
          recordDate: latest.recordDate,
          paymentDate: latest.paymentDate,
          frequency: latest.frequency,
          declarationDate: latest.declarationDate,
      }
      stockData.value.unshift(stock)
    } else {
      alert('Stock not found or no data available.')
    }
  } catch (error) {
    console.error('Error fetching stock data:', error)
    alert('Failed to fetch stock data.')
  }
}
</script>


<template>
  <div>
    <h1><img class="main-image" src="/icon-polka.png" /> Dividend Tracker</h1>

    <!-- Config section -->
    <!-- Display the API key field -->
    
   
    <div class="api-key-wrapper">
      <p>Tip: Store your FMP API key in the <strong>.env</strong> file to avoid repeatedly typing it in.</p>
      <input
        :type="showApiKey ? 'text' : 'password'"
        v-model="apiKey"
        placeholder="Enter your FMP API key"
        class="api-input"
      />
      <button @click="copyApiKey" title="Copy to clipboard">📋</button>
      <button @click="toggleApiKeyVisibility" title="Show/Hide API Key">
        {{ showApiKey ? '🙈' : '👁️' }}
      </button>
    </div>

    <!-- Search Box and Lookup Button -->
    <div>
      <input @keyup.enter="lookupStock" v-model="ticker" type="text" placeholder="Enter Stock Ticker" />
      <button @click="lookupStock">Lookup</button>
    </div>

    <!-- Table to Display Stock Info -->
    <table v-if="stockData.length > 0">
      <thead>
        <tr>
          <th>Ticker</th>
          <th>Dividend Yield</th>
          <th>Dividend Amount</th>
          <th>Declaration Date</th>
          <th>Ex-Dividend Date</th>
          <th>Record Date</th>
          <th>Next Payout Date</th>
          <th>Payment Frequency</th>
          
          <!-- Add more columns as needed -->
        </tr>
      </thead>
      <tbody>
        <tr v-for="(stock, index) in stockData" :key="index">
          <td>{{ stock.ticker }}</td>
          <td>{{ stock.dividendYield }}</td>
          <td>{{ stock.dividendAmount }}</td>
          <td>{{ stock.declarationDate }}</td>
          <td>{{ stock.computedExDivDate }}</td>
          <td>{{ stock.recordDate }}</td>
          <td>{{ stock.paymentDate }}</td>
          <td>{{ stock.frequency }}</td>
          
        </tr>
      </tbody>
    </table>
  </div>
</template>


<style scoped>

input {
  padding: 0.5em;
  margin: 1em;
  font-size: 1em;
}

table {
  width: 100%;
  margin-top: 1em;
  border-collapse: collapse;
}

th, td {
  padding: 1em;
  border: 1px solid #ddd;
}

.main-image
{
  width: 0.7em;
}

.api-key-wrapper {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.5rem;
  margin-top: 0.5rem;
  max-width: 700px;
}

.api-input {
  flex: 1 1 400px; /* Flex-grow + shrink, base width */
  min-width: 250px;
  padding: 0.5rem;
  font-family: monospace;
  font-size: 1rem;
}

button {
  padding: 0.5rem 0.6rem;
  font-size: 1.1rem;
  cursor: pointer;
  background-color: #f0f0f0;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button:hover {
  background-color: #e2e2e2;
}

.hint {
  font-size: 0.9rem;
  color: #555;
}


</style>
