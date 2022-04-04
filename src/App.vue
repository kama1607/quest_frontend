<template>
  <v-app>
    <v-app-bar
      app
      dark
    >
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
    </v-app-bar>

    <v-navigation-drawer
        app
        v-model="drawer"
        temporary
    >
      <v-list
          nav
          dense>
        <v-list-item-group
            v-model="group"
            active-class="deep-purple--text text--accent-4">

          <v-list-item link :to="{name: 'Home'}">
            <v-list-item-title>Главная страница</v-list-item-title>
          </v-list-item>

          <v-list-item link :to="{name: 'Quest'}">
          <v-list-item-title>Квесты</v-list-item-title>
          </v-list-item>

          <v-list-item link :to="{name: 'Tasks'}">
            <v-list-item-title>Задачи</v-list-item-title>
          </v-list-item>  

          <v-list-item link :to="{name: ''}">
            <v-list-item-title>
              Ответы
            </v-list-item-title>
          </v-list-item>
          

        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <router-view/>
    </v-main>

  </v-app>
</template>

<script>

export default {
  name: 'App',

  data: () => ({
    drawer: false,
    group: null,
  }),

  watch: {
    group () {
      this.drawer = false
    },
  },


};
</script>
