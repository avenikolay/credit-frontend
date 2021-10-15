<template>
  <div class="table-responsive">
    <table class="table">
      <thead>
      <tr>
        <th>#</th>
        <th>Номер кредита <button @click="sortBy = null; direction === 'ask' ? direction = 'desk' : direction = 'ask'" type="button" class="btn" :class="{'text-danger' : sortBy === null}"><i
            class="bi bi-sort-down"></i></button></th>
        <th>Сумма ближайшего погашения
          <button @click="sortBy = 'actual'; direction === 'ask' ? direction = 'desk' : direction = 'ask'" type="button" class="btn" :class="{'text-danger' : sortBy === 'actual'}"><i
              class="bi bi-sort-down"></i></button>
        </th>
        <th>Общий остаток
          <button @click="sortBy = 'amount'; direction === 'ask' ? direction = 'desk' : direction = 'ask'" type="button" class="btn" :class="{'text-danger' : sortBy === 'amount'}"><i
              class="bi bi-sort-down"></i></button>
        </th>
        <th>Дата погашения
          <button @click="sortBy = 'date'; direction === 'ask' ? direction = 'desk' : direction = 'ask'" type="button" class="btn" :class="{'text-danger' : sortBy === 'date'}"><i
              class="bi bi-sort-down"></i></button>
        </th>
      </tr>
      </thead>
      <tbody>
      <tr :key="credit.id" v-for="(credit, index) in orderedBy">
        <td>{{ index + 1 }}</td>
        <td>{{ credit.number }}</td>
        <td>{{ credit.annuity_actual_graceful }}</td>
        <td>{{ credit.termination_amount }}</td>
        <td>{{ formatDate(credit.next_payment_date) }}</td>
      </tr>
      </tbody>
      <tfoot>
      <tr>
        <th colspan="2">Всего:</th>
        <th>{{ sum }}</th>
        <th colspan="2">{{ totalSum }}</th>
      </tr>
      </tfoot>

    </table>
  </div>

</template>

<script>
import dayjs from 'dayjs'

const sortByAmountSum = (a, b) => {
  return b.termination_amount - a.termination_amount
}
const sortByDate = (a, b) => {
  return (new Date(a.next_payment_date).getTime() - new Date(b.next_payment_date).getTime())
}
const sortByActual = (a, b) => {
  return b.annuity_actual_graceful - a.annuity_actual_graceful
}
export default {
  data() {
    return {
      //sortBy: null,
      sortBy: 'actual',
      direction: 'ask'
    }
  },
  props: {
    credits: {type: Array},
  },
  computed: {
    sum() {
      return this.credits.reduce((acc, curr) => Number((acc + parseFloat(curr.annuity_actual_graceful || 0)).toFixed(2)), 0)
    },
    totalSum() {
      return this.credits.reduce((acc, curr) => Number((acc + parseFloat(curr.termination_amount)).toFixed(2)), 0)
    },
    sortedBy() {
      if (this.sortBy === 'amount') {
        return [...this.credits].sort(sortByAmountSum)
      }
      if (this.sortBy === 'date') {
        return [...this.credits].sort(sortByDate)
      }
      if (this.sortBy === 'actual') {
        return [...this.credits].sort(sortByActual)
      }
      return this.credits
    },
    orderedBy() {
      if (this.direction === 'ask') {
        return this.sortedBy.slice().reverse();
      }
      return this.sortedBy
    }
  },
  methods: {
    formatDate(date) {
      return dayjs(date, 'YYYY-DD-MM').format('DD.MM.YYYY')
    }
  },
  name: "CreditTable"
}
</script>
<style>
button:focus {
  outline: none !important;
  box-shadow: none !important;
}
</style>