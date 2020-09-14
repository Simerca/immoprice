<template>
<nav class="navbar navbar-expand-lg navbar-light fixed-top bg-light">
    <div class="container-fluid">
        <a class="navbar-brand" href="#">Immoprice</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNavAltMarkup" aria-controls="navbarNavAltMarkup" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
            <div class="navbar-nav">
                <router-link tag="a" class="nav-link" :to="menu.url" v-for="(menu, i) in menus" :key="i">{{menu.title}}</router-link>
            </div>
        </div>
    </div>
</nav>
</template>

<script>
import gql from 'graphql-tag'
export default {

    apollo: {
        menus() {
            return {
                query: gql `
                    {
                        menus{
                            title,
                            url
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
