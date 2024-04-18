<script setup>
import SaveEditor from "./components/SaveEditor.vue";
</script>

<template>
  <div class="font-prompt p-20">
    <h1 class="text-2xl text-center font-bold mx-auto">Palworld Server Config Editor</h1>
    <img
      alt="Vue logo"
      src="https://tech.palworldgame.com/img/logo.jpg"
      class="rounded-full w-32 h-32 mx-auto mt-5"
    />
    <p class="text-xs text-center font-bold mx-auto mt-5">
      Made with ❤️ by <a target="_blank" href="https://github.com/siraom15">siraom15</a>
    </p>
    <SaveEditor />
  </div>
</template>
