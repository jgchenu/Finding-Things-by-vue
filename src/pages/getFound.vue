<template>
  <div class="getFound">
    <div class="search-banner-container">
    <div class="search-banner">
      <select-place @click.native="showSelect=!showSelect"></select-place>
        <div class="search-for">
        <input type="text" placeholder="搜索物品和丢失地点" v-model="searchVal" @keyup.enter="search"><span class="icon-search iconfont" @click="search" ></span>
      </div>
    </div>
    <transition name="select-fade" mode="out-in">
    <select-detail v-show="showSelect" @selectSure="selectSure($event)"></select-detail>
    </transition>
  </div>
  <mt-loadmore :top-method="loadTop" ref="loadmore">
    <div class="search-lists" v-infinite-scroll="loadMoreFound"
  infinite-scroll-disabled="loading"
  infinite-scroll-distance="0" infinite-scroll-immediate-check="false">
      <search-list v-for="list in searchList" :key="list.sid" :list="list" @click.native="toDetail(list)"></search-list>
      <p class="no-resourse">{{noResourse}}</p>
    </div>
    
    </mt-loadmore>
    <edit-button @click.native="goRelease"></edit-button>
    
  </div>
</template>
<script>
import editButton from "@/components/comm/edit-button";
import searchList from "@/components/home/search-list";
import store from "@/vuex/store.js";
import selectPlace from "@/components/home/select-place";
import selectDetail from "@/components/home/select-detail";
import { Toast } from "mint-ui";
import { wxFn } from "@/api/tool.js";
import {loserUrl} from '@/api/variable.js'
export default {
  mounted() {
    wxFn.call(this);
    let lixun = sessionStorage.getItem("lixun");
    this.$ajax({
      method: "get",
      url: "/info",
      params: { type: 2 }
    })
      .then(res => {
        // console.log("getFound", res);
        this.searchList = res.data.data.list;
        this.nextUrl =
          (res.data.data.page &&
            res.data.data.page.url &&
            res.data.data.page.url.next) ||
          "";
        // console.log(this.nextUrl);
      })

      .catch(err => {
        // console.log(err);
        // console.log(err.response);
      });
  },
  data() {
    return {
      searchList: [],
      showSelect: false,
      searchVal: "",
      what: [],
      where: [],
      noResourse: "",
      nextUrl: "",
      loadAll: false,
      loading: false,
      isAt: true
    };
  },
  components: {
    searchList,
    editButton,
    selectPlace,
    selectDetail
  },
  methods: {
    loadMoreFound() {
      if (!this.isAt) return;
      this.loading = true;
      if (!this.nextUrl) {
        Toast({
          message: "已经到底了",
          iconClass: "iconfont icon-xiaolianchenggong",
          duration: 1000
        });
        this.loading = false;
        return;
      } else {
        let lixun = sessionStorage.getItem("lixun");
        this.$ajax({
          method: "get",
          url: "/info" + this.nextUrl
        })
          .then(res => {
            // console.log("getFound", res);
            this.searchList = this.searchList.concat(res.data.data.list);
            this.nextUrl =
              (res.data.data.page &&
                res.data.data.page.url &&
                res.data.data.page.url.next) ||
              "";
            this.loading = false;
          })
          .catch(err => {
            // console.log(err);
          });
      }
    },
    loadTop() {
      new Promise(resolve => {
        this.$refs.loadmore.onTopLoaded();
        resolve();
      }).then(() => {
        window.location.reload();
      });
    },
    toDetail(list) {
      this.$store.commit("setScrollTop", document.body.scrollTop);
      this.$store.commit(
        "changeUrl",
       loserUrl
      );
      this.$router.push({ path: "/listDetail", query: { id: list.id } });
      // console.log(this.$store.state.scrollTop);
    },
    goRelease() {
      this.$router.push({ path: "/release" });
    },
    selectSure(e) {
      this.showSelect = false;
      this.what = [];
      this.where = [];
      // console.log(e);
      if (e == []) return;
      if (!e[0] && e[1]) {
        e[1].forEach(element => {
          this.what.push(element.whattag);
        });
      } else if (!e[1] && e[0]) {
        e[0].forEach(element => {
          this.where.push(element.wheretag);
        });
      } else if (e[1] && e[0]) {
        e[0].forEach(element => {
          this.where.push(element.wheretag);
        });
        e[1].forEach(element => {
          this.what.push(element.whattag);
        });
      }
      this.search();
    },
    search() {
      let lixun = sessionStorage.getItem("lixun");
      this.$ajax({
        method: "post",
        url: "/info/search",
        data: {
          type: 2,
          what: this.what,
          where: this.where,
          query: this.searchVal
        }
      })
        .then(res => {
          // console.log("search", res);
          if (res.data.data.list.length == 0) {
            this.noResourse = "暂无此类资源，请换个词试试！";
            // console.log("暂无资源");
          } else {
            this.noResourse = "";
          }
          this.searchList = res.data.data.list;
          this.nextUrl =
            (res.data.data.page &&
              res.data.data.page.url &&
              res.data.data.page.url.next) ||
            "";
        })
        .catch(err => {
          // console.log(err);
        });
    }
  },
  beforeRouteEnter(to, from, next) {
    // console.log(document.body.scrollTop);
    document.body.scrollTop = 0;
    next(vm => {
      vm.loading = false;
      vm.isAt = true;
    });
  },
  beforeRouteLeave(to, from, next) {
    this.loading = false;
    this.isAt = false;
    next();
  },
  store
};
</script>

<style lang="scss" scoped>
@import "../assets/mixin.scss";
.getFound {
  position: relative;
  padding-bottom: 120px;
  .search-lists {
    padding-top: 130px;
    overflow: hidden;
  }
  .no-resourse {
    text-align: center;
    color: #bbbbbb;
    font-size: 26px;
  }
  .search-banner-container {
    z-index: 2;
    position: fixed;
    width: 100%;
    top: 0;
    .search-banner {
      height: 100px;
      background: #c54844;
      display: flex;
    }
    div.search-for {
      line-height: 100px;
      input {
        width: 460px;
        height: 60px;
        border-radius: 40px;
        outline: none;
        font-size: 30px;
        border: none;
        padding-left: 30px;
      }
      .icon-search {
        color: #c54844;
        font-size: 36px;
        position: relative;
        cursor: pointer;
        right: 56px;
        top: 4px;
        border-left: 1px solid #cccccc;
      }
      @include change-placeholder(#ccc,26px);
    }
  }
}
</style>

