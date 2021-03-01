<template>
  <div class="wrapper">
    <div class="card" style="max-width: 400px">
      <h5 class="card-header">Мои кредиты</h5>
      <div class="card-body">
        <div v-show="error" class="alert alert-danger" role="alert">
          Возможно логин или пароль неправильные!
        </div>
        <form @submit.prevent="auth">
          <input class="form-control mb-2" placeholder="email" v-model="email" type="email" />
          <input class="form-control mb-2" placeholder="пароль" v-model="password" type="password" />
          <button :disabled="!allowLogin" class="btn btn-primary" type="submit">Залогиниться</button>
        </form>
      </div>

    </div>
  </div>

</template>

<script>
export default {
  name: "Auth",
  data() {
    return {
      error: false,
      email: '',
      password: ''
    }
  },
  computed: {
    allowLogin() {
      return true
    }
  },
  methods: {
    auth() {
      fetch('http://online.alif/', {
        method: 'POST',
        body: JSON.stringify({
          login: this.email,
          password: this.password,
        })
      })
          .then((response) => {
            return response.json();
          })
          .then((data) => {
            if (data.token) {
              localStorage.setItem('token', data.token)
              window.location.reload();
              return
            }
            this.error = true
          }).catch(() => {
            this.error = true
      });
    }
  }
}
</script>

<style scoped>
.wrapper {
  background-color: #0093E9;
  background-image: linear-gradient(160deg, #0093E9 0%, #80D0C7 100%);
  width: 100%;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>