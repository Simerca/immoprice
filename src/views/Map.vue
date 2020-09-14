<template>
  <div class="home">
    <!-- <Navbar /> -->
    <Map />
  </div>
</template>

<script>
// @ is an alias to /src
import Map from '@/components/Map.vue'
// import Navbar from '@/components/Navbar.vue'

export default {
  name: 'Home',
  components: {
    Map,
    // Navbar
  }
}
</script>
