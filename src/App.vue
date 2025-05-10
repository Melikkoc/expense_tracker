<template>
  <main>
    <button @click="addTransactionRow">Add Transaction</button>
    <button @click="exportExpenseTable">export as CSV</button>
    <div class="box">
      <table>
        <caption>
          Expense Tracker
        </caption>
        <thead>
          <tr>
            <th>Date</th>
            <th>Category</th>
            <th>Description</th>
            <th>Type (Income/Expense)</th>
            <th>Note</th>
            <th>Payment Method</th>
            <th>Recurring</th>
            <th>Tags</th>
            <th>Amount(€)</th>
            <th>Delete</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, index) in transactionRows" :key="index">
            <td><input type="date" v-model="row.date" /></td>
            <td><input type="text" v-model="row.category" /></td>
            <td><input type="text" v-model="row.description" /></td>
            <td><input type="text" v-model="row.type" /></td>
            <td><input type="text" v-model="row.note" /></td>
            <td><input type="text" v-model="row.paymentMethod" /></td>
            <td class="centered">
              <input type="checkbox" v-model="row.recurring" />
            </td>
            <td><input type="text" v-model="row.tags" /></td>
            <td><input type="number" v-model="row.amount" /></td>
            <td class="centered">
              <button @click="deleteTransactionRow(index)">❌</button>
            </td>
          </tr>
        </tbody>
        <tbody>
          <tr>
            <th>Month</th>
            <th>Rent</th>
            <th>Subscriptions</th>
            <th>Transportation</th>
            <th>Groceries</th>
            <th>Total Expenses</th>
            <th>Savings</th>
            <th>Total Income</th>
            <th>Remaining</th>
            <th>Delete</th>
          </tr>
          <tr v-for="(row, index) in summaryRows" :key="index">
            <td><input type="month" v-model="row.month" /></td>
            <td>
              <input type="number" v-model="row.rent" />
            </td>
            <td>
              <input type="number" v-model="row.subscriptions" />
            </td>
            <td>
              <input type="number" v-model="row.transportation" />
            </td>
            <td>
              <input type="number" v-model="row.groceries" />
            </td>
            <td>
              <input
                type="number"
                disabled="disabled"
                v-model="row.total_expenses"
              />
            </td>
            <td><input type="number" v-model="row.savings" /></td>
            <td>
              <input
                type="number"
                disabled="disabled"
                v-model="row.total_income"
              />
            </td>
            <td>
              <input
                type="number"
                disabled="disabled"
                v-model="row.remaining"
              />
            </td>
            <td class="centered">
              <button @click="deleteSummaryRow(index)">❌</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <button @click="addSummaryRow">Add Summary</button>
    <button @click="getIncomeToday">get income today</button>
    <div id="box-today">
      <div id="today">
        <h2>Summary Today</h2>
        <input
          type="number"
          disabled="disabled"
          v-model="summaryToday.remainingToday"
        />
      </div>
    </div>
  </main>
</template>

<script setup>
import useLocalStorage from './composables/useLocalStorage';
import { computed, watch } from 'vue';
import * as Papa from 'papaparse';
import { errorMessages } from 'vue/compiler-sfc';
const expenseTable = useLocalStorage(
  {
    summaryRows: [
      {
        month: '',
        rent: 0,
        subscriptions: 0,
        transportation: 0,
        groceries: 0,
        total_expenses: '',
        savings: 0,
        total_income: 0,
        remaining: 0,
      },
    ],
    transactionRows: [
      {
        date: '',
        category: '',
        description: '',
        type: '',
        note: '',
        paymentMethod: '',
        recurring: false,
        tags: '',
        amount: 0,
      },
    ],
  },
  'expenseTable'
);
const summaryToday = useLocalStorage(
  {
    remainingToday: 0,
  },
  'summaryToday'
);
const transactionRows = computed(() => expenseTable.value.transactionRows);
const summaryRows = computed(() => expenseTable.value.summaryRows);

function addTransactionRow() {
  transactionRows.value.push({
    date: '',
    category: '',
    description: '',
    type: '',
    note: '',
    paymentMethod: '',
    recurring: false,
    tags: '',
    amount: 0,
  });
}

