<<<<<<< HEAD
# 📈 Stock Dividend Lookup App

A simple Vue.js app to look up dividend information for publicly traded stocks using the [Financial Modeling Prep API](https://financialmodelingprep.com/).

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/celesteg1/stocks-and-dividends.git
cd stocks-and-dividends
```

### 2. Install Dependencies

```bash
npm install
```


### 3. 🔐 Set up Environment Variables

To use the API, you need a valid API key from [Financial Modeling Prep](https://financialmodelingprep.com/).
This app uses Vite, so you'll store your API key in a .env file.

3.1. Copy the example environment file:

```bash
cp .env.example .env
```

3.2. Edit .env and paste in your API key:

VITE_API_KEY=your_actual_key_here


💡 Tip: Your .env file is listed in .gitignore and will not be committed. This keeps your API key private.

### 4. Run the App

```bash
npm run dev
```

Open your browser and go to http://localhost:5173.


## 📦 Features

- 🔍 Lookup stock dividend data
- 📋 Copy and manage your API key securely in the browser
- 🙈 Toggle API key visibility
- 💡 Persistent tips for configuration and usage


## 🧪 Example Stocks (Free Tier)

The API supports a limited set of tickers on the free tier. Some popular ones:

```r
AAPL, TSLA, MSFT, AMZN, NVDA, GOOGL, META, JPM, V, AMD, PFE, DIS, T, KO, INTC, COST
```

## 📁 Project Structure

- src/components – Vue components for Config and Lookup
- src/App.vue – Main application
- .env – Your personal API key (not committed)
- .env.example – Template for setup


## 🛡️ Security
- API key is not committed to the repo
- Input field lets you toggle visibility
- Reminder message encourages secure use via .env


## 💬 Feedback
Feel free to fork, tweak, or suggest improvements!
=======
# stocks-and-dividends
A little app to look up stocks and retrieve information such as the dividend schedule.
>>>>>>> f9a63fbe762470dd7cd30453d792957ec016aa08
