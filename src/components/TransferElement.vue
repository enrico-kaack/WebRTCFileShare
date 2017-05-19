<template>
    <!-- HTML content--->
</template>

<script>
    import Router from '../router/index'; //use it with Router.push()

    export default {
        name: 'hello', //Name that is used in router
        data () {
            return {
                msg: 'Hello World'
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h1, h2 {
        font-weight: normal;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }
</style>
