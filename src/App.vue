<template>
  <v-app>
    <!-- ヘッダエリア -->
    <header>
      <AppHeader />
    </header>
    <!-- メインエリア -->
    <v-main class="ml-3">
      <v-container fluid>
        <router-view />
      </v-container>
    </v-main>
    <!-- フッタエリア -->
    <!-- <v-footer app>フッタ</v-footer> -->
  </v-app>
</template>

<script>
import AppHeader from "@/components/AppHeader.vue";
export default {
  name: "app",
  components: {
    AppHeader,
  },
};
</script>

<style scoped>
</style>
