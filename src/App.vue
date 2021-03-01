<template>
    <Auth v-if="authRequire" />
    <Credits :loading="loading" :credits="credits" v-else />
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
      credits: []
    }
  },
  methods: {
    getMyCredits() {
      this.loading =  true
      const myHeaders = new Headers();
      myHeaders.append("token", this.token);

      const requestOptions = {
        method: 'GET',
        headers: myHeaders,
        redirect: 'follow'
      };

      fetch("http://online.alif", requestOptions)
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
    authRequire() {
      return this.token === null
    },
    token() {
      return localStorage.getItem('token');
    },
  },

  mounted() {
    if (!this.authRequire) {
      this.getMyCredits();
    }


  }
}
</script>

