<template>
  <div>
    <nav-header></nav-header>
    <nav-breadcrumb>
      <span>Goods</span>
    </nav-breadcrumb>
    <div class="accessory-result-page accessory-page">
      <div class="container">
        <div class="filter-nav">
          <span class="sortby">Sort by:</span>
          <a href="javascript:void(0)" class="default cur">Default</a>
          <a href="javascript:void(0)" class="price" @click="sortGoods">
            Price
            <img src="/static/jian.png" v-bind:class="{'jian':!sortFlag}" />
            <svg class="icon icon-arrow-short">
              <use xlink:href="#icon-arrow-short" />
            </svg>
          </a>
          <a
            href="javascript:void(0)"
            class="filterby stopPop"
            @click.stop="showFilterPop"
          >Filter by</a>
        </div>
        <div class="accessory-result">
          <!-- filter -->
          <div class="filter stopPop" id="filter" v-bind:class="{'filterby-show':filterBy}">
            <dl class="filter-price">
              <dt>Price:</dt>
              <dd>
                <a
                  href="javascript:void(0)"
                  @click="priceChecked=all"
                  v-bind:class="{'cur':priceChecked=='all'}"
                >All</a>
              </dd>
              <dd v-for="(price,index) in priceFilter" :key="price">
                <a
                  href="javascript:void(0)"
                  @click="setPriceFilter(index)"
                  v-bind:class="{'cur':priceChecked==index}"
                >{{price.startPrice}} - {{price.endPrice}}</a>
              </dd>
            </dl>
          </div>

          <!-- search result accessories list -->
          <div class="accessory-list-wrap">
            <div class="accessory-list col-4">
              <ul>
                <li v-for="item in goodsList" :key="item">
                  <div class="pic">
                    <a href="#">
                      <img v-lazy="'/static/'+item.productImage" alt />
                    </a>
                  </div>
                  <div class="main">
                    <div class="name">{{item.productName}}</div>
                    <div class="price">{{item.salePrice}}.00</div>
                    <div class="btn-area">
                      <a
                        href="javascript:;"
                        class="btn btn--m"
                        @click="addCart(item.productId)"
                      >加入购物车</a>
                    </div>
                  </div>
                </li>
              </ul>
              <div
                v-infinite-scroll="loadMore"
                infinite-scroll-disabled="busy"
                infinite-scroll-distance="20"
                class=".load-more"
              >
                <img src="./../assets/loading-spinning-bubbles.svg" v-show="loading" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <modal v-bind:mdShow="mdShow" v-on:close="closeModal">
      <p slot="message">请先登录，否则无法加入购物车</p>
      <div slot="btnGroup">
        <a class="btn btn--m" href="javascript:;" @click="mdShow = false">关闭</a>
      </div>
    </modal>
    <modal v-bind:mdShow="mdShowCart" v-on:close="closeModal">
      <p slot="message">
        <span>加入购物车成功</span>
      </p>
      <div slot="btnGroup">
        <a class="btn btn--m" href="javascript:;" @click="mdShowCart = false">继续购物</a>
        <router-link class="btn btn--m btn--red" href="javascript:;" to="/cart">查看购物车</router-link>
      </div>
    </modal>
    <div class="md-overlay" v-show="overLayFlag" @click.stop="closePop"></div>
    <nav-footer></nav-footer>
  </div>
</template>
<style scoped>
.jian {
  transform: rotate(180deg);
  transition: all 0.3s ease-out;
}
.btn:hover {
  background-color: #ffe5e6;
  transition: all 0.3s ease-out;
}
</style>
<script>
import Modal from "./../components/Modal";
import NavFooter from "./../components/NavFooter";
import NavHeader from "./../components/NavHeader";
import NavBreadcrumb from "./../components/NavBreadcrumb";
import "./../assets/css/base.css";
import "./../assets/css/product.css";
import axios from "axios";
export default {
  data() {
    return {
      goodsList: [],
      sortFlag: true,
      page: 1,
      pageSize: 8,
      mdShow: false,
      busy: true,
      mdShowCart:false,
      loading: false,
      priceFilter: [
        {
          startPrice: "0.00",
          endPrice: "100.00"
        },
        {
          startPrice: "100.00",
          endPrice: "500.00"
        },
        {
          startPrice: "500.00",
          endPrice: "1000.00"
        },
        {
          startPrice: "1000.00",
          endPrice: "5000.00"
        }
      ],
      priceChecked: "all",
      filterBy: false,
      overLayFlag: false
    };
  },
  components: {
    NavFooter,
    NavHeader,
    NavBreadcrumb,
    Modal
  },
  mounted: function() {
    this.getGoodsList();
  },
  methods: {
    getGoodsList(flag) {
      var param = {
        page: this.page,
        pageSize: this.pageSize,
        sort: this.sortFlag ? 1 : -1,
        priceLever: this.priceChecked
      };
      this.loading = true;
      //本地数据显示方法
      axios
        .get("/goods/list", {
          params: param
        })
        .then(response => {
          let res = response.data;
          this.loading = false;
          if (res.status == "0") {
            if (flag) {
              this.goodsList = this.goodsList.concat(res.result.list);
              if (res.result.count == 0) {
                this.busy = true;
              } else {
                this.busy = false;
              }
            } else {
              this.goodsList = res.result.list;
              this.busy = false;
            }
          } else {
            this.goodsList = [];
          }
        });
    },
    sortGoods() {
      //排序
      this.sortFlag = !this.sortFlag;
      this.page = 1;
      this.getGoodsList();
    },
    loadMore() {
      // 分页
      this.busy = true;
      setTimeout(() => {
        this.page++;
        this.getGoodsList(true);
      }, 500);
    },
    showFilterPop() {
      //浏览器变动价格弹窗方法
      this.filterBy = true;
      this.overLayFlag = true;
    },
    closePop() {
      //关闭浏览器变动价格弹窗方法
      this.filterBy = false;
      this.overLayFlag = false;
      this.mdShowCart = false;
    },
    setPriceFilter(index) {
      //点击价格关闭价格弹窗
      this.priceChecked = index;
      this.closePop();
      this.page = 1;
      this.getGoodsList();
    },
    addCart(productId) {
      axios
        .post("/goods/addCart", {
          productId: productId
        })
        .then(res => {
          var res = res.data;
          if (res.status == 0) {
            this.mdShowCart = true;
            // this.mdShowCart = true;
            // this.$store.commit("updateCartCount", 1);
          } else {
            // alert("msg:" + res.msg);
            this.mdShow = true;
          }
        });
    },
    closeModal() {
      this.mdShow = false;
      this.mdShowCart = false;
    },
  }
};
</script>
