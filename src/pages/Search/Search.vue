<template>
  <div>
    <TypeNav />
    <div class="main">
      <div class="py-container">
        <!--bread-->
        <div class="bread">
          <ul class="fl sui-breadcrumb">
            <li>
              <a href="#">全部结果</a>
            </li>
          </ul>
          <!--面包屑-->
          <ul class="fl sui-tag">
            <!--分类的面包屑-->
            <li class="with-x" v-if="searchParams.categoryName" @click="removeCateGoryName">
              {{searchParams.categoryName}}<i >x</i>
            </li>
            <!--关键字的面包屑-->
            <li class="with-x" v-if="searchParams.keyword"  @click="removeKeyword">
              {{searchParams.keyword}}<i>x</i>
            </li>
            <!--品牌的面包屑-->
            <li class="with-x" v-if="searchParams.trademark"  @click="removeTrademark">
              <!--split:将字符串转为数组, 注意不要使用v-show,split转undefiede会报错-->
              {{searchParams.trademark.split(":")[1]}}<i>x</i>
            </li>
            <!--平台售卖属性的面包屑-->
            <li class="with-x" v-for="(attr,index) in searchParams.props" :key="index"  @click="removeProps(index)">
              <!--split:将字符串转为数组, 注意不要使用v-show,split转undefiede会报错-->
              {{attr.split(":")[1]}}<i>x</i>
            </li>
          </ul>
        </div>

        <!--selector-->
        <SearchSelector @trademarkInfo="trademarkInfo" @attrInfo="attrInfo"/>

        <!--details-->
        <div class="details clearfix">
          <div class="sui-navbar">
            <div class="navbar-inner filter">
              <!--排序-->
              <ul class="sui-nav">
                <li :class="{active: isOrder1}"  @click="orderBy('1')">
                  <a>综合 <span v-show="isOrder1" :class="icon" ></span></a>
                </li>
                <li :class="{active: isOrder2}" @click="orderBy('2')">
                  <a>价格 <span v-show="isOrder2" :class="icon"></span></a>
                </li>
              </ul>
            </div>
          </div>
          <!-- 产品列表 -->
          <div class="goods-list">
            <ul class="yui3-g">
              <li class="yui3-u-1-5"  v-for="goods in goodsList" :key="goods.id">
                <div class="list-wrap">
                  <!-- 商品图片 -->
                  <div class="p-img">
                    <router-link :to="`/detail/${goods.id}`" >
