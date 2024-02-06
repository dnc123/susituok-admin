<template>
  <v-app class="layout">
      <v-main class="layout__main">
          <router-view />
          <Navigation class="position-fixed" />
      </v-main>
  </v-app>
</template>

<script setup>
import Navigation from "@/components/Navigation.vue";
</script>

<style lang="scss" scoped>
.layout {
    &__main {
        width: 100%;
        max-width: 1000px;
        margin-left: auto;
        margin-right: auto;
    }
}
</style>
