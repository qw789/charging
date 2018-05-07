<template>
  <div @click="test">
      test
    </div>
</template>

<script>

export default {
  methods:{
    test(){
        this.$http.post('/api/charge/getChargeFeeTimeEntity',{
        "hour": hour,
        "minute": minute,
        "stationId": 48
       }).then((response) => {
        console.log(response.data.feeType);
      })
    }
  },
  created(){
    //   this.test();
      for(var j=0;j<24;j++){
          for(var i=0;i<60;i=i+6){
              console.log(j+":"+i+"----")
               this.$http.post('/api/charge/getChargeFeeTimeEntity',{
                "hour": j,
                "minute": i,
                "stationId": 48
            }).then((response) => {
                console.log(j+":"+i+"----"+response.data.feeType);
            })
          }
      }
  }
}
</script>

<style lang="less" rel="stylesheet/less" scoped>
  
</style>