<template>
  <div>
    <Title text='Mastodon emoji search'/>
    <SearchComponent/>
    <Footer/>
  </div>
</template>

<script>
import Footer from '~/components/Footer.vue'
import SearchComponent from '~/components/SearchComponent.vue'
import Title from '~/components/Title.vue'

export default {
  components: { Footer, SearchComponent, Title }
}
</script>
