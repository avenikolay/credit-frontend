<template>
    <div>
      <Auth v-if="authRequire" />
      <Credits :usdSellValue="usdSellValue" :loading="loading" :credits="credits" v-else />
    </div>

</template>

<script>
import Auth from "@/components/Auth";
import Credits from "@/components/Credits";
export default {
  name: 'App',
  components: {Credits, Auth},
  data() {
    return {
      loading: true,
      credits: [],
      currencies: null
    }
  },
  methods: {
    getCurrencies() {
      const myHeaders = new Headers();
      myHeaders.append("token", this.token);
      return new Promise ((resolve) => {
        fetch('http://credit-api.test/currencies.php', {method: 'GET', headers: myHeaders}).then(
            r => r.json()
        ).then(r => {
          this.currencies = {...r};
          resolve()
        })
      })

    },
    getMyCredits() {
      this.loading =  true
      const myHeaders = new Headers();
      myHeaders.append("token", this.token);

      const requestOptions = {
        method: 'GET',
        headers: myHeaders,
        redirect: 'follow'
      };

      fetch("http://credit-api.test/index.php", requestOptions)
          .then(response => response.json())
          .then(result => {
            if (!Array.isArray(result) && result.error) {
              localStorage.removeItem('token');
              return window.location.reload();

            } else {
              this.credits = result
              this.loading = false
            }
          })
          .catch(error => console.log('error', error));
    }
  },
  computed: {
    usdSellValue() {
      if (!this.currencies) return 0;
      return Number(this.currencies?.['USD']?.['sell']) ?? 0
    },
    authRequire() {
      return this.token === null
    },
    token() {
      return localStorage.getItem('token');
    },
  },

  async mounted() {
    if (!this.authRequire) {
      await this.getCurrencies();
      await this.getMyCredits();
    }
  }
}
</script>

