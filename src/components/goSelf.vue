<template>
  <div class="flex-two">
      <div class="flex-item ">
        <i style="margin-right:10px;"><img :src="headerData.tenantLogo" alt="" class="wallet"></i>
        <span style="margin-right:20px;">{{headerData.tenantName}}</span>
        <a :href="'tel:'+headerData.tenantPhone" class="wrapPhone">
          <img src="../assets/phone.png" alt="" class="phone">
        </a>
      </div>
      <div class="flex-item">
        <div class="goSelf" @click="goSelf">
          <img src="../assets/user.png" alt="">
        </div>
      </div>
    </div>
</template>

<script>

export default {
  methods:{
    goSelf(){
        var mp = this.$route.query.mp;
        location.href="http://cp.gtcx-tech.com/api/user?mp="+mp
    }
  },
  props:['headerData']
}
</script>

<style lang="less" rel="stylesheet/less" scoped>
  .goSelf{
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  // border-radius: 15px;
  // border:1px solid #5b9eff;
}
.goSelf img{
  width: 100%;
  height: 100%;
  // border-radius: 15px;
}
.flex-two {
  display: flex;
  justify-content: space-between;
  padding: 0 25px;
  font-size: 0.37rem;
  margin-bottom: 0.2rem;
  background: #fff;
  height: 1.33rem;
  line-height: 1.33rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.4);
  align-items: center;
}
.phone{
  width: 0.4rem;
  height: 0.4rem;
  position: relative;
  top:2px;
}
.wallet {
  width: 60/75rem;
  height: 60/75rem;
  display: inline-block;
  vertical-align: middle;
}
.wrapPhone{
  vertical-align: middle;
}
</style>