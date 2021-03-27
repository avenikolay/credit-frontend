<template>
  <div v-show="loading" class="loader">
    <div class="lds-ripple"><div></div><div></div></div>
  </div>
  <div class="container pt-4">
    <ul class="nav nav-justified  nav-tabs">
      <li class="nav-item">
        <a @click="tab = 'next'" :class="{'active': tab === 'next'}" class="nav-link " aria-current="page" href="#">Актуальное</a>
      </li>
      <li class="nav-item">
        <a @click="tab = 'all'" :class="{'active': tab === 'all'}"  class="nav-link" href="#">Все кредиты</a>
      </li>
      <button @click="logout()">Выйти</button>
    </ul>
      <template v-if=" tab === 'next'">
        <h1>Кредиты</h1>
        <hr />
        <div>Общая сумма: {{totalSum}} сомони</div>
        <hr />
        <h2>Нужно погасить в ближайшее время:</h2>
        <CreditTable :credits="currentPeriod" />
        <hr>
        <h2>Сумма погашений на следующий зарплатный период:</h2>
        <CreditTable :credits="nextPeriodCredits" />
      </template>
      <template v-if=" tab === 'all'">
        <h2>Все кредиты</h2>
        <CreditTable :credits="actualCredits" />
      </template>
  </div>
</template>

<script>
import CreditTable from "@/components/CreditTable";
import dayjs from 'dayjs'
const today = Number(dayjs().format('D'));
//const endOfMonth = dayjs().endOf('month').subtract(1, 'day').format('YYYY-MM-DD');
const endOfMonth = dayjs().endOf('month').format('YYYY-MM-DD');
const periodDate = (date) => new Date(date).getTime();
const sortByDate = (a, b) => {
  return (new Date(a.next_payment_date).getTime() - new Date(b.next_payment_date).getTime())
}
// eslint-disable-next-line no-unused-vars
const sortBySum = (a, b) => {
  return a.annuity_actual_graceful - b.annuity_actual_graceful
}

export default {
  name: "Credits",
  components: {CreditTable},
  props: {
    credits: {
      type: Array,
      default: () => []
    },
    loading: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      tab: 'next'
    }
  },
  methods: {
    logout() {
      localStorage.removeItem('token');
      window.location.reload()
    }
  },
  computed: {
    actualCredits() {
      return this.credits.length ? this.credits.filter(p => p.next_payment_date !== null) : [];
    },
    totalSum() {
      return this.actualCredits.reduce((acc, curr) => Number((acc + parseFloat(curr.termination_amount)).toFixed(2)), 0)
    },
    currentPeriod() {
      let end = today < 15 ? dayjs().add(14 - today, 'day').format('YYYY-MM-DD') : endOfMonth;
      return this.actualCredits.filter(p => new Date(p.next_payment_date).getTime() <  periodDate(end)).sort(sortByDate)
    },
    nextPeriodCredits() {
      let start = today < 15 ? dayjs().add(15 - today, 'day').format('YYYY-MM-DD') : endOfMonth;
      let end = today < 15 ? endOfMonth : dayjs().endOf('month').add(14, 'day').format('YYYY-MM-DD');
      return this.actualCredits.filter(p => new Date(p.next_payment_date).getTime() >= periodDate(start) && new Date(p.next_payment_date).getTime() < periodDate(end)).sort(sortByDate)
    }
  }
}
</script>

<style scoped>
.loader {
  width: 100%;
  height: 100vh;
  position: absolute;
  background-color: rgba(0, 147, 233, 0.5);
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
}
.lds-ripple {
  display: inline-block;
  position: relative;
  width: 180px;
  height: 180px;
}
.lds-ripple div {
  position: absolute;
  border: 4px solid #fff;
  opacity: 1;
  border-radius: 50%;
  animation: lds-ripple 1s cubic-bezier(0, 0.2, 0.8, 1) infinite;
}
.lds-ripple div:nth-child(2) {
  animation-delay: -0.5s;
}
@keyframes lds-ripple {
  0% {
    top: 36px;
    left: 36px;
    width: 0;
    height: 0;
    opacity: 1;
  }
  100% {
    top: 0;
    left: 0;
    width: 72px;
    height: 72px;
    opacity: 0;
  }
}
</style>