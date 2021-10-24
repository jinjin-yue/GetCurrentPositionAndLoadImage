<template>  
    <!-- ヘッダエリア -->
    <v-app-bar app dark dense>
      <v-toolbar-title>SampleApp</v-toolbar-title>
      <v-tabs align-with-title>
        <v-tab to="/">Map</v-tab>
      </v-tabs>
    </v-app-bar>    
</template>

<script>
export default {
  name: "app-header"
};
</script>

<style scoped>
.v-toolbar__title {
  overflow: visible !important;
}
</style>
