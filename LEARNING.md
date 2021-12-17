# 21. Redux

## Refactoring with Redux Toolkit

*   `npm install redux`
*   `npm install @reduxjs/toolkit`
*   `npx create-react-app my-app --template redux`
*   `configureStore(options)`!

crazy redux

## Project Structure

<img src="../../../../../../../../../Library/Application%20Support/typora-user-images/image-20211216090453030.png" alt="image-20211216090453030" style="zoom:50%;" />

### Order:

index.js > app (`app.js`, `store.js`) > features ((budgets: `Budgets.js`, `budgetsSlice.js`), (transactions: `Transactions.js`, `transactionsSlice.js`)) > components (`Budget.js`, `Transactions.js`, `TransactionForm.js`, `TransactionList.js`)

```react
//index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './app/App';
import { Provider } from 'react-redux';
import store from './app/store';

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>,
  document.getElementById('app')
);
```

## app

```react
//App.js
import Transactions from '../features/transactions/Transactions';
import Budgets from '../features/budgets/Budgets';
import React from 'react';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <h1>Expense Tracker</h1>
        <Budgets />
        <Transactions />
      </header>
    </div>
  );
}

export default App;
```

```react
//store.js
import { configureStore } from '@reduxjs/toolkit';
import transactionsReducer from '../features/transactions/transactionsSlice';
import budgetsReducer from '../features/budgets/budgetsSlice';

export default configureStore({
  reducer: {
    transactions: transactionsReducer,
    budgets: budgetsReducer,
  },
});
```

## components (some functional & presentational)

```react
//Budget.js
import React, { useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { editBudget } from '../features/budgets/budgetsSlice';
import { selectTransactions } from '../features/transactions/transactionsSlice';

export default function Budget({ budget }) {
  const dispatch = useDispatch();
  const [amount, setAmount] = useState(budget.amount);
  const transactions = useSelector(selectTransactions);

  const handleEdit = (e) => {
    e.preventDefault();
    dispatch(editBudget({ category: budget.category, amount: amount }));
  };

  const calculateTotalExpenses = () => {
    return transactions[budget.category]
      .map((transaction) => transaction.amount)
      .reduce((amount1, amount2) => amount1 + amount2, 0);
  };

  const getFundsRemainingClassName = (amount) => {
    if (parseFloat(amount) === 0) {
      return null;
    }

    return parseFloat(amount) > 0 ? 'positive' : 'negative';
  };

  const remainingFunds = Number.parseFloat(budget.amount - calculateTotalExpenses()).toFixed(2);
  const fundsRemainingClassName = getFundsRemainingClassName(remainingFunds);

  return (
    <li className="budget-container">
      <div className="category-label">Category</div>{' '}
      <div className="category-wrapper">
        <h3 className="category-value">{budget.category}</h3>
        <form onSubmit={handleEdit} className="budget-form">
          <input
            className="amount-input"
            value={amount}
            onChange={(e) => setAmount(e.currentTarget.value)}
            type="number"
            inputmode="numeric"
            step="0.01"
          />
          <button className="update-button">Update</button>
        </form>
      </div>
      <h4 className={`remaining-funds ${fundsRemainingClassName}`}>
        Funds Remaining: {remainingFunds}
      </h4>
    </li>
  );
}

```

```react
//TransactionForm.js
import React, { useState } from 'react';
import { useDispatch } from 'react-redux';
import {
  addTransaction,
  CATEGORIES,
} from '../features/transactions/transactionsSlice';
import { v4 as uuidv4 } from 'uuid';

export default function TransactionForm({ categories }) {
  const dispatch = useDispatch();
  const [category, setCategory] = useState(CATEGORIES[0]);
  const [description, setDescription] = useState('');
  const [amount, setAmount] = useState(0);

  const handleSubmit = (e) => {
    e.preventDefault();
    dispatch(
      addTransaction({
        category: category,
        description: description,
        amount: parseFloat(amount),
        id: uuidv4(),
      })
    );
    setDescription('');
    setAmount(0);
  };

  return (
    <section className="new-transaction-section">
      <h2>New Transaction</h2>
      <form onSubmit={handleSubmit}>
        <div className="form-wrapper">
          <div>
            <label htmlFor="category">Category</label>
            <select
              id="category"
              value={category}
              onChange={(e) => setCategory(e.currentTarget.value)}
            >
              {CATEGORIES.map((c) => (
                <option key={c} value={c}>
                  {c}
                </option>
              ))}
            </select>
          </div>

          <div>
            <label htmlFor="description">Description</label>
            <input
              id="description"
              value={description}
              onChange={(e) => setDescription(e.currentTarget.value)}
              type="text"
            />
          </div>

          <div>
            <label htmlFor="amount">Amount</label>
            <input
              id="amount"
              value={amount}
              onChange={(e) => setAmount(e.currentTarget.value)}
              type="number"
              inputmode="numeric"
              step="0.01"
            />
          </div>
        </div>

        <button>Add Transaction</button>
      </form>
    </section>
  );
}
```

