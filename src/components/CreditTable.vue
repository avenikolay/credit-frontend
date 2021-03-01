<template>
  <div class="table-responsive">
    <table class="table">
      <thead>
      <tr>
        <th>#</th>
        <th>Номер кредита</th>
        <th>Сумма ближайшего <br /> погашения</th>
        <th>Общий остаток</th>
        <th>Дата погашения</th>
      </tr>
      </thead>
      <tbody>
      <tr :key="credit.id" v-for="(credit, index) in sortedBy">
        <td>{{index + 1}}</td>
        <td>{{ credit.number }}</td>
        <td>{{ credit.annuity_actual_graceful }}</td>
        <td>{{ credit.termination_amount }}</td>
        <td>{{ formatDate(credit.next_payment_date) }}</td>
      </tr>
      </tbody>
      <tfoot>
      <tr>
        <th>Всего:</th>
        <th>{{ sum }}</th>
        <th>{{ totalSum }}</th>
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
export default {
  props: {
    credits: {type: Array},
  },
  computed: {
    sum() {
      return this.credits.reduce((acc, curr) => Number((acc + parseFloat(curr.annuity_actual_graceful)).toFixed(2)), 0)
    },
    totalSum() {
      return this.credits.reduce((acc, curr) => Number((acc + parseFloat(curr.termination_amount)).toFixed(2)), 0)
    },
    sortedBy() {
      return [...this.credits].sort(sortByAmountSum)
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