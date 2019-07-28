<template>
    <div class="home">
        <DatawalletLogin/>
    </div>
</template>

<script>
    import DatawalletLogin from '@/components/DatawalletLogin.vue'

    export default {
        name: 'home',
        components: {
            DatawalletLogin
        }
    }
</script>
