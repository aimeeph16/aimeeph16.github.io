<template>
  <div>
    <FrontHeader></FrontHeader>
    <Nuxt />
    <FrontFooter></FrontFooter>
  </div>
</template>
<style>
</style>
<script>
  import FrontHeader from "../components/partials/FrontHeader";
  import FrontFooter from "../components/partials/FrontFooter";
  export default {
    components: {FrontFooter, FrontHeader},
    methods: {
      isAuthenticated() {
        if(process.browser) {
          const user = JSON.parse(localStorage.getItem('user_data'));
          if (localStorage.getItem('is_authenticated') === "1" && localStorage.getItem("auth_token") != null && user.id) {
            this.$store.dispatch('general/storeAuthData', {auth_token: localStorage.getItem("auth_token"), user_data: user});
          } else {
            this.$store.dispatch('general/resetAuthData');
            this.$store.commit('cart/clear');
          }
        }
      },
      checkUserLogin() {
        if(process.browser) {
          // send request to check if the user is logged
          if (localStorage.getItem('auth_token') !== null && localStorage.getItem('auth_token') !== undefined) {
            this.$axios.setHeader('Authorization', "Bearer " + localStorage.getItem('auth_token'));
            this.$axios.$get('api/check-login').then(response => {
              if (response.success !== 1) {
                localStorage.removeItem('auth_token');
                localStorage.removeItem('is_authenticated');
                localStorage.removeItem('user_data');
                this.$store.dispatch('general/resetAuthData');
                this.$store.commit('cart/clear');
                this.$router.push("/");
              }
            }).catch(err => {
              localStorage.removeItem('auth_token');
              localStorage.removeItem('is_authenticated');
              localStorage.removeItem('user_data');
              this.$store.dispatch('general/resetAuthData');
              this.$store.commit('cart/clear');
              this.$router.push("/");
            });
          } else {
            this.$store.dispatch('general/resetAuthData');
          }
        }
      }
    },
    mounted() {
      this.isAuthenticated();
      const verify = setInterval(this.checkUserLogin, 8000);
      if(!this.$store.state.general.auth.is_logged || !this.$store.state.general.auth.auth_token) {
        clearInterval(verify);
      }
      // fetch shopping cart
      if(localStorage.getItem('is_authenticated') === "1" && localStorage.getItem("auth_token") != null) {
        this.$store.dispatch('cart/getAll');
      }
    }
  }
</script>