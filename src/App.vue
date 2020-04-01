<template>
    <div id="app">
        <div class="title">企业级可扩展Vue瀑布流</div>
        <Waterfall :merge=true :mergeHeight=415 :mergeColumns=[2,3]>
            <template slot="first-col">
                <div class="col-other col-other-first">第一列非瀑布流卡片内容</div>
            </template>
            <template slot="second-col">
                <!-- <div class="col-other col-other-second">第二列非瀑布流卡片内容</div> -->
            </template>
            <template slot="third-col">
                <!-- <div class="col-other col-other-third">第三列非瀑布流卡片内容</div> -->
            </template>
            <template slot="last-col">
                <div class="col-other col-other-last">第四列非瀑布流卡片内容</div>
            </template>
            <template slot="merge-col">
                <!-- 合并内容...     -->
                <div class="col-other col-other-merge">非瀑布流卡片合并列内容(运营可配置资源位)</div>
            </template>
        </Waterfall>
    </div>
</template>

<script>
import Waterfall from './components/waterfall.vue'

export default {
    name: 'App',
    components: {
        Waterfall
    },
    created() {
        this._timer = null;

        // 是否请求中
        this._requesting = false;

        // 是否执行节流函数
        this._throttleFnRun = true;
    },

    mounted() {
    },
    methods: {
    }
}
</script>

<style lang="less" scoped>
.title{
    text-align: center;
    font-size: 32px;
    margin: 20px;
}
.col-other{
    width: 220px;
    border-radius: 6px;
    margin-bottom: 15px;
    text-align: center;
}
.col-other-first{
    height: 330px;
    line-height: 330px;
    background: #669c8a;
    color: #fff;
}
.col-other-second{
    height: 230px;
    line-height: 230px;
    background: coral;
    color: #fff;
}
.col-other-third{
    height: 330px;
    line-height: 330px;
    background: forestgreen;
    color: #fff;
}
.col-other-last{
    height: 130px;
    line-height: 130px;
    background: #806271;
    color: #fff;
}
.col-other-merge{
    width: 460px;
    height: 400px;
    line-height: 400px;
    background: #583044;
    color: #fff;
}
</style>