<!--                      <img :src="goods.defaultImg" />-->
                      <img v-lazy="goods.defaultImg" />
                    </router-link>
                  </div>
                  <div class="price">
                    <strong>
                      <em>¥</em>
                      <i>{{goods.price}}</i>
                    </strong>
                  </div>
                  <div class="attr">
                    <a target="_blank" href="item.html" title="促销信息，下单即赠送三个月CIBN视频会员卡！【小米电视新品4A 58 火爆预约中】">
                      {{goods.title}}
                    </a>
                  </div>
                  <div class="commit">
                    <i class="command">已有<span>2000</span>人评价</i>
                  </div>
                  <div class="operate">
                    <a href="success-cart.html" target="_blank" class="sui-btn btn-bordered btn-danger">加入购物车</a>
                    <a href="javascript:void(0);" class="sui-btn btn-bordered">收藏</a>
                  </div>
                </div>
              </li>
            </ul>
          </div>
          <!--分页器组件-->
          <Pagination    :pageNo ='searchParams.pageNo'
                         :pageSize="searchParams.pageSize"
                         :total="getTotal"
                         :continues="5"
                         @getPageNo="getPageNo"/>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import SearchSelector from './SearchSelector/SearchSelector'
  import {mapGetters} from "vuex";
  import throttle from 'lodash/throttle'//按需引入节流
  export default {
    // eslint-disable-next-line vue/multi-word-component-names
    name: 'Search',
    data(){
      return {
        searchParams:{
          //一级分类的id
          category1Id: "",
          //二级分类id
          category2Id:"",
          //三级分类的id
          category3Id:"",
          //分类名字
          categoryName:"",
          //关键字
          keyword: "",
          //排序, 默认是综合降序
          order: "1:asc",
          //分页器用的:代表的是当前是第几页
          pageNo: 1,
          //代表的是每一个展示数据个数
          pageSize: 3,
          //平台售卖属性换作带的参数
          props: [],
          //品牌
          trademark: "",
        }
      }
    },
    components: {
      SearchSelector
    },
    computed:{
      //mapGetters写法: 传递数组, 因为getters计算没有划分模块
      ...mapGetters(['goodsList','getTotal']),
      isOrder1(){//控制综合排序的显示(1)隐藏(2)
        return this.searchParams.order.indexOf('1')!==-1
      },
      isOrder2(){//控制价格排序的显示(2)隐藏(1)
        return this.searchParams.order.indexOf('2')!==-1
      },
      icon(){//控制矢量图的上下, asc上, desc下
        let iconName = "iconfont ";
        if (this.searchParams.order.indexOf('asc')!==-1){
          iconName += "icon-xiangshang";
        }else if(this.searchParams.order.indexOf('desc')!==-1){
          iconName += "icon-xiangxia";
        }
        return iconName;
      }
    },
    mounted() { //仅执行一次
      //放到函数中
      // this.$store.dispatch("getSearchList",this.searchParams)
      //第一次需要合并到searchParams中, 让面包屑显示
      Object.assign(this.searchParams, this.$route.query, this.$route.params)
      this.getSearchData()
    },
    methods:{
      getSearchData(){
        //挂载完毕之前把query, params参数合并到searchParams中
        //复杂写法
        // this.searchParams. categorylId = this.Sroute.query.category1Id;
        // this.searchParams.category2Id = this.$route.query.category2Id;
        // this.searchParams.category3Id = this.Sroute.query.category3Id;
        // this.searchParams.categoryName = this.Sroute.query.categoryName;
        // this.searchParams. keyword = this. Sroute.params. keyword;
        //Object.assign: ES6新增的语法,合并对象
        //Object.assign(this.searchParams, this.$route.query, this.$route.params)
        //获取search数据
        this.$store.dispatch("getSearchList",this.searchParams)
      },
      removeCateGoryName(){
        //undefined请求不会带给服务器
        this.searchParams.categoryName = undefined;//清空对应名字
        // this.getSearchData();//重新发请求
        //地址栏也需要改,进行路由跳转,watch监听到有改变也会发请求
        this.$router.push({name :'search',params:this.$route.params})

      },
      removeKeyword(){
        this.searchParams.keyword = undefined;//清空对应名字
        //地址栏也需要改,进行路由跳转,watch监听到有改变也会发请求
        this.$router.push({name :'search',query:this.$route.query});
        //清空header组件的文本框
        this.$bus.$emit('clearKeyword');
      },
      trademarkInfo(trademark){
        //整理品牌名称字段参数,  ID: 品牌名称
        this.searchParams.trademark = `${trademark.tmId}:${trademark.tmName}`;
        // this.$router.push({name :'search',query:this.$route.query,params:this.$route.params})
        this.getSearchData()
      },
      removeTrademark(){//删除品牌信息
        this.searchParams.trademark = undefined;//要使用v-show,这里要 = 空字符串""
        //重新发起请求
        this.getSearchData();
      },
      attrInfo(attr,attrValue){
        //属性ID:属性值:属性名
        let props =  `${attr.attrId}:${attrValue}:${attr.attrName}`;
        //判断数组中是否存在属性, 存在再添加
        if (this.searchParams.props.indexOf(props)==-1){
          this.searchParams.props.push(props);
          this.getSearchData()
        }
      },
      removeProps(index){
        //再次整理参数,删除对应下标的数据
        this.searchParams.props.splice(index,1);
        this.getSearchData()//再发请求
      },
      orderBy:throttle(function (flag){//排序+节流
        //flag:1是综合, 2是价格
        let oldOrder = this.searchParams.order.split(':')[1];//查询之前的排序是asc还是desc
        if (this.searchParams.order.includes(flag)){//如果排序类型没有改变就取反
          oldOrder = oldOrder==='asc'? 'desc':'asc';//取反, asc就取desc
        }
        this.searchParams.order=`${flag}:${oldOrder}`;//如果改变了类型, 赋值给新的flag
        console.log(this.searchParams.order);
        this.getSearchData();//重新发起请求
      },1000),
      getPageNo(num){
        console.log('@@',num)
        this.searchParams.pageNo=num;
        this.getSearchData()
      }
    },
    watch:{
      $route:{//路由信息变化就重新发起请求
        handler(){
          // console.log(this.searchParams)
          //合并参数
          Object.assign(this.searchParams, this.$route.query, this.$route.params)
          this.getSearchData()
          //每次请求完毕后清空1,2,3级分类id
          this.searchParams.category1Id= undefined;
          this.searchParams.category2Id= undefined;
          this.searchParams.category3Id= undefined;
        }
      }
    }
  }
</script>