function addSummaryRow() {
  summaryRows.value.push({
    month: '',
    rent: 0,
    subscriptions: 0,
    transportation: 0,
    groceries: 0,
    total_expenses: 0,
    savings: 0,
    total_income: 0,
    remaining: 0,
  });
}

function getTotalExpenses() {
  summaryRows.value.forEach((row) => {
    const rent = row.rent;
    const subscriptions = row.subscriptions;
    const transportation = row.transportation;
    const groceries = row.groceries;

    row.total_expenses = rent + subscriptions + transportation + groceries;
  });
}

function getTotalIncome() {
  summaryRows.value.forEach((row) => {
    const summaryMonth = row.month;

    const matchingTransactions = transactionRows.value.filter((transaction) => {
      const transactionMonth = transaction.date?.slice(0, 7);
      return transactionMonth === summaryMonth;
    });

    const total = matchingTransactions.reduce(
      (acc, t) => acc + (t.amount || 0),
      0
    );
    row.total_income = total;
  });
}

function getRemaining() {
  summaryRows.value.forEach((row) => {
    const total_expenses = row.total_expenses;
    const savings = row.savings;
    const total_income = row.total_income;

    row.remaining = total_income - total_expenses - savings;
  });
}

function deleteTransactionRow(index) {
  transactionRows.value.splice(index, 1);
}

function deleteSummaryRow(index) {
  summaryRows.value.splice(index, 1);
}
function exportExpenseTable() {
  const transactionCsv = Papa.unparse(transactionRows.value);
  const summaryCsv = Papa.unparse(summaryRows.value);

  const combinedCsv = [transactionCsv, summaryCsv].join('\n');

  downloadCSV(combinedCsv, 'expenseTable.csv');
}

function downloadCSV(csvData, filename) {
  const blob = new Blob([csvData], { type: 'text/csv;charset=utf-8;' });
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');

  link.setAttribute('href', url);
  link.setAttribute('download', filename);
  document.body.appendChild(link);
  link.click();
  document.body.removeChild();
}

function getIncomeToday() {
  const formatDate = (date) => {
    const year = String(date.getFullYear());
    const month = String(date.getMonth() + 1).padStart(2, '0'); // Months are 0-indexed
    const day = String(date.getDate()).padStart(2, '0');

    return `${year}-${month}-${day}`;
  };
  const currentDate = formatDate(new Date());
  const todaysTransactions = transactionRows.value.filter(
    (transaction) => transaction.date === currentDate
  );
  const total = todaysTransactions.reduce((acc, t) => acc + (t.amount || 0), 0);

  console.log(total);

  summaryToday.value.remainingToday = total;
}

watch(
  transactionRows,
  () => {
    getTotalIncome();
    getRemaining();
  },
  { deep: true }
);

watch(
  summaryRows,
  () => {
    getTotalExpenses();
    getRemaining();
  },
  { deep: true }
);
</script>

<style scoped>
/* Layout Container */
main {
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f8f8f8;
  color: #333;
  min-height: 100vh;
}

/* Buttons */
button {
  margin: 10px 10px 20px 0;
  padding: 10px 18px;
  background-color: #5741e5;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: transform 0.3s ease, background-color 0.3s ease;
}
button:hover {
  background-color: #3f2ac4;
  transform: translateY(-3px);
}
button:active {
  background-color: #3b279d;
  transform: translateY(1px);
}

/* Box around table */
.box {
  overflow-x: auto;
  background-color: #ffffff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s ease;
}
.box:hover {
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

/* Table Layout */
table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0 12px;
  margin-bottom: 30px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
  animation: fadeIn 0.8s ease-out;
}
caption {
  caption-side: top;
  font-size: 1.6em;
  margin-bottom: 10px;
  font-weight: bold;
  color: #5741e5;
}

/* Table Header */
th {
  background-color: #5741e5;
  color: white;
  padding: 12px;
  font-size: 14px;
  border: none;
  text-align: center;
  animation: fadeIn 0.8s ease-out;
}

