<template>
  <div class="demo">
    <div id="demo">
      <GoSelf :headerData="aboutData"></GoSelf>
      <div class="flex-center">
        <div>
          <div class="font-32" style="padding-top:10px;">{{aboutData.name}}</div>
          <water v-if="aboutData.carElectricity!=null" :cardata="aboutData.carElectricity"></water>
          <div class="font-24">{{aboutData.currTime}}更新</div>
        </div>
      </div>
      <div class="flex-group">
        <div class="flex-item">
          <div class="font-38 color-blue">{{aboutData.electricity|returnFloat}}
            <span class="font-24">度</span>
          </div>
          <div>
            <i class="icon"><img src="../assets/battary.png" alt=""></i>用电量</div>
        </div>
        <div class="flex-item">
          <div class="font-38 color-blue">{{aboutData.duration}}
            <span class="font-24">分钟</span>
          </div>
          <div>
            <i class="icon"><img src="../assets/时间.png" alt=""></i>充电时间</div>
        </div>
        <div class="flex-item">
          <div class="font-38 color-blue">{{aboutData.consume|returnFloat}}
            <span class="font-24">元</span>
          </div>
          <div>
            <i class="icon"><img src="../assets/金额.png" alt=""></i>消费金额</div>
        </div>
      </div>
      <div class="flex-group" style="margin-bottom:1px;">
        <div class="flex-item" style="border-bottom:0;">
          <div class="font-32">{{aboutData.feeType|returnFeeType}}</div>
          <div>
            <i class="icon"><img src="../assets/shiduan.png" alt=""></i>当前时段</div>
        </div>
        <div class="flex-item" style="border-bottom:0;">
          <div class="font-32">{{aboutData.fee|returnFloat}}
            <span class="font-24">元</span>
          </div>
          <div>
            <i class="icon"><img src="../assets/dianjia.png" alt=""></i>电价(元/度)</div>
        </div>
        <div class="flex-item" style="border-bottom:0;">
          <div class="font-32">{{aboutData.available|returnFloat}}
            <span class="font-24">元</span>
          </div>
          <div>
            <i class="icon"><img src="../assets/zhanghuyue .png" alt=""></i>账户余额</div>
        </div>
      </div>
      <div class="bottom-radius">
        <div id="bottom-red" ref="endButton">
          <x-button type="primary" action-type="button" :disabled="end" style="border-radius:50px;background:#4582ff;width:auto;" @click.native="endCharge()">{{endChargeText}}
          </x-button>
        </div>
      </div>
    </div>
    <Shadow v-if="isShadowing"></Shadow>
    <OrderVoid v-if="noDataPopup"></OrderVoid>
    <ToggleLoad v-if="isToggleLoading"></ToggleLoad>
    <div id="sidebar">
      <ul>
        <li class="totalOrder">我的订单</li>
        <li v-for='(item,index) in mutileCharge' :key="index" @click="toggleData(item)" :class="{active: activeName == item.addr}">
          {{item.addr}}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { Group, Cell, CellBox, XButton } from "vux";