<style lang="less" scoped>
  .main {
    margin: 10px 0;

    .py-container {
      width: 1200px;
      margin: 0 auto;

      .bread {
        margin-bottom: 5px;
        overflow: hidden;

        .sui-breadcrumb {
          padding: 3px 15px;
          margin: 0;
          font-weight: 400;
          border-radius: 3px;
          float: left;

          li {
            display: inline-block;
            line-height: 18px;

            a {
              color: #666;
              text-decoration: none;

              &:hover {
                color: #4cb9fc;
              }
            }
          }
        }

        .sui-tag {
          margin-top: -5px;
          list-style: none;
          font-size: 0;
          line-height: 0;
          padding: 5px 0 0;
          margin-bottom: 18px;
          float: left;

          .with-x {
            font-size: 12px;
            margin: 0 5px 5px 0;
            display: inline-block;
            overflow: hidden;
            color: #000;
            background: #f7f7f7;
            padding: 0 7px;
            height: 20px;
            line-height: 20px;
            border: 1px solid #dedede;
            white-space: nowrap;
            transition: color 400ms;
            cursor: pointer;

            i {
              margin-left: 10px;
              cursor: pointer;
              font: 400 14px tahoma;
              display: inline-block;
              height: 100%;
              vertical-align: middle;
            }

            &:hover {
              color: #28a3ef;
            }
          }
        }
      }

      .details {
        margin-bottom: 5px;

        .sui-navbar {
          overflow: visible;
          margin-bottom: 0;

          .filter {
            min-height: 40px;
            padding-right: 20px;
            background: #fbfbfb;
            border: 1px solid #e2e2e2;
            padding-left: 0;
            border-radius: 0;
            box-shadow: 0 1px 4px rgba(0, 0, 0, 0.065);

            .sui-nav {
              position: relative;
              left: 0;
              display: block;
              float: left;
              margin: 0 10px 0 0;

              li {
                float: left;
                line-height: 18px;

                a {
                  display: block;
                  cursor: pointer;
                  padding: 11px 15px;
                  color: #777;
                  text-decoration: none;
                }

                &.active {
                  a {
                    background: #e1251b;
                    color: #fff;
                  }
                }
              }
            }
          }
        }

        .goods-list {
          margin: 20px 0;

          ul {
            display: flex;
            flex-wrap: wrap;

            li {
              height: 100%;
              width: 20%;
              margin-top: 10px;
              line-height: 28px;

              .list-wrap {
                .p-img {
                  padding-left: 15px;
                  width: 215px;
                  height: 255px;

                  a {
                    color: #666;

                    img {
                      max-width: 100%;
                      height: auto;
                      vertical-align: middle;
                    }
                  }
                }

                .price {
                  padding-left: 15px;
                  font-size: 18px;
                  color: #c81623;

                  strong {
                    font-weight: 700;

                    i {
                      margin-left: -5px;
                    }
                  }
                }

                .attr {
                  padding-left: 15px;
                  width: 85%;
                  overflow: hidden;
                  margin-bottom: 8px;
                  min-height: 38px;
                  cursor: pointer;
                  line-height: 1.8;
                  display: -webkit-box;
                  -webkit-box-orient: vertical;
                  -webkit-line-clamp: 2;

                  a {
                    color: #333;
                    text-decoration: none;
                  }
                }

                .commit {
                  padding-left: 15px;
                  height: 22px;
                  font-size: 13px;
                  color: #a7a7a7;

                  span {
                    font-weight: 700;
                    color: #646fb0;
                  }
                }

                .operate {
                  padding: 12px 15px;

                  .sui-btn {
                    display: inline-block;
                    padding: 2px 14px;
                    box-sizing: border-box;
                    margin-bottom: 0;
                    font-size: 12px;
                    line-height: 18px;
                    text-align: center;
                    vertical-align: middle;
                    cursor: pointer;
                    border-radius: 0;
                    background-color: transparent;
                    margin-right: 15px;
                  }

                  .btn-bordered {
                    min-width: 85px;
                    background-color: transparent;
                    border: 1px solid #8c8c8c;
                    color: #8c8c8c;

                    &:hover {
                      border: 1px solid #666;
                      color: #fff !important;
                      background-color: #666;
                      text-decoration: none;
                    }
                  }

                  .btn-danger {
                    border: 1px solid #e1251b;
                    color: #e1251b;

                    &:hover {
                      border: 1px solid #e1251b;
                      background-color: #e1251b;
                      color: white !important;
                      text-decoration: none;
                    }
                  }
                }
              }
            }
          }
        }

        .page {
          width: 733px;
          height: 66px;
          overflow: hidden;
          float: right;

          .sui-pagination {
            margin: 18px 0;

            ul {
              margin-left: 0;
              margin-bottom: 0;
              vertical-align: middle;
              width: 490px;
              float: left;

              li {
                line-height: 18px;
                display: inline-block;

                a {
                  position: relative;
                  float: left;
                  line-height: 18px;
                  text-decoration: none;
                  background-color: #fff;
                  border: 1px solid #e0e9ee;
                  margin-left: -1px;
                  font-size: 14px;
                  padding: 9px 18px;
                  color: #333;
                }

                &.active {
                  a {
                    background-color: #fff;
                    color: #e1251b;
                    border-color: #fff;
                    cursor: default;
                  }
                }

                &.prev {
                  a {
                    background-color: #fafafa;
                  }
                }

                &.disabled {
                  a {
                    color: #999;
                    cursor: default;
                  }
                }

                &.dotted {
                  span {
                    margin-left: -1px;
                    position: relative;
                    float: left;
                    line-height: 18px;
                    text-decoration: none;
                    background-color: #fff;
                    font-size: 14px;
                    border: 0;
                    padding: 9px 18px;
                    color: #333;
                  }
                }

                &.next {
                  a {
                    background-color: #fafafa;
                  }
                }
              }
            }

            div {
              color: #333;
              font-size: 14px;
              float: right;
              width: 241px;
            }
          }
        }
      }
    }
  }
</style>
