<template>
  <v-app>
      <v-main>
          <router-view />

          <Navigation class="position-fixed" />
      </v-main>
  </v-app>
</template>

<script lang="ts" setup>
import Navigation from "@/components/Navigation.vue";
</script>