/* Table Body Cells */
td {
  padding: 12px;
  background-color: #f9f9f9;
  color: #000000;
  border: none;
  border-bottom: 1px solid #e0e0e0;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.05);
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
}
td input {
  transition: border-color 0.3s ease, background-color 0.3s ease;
}
td:hover {
  background-color: #f0f0f0;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

tr:hover td {
  background-color: #ecebff;
}

/* Inputs inside table */
input[type='text'],
input[type='number'],
input[type='date'],
input[type='month'] {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 14px;
  background-color: #ffffff;
  color: #333;
  transition: all 0.3s ease;
}

input:hover {
  transform: scale(1.1);
}

input:disabled {
  background-color: #e9ecef; /* Light gray background */
  color: #6c757d; /* Dimmed text color */
  border: 1px solid #ccc; /* Optional: make border more subtle */
  cursor: not-allowed; /* Shows a "not-allowed" cursor */
  opacity: 0.7; /* Optional: make it slightly transparent */
  transform: none;
}

input[type='checkbox'] {
  -webkit-appearance: none; /* Remove default checkbox styling */
  -moz-appearance: none;
  appearance: none;
  width: 24px; /* Size of the checkbox */
  height: 24px;
  border: 2px solid #5741e5; /* Border color */
  border-radius: 4px; /* Rounded corners */
  position: relative;
  background-color: #fff; /* Background color */
  transition: all 0.3s ease; /* Smooth transitions */
  cursor: pointer;
}

/* Checkbox when checked */
input[type='checkbox']:checked {
  background-color: #5741e5; /* Background color when checked */
  border-color: hsl(248, 76%, 58%); /* Border color when checked */
}

input[type='checkbox']:hover {
  background-color: hsl(0, 0%, 85%);
  border-color: hsla(248, 76%, 58%, 0.9); /* Border color when checked */
}

/* Creating a custom checkmark */
input[type='checkbox']:checked::before {
  content: '✔'; /* The checkmark character */
  position: absolute;
  top: 0;
  left: 3px;
  color: white;
  font-size: 18px; /* Size of the checkmark */
}

/* Hover effect */
input[type='checkbox']:hover {
  border-color: #3f2ac4; /* Border color on hover */
  transform: scale(1.1);
}
input[type='checkbox']:active {
  transform: scale(0.95);
}

/* Focus effect */
input[type='checkbox']:focus {
  outline: none;
  border-color: #5741e5; /* Border color on focus */
}

input[type='checkbox']:checked {
  background-color: #5741e5;
  color: #5741e5; /* Background color when checked */
  border-color: #5741e5; /* Border color when checked */
}

input[type='text']:focus,
input[type='number']:focus,
input[type='date']:focus,
input[type='month']:focus {
  border-color: #5741e5;
  background-color: #fafbff;
}

tr th:first-child {
  border-radius: 5px 0 0 5px;
}

tr th:last-child {
  border-radius: 0 5px 5px 0;
}

/* Delete Button inside table */
td button {
  background-color: rgba(255, 159, 159, 0.75);
  color: white;
  border-radius: 4px;
  transition: transform 0.2s ease, background-color 0.2s ease;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
td button:hover {
  background-color: rgb(255, 159, 159);
  transform: scale(1.1);
}
td button:active {
  transform: scale(0.95);
}
td.centered {
  text-align: center;
  vertical-align: middle;
}
/* Summary Today Section */
#box-today {
  margin-top: 40px;
  display: flex;
  justify-content: center;
}
#today {
  background-color: #f1efff;
  padding: 20px;
  border-radius: 6px;
  display: inline-block;
  border: 1px solid #5741e5;
  text-align: center;
  animation: fadeIn 1s ease-out;
}
#today h2 {
  margin-bottom: 10px;
  font-size: 1.5em;
  color: #5741e5;
}
#today input {
  font-size: 16px;
  font-weight: bold;
  border: none;
  background-color: #ffffff;
  color: #000000;
  padding: 8px;
  border-radius: 4px;
  width: 100%;
}

/* Fade-in animation */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Responsive Design */
@media (max-width: 768px) {
  table {
    font-size: 12px;
  }
  button {
    padding: 6px 12px;
  }
  #today {
    padding: 15px;
  }
}
</style>
