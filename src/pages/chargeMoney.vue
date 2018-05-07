<template>
  <div>
    <scroller use-pullup :pullup-config="pullupDefaultConfig" @on-pullup-loading="loadMore" use-pulldown :pulldown-config="pulldownDefaultConfig" @on-pulldown-loading="refresh" lock-x ref="scrollerBottom" height="-48">
      <div>
        <ul class="flex-group" v-for="item in list" :key="item.id">
          <li class="flex-item">
            <div style="font-size:0;"><span class="font-w" >{{item.bizType|returnType}}</span><span class="colorGray">{{item.remark}}</span></div>
            <div class="kwh">{{item.updated}}</div>
          </li>
          <li class="flex-item-right">
            <div class="colorBlue" :class="{colorRed:(item.bizType==1||item.bizType==5||item.bizType==7||item.bizType==8||item.bizType==9)}">{{item.bizType|returnMark}}{{item.money|returnFloat}}</div>
            <div class="kwh">余额&nbsp;{{item.balanceAfter|returnFloat}}</div>
          </li>
        </ul>
        <div v-if="list === undefined || list.length == 0" class="text-center">
          您还没有充值记录哦
        </div>
      </div>
    </scroller>
  </div>
</template>

<script>
import { Scroller } from "vux";
const pulldownDefaultConfig = {
  content: "",
  height: 40,
  autoRefresh: false,
  downContent: "",
  upContent: "",
  loadingContent: "...",
  clsPrefix: "xs-plugin-pulldown-"
};
const pullupDefaultConfig = {
  content: "",
  pullUpHeight: 60,
  height: 40,
  autoRefresh: false,
  downContent: "",
  upContent: "",
  loadingContent: "",
  clsPrefix: "xs-plugin-pullup-"
};
export default {
  components: {
    Scroller
  },
  data() {
    return {
      list: [],
      pullupDefaultConfig: pullupDefaultConfig,
      pulldownDefaultConfig: pulldownDefaultConfig,
      page: 1,
      blue:""
    };
  },
  mounted() {
    this.$nextTick(() => {
      this.$refs.scrollerBottom.reset({ top: 0 });
      document.querySelector(".xs-container").parentElement.style.height=window.innerHeight+"px"
    });
    this.loadMore();
  },
  methods: {
    fetchData(cb) {
       var mp = this.$route.query.mp;
      this.$http
        .post("/api/recharge/list", {
          page: this.page,
          limit: 10,
          mp:mp
        })
        .then(response => {
          this.$nextTick(() => {
            this.$refs.scrollerBottom.reset();
             document.querySelector(".xs-plugin-pullup-container").innerHTML =
              " ";
          });
          cb(response.data);
        });
    },
    refresh() {
      this.page=1;
      this.fetchData(data => {
        this.list = data.data.list;
        this.$refs.scrollerBottom.enablePullup();
        this.$refs.scrollerBottom.donePulldown();
        this.page=2;
      });
    },
    loadMore() {
      this.fetchData(data => {
        if (this.list.length >= 10) {
          this.$refs.scrollerBottom.disablePullup();
        }
        this.list = this.list.concat(data.data.list);
        this.page++;
        this.$refs.scrollerBottom.donePullup();
      });
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
    },
    sucessOrFail(value) {
      if (value) {
        return "充值成功";
      } 
    },
    returnType(value){
      switch (value) {
        case 1:
          return "充值"
          break;
       case 2:
          return "提现"
          break;
       case 3:
          return "退还"
          break;
       case 4:
          return "消费"
          break;
       case 5:
          return "调入"
          break;
       case 6:
          return "清零"
          break;
       case 7:
          return "返奖"
          break;
       case 8:
          return "销售收入"
          break;
       case 9:
          return "退来"
          break;
       case 10:
          return "借贷"
          break;
       case 11:
          return "调出"
          break;
        default:
          break;
      }
    },
     returnMark(value){
      switch (value) {
        case 1:
          return "+"
          break;
       case 2:
          return "-"
          break;
       case 3:
          return "-"
          break;
       case 4:
          return "-"
          break;
       case 5:
          return "+"
          break;
       case 6:
          return "-"
          break;
       case 7:
          return "+"
          break;
       case 8:
          return "+"
          break;
       case 9:
          return "+"
          break;
       case 10:
          return "-"
          break;
       case 11:
          return "-"
          break;
        default:
          break;
      }
    }
  },
  created() {
    // document.title="充值记录";
  }
};
</script>

<style lang="less" rel="stylesheet/less" scoped>
ul,
li {
  list-style: none;
}

.vux-demo {
  text-align: center;
  background: #fff;
}
.flex-two {
  display: flex;
  justify-content: space-between;
  padding: 0 30px;
  font-size: 0.37rem;
  height: 1.33rem;
  line-height: 1.33rem;
  background: #f2f2f2;
}
.flex-group {
  display: flex;
  justify-content: space-between;
  padding: 0 30px;
  font-size: 0.37rem;
  height: 1.86rem;
  border-bottom: 1px solid #eee;
}
.flex-item {
  padding: 10px 0;
}
.flex-item-right{
  padding: 10px 0;
  text-align: right;
}
.date {
  line-height: 1.86rem;
}
.kwh {
  font-size: 24/75rem;
  color: #999;
}
.colorBlue {
  font-size: 32/75rem;
}
.flex-middel {
  padding: 20px 0;
}
.font-w {
  font-weight: 600;
  font-size: 32/75rem;
}
.text-center{
  text-align: center;
  font-size: 18px;
   margin-top: 50px;
}
.colorGray{
  color:#999;
  width: 260/75rem;  
  font-size: 28/75rem;
  margin-left: 10px;
  overflow: hidden;
  text-overflow:ellipsis;
  white-space: nowrap;
  display: inline-block;
  vertical-align: -22%;
}
.colorRed{
  color: red;
  font-size: 32/75rem;
}
</style>
