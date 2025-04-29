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

const showManualForm = ref(false)

const manualStock = ref({
  ticker: '',
  dividendYield: '',
  dividendAmount: '',
  declarationDate: '',
  computedExDivDate: '',
  recordDate: '',
  paymentDate: '',
  frequency: '',
})

const manualFormErrors = ref([])
const tickerError = ref('')

const showEditModal = ref(false)
const editingStockIndex = ref(null)

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
      // Prevent duplicates
      const alreadyExists = stockData.value.some(s => s.ticker === stock.ticker)
      if (!alreadyExists) {
        stockData.value.unshift(stock)
      } else {
        alert(`Ticker ${stock.ticker} is already in the table.`)
      }
     
    } else {
      alert('Stock not found or no data available.')
    }
  } catch (error) {
    console.error('Error fetching stock data:', error)
    alert('Failed to fetch stock data.')
  }
}
  const refreshStocks = async () => {

    try{

      for (let i=0; i<stockData.value.length; i++) {
        const symbol = stockData.value[i].ticker
        const url = `https://financialmodelingprep.com/stable/dividends?symbol=${symbol}&apikey=${apiKey.value}`
        const response = await axios.get(url)

        if (response.data && response.data.length > 0) {
          const latest = response.data[0]
          const recordDate = new Date(latest.recordDate)
          const exDivDate = new Date(recordDate)
          exDivDate.setDate(recordDate.getDate() - 1)
          const now = new Date();

          stockData.value[i] = {
            ticker: latest.symbol,
            dividendYield: `${latest.yield.toFixed(2)}%`,
            dividendAmount: `$${latest.dividend.toFixed(2)}`,
            computedExDivDate: exDivDate.toISOString().split('T')[0],
            recordDate: latest.recordDate,
            paymentDate: latest.paymentDate,
            frequency: latest.frequency,
            declarationDate: latest.declarationDate,
            lastRefreshed: now.toISOString().split('T')[0] + ' ' + now.toLocaleTimeString().split(' ')[0], // "YYYY-MM-DD HH:MM:SS"
            

          }
        }
      }

    } catch (error) {
      console.error('Error refreshing stock data:', error)
      alert('Failed to refresh one or more stocks.')
    }

  } //refreshStocks()


  const addManualStock = () => {

    manualFormErrors.value = [] //resets errors
    
    const requiredFields = ['ticker', 'dividendYield', 'dividendAmount']

    requiredFields.forEach(field => {
      if (!manualStock.value[field] || manualStock.value[field].trim() === '') {
        manualFormErrors.value.push(`${field} is required.`)
      }
    })

    const yieldValue = parseFloat(manualStock.value.dividendYield)
    const amountValue = parseFloat(manualStock.value.dividendAmount)

    if (isNaN(yieldValue) || yieldValue < 0) {
      manualFormErrors.value.push('Dividend Yield must be a valid positive number')
    }

    if (isNaN(amountValue) || amountValue < 0) {
      manualFormErrors.value.push('Dividend Amount must be a valid positive number')
    }

    if (manualFormErrors.value.length > 0 ){
      return
    }


    const stock = {
      ...manualStock.value,
      ticker: manualStock.value.ticker.toUpperCase().trim(),
      dividendYield: `${parseFloat(manualStock.value.dividendYield).toFixed(2)}%`,
      dividendAmount: `$${parseFloat(manualStock.value.dividendAmount).toFixed(2)}`,
      source: 'manual',
      
    }

    stockData.value.unshift(stock),
    tickerError.value = '' //clear message that ticker already exists
    showManualForm.value = false
    

    //Clear the form
    Object.keys(manualStock.value).forEach(key => {
      manualStock.value[key] = ''
    })


  } //addManualStock()

  const resetManualForm = () => {

    manualStock.value = {
      ticker: '',
      dividendYield: '',
      dividendAmount: '',
      declarationDate: '',
      computedExDivDate: '',
      recordDate: '',
      paymentDate: '',
      frequency: '',
    }
    manualFormErrors.value = ''
    tickerError.value = ''
  } // resetManualForm()

const cancelManualForm = () => {

  resetManualForm()
  showManualForm.value = false
  showEditModal.value = false
  editingStockIndex.value = null
}

const checkDuplicateTicker = () => {
  const ticker = manualStock.value.ticker.toUpperCase().trim()
  const exists = stockData.value.some(s => s.ticker === ticker)

  tickerError.value = exists ? `${ticker} already exists in your table` : ''
}

const onTickerInput = (event) => {
  manualStock.value.ticker = event.target.value.toUpperCase()
}

const deleteRow = (index) => {
  const confirmDelete = confirm(`Are you sure you want to delete the entry for ${stockData.value[index].ticker}?`)
  if (confirmDelete) {
    stockData.value.splice(index, 1)
  }
}

