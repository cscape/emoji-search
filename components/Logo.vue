<template>
  <div class="cyberscape-logo" :style="{ display: inline === true ? 'inline-block' : null }">
    <img alt="Cyberscape" src="/logo.png" :height="height" :width="width"/>
  </div>
</template>

<script>
export default {
  props: {
    'width': {
      default: 'auto',
      type: String
    },
    'height': {
      default: 'auto',
      type: String
    },
    'inline': {
      default: false,
      type: Boolean
    }
  }
}
</script>


<style>
.cyberscape-logo {
  line-height: 1.2;
}
</style>
