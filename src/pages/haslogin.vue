<template>
  <div class="demo">
    <GoSelf :headerData="aboutData"></GoSelf>
    <div class="flex-start" style="border:1px solid #eee;">
      <span style="margin-right:10px;">
        <img src="../assets/chongdianz.png" alt="" class="chongdian">
      </span>
      <span>
        <div class="font-32 font-w">
          {{aboutData.name}}
        </div>
        <div class="font-26 tips" style="margin:5px 0">
          {{ifBusy}}
        </div>
        <div class="font-26" v-if="aboutData.pileType==1">
          · 直流桩
        </div>
        <div class="font-26" v-else>
          · 交流桩
        </div>
        <div class="font-26">
          · 最大功率 {{aboutData.power}}
        </div>
      </span>
    </div>
    <div class="flex-g">
      <div class="font-28">峰时</div>
      <div class="font-24 middle-wi">{{aboutData.peakTime}}</div>
      <div class="flex-item font-24">
        {{aboutData.peakPrice|returnFloat}}元/度
      </div>
    </div>
    <div class="flex-g">
      <div class="font-28">平时</div>
      <div class="font-24 middle-wi">{{aboutData.usuallyTime}}</div>
      <div class="flex-item font-24">
        {{aboutData.usuallyPrice|returnFloat}}元/度
      </div>
    </div>
    <div class="flex-g">
      <div class="font-28">谷时</div>
      <div class="font-24 middle-wi">{{aboutData.valleyTime}}</div>
      <div class="flex-item font-24">
        {{aboutData.valleyPrice|returnFloat}}元/度
      </div>
    </div>
    <div class="flex-twos" style="margin-bottom:10px;">
      <div class="flex-item ">
        <div class="font-28">服务费</div>
      </div>
      <div class="flex-item font-24">
        {{aboutData.service|returnFloat}}元/度
      </div>
    </div>
    <!-- 底部 -->
    <div class="flex-center">
      <div>
        <div class="circle" :class="{ blueBack: blueBack }" @click="debounceClick">
          <div :class="{status1:status1}">{{btText}}</div>
          <img src="../assets/chatou.png" alt="" class="chatou" v-if="aboutData.hasLogin==true && (aboutData.chargingCount==0 || stationId=='创新智慧港') && aboutData.status==0">
        </div>
        <div class="font-24" v-show="aboutData.hasLogin==true">余额
          <span>{{aboutData.available|returnFloat}}</span>
        </div>
        <div class="color-red" v-show="aboutData.hasLogin==true" @click="goRecharge()">立即充值></div>
      </div>
    </div>
    <div class="bottom2" v-if="aboutData.hasLogin==true && aboutData.chargingCount!=0">
      <div>
        您已有{{aboutData.chargingCount}}个正在充电的订单
      </div>
      <div @click="checkOrder()">
        查看
      </div>
    </div>

  </div>
</template>

