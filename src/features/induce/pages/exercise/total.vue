<template>
  <div>
    <aside v-if="list.length" style="padding:0 .5rem;font-size:.7rem;line-height:1.5rem;margin-bottom:-10px;">
      <p>本节共有<b style="color:#F8BF4D">{{totalCount}}</b>个题型，未处理<b style="color:#F8BF4D">{{recordSize}}</b>题</p>
    </aside>
    <card v-for="(item, index) in list" :key="index">
      <div class="weui-panel__hd" slot="header">
        <flexbox>
          <flexbox-item :span="10" style="color:#4bb7aa">{{item.chapter_name}}</flexbox-item>
          <flexbox-item :span="2">难度: {{item.degree}}</flexbox-item>
        </flexbox>
      </div>
      <div slot="content" @click="$router.push({name:'example', params: {subjectId: Route.params.subject.indexOf('math') !== -1 ? 2 : 7, id: item.exercises_id}})">
        <div v-html="item.stem"></div>
        <div v-if="item.opt_jo.hasOwnProperty('A')">
          <template v-for="(value, key) in item.opt_jo">
            <div style="padding-top:5px;">{{ key }}： <p v-html="value" style="display:inline-block"></p></div>
          </template>
        </div>
      </div>
      <div slot="footer">
        <div class="weui-cell weui-cell_link" style="padding:0;">
          <div class="weui-cell__bd">
            <flexbox style="text-align:center" :gutter="0">
              <flexbox-item :span="3"></flexbox-item>
              <flexbox-item :span="3" @click.native="_operate('practice', item, index)" class="weui-cell_access" style="padding:10px 0px">
                <i class="icon iconfont icon-jinrulianxi" style="font-size:17px;"></i>练习
              </flexbox-item>
              <flexbox-item :span="3" @click.native="_operate('break', item, index)" class="weui-cell_access" style="padding:10px 0px">
                <i class="icon iconfont icon-brush1" style="font-size:17px;"></i>斩题
              </flexbox-item>
              <flexbox-item :span="3" @click.native="_operate('pass', item, index)" class="weui-cell_access" style="padding:10px 0px">
                <i class="icon iconfont icon-lajitong16" style="font-size:17px;"></i>弃题
              </flexbox-item>
            </flexbox>
          </div>
        </div>
      </div>
    </card>
    <div style="text-align:center;padding:20px 0;">
      <spinner v-if="loading" type="lines"></spinner>
      <div>
        <p style="font-size:16px;color:#4BB7AA" v-if="loadingNoData">已经加载全部题型~</p>
        <p style="font-size:16px;color:#4BB7AA" v-if="!loadingNoData && !loading" @click="_getData">点我加载更多</p>
      </div>
    </div>
  </div>
</template>

<script>
import {Card, Spinner, Flexbox, FlexboxItem, TransferDomDirective as TransferDom} from 'vux'
import {mapActions, mapGetters} from 'vuex'

export default {
  name: 'total',
  components: {
    Card, Spinner, Flexbox, FlexboxItem
  },
  computed: {
    ...mapGetters(['Route', 'induceTotal']),
    list () {
      return this.induceTotal.list
    },
    recordSize () {
      return this.induceTotal.recordSize
    },
    totalCount () {
      return this.induceTotal.totalCount
    }
  },
  directives: {
    TransferDom
  },
  data () {
    return {
      loading: true,
      loadingNoData: false
    }
  },
  methods: {
    ...mapActions(['getInduceList', 'setInduceListScroll', 'induceListClear', 'induceTotalAction']),
    _operate (type, item, index) {
      let msg
      if (type === 'practice') msg = '已放入练习本'
      else if (type === 'break') msg = '已放入斩题本'
      else msg = '已放入弃题本'
      this.induceTotalAction({index: index, type: type, id: item.exercises_id, chapter_id: item.chapter_id}).then(() => {
        this.$vux.toast.show({text: msg, type: 'text', time: 1500, position: 'bottom'})
      })
    },
    _getData () {
      this.loading = true
      this.getInduceList({type: 'total'}).then((res) => {
        if (res.data.data.list.length < 10) {
          this.loadingNoData = true
        }
        this.loading = false
      }).catch(() => {
        this.loading = false
      })
    }
  },
  beforeRouteEnter (to, from, next) {
    next(vm => {
      // 不来自题目详情全部清除数据
      if (from.name !== 'example') {
        vm.loadingNoData = false
        vm.induceListClear({type: 'total'})
      }
      // 是否需要加载
      if (vm.induceTotal.isReset) {
        vm._getData()
      }
      vm.$parent.$refs.viewBoxBody.scrollTop = vm.induceTotal.scroll
    })
  },
  beforeRouteLeave (to, from, next) {
    this.setInduceListScroll({type: 'total', height: this.$parent.$refs.viewBoxBody.scrollTop})
    next()
  }
}
</script>
