<template>
  <v-app>
    <v-layout class="rounded rounded-md">
      <v-app-bar color="surface-variant" title="Application bar">
        <template v-slot:prepend>
          <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
        </template>
      </v-app-bar>
      <v-navigation-drawer
        v-model="drawer"
        :location="$vuetify.display.mobile ? 'bottom' : undefined"
        temporary
      >
        <v-list
          density="compact"
          nav
        >
          <v-list-item prepend-icon="mdi-view-dashboard" title="Home" to="/"></v-list-item>
          <v-list-item prepend-icon="mdi-forum" title="About" to="/about"></v-list-item>
        </v-list>
      </v-navigation-drawer>
      <v-main class="d-flex align-center justify-center" style="min-height: 300px;">
        <slot />
      </v-main>
    </v-layout>
  </v-app>
</template>

<script lang="ts">
  import { defineComponent } from 'vue'

  export default defineComponent({
    data: () => ({
      drawer: false
    })
  })
</script>