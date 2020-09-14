<template>
  <div class="row row-cols-1 row-cols-md-3 g-4" v-if="blogs">
  <div class="col" v-for="(blog,i) in blogs" :key="i">
    <div class="card">
      <img :src="'http://localhost:1337'+blog.image.url" class="card-img-top" alt="...">
      <div class="card-body">
        <h5 class="card-title">{{ blog.title }}</h5>
        <p class="card-text">{{ blog.content }}</p>
      </div>
    </div>
  </div>
</div>
</template>

<script>
import gql from 'graphql-tag'
export default {

    apollo: {
        blogs(){
            return {
                query:gql`
                {
                    blogs(limit:3){
                        title,
                        content,
                        image {
                            url
                        }
                    }
                }
                `
            }
        }
    }

}
</script>

<style>

</style>