<template>
  <div id="app">
    <SqlEditor />
  </div>
</template>

<script>
import SqlEditor from "./components/SqlEditor.vue";

export default {
  name: "app",
  components: {
    SqlEditor
  }
};
</script>

<style>
#app {
  width: 100%;
  height: 50vh;
}
</style>
