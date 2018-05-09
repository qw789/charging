<template>
    <div class="popup">
        <div class="content">
            <div style="padding:30px;">
                <div class="flex-center" style="padding-top:150px;">
                    <img src="../assets/chongzhishibai.png" height="118" width="118" alt="">
                </div>
                <div style="text-align:center;padding-top:30px;">
                    您没有正在充电的订单
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
  methods: {}
};
</script>

<style lang="less" rel="stylesheet/less" scoped>
.popup {
  width: 10rem;
  height: 100%;
  background: #fff;
  position: absolute;
  top: 0;
  left: 0;
  display: flex;
  justify-content: center;
  align-items: flex-start;
}
.content {
  width: 700/75rem;
  background: #fff;
  border-radius: 10px;
}
.content div {
  font-size: 30/75rem;
}
.flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  background: #fff;
}
</style>