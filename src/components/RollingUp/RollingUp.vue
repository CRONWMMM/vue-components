<!-- 向上滚动的消息提示组件 -->
<template>
    <div class="rolling-up-wrap" id="rolling-up-wrap">
        <div class="rolling-up" id="rolling-up">
            <p class="rolling-up-message" v-for="(item, index) in currentList" :key="index">{{item}}</p>
        </div>
    </div>
</template>

<script>
    // utils
    import { copypart, prefixStyle } from '../utils/utils'

    export default {
        props: {
            height: {   // 高度
                type: Number,
                default: 30
            },
            interval: { // 滚动间歇间隔，单位（毫秒）
                type: Number,
                default: 3000
            },
            messageList: {
                type: Array,
                required: true
            }
        },
        mounted() {
            this.rollingUpWrapDOM = document.getElementById('rolling-up-wrap')
            this.rollingUpDOM = document.getElementById('rolling-up')
            this.styleInit()
            if (this.currentList.length > 1) this.iteration()
        },
        data() {
            return {
                currentNum: 0,
                duration: 600,  // 滚动动画间隔，300 ms
                timer: null,
            }
        },
        methods: {
            // 样式初始化
            styleInit() {
                const rollingUpWrapDOM = this.rollingUpWrapDOM
                rollingUpWrapDOM.style.height = rollingUpWrapDOM.style.lineHeight = `${this.height}px`
            },
            // rolling-up 位置改变
            positionChange() {
                const rollingUpDOM = this.rollingUpDOM
                rollingUpDOM.style[prefixStyle('transition')] = `transform ${this.duration / 1000}s`
                rollingUpDOM.style[prefixStyle('transform')] = `translate(0, -${this.height}px)`
            },
            // rolling-up 位置重置
            positionReset() {
                const rollingUpDOM = this.rollingUpDOM
                setTimeout(() => {
                    rollingUpDOM.style[prefixStyle('transition')] = ''
                    rollingUpDOM.style[prefixStyle('transform')] = ''
                }, this.duration)
            },
            // 改变 currentNum
            currentNumChange() {
                const messageListLength = this.messageList.length
                if (this.currentNum < messageListLength - 1)
                    this.currentNum += 1
                else
                    this.currentNum = 0
            },
            // 循环
            iteration() {
                this.timer = setTimeout(() => {
                    this.positionChange()
                    this.positionReset()
                    this.iteration()
                    setTimeout(() => {
                        this.currentNumChange()
                        // 向外发送变更事件
                        this.$emit('change', this.messageList[this.currentNum])
                    }, this.duration)
                }, this.interval)
            }
        },
        computed: {
            // 当前展示的列表
            currentList() {
                const messageListLength = this.messageList.length
                let ret = null
                if (messageListLength === 1) {
                    ret = [...this.messageList]
                    return ret
                }
                if (this.currentNum === messageListLength - 1) {     // 尾
                    ret = [this.messageList[ messageListLength - 1], this.messageList[0]]
                } else {    // 正常
                    ret = copypart(this.messageList, this.currentNum, 2)
                }
                return ret
            }
        }
    }
</script>

<style lang="scss" scoped>
    .rolling-up-wrap{
        height: 20px;
        line-height: 20px;
        overflow: hidden;
    }
    .rolling-up{}
    .rolling-up-message{}
</style>