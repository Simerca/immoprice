<template>
<div class="home">
    <Navbar />
    <Carousel />
    <div class="container" v-if="options">
        <div class="col-12 my-2" v-for="(option,i) in options" :key="i">
           <vue-markdown>{{ option.value }}</vue-markdown>
        </div>
        <BlogsPosts />
    </div>
    <Footer />
</div>
</template>

<script>
// @ is an alias to /src
import Navbar from '@/components/Navbar.vue'
import Carousel from '@/components/Carousel.vue'
import BlogsPosts from '@/components/BlogsPosts.vue'
import Footer from '@/components/Footer.vue'
import gql from 'graphql-tag'
import VueMarkdown from 'vue-markdown'
export default {
    name: 'Home',
    components: {
        Navbar,
        BlogsPosts,
        VueMarkdown,
        Footer,
        Carousel
    },
    apollo: {
        options() {
            return {
                query: gql `
                {options(where:{key_contains:"home_header_text"}){
                  value
                }}
          `
            }
        }
    }
}
</script>
