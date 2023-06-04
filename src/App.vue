<template>
  <Three_js />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import Three_js from "./components/Three_js.vue";

export default defineComponent({
  name: "App",
  components: {
    Three_js,
  },
});
</script>