```react
//TransactionList.js
import React from 'react';
import Transaction from './Transaction';

export default function TransactionList({ transactions }) {
  return (
    <section className="new-transactions-section">
      <h2>Transactions</h2>
      <ul className="new-transaction-list">
        {transactions.map((t) => (
          <Transaction transaction={t} key={t.id} />
        ))}
      </ul>
    </section>
  );
}
```

```react
//Transaction.js
import React from 'react';
import { useDispatch } from 'react-redux';
import { deleteTransaction } from '../features/transactions/transactionsSlice';

export default function Transaction({ transaction }) {
  const dispatch = useDispatch();

  const handleDelete = (e) => {
    dispatch(deleteTransaction(transaction));
  };

  return (
    <li className="new-transaction">
      <span>
        {transaction.amount} – {transaction.category}{' '}
        <span className="description">( {transaction.description} )</span>
      </span>
      <button onClick={handleDelete} aria-label="Remove">
        X
      </button>
    </li>
  );
}
```

## features (slice + actual feature)

### budgets

```react
//Budgets.js
import React from 'react';
import { useSelector } from 'react-redux';
import { selectBudgets } from './budgetsSlice';
import Budget from '../../components/Budget';

const Budgets = () => {
  const budgets = useSelector(selectBudgets);
  return (
    <ul className='comments-container'>
      { budgets.map(budget => <Budget budget={budget} key={budget.category}/>) }
    </ul>
  );
};

export default Budgets;
```

```react
//budgetsSlice.js
import { createSlice } from '@reduxjs/toolkit';
export const CATEGORIES = ['housing', 'food', 'transportation', 'utilities', 'clothing', 'healthcare', 'personal', 'education', 'entertainment'];
const initialState = CATEGORIES.map(category => ({ category: category, amount: 0 }))

const budgetsSlice = createSlice({
  name: 'budgets',
  initialState: initialState,
  reducers: {
    editBudget: (state, action) => {
      const cat = action.payload.category;
      return state.map(budget => {
        if (budget.category === cat)
          return action.payload;
        else;
          return budget;
      });
    }
  }
})

export const selectBudgets = (state) => state.budgets;
export const { editBudget } = budgetsSlice.actions;
export default budgetsSlice.reducer;
```

### transactions

```react
//Transactions.js
import React from 'react';
import { useSelector } from 'react-redux';
import { selectFlattenedTransactions } from './transactionsSlice';
import TransactionForm from '../../components/TransactionForm';
import TransactionList from '../../components/TransactionList';

const Transactions = () => {
  const transactions = useSelector(selectFlattenedTransactions);
  return (
    <div className="comments-container">
      <TransactionList transactions={transactions} />
      <TransactionForm />
    </div>
  );
};

export default Transactions;
```

```react
//transactionsSlice.js
import { createSlice } from '@reduxjs/toolkit';
export const CATEGORIES = ['housing', 'food', 'transportation', 'utilities', 'clothing', 'healthcare', 'personal', 'education', 'entertainment'];
const initialState = Object.fromEntries(CATEGORIES.map(category => [category, []]))

const transactionsSlice = createSlice({
  name: 'transactions',
  initialState: initialState,
  reducers: {
    addTransaction: (state, action) => {
      const cat = action.payload.category;
      state[cat].push(action.payload);
    },
    deleteTransaction: (state, action) => {
      const deletedIndex = state[action.payload.category].findIndex(transaction => transaction.id === action.payload.id);
      const newTransactionsForCategory = state[action.payload.category].filter((item, index) => index !== deletedIndex)
      return { ...state, [action.payload.category]: newTransactionsForCategory };
    }
  }
})

export const selectTransactions = (state) => state.transactions;
export const selectFlattenedTransactions = (state) => Object.values(state.transactions).reduce((a,b) => [...a, ...b], []);
export const { addTransaction, deleteTransaction} = transactionsSlice.actions;
export default transactionsSlice.reducer;
```



## *styles.css*

