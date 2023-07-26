<template>
  <v-app>
    <Navbar :user="user" />
    <v-main  style=" background-image: linear-gradient(to right, #22889d, #138897, #038790, #008688, #03857f);"
  >
      <router-view :key="$route.path" />
    </v-main>
    <Footer />
  </v-app>
</template>

<script>
import Navbar from "@/components/Navbar";
import Footer from "@/components/Footer";
export default {
  name: "App",
  data() {
    return {
      user: {
        admin: false,
        steam_id: "",
        id: -1,
        super_admin: false,
        name: "",
        small_image: "",
        medium_image: "",
        large_image: ""
      } // should be object from JSON response
    };
  },
  components: {
    Navbar,
    Footer
  },
  async mounted() {
    this.user = await this.IsLoggedIn();
  }
};
</script>

<style>
a {
  text-decoration: none;
}
.theme--dark.v-card {
  background-color: unset;
  background-image: linear-gradient(to right, #1d856f, #118072, #067c74, #017775, #047275, #006f77, #006c78, #016979, #00687c, #00667f, #006582, #026384);
}
.theme--dark.v-data-table {
  background-color: unset;
}
.v-btn.v-btn--is-elevated.v-btn--fab.v-btn--has-bg.v-btn--round.v-btn--rounded.theme--dark.v-size--small.grey.darken-2 {
  display:none;
}
</style>
