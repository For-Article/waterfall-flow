<template>
    <div class="item">
        <img :src="data.img"/>
    </div>
</template>

<script>
export default {
    props: {
        data: {
            type: Object,
            require: true
        }
    }
}
</script>

<style scoped>
.item{
    width: 220px;
    height: auto;
    border-radius: 6px;
    overflow: hidden;
    margin-bottom: 15px;
}
.item img{
    display: block;
    width: 100%;
    border-radius: 6px;
}
</style>