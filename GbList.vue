<template>
    <div class="app-container">
        <div class="sticky-header">
            <x-header :left-options="{showBack: true, backText: $i18n.components.satTabbar.home}">
                {{$i18n.pages.gbList.title}}
            </x-header>
            <tab :line-width="5" :animate="false" active-color="#fb7719">
                <tab-item :selected="true" @click="loadData('OPENED')">{{$i18n.pages.gbList.published}}</tab-item>
                <tab-item :selected="false" @click="loadData('DRAFT')">{{$i18n.pages.gbList.unPublished}}</tab-item>
                <tab-item :selected="false" @click="loadData('CLOSED')">{{$i18n.pages.gbList.closed}}</tab-item>
            </tab>
        </div>
        <div class="mt-82 mb-42">
            <h3 class="no-data" v-if="(gbs.length == 0 && isEmpty)">{{$i18n.pages.common.noData}}</h3>
            <scroller lock-x scrollbar-y :bounce=false height="-132" use-pullup @pullup:loading="loadMore" v-ref:scroller>
                <div style="width:100%;">
                    <group v-else>
                        <cell is-link :title="gb.name" @click="toEdit(gb.id)" v-for="gb in gbs">
                            <div slot="after-title" style="font-size: 15px;color: #888888;">{{gb.brief}}</div>
                            <div slot="after-title" style="font-size: 11px;">
                                <div style="float: left;">
                                    <i slot="icon" class="iconfont icon-time"></i>
                                    <span v-text="new Date(gb.startTime) | date-formatter 'MM/DD/YYYY'"></span>-<span v-text="new Date(gb.endTime) | date-formatter 'MM/DD/YYYY'"></span></div>
                                <div style="float: right;">
                                    <i slot="icon" class="iconfont icon-icon"></i>
                                    <span v-text="new Date(gb.pickupEndTime) | date-formatter 'MM/DD/YYYY'"></span>
                                </div>
                            </div>
                        </cell>
                    </group>
                </div>
            </scroller>
        </div>
        <div class="sticky-button">
            <x-button class="sticky-footer clear-border" type="warn" v-link="{path: '/gb/new'}">{{$i18n.pages.gbNew.title}}</x-button>
        </div>
    </div>
</template>
<script>
import Vue from 'vue/dist/vue'
import {
    Group,
    Cell,
    XHeader,
    Tab,
    TabItem,
    Flexbox,
    FlexboxItem,
    DateFormatter,
    XButton,
    Scroller
} from 'vux/src/components'
import SatTabbar from '../components/SatTabbar'
import analytics from '../js/analytics';

export default {
    components: {
        Group,
        Cell,
        XHeader,
        Tab,
        TabItem,
        Flexbox,
        FlexboxItem,
        DateFormatter,
        XButton,
        Scroller
    },
    data() {
        return {
            currentPage: 1,
            currentStatus: 'OPENED',
            totalCount: 0,
            isEmpty: true,
            gbs: []
        }
    },
    filters: {
        DateFormatter
    },
    methods: {
        loadMore(uuid) {
            if ((this.currentPage - 1) * 10 <= this.totalCount) {
                this.$vux.loading.show(this.$i18n.pages.common.loading);
                setTimeout(() => {
                    this.loadData(this.currentStatus, ++this.currentPage);
                    this.$nextTick(() => {
                        this.$broadcast('pullup:reset', uuid);
                    })

                }, 1000);
            } else {
                this.$broadcast('pullup:disable', uuid);
            }
        },
        toEdit(id) {
            this.$router.go('/gb/edit/' + id);
        },
        loadData(currentStatus, currentPage) {
            debugger;

            setTimeout(() => {
                this.$nextTick(() => {
                    this.$refs.scroller.reset();
                })
            }, 1000);
            this.isEmpty = false;
            var list = [];
            this.$vux.loading.show(this.$i18n.pages.common.loading);
            if (this.currentStatus !== currentStatus) {
                this.currentPage = 1;
            }
            currentStatus ? (this.currentStatus = currentStatus) : this.currentStatus;
            let shopId = localStorage.getItem("shopId");
            this.$http.get('/businesses/' + shopId + '/sellPacks?size=10&page=' + this.currentPage + '&status=' + currentStatus).then((resp) => {
                list = resp.data.result;
                this.totalCount = resp.data.total;
                if (this.currentPage === 1) {
                    this.gbs = list;
                    this.toTop = true;
                } else {
                    this.gbs = this.gbs.concat(list);
                }
                this.$vux.loading.hide();
                setTimeout(() => {
                    this.$nextTick(() => {
                        this.$refs.scroller.reset();
                    })
                }, 1000);
                this.currentPage++;
            });
        }
    },
    created: function() {
        setTimeout(() => {
            this.loadData("OPENED");
        }, 1000);
    }
}
</script>
<style lang="less" scoped>
@import "../assets/css/variable.less";
@import "../assets/css/style.less";
.mb-42 {
    margin-bottom: 0px;
}

.mt-82 {
    margin-top: 90px;
}

.sticky-header {
    width: 100%;
    position: absolute;
    left: 0px;
    top: 0px;
    z-index: 100;
}

.sticky-button {
    width: 100%;
    position: absolute;
    left: 0px;
    bottom: 0px;
    z-index: 1000;
}

.app-container {
    position: fixed;
    left: 0px;
    right: 0px;
    height: 100%;
}
</style>
