<template>
  <div id="app">
    <dashboard/>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import HelloWorld from "./components/HelloWorld.vue";
import Dashboard from "./components/Dashboard.vue";

@Component({
  components: {
    HelloWorld,
    Dashboard
  }
})
export default class App extends Vue {}
</script>
