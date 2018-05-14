<template>
  <div>
    <div id="top">
      <img src="../assets/login_logo@2x.png"  alt="">
    </div>
    <div id="middle">
      <group>
        <x-input type="number" v-model="maskValue1" title="+86" name="mobile" placeholder="请输入手机号" keyboard="number" is-type="china-mobile" @on-change="onBlur"></x-input>
      </group>
      <group>
        <x-input  type="number" placeholder="请输入验证码" class="weui-vcode" v-model="maskValue2" :disabled="check">
             <span slot="right" @click="debounceClick" class="checkNubmber" :class="{ colorC: iscolorC }" >
               {{word}}
              </span>
        </x-input>
      </group>
    </div>
    <div id="bottom">
      <x-button type="primary" action-type="button" :disabled="disable001" style="border-radius:99px;background:#4582ff;" @click.native="login()">登录</x-button>
    </div>
    <group>
      <alert v-model="show" @on-show="onShow" @on-hide="onHide">请输入手机号和验证码！</alert>
    </group>
  </div>
</template>

<script>
import { debounce } from "../common/util.js";
import { Group, XInput, XButton, Alert, XSwitch } from "vux";
export default {
  components: {
    XInput,
    Group,
    XButton,
    Alert,
    XSwitch
  },
  data() {
    return {
      maskValue1: "", //手机号值
      maskValue2: "", //验证码值
      show: false,
      word: "发送验证码",
      isOvertime: false, //是否超时
      msgCode: "",
      iscolorC:false,
      check:true,
      flag:false,
      disable001:false
    };
  },
  methods: {
    sendMessage() {
      if(!this.flag){
        return false;
      }else{
        if(this.maskValue1!=""){
        if (this.isOvertime) {
          return false;
        }
        let that = this,
          time = 60;
        var sendTimer = setInterval(function() {
          that.isOvertime = true;
          that.iscolorC=true;          
          time--;
          that.word = time + "s";
          if (time < 0) {
            that.isOvertime = false;
            clearInterval(sendTimer);
            that.word = "获取验证码";
          }
        }, 1000);
        var mp = this.$route.query.mp;
         this.$http
          .post("/api/user/msgCode", {
            phone: this.maskValue1,
            mp:mp
          })
          .then(res=>{
            this.$vux.toast.text('验证码已发送', 'middle')
          })
          .catch(function(err) {
            console.log(err);
          });
      }
      }
    },
     debounceClick: debounce(function(e) {
        this.sendMessage();
    }, 300),
    login() {
      if (this.maskValue1 == "" || this.maskValue2 == "") {
        this.show = true;
      } else {
        this.disable001=true;
        var state = this.$route.query.state;
        var mp = this.$route.query.mp;  
        this.$http
          .post("/api/user/login", {
            phone: this.maskValue1,
            msgCode: this.maskValue2,
            state: state,
            mp:mp
          })
          .then(function(res) {
            if (res.data.code == 0) {
              if(state && state.length>0){
                if(state.indexOf("?")!=-1){
                  location.href = state+"&mp="+mp;//跳到扫码充电页面，state里面包含addr
                }else{
                  location.href = state+"?mp="+mp; //跳到个人中心
                }
              }else{
                location.href="/api/user?mp="+mp;
              }
            } else {
              this.disable001=false;
              this.$msgbox(res.data.msg);
            }
          }.bind(this))
          .catch(function(err) {
            console.log(err);
          });
      }
    },
    onHide() {
      console.log("on hide");
    },
    onShow() {
      console.log("on show");
    },
    onBlur (val) {
      var reg = /^1[3|4|5|6|7|8|9][0-9]{9}$/;
      this.flag = reg.test(val); 
      if(this.flag){
        this.check=false;
      }
    }
  },
   created() {
  }
};
</script>

<style lang="less" rel="stylesheet/less">
#top {
  width: 10rem;
  height: 350/75rem;
  display: flex;
  align-items: center;
}
img {
  width: 144/75rem;
  height: 144/75rem;
  margin: 0 auto;
}
.btn {
  display: block;
  width: 500/75rem;
  height: 80/75rem;
  border-radius: 40/75rem;
  margin: 10px auto;
  background: #02bd84;
  color: #fff;
  margin-top: 150/75rem;
  font-size: 34/75rem;
}
.weui-cells {
  margin: 0 auto;
  border-left: 1px solid #e4e4e4;
  border-right: 1px solid #e4e4e4;
  border-radius: 10/75rem;
}
#bottom,
#middle {
  padding: 0 120/75rem;
}
#bottom {
  margin-top: 120/75rem;
}
.checkNubmber{
  color: #4582FF;
  font-size: 28/75rem;
}
.colorC{
  color:#999;
}
</style>
