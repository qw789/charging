<template>
  <div>
    <div class="flex-center">
      <div class="flex-item">
        <img src="../assets/chongzhichenggong.png" height="118" width="118" alt="">
        <div class="charging-success">{{status}}</div>
      </div>
    </div>
    <div class="flex-group">
         <flexbox>
          <flexbox-item v-show="ChargeContinue">
            <x-button type="primary" style="border-radius:99px;background:#4582ff;" @click.native="recharge()">继续充电</x-button>
          </flexbox-item>
          <flexbox-item style="padding:0 30px">
            <x-button plain style="border-radius:99px;" @click.native="goback()">返回账户</x-button>
          </flexbox-item>
        </flexbox>
      </div>
  </div>
</template>

<script>
import { XButton,Flexbox, FlexboxItem} from 'vux'

export default {
   components: {
    XButton,
    Flexbox, FlexboxItem
  },
  data () {
    return{
      status:"充值成功",
      ChargeContinue:false
    }
  },
  methods:{
    goback(){
       var mp = this.$route.query.mp;
      this.$router.push({name:'selfCenter', query: { mp: mp }})
    },
    recharge(){
       var mp = this.$route.query.mp;
       var addr = this.$route.query.addr;
      this.$router.push({name:"haslogin", query: { mp: mp,addr:addr }})
    }
   },
    created() {
    var center=this.$route.query.center;
    if(center=="self"){
      this.ChargeContinue=true;
    }
  }
}
</script>

<style lang="less" rel="stylesheet/less" scoped>
  .flex-center{
    display:flex;
    justify-content:center;
    align-items:center;
    height: 300px;
  }
  img{
     width:118/75rem;
     height:118/75rem;
  }
  .flex-item{
    width:130/75rem;
  }
  .charging-success{
    font-size:32/75rem;
  }
  .flex-group{
    padding:0 30px;
  }
</style>
