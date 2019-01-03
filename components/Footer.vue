<template>
  <footer>
    <p class="text-md-right mb-1">Explore any fediverse instance's emojis.</p>
    <p class="text-md-right">Created with ❤ at <a href="https://cyberscape.co/"><Logo height='20' :inline='true'/></a></p>
  </footer>
</template>

<script>
import Logo from '~/components/Logo.vue'

export default {
  components: { Logo }
}
</script>

<style>
footer {
  margin-top: 20pt;
  font-size: 20px;
}
</style>