const editStock = (index) => {
  const stock = stockData.value[index]
  editingStockIndex.value = index
  manualStock.value = { ...stock } // Copy the stock into the manualStock form
  showEditModal.value = true
}

const updateStock = () => {
  if (editingStockIndex.value !== null) {
    const updatedStock = {
      ...manualStock.value,
      ticker: manualStock.value.ticker.toUpperCase().trim(),
      dividendYield: `${parseFloat(manualStock.value.dividendYield).toFixed(2)}%`,
      dividendAmount: `$${parseFloat(manualStock.value.dividendAmount).toFixed(2)}`,
      source: 'manual', // Keep it as manual
    }

    stockData.value[editingStockIndex.value] = updatedStock

    // Close modal and reset form
    showEditModal.value = false
    resetManualForm()
    editingStockIndex.value = null
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

    <!-- Refresh all dividend data in table-->
     <div>
     <button @click="refreshStocks" :disabled="stockData.length === 0">🔄 Refresh All</button>
     </div>

    <!-- Add a dividend manually. Opens modal to enter dividend data.-->
     <button @click="showManualForm = true">+ Manual Entry</button>

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
          <th>Last Refreshed</th>
          <th>Action</th>
          
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
          <td>{{ stock.lastRefreshed }}</td>
          <td>
            <button v-if="stock.source === 'manual'" @click="editStock(index)">✏️</button>
            <button @click="deleteRow(index)">🗑️</button>
          </td>

         
        </tr>
      </tbody>
    </table>
  </div>

  <div v-if="showManualForm" class="modal-backdrop">
  <div class="modal">
    <h2>Manual Dividend Entry</h2>
    
    <div v-if="manualFormErrors.length" class="error-message">
      <ul>
        <li v-for="(error, index) in manualFormErrors" :key="index">{{ error }} </li>
      </ul>
    </div>

    <label>Ticker: 
      <input
        v-model="manualStock.ticker"
        @input="onTickerInput()"
        @blur="checkDuplicateTicker"
        placeholder="Ticker"
      />
      <span v-if="tickerError" class="ticker-error">{{ tickerError }}</span>
    </label>
    <label>Dividend Yield (%): <input v-model="manualStock.dividendYield" /></label>
    <label>Dividend Amount ($): <input v-model="manualStock.dividendAmount" /></label>
    <label>Declaration Date: <input type="date" v-model="manualStock.declarationDate" /></label>
    <label>Ex-Dividend Date: <input type="date" v-model="manualStock.computedExDivDate" /></label>
    <label>Record Date: <input type="date" v-model="manualStock.recordDate" /></label>
    <label>Payment Date: <input type="date" v-model="manualStock.paymentDate" /></label>
    <label>Frequency: <input v-model="manualStock.frequency" /></label>

    <div class="modal-buttons">
      <button @click="addManualStock">Add</button>
      <button @click="cancelManualForm">Cancel</button>
    </div>
  </div>
</div>

<div v-if="showEditModal" class="modal-backdrop">
  <div class="modal">
    <h2>Edit Manual Stock</h2>

    <!-- You can reuse your manual form fields here -->
    <div>
      <label>Ticker:</label>
      <input v-model="manualStock.ticker" type="text" />

      <label>Dividend Yield (%):</label>
      <input v-model="manualStock.dividendYield" type="text" />

      <label>Dividend Amount ($):</label>
      <input v-model="manualStock.dividendAmount" type="text" />

      <label>Declaration Date:</label>
      <input v-model="manualStock.declarationDate" type="date" />

      <label>Ex-Dividend Date:</label>
      <input v-model="manualStock.computedExDivDate" type="date" />

      <label>Record Date:</label>
      <input v-model="manualStock.recordDate" type="date" />

      <label>Payment Date:</label>
      <input v-model="manualStock.paymentDate" type="date" />

      <label>Frequency:</label>
      <input v-model="manualStock.frequency" type="text" />
    </div>

    <div class="modal-buttons">
      <button @click="updateStock">Update</button>
      <button @click="cancelManualForm">Cancel</button>
    </div>
  </div>
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

.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.6);
  display: flex;
  align-items: center;
  justify-content: center;

}

.modal {
  background: white;
  padding: 2em;
  border-radius: 8px;
  max-width: 600px;
  width: 90%;
  max-height: 90vh; /* or 80vh */
  overflow-y: auto;
}

.modal label {
  display: block;
  margin-bottom: 1em;
}

.modal input {
  width: 90%;
  padding: 0.5em;
}

.modal-buttons {
  margin-top: 20px;
  display: flex;
  gap: 10px;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
}

.error-messages {
  background-color: #ffe0e0;
  color: #a00000;
  padding: 1em;
  margin-bottom: 1em;
  border-radius: 4px;
}

.ticker-error {
  color: #a00000;
  font-size: 0.9em;
  margin-top: 0.25em;
  display: block;
}


</style>
