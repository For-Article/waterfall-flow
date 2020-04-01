<template>
    <div>
        <div class="waterfall-wrapper">
            <!-- 第一列 -->
            <div class="column-wrapper" ref="column1" :style="{marginTop: merge && mergeColumns.indexOf(1) > -1 ? mergeHeight + 'px':''}">
                <template v-if="$slots['first-col']">
                    <slot name="first-col"></slot>
                </template>
                <template v-for="(item, index) in columnList1">
                    <!-- 第一列瀑布流内容... -->
                    <Item :data="item.data" v-bind:key="index"/>
                </template>
            </div>
            <!-- 第二列 -->
            <div class="column-wrapper" ref="column2" :style="{marginTop: merge && mergeColumns.indexOf(2) > -1 ? mergeHeight + 'px':''}">
                <template v-if="$slots['second-col']">
                    <slot name="second-col"></slot>
                </template>
                <template v-for="(item, index) in columnList2">
                    <!-- 第二列瀑布流内容... -->
                    <Item :data="item.data" v-bind:key="index"/>
                </template>
            </div>
            <!-- 第三列 -->
            <div class="column-wrapper" ref="column3" :style="{marginTop: merge && mergeColumns.indexOf(3) > -1 ? mergeHeight + 'px':''}">
                <template v-if="$slots['third-col']">
                    <slot name="third-col"></slot>
                </template>
                <template v-for="(item, index) in columnList3">
                    <!-- 第三列瀑布流内容... -->
                    <Item :data="item.data" v-bind:key="index"/>
                </template>
            </div>
            <!-- 第四列 -->
            <div class="column-wrapper" ref="column4">
                <template v-if="$slots['last-col']">
                    <slot name="last-col"></slot>
                </template>
                <template v-for="(item, index) in columnList4">
                    <!-- 第四列瀑布流内容... -->
                    <Item :data="item.data" v-bind:key="index"/>
                </template>
            </div>
            <!-- 合并块非瀑布流内容 -->
            <div class="column-merge" v-if="merge" :style="{left: (mergeColumns[0] - 1)*240 + 'px'}">
                <slot name="merge-col"></slot>
            </div>
        </div>
    </div>
</template>

<script>
import walldata from '../data.js';
import Item from './item.vue';

// 可视区域高度
const winHeight = document.documentElement.clientHeight || document.body.clientHeight || window.innerHeight;

// 可视区域可展示item预估个数
const canShowFeedNum = (Math.ceil(winHeight/500) + 1) * 4;

// 上一次渲染的最小高度
let renderMinHeight = 0;

export default {
    name: 'waterfall',
    components: {
        Item
    },
    props: {
        requestParams: {
            type: Object,
            require: true
        },
        merge: {
            type: Boolean
        },
        mergeHeight:{
            type: Number
        },
        mergeColumns:{
            type: Array
        }
    },
    data(){
        return{
            pageIndex: 1,
            pageTotal: 3,
            allList: [], // 拿到的全部数据列表
            columnList1: [],
            columnList2: [],
            columnList3: [],
            columnList4: [],
            dataListLength: 0, // item长度，配合renderIndex使用
            renderIndex: -1, // 渲染第几个item
            isRendering: false, // 是否正在渲染
            detectiveDesc: '精彩加载中...',
            isEnd: false,
            mbook: ''
        };
    },
    mounted() {
        this.getData();
        window.addEventListener('scroll', this.handleScroll);
    },
    watch:{
        renderIndex(index){
            // 当前滚动高度
            const scrollTop = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop;

            // 如果item最小高度在可是范围内，则继续加载
            if(renderMinHeight - scrollTop < winHeight){
                setTimeout(() => {
                    this.requestAnimationFrame(this.renderWall);
                }, 1000);
            }

            // 如果再加载一屏幕就加载完成item,则继续请求数据
            if(index + canShowFeedNum > this.allList.length && this.pageIndex <= this.pageTotal){
                // 图墙加载
                this.getData();
            }
        }
    },
    methods: {
        getData(){
            let newData = walldata[`page${this.pageIndex}`] || [];
            console.log(this.allList.length)
            if(this.pageIndex > this.pageTotal){
                console.log('没有更多数据了');
            }else{
                this.allList = this.allList.concat(newData);
                this.pageIndex = this.pageIndex + 1;
            }
            this.renderWall();
        },
        renderWall(){

            // 如果已经将所有数据全部渲染完成或者是正在渲染其他数据则停止新的渲染
            if(this.renderIndex >= this.allList.length - 1 || this.isRendering){
                return;
            }

            this.isRendering = true;

            // 计算4列的高度
            let arr = [this.$refs.column1.offsetHeight, this.$refs.column2.offsetHeight,this.$refs.column3.offsetHeight, this.$refs.column4.offsetHeight];
            if(this.merge){
                arr[0] = this.mergeColumns.indexOf(1) > -1 ? arr[0] + this.mergeHeight : arr[0];
                arr[1] = this.mergeColumns.indexOf(2) > -1 ? arr[1] + this.mergeHeight : arr[1];
                arr[2] = this.mergeColumns.indexOf(3) > -1 ? arr[2] + this.mergeHeight : arr[2];
                arr[3] = this.mergeColumns.indexOf(4) > -1 ? arr[3] + this.mergeHeight : arr[3];
            }

            // 获取高度排序索引，按照索引排序
            let arrSortNum = [];
            let arrCopy = JSON.parse(JSON.stringify(arr));
            for(let i = 0; i < arrCopy.length; i++){
                let minIndex = arrCopy.indexOf(Math.min.apply(Math, arrCopy));
                arrCopy.splice(minIndex, 1, 99999999);
                arrSortNum.push(minIndex);
            }

            // 最小列高度
            renderMinHeight = arr[arrSortNum[0]] + 15;

            // 判断本次渲染可插入几条数据, 如果相邻两列高度相差很大,则不渲染,继续进行比较
            let addNum = 0;
            for(let i = 0; i < arrSortNum.length; i++){
                if(i > 0 && arr[arrSortNum[i]] - arr[arrSortNum[i - 1]] > 500 ){
                    break;
                }else if(this.renderIndex + addNum + 1 < this.allList.length){
                    addNum++;
                    const key = `columnList${arrSortNum[i] + 1}`;
                    let itemData = this.allList[this.renderIndex + addNum];
                    this[key] = this[key].concat({
                        i: this.renderIndex + addNum,
                        data: itemData
                    });
                }
            }

            this.$nextTick(() => {
                this.renderIndex = this.renderIndex + addNum;
                this.isRendering = false;
            });


        },
        requestAnimationFrame(callback) {
            const f = window.requestAnimationFrame ||
                window.webkitRequestAnimationFrame ||
                window.mozRequestAnimationFrame ||
                function(c) {
                    window.setTimeout(c, 1000 / 60);
                };
            f(callback);
        },
        handleScroll(){

            // 当前滚动条高度
            const scrollTop = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop;
            const bottomDetectionTop = this.$refs.bottomDetection.offsetTop;

            if(bottomDetectionTop - scrollTop < winHeight){
                this.requestAnimationFrame(this.renderWall)
            }
        }
    }
}
</script>
<style lang="less">
.waterfall-wrapper{
    width: 960px;
    height: 100%;
    position: relative;
    overflow: hidden;
    margin: 0 auto;
}
.column-wrapper{
    width: 220px;
    margin-right: 20px;
    float: left;
    overflow: hidden;
    &:last-child {
        margin-right: 0;
    }
}
.column-merge{
    position: absolute;
    top: 0;
}
</style>
