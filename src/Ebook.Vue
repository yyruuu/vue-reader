<template>
  <div class="ebook">
    <!-- 注意组件的名称 -->
    <title-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"></title-bar>
    <menu-bar :ifTitleAndMenuShow="ifTitleAndMenuShow" :fontSizeList="fontSizeList" 
    :defaultFontSize="defaultFontSize" @setFontSize="setFontSize" :themeList="themeList" 
    :defaultTheme="defaultTheme" @setTheme="setTheme" :bookAvailable="bookAvailable" 
    @onProgressChange="onProgressChange" @jumpTo="jumpTo" :navigation="navigation" ref="menuBar"></menu-bar>
    <div class="read-wrapper">
      <div id="read"></div>
      <!-- 加入浮层，实现上一页和下一页 -->
      <div class="mask" >
        <div class="left" @click="prevPage"></div>
        <div class="center" @click="toggleTitleAndMenu"></div>
        <div class="right" @click="nextPage"></div>
      </div>
    </div>
  
  </div>
</template>

<script>
//引入titlebar和menubar组件
import TitleBar from '@/components/TitleBar'
import MenuBar from '@/components/MenuBar'
//将阅读器引擎导入到Ebook中
import Epub from 'epubjs'
//指向电子书的下载路径
const DOWNLOAD_URL = '/static/2018_Book_AgileProcessesInSoftwareEngine.epub';
global.epub = Epub;
/*eslint-disable space-before-function-paren */
export default {
  components:{
    //注册组件
    TitleBar,
    MenuBar
  },
  data(){
    return {
      ifTitleAndMenuShow: false,
      //用来表示所有的字号
      fontSizeList: [
        //字号加2
        {fontSize: 12},
        {fontSize: 14},
        {fontSize: 16},
        {fontSize: 18},
        {fontSize: 20},
        {fontSize: 22},
        {fontSize: 24}
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: 'default',
          style: {
            body:{
              //样式作用于body
              'color': '#000',
              'background': '#fff'
            }
          }
        },
        {
          name: 'eye',
          style: {
            body:{
              //样式作用于body
              'color': '#000',
              'background': '#ceeaba'
            }
          }
        },
        {
          name: 'night',
          style: {
            body:{
              //样式作用于body
              'color': '#fff',
              'background': '#000'
            }
          }
        },
        {
          name: 'gold',
          style: {
            body:{
              //样式作用于body
              'color': '#000',
              'background': 'rgb(241, 236, 226)'
            }
          }
        }

      ],
      //这里用的是数组的序号
      defaultTheme: 0,
      //图书是否处于可用状态
      bookAvailable: false
    }
  },
  methods:{
    
    jumpTo(href){
      //根据链接跳转到指定的位置 实现目录功能
      this.rendition.display(href);
      this.hideTitleAndMenun();//跳转的时候会自动隐藏标题栏和菜单栏
    },
    hideTitleAndMenun(){
      //隐藏标题和菜单栏
      this.ifTitleAndMenuShow = false;
      this.$refs.menuBar.hideSetting();
      this.$refs.menuBar.hideContent();
    },
    onProgressChange(progress){
      //进度条变化时调用 progress是一个数值(0-100)
      //将progress变成百分比
      const percentage = progress / 100;
      //去获取当前的页数 
      const location = percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
      this.rendition.display(location);

    },
    setTheme(index){
      this.themes.select(this.themeList[index].name);
      this.defaultTheme = index;
    },
    registerTheme(){
      //注册主题

      //遍历数组
      this.themeList.forEach(theme => {
        this.themes.register(theme.name, theme.style);
      });
    },
     setFontSize(fontSize){
       this.defaultFontSize = fontSize;
        //1. 获取Theme对象
        if(this.themes){
          this.themes.fontSize(fontSize + 'px');//这里的fontSize是数字型的 所以要在后面加上px变成string类型
        }
      },
    toggleTitleAndMenu(){
      //点击显示隐藏标题和菜单栏
      this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow;
      if(!this.ifTitleAndMenuShow){
        //如果标题和菜单栏是隐藏的，那么就调用MenuBar中的hideSetting方法
        //下面这个相当于DOM中选择元素，从而使用对应的方法
        this.$refs.menuBar.hideSetting();
      }
    },
    prevPage(){
      //翻到上一页 通过rendition.prev
      if(this.rendition){
        //如果rendition对象存在，则调用它的prev方法 
        this.rendition.prev();
      }
    },
    nextPage(){
      if(this.rendition){
        this.rendition.next();
      }
    },
    showEpub(){
      //电子书的解析和渲染
      //1.生成book对象 2.通过book对象声称Rendition对象，3.通过Rendition.display渲染电子书
      this.book = new Epub(DOWNLOAD_URL);
      //Rendition对象通过book对象的renderTo生成的 ('Dom对象的id',充满全屏 )
      this.rendition = this.book.renderTo('read', {
        width: window.innerWidth,
        height: window.innerHeight
      });
      //渲染
      this.rendition.display();

      //1. 获取Theme对象
      this.themes = this.rendition.themes;
      //设置默认字体
      this.setFontSize(this.defaultFontSize);
      //this.themes.register(name, styles)用来设置主题，参数：（名称，样式）
      //this.themes.select(name)通过主题的名称切换主题
      this.registerTheme();
      //设置默认样式
      this.setTheme(this.defaultTheme);
      //获取locations对象 实现进度条的功能 默认不会生成location 比较耗内存 so👇
      //通过epub的钩子函数来实现 ready--->电子书解析完成后会调用，返回一个promise对象
      //epubcfi 用来定位
      this.book.ready.then(()=>{
        //获取目录 toc下是目录的信息
       this.navigation = this.book.navigation;
       return this.book.locations.generate();
      }).then(result => {
        //将它存入本地的locations变量中
        this.locations = this.book.locations;
        //默认图书不可用 当locations对象生成以后，设置为true
        this.bookAvailable = true;
        
      });

      
    }
  },
  mounted(){
    //第四个生命周期函数，表示内存中的模版以放置到页面中，用户看到已渲染好的页面。执行了这个函数，表示Vue实例初始化已经完成了

    this.showEpub();
  }
  
}
</script>

<style lang="scss" scoped>
@import 'assets/styles/global';

.ebook{
  position: relative;
  
  .read-wrapper{
    .mask{
      //子绝父相
      position: absolute;
      top: 0;
      left: 0;
      display: flex;
      width: 100%;
      height: 100%;
      z-index: 100;
      // background-color: yellow;

      //采用弹性布局 让他们水平排列
      .left{
        //左边100px
        flex: 0 0 px2rem(100);
        
      }
      .center{
        //中间填充满
        flex: 1;
        

      }
      .right{
        flex: 0 0 px2rem(100);
        
      }

    }
  }

 
}
</style>