<script>
import { debounce } from "../common/util.js";
import GoSelf from "@/components/goSelf";
export default {
  components: {
    GoSelf
  },
  data() {
    return {
      aboutData: {
        peakPrice: 0,
        usuallyPrice: 0,
        valleyPrice: 0,
        service: 0
      },
      blueBack: false,
      btText: "开始充电",
      ifBusy: "",
      status1: false,
      stationId: ""
    };
  },
  methods: {
    getApi() {
      var addr = this.$route.query.addr;
      var mp = this.$route.query.mp;
      this.$http
        .post("/api/charging/info", {
          addr: addr,
          mp: mp
        })
        .then(
          function(res) {
            if (res.data.code == 0) {
              this.aboutData = res.data.data;
              this.stationId = res.data.data.name.split("-")[0];
              //  业务逻辑
              if (!this.aboutData.tenantLogo) {
                this.aboutData.tenantLogo = "../static/recharge_logo.png";
              }
              switch (this.aboutData.status) {
                case 0:
                  this.ifBusy = "空闲";
                  window.intervalid2 = setTimeout(this.getApi, 1000);
                  break;
                case 1:
                  this.ifBusy = "已插枪"; //已经插枪
                  break;
                case 2:
                  this.ifBusy = "充电中";
                  break;
                case 3:
                  this.ifBusy = "禁用";
                  break;
                case 4:
                  this.ifBusy = "故障";
                  break;
                case 5:
                  this.ifBusy = "锁定";
                  break;
                case 6:
                  this.ifBusy = "未连接";
                  break;
              }
              //开始判断
              if (this.aboutData.hasAuthorize == false) {
                this.btText = "请先授权登陆";
                return false;
              } else {
                if (this.aboutData.status == 3) {
                  this.btText = "已停用";
                  return false;
                }
                if (this.aboutData.chargingCount != 0) {
                  //有订单，跳转到充电中
                  var flag = true;
                  if (this.aboutData.chargingAddr && this.aboutData.chargingAddr.indexOf(addr) > -1) {
                    this.$router.push({
                      name: "redCharging",
                      query: {
                        mp: mp,
                        addr: addr,
                        chargingCount: this.aboutData.chargingCount,
                        flag: flag
                      }
                    });
                  }
                  //临时测试
                  if (this.stationId == "创新智慧港") {
                    if (
                      this.aboutData.status == 1 ||
                      this.aboutData.status == 5 ||
                      this.aboutData.status == 0
                    ) {
                      if (this.aboutData.hasLogin == false) {
                        this.btText = "登录后充电";
                      } else {
                        this.btText = "开始充电"; //已经登录也插枪了
                        this.blueBack = true;
                        if (this.aboutData.status == 0) {
                          this.btText = "请插枪"; //已经登录却没插枪
                          this.status1 = true;
                        }
                      }
                    }
                  }
                } else {
                  //是否可用
                  if (
                    this.aboutData.status == 1 ||
                    this.aboutData.status == 5 ||
                    this.aboutData.status == 0
                  ) {
                    //只有这几个状态颜色是蓝色
                      this.blueBack = true;
                    if (this.aboutData.hasLogin == false) {
                      this.btText = "登录后充电";
                    } else {
                      this.btText = "开始充电"; //已经登录也插枪了
                      if (this.aboutData.status == 0) {
                        this.btText = "请插枪"; //已经登录却没插枪
                        this.status1 = true;
                      }
                    }
                  }
                }
              }
            } else {
              this.$msgbox(res.data.msg);
            }
          }.bind(this)
        )
        .catch(function(err) {
          console.log(err);
        });
    },
    gologin() {
      var addr = this.$route.query.addr;
      var mp = this.$route.query.mp;
      if (this.aboutData.chargingCount == 0 ||this.stationId == "创新智慧港") {
        if (
          this.aboutData.status == 1 ||
          this.aboutData.status == 5
        ) {
          if (this.aboutData.hasLogin == false) {
            location.href =
              "/api/login?state=http://test.gtcx-tech.com/api/scanCharge?addr=" +
              addr +
              "&mp=" +
              mp;
          } else {
            this.$http
              .post("api/charging/start", {
                addr: addr,
                mp: mp
              })
              .then(
                function(res) {
                  if (res.data.code == 0) {
                    var flag = true;
                    this.$router.push({
                      name: "redCharging",
                      query: {
                        mp: mp,
                        addr: addr,
                        flag: flag,
                        chargingCount: this.aboutData.chargingCount
                      }
                    });
                  } else {
                    this.$msgbox(res.data.msg);
                  }
                }.bind(this)
              )
              .catch(function(err) {
                console.log(err);
              });
          }
        }
      }
    },
    debounceClick: debounce(function(e) {
      this.gologin();
    }, 300),
    goRecharge() {
      var addr = this.$route.query.addr;
      var mp = this.$route.query.mp;
      var center = "self";
      this.$router.push({
        name: "charging",
        query: { mp: mp, center: center, addr: addr }
      });
    },
    checkOrder() {
      var mp = this.$route.query.mp;
      this.$router.push({
        name: "redCharging",
        query: { mp: mp }
      });
    }
  },
  created() {
    this.getApi();
  },
  beforeDestroy() {
    if (this.ifBusy == "空闲") {
      clearInterval(intervalid2);
    }
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
    }
  }
};
</script>

<style lang="less" rel="stylesheet/less">
ul,
li {
  list-style: none;
}
.demo {
  background: #f2f2f2;
}

.flex-twos {
  display: flex;
  justify-content: space-between;
  padding: 0 25px;
  font-size: 0.37rem;
  height: 1rem;
  line-height: 1rem;
  background: #fff;
}
.date {
  line-height: 1.86rem;
}
.kwh {
  font-size: 24/75rem;
  color: #999;
}
.flex-middel {
  padding: 20px 0;
}
.font-w {
  font-weight: 600;
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
  padding: 10px 25px;
}
.font-32 {
  font-size: 32/75rem;
}
.font-28 {
  font-size: 28/75rem;
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
  width: 80/75rem;
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

.bottom2 {
  position: absolute;
  bottom: 0;
  width: 100%;
  display: flex;
  justify-content: space-between;
  padding: 0 30px;
  font-size: 0.37rem;
  height: 1.33rem;
  line-height: 90/75rem;
  height: 90/75rem;
  background: #4582ff;
  color: #fff;
  box-sizing: border-box;
}
.color-red {
  font-size: 28/75rem;
  color: #e01a1a;
  margin: 5px 0;
}
.circle {
  width: 250/75rem;
  height: 250/75rem;
  line-height: 250/75rem;
  background: #d6d6d6;
  border-radius: 125/75rem;
  color: #fff;
  font-size: 32/75rem;
  font-weight: 600;
  margin: 20px 0;
  position: relative;
  text-align: center;
}
.chatou {
  position: absolute;
  left: 70/75rem;
  width: 108/75rem;
  animation: updown 1s infinite linear;
}
@keyframes updown {
  0% {
    top: 50px;
  }
  50% {
    top: 70px;
  }
  100% {
    top: 50px;
  }
}

.status1 {
  position: absolute;
  top: -30px;
  width: 100%;
}
.speci-w {
  height: auto;
  line-height: 0.7rem;
}
.speic-inner {
  display: inline-block;
  width: 70/75rem;
}
.speic-inner2 {
  display: inline-block;
  width: 78/75rem;
}
.flex-starts {
  display: flex;
  justify-content: flex-start;
}
.flex-g {
  display: flex;
  justify-content: space-between;
  padding: 2px 25px;
  font-size: 0.37rem;
  background: #fff;
}
.middle-wi {
  width: 6rem;
}
.blueBack {
  background: #5b9eff;
}
</style>
