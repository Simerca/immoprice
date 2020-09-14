<template>
<div id="carouselExampleCaptions" class="carousel slide" data-ride="carousel">
    <ol class="carousel-indicators">
        <li data-target="#carouselExampleCaptions" v-for="(carousel, i) in carousels" :key="i" :class="{'active':i===0}"></li>
    </ol>
    <div class="carousel-inner" >
        <div class="carousel-item" v-for="(carousel, i) in carousels" :key="i" :class="{'active':i===0}">
            <img :src="'http://localhost:1337'+carousel.image.url" class="d-block w-100" alt="...">
            <div class="carousel-caption d-none d-md-block">
                <h5>{{carousel.title}}</h5>
                <p>{{carousel.content}}</p>
            </div>
        </div>
    </div>
    <a class="carousel-control-prev" href="#carouselExampleCaptions" role="button" data-slide="prev">
        <span class="carousel-control-prev-icon" aria-hidden="true"></span>
        <span class="sr-only">Previous</span>
    </a>
    <a class="carousel-control-next" href="#carouselExampleCaptions" role="button" data-slide="next">
        <span class="carousel-control-next-icon" aria-hidden="true"></span>
        <span class="sr-only">Next</span>
    </a>
</div>
</template>

<script>
import gql from 'graphql-tag'
export default {

    apollo: {
        carousels() {
            return {
                query: gql `
                {
                    carousels{
                        title,
                        content,
                        image{
                            url
                        },
                        button_link
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
