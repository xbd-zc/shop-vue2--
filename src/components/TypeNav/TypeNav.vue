<template>
  <!-- 商品分类导航 -->
  <div class="type-nav">
<!--    <h1>{{categoryList}}</h1>-->
    <div class="container">
      <!--事件委派给父亲完成      -->
      <div  @mouseleave="leaveIndex" @mouseenter="enterIndex" >
        <h2 class="all">全部商品分类</h2>
        <!--过度动画        -->
        <transition name="sort">
          <!--三级联动        -->
          <div class="sort" v-show="showNav">
            <!--goSearch事件委派给父亲完成      -->
            <div class="all-sort-list2" @click="goSearch($event)">
              <div class="item" v-for="(c1,index) in categoryList"
                   :key="c1.categoryId"
                   :class="{cur:currentIndex===index}"
              >
                <h3 @mouseenter="changeIndex(index)">
                  <a :data-categoryname="c1.categoryName"
                     :data-category1Id="c1.categoryId">
                    {{c1.categoryName}}
                  </a>
                  <!--                <router-link to="/search">{{c1.categoryName}}</router-link>-->
                </h3>
                <!-- 二三级分类             -->
                <div class="item-list clearfix" :style="{display:currentIndex===index?'block':'none'}">
                  <div class="subitem" v-for="c2 in c1.categoryChild" :key="c2.categoryId">
                    <dl class="fore">
                      <dt>
                        <a :data-categoryname="c2.categoryName"
                           :data-category2Id="c2.categoryId" >
                          {{c2.categoryName}}
                        </a>
                        <!--                      <router-link to="/search">{{c2.categoryName}}</router-link>-->

                      </dt>
                      <dd>
                        <em v-for="c3 in c2.categoryChild" :key="c3.categoryId">
                          <a :data-categoryname="c3.categoryName"
                             :data-category3Id="c3.categoryId">
                            {{ c3.categoryName }}
                          </a>
                          <!--                        <router-link to="/search">{{c3.categoryName}}</router-link>-->
                        </em>
                      </dd>
                    </dl>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </transition>
      </div>
      <nav class="nav">
        <a >服装城</a>
        <a >美妆馆</a>
        <a >尚品汇超市</a>
        <a >全球购</a>
        <a >闪购</a>
        <a >团购</a>
        <a >有趣</a>
        <a >秒杀</a>
      </nav>

    </div>
  </div>

</template>

<script>
import {mapState} from 'vuex'
//全部引入lodash
// import _ from 'lodash'
//按需引入lodash
import throttle from 'lodash/throttle'

export default {
  name: "TypeNav",
  data(){
    return {
      currentIndex : -1,
      showNav:true
    }
  },
  //组件挂载完毕, 向服务器发请求
  mounted() {
    //通知vuex发请求, 获取数据, 存储于仓库中
    // this.$store.dispatch('categoryList');
    //如果不是home组件, 将nav隐藏
    if (this.$route.path !=='/home'){
      this.showNav= false
    }
  },
  computed:{
      ...mapState({
        //右侧需要是一个函数, 当使用这个计算属性时, 右侧函数就会立即执行一次
        //注入一个参数state, 这个state是大仓库的数据, 要指向对应小仓库
/*        categoryList:(state)=>{
          return state.store_home.categoryList;
        }*/
        //简写
        categoryList:state=>state.store_home.categoryList

      })
  },
  methods:{
    // changeIndex(index){
    //   console.log(index)
    //   this.currentIndex = index
    // },
    //节流throttle回调函数别用箭头函数,可能出现上下文this
    changeIndex:throttle(function (index) {
        this.currentIndex = index
    },50),
    leaveIndex(){
      this.currentIndex = -1;
      if (this.$route.path!=='/home'){
        this.showNav = false
      }
    },
    enterIndex(){
      if (this.$route.path!=='/home'){
        this.showNav = true
      }
    },
    goSearch(event){
      //第一个问题:把子节点当中a标签,我加上自定义属性data-categoryName,其余的子节点是没有的
      let elment = event.target;
      //获取到当前出发这个事件的节点【h3、a、dt、dl】,需要带有data-categoryname这样节点【一定是a标签】
      //节点有一个属性dataset属性,可以获取节点的自定义属性与属性值
      let {categoryname,category1id,category2id,category3id} = elment.dataset;
      //如果标签身上拥有categoryname一定是a标签
      if (categoryname){
        //整理路由跳转的参数
        let location = {name:'search'};
        let query = {categoryName:categoryname};
        //一级分类、二级分类、三级分类的a标签
        if (category1id){
          // console.log('@@',category1id);
          query.category1Id = category1id;
        }else if (category2id){
          // console.log('@@',category2id)
          query.category2Id = category2id;
        }else if (category3id) {
          // console.log(category3id)
          query.category3Id = category3id;
        }
        //整理完参数
        if (this.$route.params){//如果有params参数合并到参数中
          location.params = this.$route.params;
        }
        location.query = query
        this.$router.push(location);
      }
      this.showNav = false;//点击搜索隐藏三级菜单
    }
  }
}
</script>

<style scoped lang="less">
.type-nav {
  border-bottom: 2px solid #e1251b;

  .container {
    width: 1200px;
    margin: 0 auto;
    display: flex;
    position: relative;

    .all {
      width: 210px;
      height: 45px;
      background-color: #e1251b;
      line-height: 45px;
      text-align: center;
      color: #fff;
      font-size: 14px;
      font-weight: bold;
    }

    .nav {
      a {
        height: 45px;
        margin: 0 22px;
        line-height: 45px;
        font-size: 16px;
        color: #333;
      }
    }

    .sort {
      position: absolute;
      left: 0;
      top: 45px;
      width: 210px;
      height: 461px;
      position: absolute;
      background: #fafafa;
      z-index: 999;

    .all-sort-list2 {
      height: 450px;
      overflow: hidden;
      .item {
        h3 {
          line-height: 30px;
          font-size: 14px;
          font-weight: 400;
          overflow: hidden;
          padding: 0 20px;
          margin: 0;

          a {
            color: #333;
          }
        }

        .item-list {
          display: none;
          position: absolute;
          width: 734px;
          min-height: 460px;
          background: #f7f7f7;
          left: 210px;
          border: 1px solid #ddd;
          top: 0;
          z-index: 9999 !important;

        .subitem {
          float: left;
          width: 650px;
          padding: 0 4px 0 8px;

        dl {
          border-top: 1px solid #eee;
          padding: 6px 0;
          overflow: hidden;
          zoom: 1;

        &.fore {
           border-top: 0;
         }

        dt {
          float: left;
          width: 54px;
          line-height: 22px;
          text-align: right;
          padding: 3px 6px 0 0;
          font-weight: 700;
        }

        dd {
          float: left;
          width: 415px;
          padding: 3px 0 0;
          overflow: hidden;

        em {
          float: left;
          height: 14px;
          line-height: 14px;
          padding: 0 8px;
          margin-top: 5px;
          border-left: 1px solid #ccc;
        }
        }
        }
        }
        }


      }
      .cur{
        background: skyblue;
      }
    }
    }
    //过度动画的样式
    //开始状态
    .sort-enter{
      height: 0 ;
      //transform: rotate(0deg);/*旋转*/
    }
    //结束状态
    .sort-enter-to{
      height: 461px;
      //transform: rotate(360deg);/*旋转*/

    }
    //定义动画时间, 速率
    .sort-enter-active{
      transition: all 0.5s linear;
      overflow: hidden;
    }
  }
}

</style>
