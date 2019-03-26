<template>
    <div id="app" class="flex items-center justify-center h-screen w-screen bg-grey-lighter">
        <HelloWorld message="Hello" />
    </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
    components: {
        HelloWorld,
    },
}
</script>