import { mapState } from "vuex";
import Shadow from "@/components/shadow.vue";
import OrderVoid from "@/components/orderVoid.vue";
import ToggleLoad from '@/components/popup/load.vue'
import water from "@/components/water";
import GoSelf from "@/components/goSelf";
export default {
  components: {
    Group,
    Cell,
    XButton,
    water,
    GoSelf,
    Shadow,
    OrderVoid,
    ToggleLoad
  },
  data() {
    return {
      aboutData: {
        carElectricity: 0,
        electricity: 0,
        duration: 0,
        consume: 0,
        feeType: 0,
        fee: 0,
        available: 0,
        name: "",
        tenantName: ""
      },
      addr: "",
      hasData: {},
      number: null,
      noCharge: true,
      flag: null,
      endChargeText: "结束充电",
      end: false,
      mutileCharge: [],
      activeName: "",
      chargingLength: "",
      chargingIndex: ""
    };
  },
  computed: {
    ...mapState({
      isShadowing: state => state.isShadowing,
      noDataPopup: state => state.noDataPopup,
      isToggleLoading:state=>state.isToggleLoading
    })
  },
  methods: {
    getApi() {
      var mp = this.$route.query.mp;
      var chargingIndex = this.chargingIndex;
      this.$http
        .post("/api/charging/charging", { mp: mp, addr: chargingIndex })
        .then(res => {
          if (res.data.code == 0) {
            this.$store.commit("toggleLoading", { isToggleLoading: false });
            if (res.data.data) {
              //timer中的逻辑，当有数据时返回 且hasData==false，1 hasData设置为true 2 关闭等待状态层 3 timer间隔设置为10s
              this.aboutData = res.data.data;
              this.addr = res.data.data.addr;
              this.number = res.data.data.number;
              if (!this.aboutData.tenantLogo) {
                this.aboutData.tenantLogo = "../static/recharge_logo.png";
              }
              //如果这个桩有数据后，把他的hasData设为true
              if (!this.hasData[chargingIndex]) {
                this.hasData[chargingIndex] = true;

                //关闭状态等待层???
                this.$store.commit("waitingControl", { isShadowing: false });
                //停止从扫码充电进来请求数据的定时器
                if (window.intervalid1) {
                  clearInterval(intervalid1);
                }
                window.intervalid1 = setInterval(this.getApi, 10000);
              }
            }
            //-------------------------------------
            if (!res.data.data) {
              //充电自动结束
              if (this.hasData[chargingIndex] == true) {
                var number = this.number;
                this.$router.push({
                  name: "chargingFinish",
                  query: { orderId: number, mp: mp, addr: chargingIndex }
                });
                return;
              }
              //第一次进来无数据
              // if (!this.hasData[chargingIndex]) {
              //   console.log("无数据");
              // }
              //显示无数据层,直接点充电中的
              if (!this.flag) {
                this.$store.commit("waitingControl", { isShadowing: false });
                this.$store.commit("noDataControl", { noDataPopup: true });
              }
            }
          } else {
            this.$msgbox(res.data.msg);
          }
        })
        .catch(function(err) {
          console.log(err);
        });
    },
    chargingList() {
      var mp = this.$route.query.mp;
      this.$http.post("/api/charging/chargingList", { mp: mp }).then(res => {
        //箭头函数绑定的是父级作用域的上下文及vue实例
        //如果列表<=1，不用显示列表
        if (res.data.data && res.data.data.length) {
          this.mutileCharge = res.data.data;
          this.chargingLength = res.data.data.length;
          if (this.chargingLength > 1) {
            document.querySelector("#sidebar").style.display = "block";
            document.querySelector("#demo").style.paddingLeft = "75px";
            //没有指定addr 取第一个
            if (!this.$route.query.addr) {
              this.chargingIndex = this.mutileCharge[0].addr;
              this.activeName = this.mutileCharge[0].addr;
            }
          }
        }
        this.start();
      });
    },
    endCharge() {
      var that = this;
      this.$vux.confirm.show({
        content: "确认结束充电？",
        onConfirm() {
          if (that.flag == "true") {
            clearInterval(intervalid1);
          } else {
            clearInterval(intervalid2);
          }
          if (that.addr) {
            that.endChargeText = "结束中...";
            that.end = true;
            that.$refs.endButton.children[0].style.background = "gray";
            var addr = that.addr;
            var mp = that.$route.query.mp;
            that.$http
              .post("/api/charging/end", {
                addr: addr,
                mp: mp
              })
              .then(
                function(res) {
                  if (res.data.code == 0) {
                    var number = res.data.data; //可能有问题
                    that.$router.push({
                      name: "chargingFinish",
                      query: { orderId: number, mp: mp }
                    });
                  } else {
                    that.$msgbox(res.data.msg);
                  }
                }.bind(that)
              )
              .catch(function(err) {
                console.log(err);
              });
          }
        }
      });
    },
    start() {
      //启动计时
      var timer = 15;
      //从扫码充电进来查看充电订单
      this.flag = this.$route.query.flag;
      if (this.flag == "true") {
        //创建一个间隔2s的timer
        this.getApi();
        window.intervalid1 = setInterval(this.getApi, 2000);
        //设置一个倒计时timeout，执行：hasData为false时 显示无结果状态层，把timer停止；hasData为true时，return
        setTimeout(() => {
          if (!this.hasData[this.chargingIndex]) {
            document.getElementById("demo").style.display = "none";
            this.$store.commit("waitingControl", { isShadowing: false });
            this.$store.commit("noDataControl", { noDataPopup: true });
            clearInterval(intervalid1);
          }
        }, 20000);
      } else {
        //直接点充电中
        this.getApi();
        window.intervalid2 = setInterval(this.getApi, 10000);
      }
    },
    toggleData(item) {
      this.$store.commit("toggleLoading", { isToggleLoading: true });            
      this.activeName = item.addr;
      this.chargingIndex = item.addr;
      this.getApi();
    }
  },
  created() {
    // this.chargingList();
  },
  mounted() {
    var chargingCount = this.$route.query.chargingCount;
    if (chargingCount > 1) {
      document.querySelector("#sidebar").style.display = "block";
      document.querySelector("#demo").style.paddingLeft = "75px";
    }
    if (this.$route.query.addr) {
      this.chargingIndex = this.$route.query.addr;
      this.activeName = this.$route.query.addr;
    }
    this.chargingList();
  },
  beforeDestroy() {
    clearInterval(intervalid1);
    clearInterval(intervalid2);
  },
  filters: {
    returnFloat(value) {
      var value = Math.round(parseFloat(value) * 100) / 100;
      var xsd = value.toString().split(".");
      if (xsd.length == 1) {
        value = value.toString() + ".00";
        return value;
      }
      if (xsd.length > 1) {
        if (xsd[1].length < 2) {
          value = value.toString() + "0";
        }
        return value;
      }
    },
    returnFeeType(value) {
      switch (value) {
        case 0:
          return "未知";
          break;
        case 1:
          return "峰时";
          break;
        case 2:
          return "平时";
          break;
        case 3:
          return "谷时";
          break;
      }
    }
  }
};
</script>

