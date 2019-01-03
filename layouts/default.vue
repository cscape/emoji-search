<template>
  <div class="container">
    <nuxt/>
  </div>
</template>

<style>
@import '~/assets/face.css';

:root, html, body {
  background-color: #0f1414;
  color: #e8467a;
  font-size: 18px;
  -ms-text-size-adjust: 100%;
  -webkit-text-size-adjust: 100%;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  box-sizing: border-box;
}

body {
  padding: 4rem 0;
}

::selection {
  background: hsla(341, 88%, 79%, 1);
  color: #FFFFFF;
}

*::selection {
  background: hsla(341, 88%, 79%, 1);
  color: #FFFFFF;
}

.nuxt-progress {
  opacity: 1;
  height: 100vh !important;
  bottom: 100% !important;
  top: auto !important;
  position: absolute;
  box-shadow: 0 100vh rgba(232, 70, 122, 0.25);
}

</style>
