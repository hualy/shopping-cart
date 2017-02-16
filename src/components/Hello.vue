<template>
  <form>
    {{message}}
    <slot>我是替补内容</slot>
    <table class="detial-wrap">
      <tr>
        <th>商品信息</th>
        <th>商品金额</th>
        <th>商品数量</th>
        <th>总金额</th>
        <th>编辑</th>
      </tr>
      <tr v-for="item in cartList">
        <td class="goods-detial-wrap">
          <div class="checkbox-wrap left">
            <span class="checkbox" v-bind:class="{'checked':item.checked}" @click="selectedProduct(item)" ></span>
          </div>
          <div class="goods-detial right">
            <div class="good-img left"><img :src="item.imgPic"/></div>
            <div class="good-text left">
              <div class="name">{{item.name}}</div>
              <dl class="gifts">
                <dt>赠送：</dt>
                <dd v-for="gift in item.gifts">{{gift.giftName}}</dd>
              </dl>
            </div>
          </div>
        </td>
        <td class="unitprice">{{item.price | round}}</td>
        <td class="buy-num">
          <div class="choosenum-handler">
            <i class="icon-minus" @click="changeMoney(item,-1)"></i>
            <span class="countbox">{{item.count}}</span>
            <!--<input type="text" v-model="item.count" disabled>-->
            <i class="icon-plus" @click="changeMoney(item,1)"></i>
          </div>
          <div class="stock onsell"></div>
        </td>
        <td class="amount">{{item.price * item.count | round}}</td>
        <!--<td class="icon icon-delete" @click="delFlag=true"></td>-->
        <td class="icon icon-delete" @click="delConfirm(item)"></td>

      </tr>
    </table>
    <footer class="checkout-wrap">
      <div class="total-check-wrap left">
        <div class="checkbox-wrap left"><span class="checkbox " :class="{'checked':checkAllFlag}" @click="checkAll(true)"></span></div>
        <div class="check-text">
          <span class="checked-all" @click="checkAll(true)">全选</span>
          <span class="unchecked-all" @click="checkAll(false)">取消全选</span>
        </div>
      </div>
      <div class="checkout right">
        <div class="total-money"><span class="name">总金额 :</span><span class="amount">{{ totalMoney | money}}元</span></div>
        <a href="#"><input type="submit" value="结账" class="danger"/></a>

      </div>
    </footer>
    <div class="md-modal modal-msg md-modal-transition" id="showModal" :class="{'md-show':delFlag}">
      <div class="md-modal-inner">
        <div class="md-top">
          <button class="md-close">关闭</button>
        </div>
        <div class="md-content">
          <div class="confirm-tips">
            <p id="cusLanInfo">你确定删除此订单信息吗？</p>
          </div>
          <div class="btn-wrap col-2">
            <button class="btn btn--m" id="btnModalConfirm">Yes</button>
            <button class="btn btn--m btn--red" id="btnModalCancel">No</button>
          </div>
        </div>
      </div>
    </div>
  </form>

</template>

<script>
  import Vue from 'vue'

export default {
  name: 'hello',
  props:['message'],
  data(){
    return{
        cartList:[],
        checkAllFlag:false,
        totalMoney:0,
        delFlag:false,
        curProduct:''
    }
  },
  mounted: function () {
    this.$nextTick(function () {
      this.getDataList();
    })
  },
  filters:{
    round:function (value) {
      return "$"+value.toFixed(2);
    }
  },
  methods: {
    getDataList: function () {
//      this.$http.get('static/resource/cart.json').then(function (response) {
//        this.cartList=response.body.result;
//      })
      this.$http.get('static/resource/cart.json').then(response=>{this.cartList=response.body.result;})
    },
    plus:function (item) {
      item.count++;
      this.$emit('change')
    },
    minus:function (item) {
      if(item.count>1){
        item.count--;
        this.$emit('change')
      }else{
        item.count=1;
      }
    },

    //不能的方法
    changeMoney:function (product,way) {
      if (way>0) {
        product.count++;
        this.$emit('change');

      }else{
          product.count--;
        this.$emit('change');
        if(product.count<1){
              product.count=1;
          }
      }
      this.getTotalMount();
    },
    getTotalMount:function () {
//      var sum=0;
//      this.cartList.forEach(function (value,index) {
//        sum+=value.count*value.price;
//      });
//      return sum;
      var _this=this;
      _this.totalMoney=0;
      this.cartList.forEach(function (item,index) {
        if(item.checked){
            _this.totalMoney+=item.price*item.count;
        }
      })
    },
    selectedProduct:function (item) {
      if(typeof item.checked =='undefined'){
//        Vue.set(item,"checked",true);
        this.$set(item,"checked",true);
      }else {
        item.checked = !item.checked;
      }
      this.getTotalMount();
    },
    checkAll:function (flag) {
      this.checkAllFlag =flag;
      var _this =this;
      this.cartList.forEach(function (item,index) {
        //如果第一次直接点击全选
        if(typeof item.checked =='undefined'){
//        Vue.set(item,"checked",true);
          _this.$set(item,"checked",_this.checkAllFlag);
        }else {
          item.checked = _this.checkAllFlag;
        }
      });
      this.getTotalMount();
    },
    delConfirm:function (item) {
      //保存对象才知道删除那个对象(用于模态框)
      this.curProduct=item;
//      alert("hello");
//      获取缩影
      var index = this.cartList.indexOf(item);
//      alert(index);
      this.cartList.splice(index,1);
    }

  }
}
Vue.filter('money',function (value) {
  return "￥"+value;
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .md-show {
    display: none;
  }
/*h1, h2 {*/
  /*font-weight: normal;*/
/*}*/

/*ul {*/
  /*list-style-type: none;*/
  /*padding: 0;*/
/*}*/

/*li {*/
  /*display: inline-block;*/
  /*margin: 0 10px;*/
/*}*/

/*a {*/
  /*color: #42b983;*/
/*}*/
</style>