```css
/* styles.css */
@import url('https://fonts.googleapis.com/css2?family=Oxygen:wght@400;700&display=swap');
* {
  margin: 0px;
  padding: 0px;
}
:root {
  --main-text-color: #141c3a;
  --blue: #3069f0;
  --red: #fd4d3f;
  --gray: rgba(20, 28, 58, 0.5);
  --shadow: 0 0 21px 0 rgba(20, 28, 58, 0.7);
  --white: #ffffff;
}

body {
  font-family: 'Oxygen', sans-serif;
  color: var(--main-text-color);
  padding: 24px 20px 0px 20px;
  margin: 0;
}

h1 {
  font-size: clamp(35px, 4.5vw, 60px);
  font-weight: bold;
  margin-top: 0;
  margin-bottom: 32px;
}

h2 {
  font-size: clamp(25px, 4vw, 48px);
  font-weight: bold;
  margin-top: 0px;
  margin-bottom: 32px;
}

h3 {
  font-size: clamp(20px, 4vw, 30px);
  font-weight: bold;
  margin: 2px 0px 0px 0px;
}

h4 {
  font-size: clamp(15px, 4vw, 24px);
  font-weight: bold;
  margin: 0;
}

.category-label {
  color: var(--gray);
  font-size: 15px;
  font-weight: bold;
}

.budget-form .amount-input{
  width: 70px;
  margin-right: 10px;
}

.budget-container {
  list-style: none;
  padding: 16px 0px 24px 0px;
  border-bottom: solid 1px var(--main-text-color);
}

.comments-container {
  padding: 0;
}

.category-value {
  text-transform: capitalize;
}

input {
  padding: 7px 12px;
  border-radius: 6px;
  border: solid 1px #141c3a;
  font-size: 14px;
}

.new-transaction-section input {
  width: clamp(20px, 20vw, 400px);
}

.update-button {
  padding: 6px 12px;
  border-radius: 6px;
  border: solid 1px #141c3a;
  font-size: 18px;
  font-weight: bold;
  background: var(--white);
  cursor: pointer;
}

.update-button:hover,
.update-button:focus {
  background-color: #141c3a;
  color: #ffffff;
}

.category-wrapper {
  display: flex;
  align-items: center;
  margin-bottom: 24px;
  flex-wrap: wrap;
  justify-content: space-between;
  max-width: 1000px;
  margin-left: 15px;
  margin-right: 15px;
}

.remaining-funds.positive {
  color: var(--blue);
}

.remaining-funds.negative {
  color: var(--red);
}

.new-transaction-section {
  background: var(--main-text-color);
  color: var(--white);
  margin-left: -20px;
  margin-right: -20px;
  padding: 24px 20px;
  padding-bottom: 38px;
  box-shadow: var(--shadow);
  position: fixed;
  bottom: 0;
  box-sizing: border-box;
  max-height: 33%;
  width: 100%;
  overflow-y: scroll;
}

.new-transaction-section label {
  display: block;
  font-size: clamp(12px, 3.5vw, 18px);
  font-weight: bold;
  margin-bottom: 8px;
}

.form-wrapper {
  display: flex;
  margin-bottom: 28px;
  justify-content: space-around;
  min-width: 0;
}

.new-transaction-section button {
  padding: 9px 12px;
  border-radius: 6px;
  border: solid 1px #ffffff;
  background-color: #141c3a;
  font-size: 18px;
  font-weight: bold;
  color: #ffffff;
  cursor: pointer;
}

.new-transaction-section button:hover,
.new-transaction-section button:focus {
  background-color: #ffffff;
  color: #141c3a;
}

select {
  padding: 7px 16px 8px 12px;
  border-radius: 6px;
  border: solid 1px #141c3a;
  background-color: #ffffff;
  text-transform: capitalize;
}

.new-transaction-list {
  list-style: none;
  padding: 0;
}

.new-transaction {
  padding: 12px 10.5px 11px 16px;
  border-radius: 6px;
  border: solid 1px #141c3a;
  background-color: #ffffff;
  font-size: 16px;
  font-weight: bold;
  display: flex;
  margin-bottom: 16px;
}

.description {
  font-style: italic;
  font-size: 16px;
  font-weight: normal;
}

.new-transaction button {
  background: white;
  border: solid 1px #141c3a;
  border-radius: 50%;
  height: 20px;
  width: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-weight: bold;
}

.new-transaction button:hover,
.new-transaction button:focus {
  color: white;
  background-color: #141c3a;
}

.new-transactions-section {
  padding-bottom: 350px;
  height: 30%;
}
```

