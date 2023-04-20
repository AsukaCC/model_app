<template>
    <div id="app">
        <HelloWorld />
    </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
    name: 'App',
    components: {
        HelloWorld
    }
}
</script>

<style>
* {
    padding: 0 0;
    margin: 0 0;
}

#app {
    width: 100%;
    height: 100%;
}
</style>
