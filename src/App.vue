<template>
  <v-app>
    <v-main>
      <ShowUser/>
    </v-main>
  </v-app>
</template>

<script>
import ShowUser from './components/ShowUser.vue'

export default {
  name: 'App',

  components: {
    ShowUser
},

  data: () => ({
    //
  }),
};
</script>
