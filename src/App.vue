<template>
  <v-app>
    <v-main>
      <Home file-type=".pdf"/>
    </v-main>
  </v-app>
</template>

<script>
import Home from './views/Home.vue'

export default {
  name: 'App',

  components: {
    Home,
  },

  data: () => ({
    //
  }),
}
</script>
