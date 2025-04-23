<template>
  <main>
    <button @click="addTransactionRow">Add Transaction</button>

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
            <td><input type="checkbox" v-model="row.recurring" /></td>
            <td><input type="text" v-model="row.tags" /></td>
            <td><input type="number" v-model="row.amount" /></td>
            <td><button @click="deleteTransactionRow(index)">❌</button></td>
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
            <td><button @click="deleteSummaryRow(index)">❌</button></td>
          </tr>
        </tbody>
      </table>
    </div>
    <button @click="addSummaryRow">Add Summary</button>
  </main>
</template>

<script setup>
import useLocalStorage from './composables/useLocalStorage';
import { computed, watch } from 'vue';
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

<style>
.box {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
table,
th,
td {
  text-align: center;
  border: solid white 1px;
  border-collapse: collapse;
}
th {
  font-size: 20px;
  font-weight: bold;
  width: 180px;
  height: 100px;
}
td {
  height: 50px;
}
</style>
