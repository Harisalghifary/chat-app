<template>
  <div id="app">
    <ChatWindow />
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import ChatWindow from "./components/ChatWindow.vue";

export default defineComponent({
  name: "App",
  components: {
    ChatWindow,
  },
});
</script>

<style>
#app {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f9f9f9;
}
</style>