<style lang="less" rel="stylesheet/less" scoped>
#myCanvas {
  width: 10rem;
  height: 300px;
}
ul,
li {
  list-style: none;
}
input[type="range"]::before {
  content: attr(min);
  color: #000;
  padding-right: 5px;
}
input[type="range"]::after {
  content: attr(max);
  color: #000;
  padding-left: 5px;
}
.demo {
  background: #f2f2f2;
}
.bottom-radius {
  background-image: url("../assets/bottom.png");
  background-size: 100% 100%;
}
.flex-group {
  display: flex;
  justify-content: center;
  padding: 0 0.2rem;
  font-size: 0.37rem;
  background: #fff;
}
.flex-item {
  width: 200/75rem;
  border-bottom: 1px solid #eee;
  padding: 10px 0;
  text-align: center;
  color: #999;
}
.flex-item:not(:last-child) {
  border-right: 1px solid #eee;
}
.date {
  line-height: 1.86rem;
}
.kwh {
  font-size: 24/75rem;
  color: #999;
}
.color-green {
  color: #02bd84;
  font-size: 32/75rem;
}
.flex-middel {
  padding: 20px 0;
}
.font-w {
  font-weight: 600;
}
.wallet {
  width: 60/75rem;
  height: 60/75rem;
  display: inline-block;
  vertical-align: middle;
}
.chongdian {
  display: inline-block;
  width: 180/75rem;
  height: 180/75rem;
}
.flex-start {
  display: flex;
  justify-content: flex-start;
  background: #fff;
  padding: 10px 0 20px 10px;
  border-bottom: 1px solid #eee;
}
.font-38 {
  font-size: 38/75rem;
  font-weight: 600;
  color: #666;
}
.font-32 {
  font-size: 32/75rem;
}
.font-24 {
  font-size: 24/75rem;
  color: #999;
}
.font-26 {
  font-size: 26/75rem;
  color: #f8a631;
}
.font-w {
  font-weight: 600;
}
.tips {
  display: inline-block;
  width: 80/75rem;
  height: 35/75rem;
  line-height: 35/75rem;
  background: #5b9eff;
  vertical-align: middle;
  font-size: 20/75rem;
  color: #fff;
  text-align: center;
  font-weight: normal;
  border-radius: 2px;
}
.flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  background: #fff;
}
.circle {
  width: 250/75rem;
  height: 250/75rem;
  line-height: 250/75rem;
  background: #5b9eff;
  border-radius: 125/75rem;
  color: #fff;
  font-size: 32/75rem;
  margin: 20px 0;
}
.bottom {
  display: flex;
  justify-content: space-between;
  padding: 0 30px;
  font-size: 0.37rem;
  height: 1.33rem;
  line-height: 90/75rem;
  height: 90/75rem;
  background: #4582ff;
  color: #fff;
}
.color-red {
  font-size: 28/75rem;
  color: #e01a1a;
  margin: 5px 0;
}
.color-blue {
  color: #4582ff;
}
.icon {
  display: inline-block;
  width: 24/75rem;
  height: 24/75rem;
  margin-right: 5px;
  vertical-align: middle;
}
.icon img {
  width: 100%;
  height: 100%;
}
#bottom-red {
  padding: 0.5rem 0;
}

#sidebar {
  position: absolute;
  left: 0;
  top: 0;
  font-size: 0.4rem;
  height: 100%;
  display: none;
}
#sidebar ul {
  height: 100%;
  background: #f2f2f2;
  overflow: scroll;
}
#sidebar li {
  height: 45px;
  line-height: 45px;
  background: #f2f2f2;
  text-align: center;
  border-bottom: 1px solid #fff;
  color: #fff;
  padding: 0 10px;
  color: #999;
}
#sidebar li.active {
  background: #fff;
  border-left: 3px solid #4582ff;
  color: #4582ff;
}
#sidebar li.totalOrder {
  height: 1.53rem;
  line-height: 1.53rem;
  color: #999;
  background: #f2f2f2;
}
.flex-item div:nth-child(2) {
  font-size: 0.25rem;
}
</style>