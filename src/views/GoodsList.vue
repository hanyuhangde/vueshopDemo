<template>
    <div>
      <nav-header></nav-header>
      <nav-bread>
        <span>Goods</span>
      </nav-bread>
      <div class="accessory-result-page accessory-page">
        <div class="container">
          <div class="filter-nav">
            <span class="sortby">Sort by:</span>
            <a href="javascript:void(0)" class="default cur">Default</a>
            <a href="javascript:void(0)" @click="sortGoods" class="price">
              Price
              <img src="./../../static/up.png" class="icon icon-arrow-short" v-bind:class="{'sort-up':sortFlag}">
            </a>
            <a href="javascript:void(0)" class="filterby stopPop" @click="showFilterPop">Filter by</a>
          </div>
          <div class="accessory-result">
            <!-- filter -->
            <div class="filter stopPop" id="filter" :class="{'filterby-show':filterBy}">
              <dl class="filter-price">
                <dt>Price:</dt>
                <dd><a href="javascript:void(0)" :class="{'cur':priceChecked=='all'}" @click="setPriceFilter('all')">All</a></dd>
                <dd v-for="(price,index) in priceFilter" >
                  <a href="javascript:void(0)" @click="setPriceFilter(index)" :class="{'cur':priceChecked==index}">{{price.startPrice}}--{{price.endPrice}}</a>
                </dd>
              </dl>
            </div>

            <!-- search result accessories list -->
            <div class="accessory-list-wrap">
              <div class="accessory-list col-4">
                <ul>
                  <li v-for="item in goodsList">
                    <div class="pic">
                      <a href="#"><img v-lazy="'/static/'+item.productImage" alt=""></a>
                    </div>
                    <div class="main">
                      <div class="name">{{item.productName}}</div>
                      <div class="price">{{item.salePrice}}</div>
                      <div class="btn-area">
                        <a href="javascript:;" class="btn btn--m" @click="addCart(item.productId)">加入购物车</a>
                      </div>
                    </div>
                  </li>
                </ul>
                <div class="load-more" v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="30">
                     <img src="./../assets/svg/loading-spinning-bubbles.svg" v-show="loading">
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="md-overlay" v-show="overLayFlag" @click="closePop"></div>
      <modal :mdShow="mdShow" v-on:close="closeModal">
        <p slot="message">
          请先登录，否则无法加入购物车
        </p>
        <div slot="btnGroup">
          <a class="btn btn--m" href="javascript:;" @click="mdShow=false">关闭</a>
        </div>
      </modal>
      <modal :mdShow="mdShowCart" @close="closeModal">
        <p slot="message">
          <img src="./../../static/up.png" class="icon-status-ok">
          <span>加入购物车成功！</span>
        </p>
        <div slot="btnGroup">
          <a class="btn btn--m" href="javascript:;" @click="mdShowCart=false">继续购物</a>
          <router-link class="btn btn--m" href="javascript:;" to="/cart">查看购物车</router-link>
        </div>
      </modal>
      <nav-footer></nav-footer>
    </div>
</template>

<script>
  import './../assets/css/base.css'
  import './../assets/css/product.css'
  import NavHeader from './../components/Header'
  import NavFooter from './../components/Footer'
  import NavBread from './../components/Bread'
  import Modal from './../components/Modal'
  import axios from 'axios'
    export default {
       data(){
         return{
           goodsList:{},
           sortFlag:true,
           page:1,
           pageSize:8,
           busy:true,
           loading:false,
           mdShow:false,
           mdShowCart:false,
           priceFilter:[
             {
               startPrice:'0.00',
               endPrice:'100.00'
             },
             {
               startPrice:'100.00',
               endPrice:'500.00'
             },
             {
               startPrice:'500.00',
               endPrice:'1000.00'
             },
             {
               startPrice:'1000.00',
               endPrice:'5000.00'
             }
             ],
           priceChecked:'all',
           filterBy:false,
           overLayFlag:false

         }
       },
      components:{NavHeader,NavFooter,NavBread,Modal},
      mounted:function () {
         this.getGoodsList();
      },
      methods:{
         getGoodsList(flag){
           var param = {
             page:this.page,
             pageSize:this.pageSize,
             sort:this.sortFlag?1:-1,
             priceLevel:this.priceChecked
           };
           this.loading = true;
           axios.get("/goods/list",{
             params:param
           }).then((response)=>{
             let res = response.data;
             this.loading = false;
             if (res.status == "0"){
               if (flag){
               this.goodsList = this.goodsList.concat(res.result.list);
               if (res.result.count == 0){
                 this.busy = true;
               }else{
                 this.busy = false;
               }
               }else{
                 this.goodsList = res.result.list;
                 this.busy = false;
               }
           } else{
               this.goodsList = [ ];
             }
           });
         },
        sortGoods(){
          this.sortFlag = !this.sortFlag;
          this.page = 1;
          this.getGoodsList();
        },
        setPriceFilter(index){
          console.log(111);
          this.priceChecked = index;
          this.page = 1;
          this.getGoodsList();
          this.closePop();
        },
        addCart(productId){
          axios.post('/goods/addCart',{
            productId:productId
          }).then((res)=>{
            var res = res.data;
            if (res.status==0){
              // alert("加入成功")
              this.mdShowCart=true;
              this.$store.commit("updateCartCount",1);
            }else {
              this.mdShow = true;
            }
          });
        },
        closeModal(){
          this.mdShowCart = false;
        },
        loadMore(){
          this.busy = true;
          setTimeout(() => {
            this.page++;
            this.getGoodsList(true);
          }, 500);
        },
        showFilterPop(){
          this.filterBy = true;
          this.overLayFlag = true;
        },
        closePop(){
           this.filterBy = false;
           this.overLayFlag = false;
        }
      }
    }
</script>

<style scoped>
.load-more{
  height: 100px;
  line-height: 100px;
  text-align: center;
}
  .sort-up{
    transform: rotate(180deg);
    transition: all 0.3s ease-out;
  }
  .icon-arrow-short{
    transition: all 0.3s ease-out;
  }
</style>

