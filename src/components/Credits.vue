<template>
  <div v-show="loading" class="loader">
    <div class="lds-ripple">
      <div></div>
    </div>
  </div>
  <div v-if="!loading" class="container pt-4">
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
        <div>Общая сумма: {{totalSum}}</div>
        <hr />
        <h2>Нужно погасить в ближайшее время:</h2>
        <CreditTable :usdSellValue="usdSellValue" :credits="currentPeriodCredits" />
        <hr>
        <h2>Сумма погашений на следующий зарплатный период:</h2>
        <CreditTable :usdSellValue="usdSellValue" :credits="nextPeriodCredits" />
      </template>
      <template v-if=" tab === 'all'">
        <h2>Все кредиты</h2>
        <CreditTable  :usdSellValue="usdSellValue" :credits="actualCredits" />
      </template>
  </div>
</template>

<script>
import CreditTable from "@/components/CreditTable";
import dayjs from 'dayjs'
const today = Number(dayjs().format('D'));
//const endOfMonth = dayjs().endOf('month').subtract(1, 'day').format('YYYY-MM-DD');
const endOfMonth = dayjs().endOf('month').format('YYYY-MM-DD');
const latestDayOfMonth = dayjs().endOf('month').get('date');
const periodDate = (date) => new Date(date).getTime();
const sortByDate = (a, b) => {
  return (new Date(a['next_payment_date']).getTime() - new Date(b['next_payment_date']).getTime())
}
// eslint-disable-next-line no-unused-vars
const sortBySum = (a, b) => {
  return a['annuity_actual_graceful'] - b['annuity_actual_graceful']
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
    },
    usdSellValue: Number
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
      // return this.credits.length ? this.credits.filter(p => p.next_payment_date !== null) : [];
      return this.credits.length ? this.credits.filter(p => p['termination_amount'] !== "0.00" && p['termination_amount'] !== 0 && p['next_payment_date'] !== null) : [];
    },
    endDateForCurrentPeriod() {
      if (today < 15) {
         return dayjs().add(15 - today, 'day').format('YYYY-MM-DD')
      } else if (today >= 15  && today < latestDayOfMonth) {
        return endOfMonth;
      }
      return  dayjs().endOf('month').add('15', 'day').format('YYYY-MM-DD')
    },
    totalSum() {
      const helper = (acc, curr) => Number((acc + parseFloat(curr['termination_amount'])).toFixed(2));
      const usd = this.actualCredits.filter(credit => credit.currency === 'USD');
      const tjs = this.actualCredits.filter(credit => credit.currency === 'TJS');
      return `${tjs.reduce(helper, 0)} TJS, ${(usd.reduce(helper, 0) * this.usdSellValue).toFixed(2)} TJS (долларовый)`;
    },
    currentPeriodCredits() {
      return this.actualCredits.filter(p => new Date(p['next_payment_date']).getTime() <  periodDate(this.endDateForCurrentPeriod)).sort(sortByDate)
    },
    nextPeriodCredits() {
      const start = dayjs(this.endDateForCurrentPeriod).format('YYYY-MM-DD');
      const separator = dayjs(this.endDateForCurrentPeriod).get('date')
      const end = separator === 15 ? dayjs(start).endOf('month').format('YYYY-MM-DD') : dayjs(this.endDateForCurrentPeriod).add(14, 'day').format('YYYY-MM-DD');
      return this.actualCredits.filter(p => new Date(p['next_payment_date']).getTime() >= periodDate(start) && new Date(p['next_payment_date']).getTime() <= periodDate(end)).sort(sortByDate)
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