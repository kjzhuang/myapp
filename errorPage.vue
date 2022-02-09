<template>
    <div class="error-page" orient="vertical" wrap="wrap" justify="center" align="center" direction="column">
         
        <div class="error-page__msg">{{ errorMsg }}</div>
        <router-link class="error-page__btn vux-1px"
                     :to="$route.query.to"
                     v-if="$route.query.to"
                     replace>重试
        </router-link>
    </div>
</template>
<script lang="babel">
    import { mapState } from 'vuex';

    export default {
        computed: {
            ...mapState({
                isOnLine: state => state.page.isOnLine,
            }),
            errorMsg() {
                return this.isOnLine ? '网络不给力，请稍后重试' : '没有网络，请检查网络设置后重试';
            },
        },
    };
</script>
<style lang="less" scoped>
    .error-page {
        height: 100%;
        height: 100vh;
        background: #F6F6F6;
        font-size: 14px;
        color: #AAAAAA;
    }

    .error-page__msg {
        margin-top: 18px;
    }

    .error-page__btn {
        display: block;
        margin-top: 30px;
        width: 140px;
        height: 36px;
        line-height: 36px;
        text-align: center;
        color: #AAAAAA;
    }
</style>
