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
          class="shrink mr-2"
          src="./assets/logo.png"
          transition="scale-transition"
          width="20"
        />
        <h1 class="headline mr-4">
          Clash Royale
        </h1>
      </div>

      <v-spacer></v-spacer>

      <span class="mr-1">{{lastWarDateFormatted}}</span>
    </v-app-bar>

    <v-content>
      <WarLog @updateLastWarDate="lastWarDate = $event"/>
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
    lastWarDate: ''
  }),
};
</script>

<style>
.theme--light.v-application {
  background-color: #FFF;
}
</style>
