<template>
  <v-app>
    <v-app-bar
      app
      color="secondary"
      dark
    >
      <div class="d-flex align-center">
        <v-img
          alt="Clash Royale Logo"
          class="logo mr-2"
          src="./assets/logo.png"
          transition="scale-transition"
          width="20"
        />
        <h1 class="headline mr-2 text-no-wrap">
          Clash Royale
        </h1>
      </div>

      <v-spacer></v-spacer>

      <span class="mr-1 caption">{{lastWarDateFormatted}}</span>
    </v-app-bar>

    <v-content>
      <WarLog v-bind:clanTag=clanTag @updateLastWarDate="lastWarDate = $event"/>
    </v-content>
  </v-app>
</template>

<script>
import WarLog from './components/WarLog';

export default {
  name: 'App',
  components: {
    WarLog,
  },
  computed: {
    lastWarDateFormatted: function () {
      return this.lastWarDate.replace("T", " ");
    }
  },
  data: () => ({
    lastWarDate: '',
    clanTag: '',
  }),
  created() {
    var path = window.location.pathname
    this.clanTag = path.length > 1 ? path.replace('/','') : '2YYYGL';
    
  }
};
</script>

<style>
.logo {
  border-radius: 4px;
}
.theme--light.v-application {
  background-color: #FFF;
}
</style>
