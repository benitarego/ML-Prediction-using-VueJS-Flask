<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-toolbar-title>
        <v-icon class="ml-4 mr-4">mdi-auto-fix</v-icon>ML Predictions with VueJS and Flask
      </v-toolbar-title>

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-content>
      <router-view></router-view>
    </v-content>
  </v-app>
</template>

<script>
export default {
  name: "App",

  components: {},

  data: () => ({
    //
  })
};
</script>
