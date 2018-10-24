<template>
  <div class="ebook">
    <TitleBar :nee="ifTitleAndMenuShow" />
    <div class="read-wrapper">
      <div id="read"></div>
      <div class="mask">
        <div class="left" @click="prevPage"></div>
        <div class="center" @click="toggleTitleMenu"></div>
        <div class="right" @click="nextPage"></div>
      </div>
    </div>
    <MenuBar :nee="ifTitleAndMenuShow" :fontSizeList="fontSizeList" :defaultFontSize="defaultFontSize" @setFontSize="setFontSize" :themeList="themeList" :defaultTheme="defaultTheme" @setTheme="setTheme" :bookAvailable="bookAvailable" @onProgressChange="onProgressChange" @jumpTo="jumpTo" :navigation="navigation" ref="menuBar" />
  </div>
</template>

<script type="text/ecmascript-6">
import Epub from "epubjs";
import TitleBar from "./components/TitleBar";
import MenuBar from "./components/MenuBar";
const DOWNLOAD_URL = "/2018_Book_AgileProcessesInSoftwareEngine.epub";

export default {
  name: "Ebook",
  data() {
    return {
      ifTitleAndMenuShow: false,
      fontSizeList: [
        { fontSize: 12 },
        { fontSize: 14 },
        { fontSize: 16 },
        { fontSize: 18 },
        { fontSize: 20 },
        { fontSize: 22 },
        { fontSize: 24 }
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: "default",
          style: {
            body: {
              color: "#000",
              background: "#fff"
            }
          }
        },
        {
          name: "eye",
          style: {
            body: {
              color: "#000",
              background: "#ceeaba"
            }
          }
        },
        {
          name: "night",
          style: {
            body: {
              color: "#fff",
              background: "#000"
            }
          }
        },
        {
          name: "gold",
          style: {
            body: {
              color: "#000",
              background: "rgb(241, 236, 226)"
            }
          }
        }
      ],
      defaultTheme: 0,
      // 图书是否处于可用状态
      bookAvailable: false,
      navigation: {}
    };
  },
  components: {
    TitleBar,
    MenuBar
  },
  methods: {
    // 根据链接跳转到指定地方
    jumpTo(href) {
      this.rendition.display(href);
      this.hideTitleAndMenu();
    },
    hideTitleAndMenu() {
      // 隐藏标题栏和菜单栏
      this.ifTitleAndMenuShow = false;
      // 隐藏菜单栏弹出的设置栏
      this.$refs.menuBar.hideSetting();
      // 隐藏目录
      this.$refs.menuBar.hideContent();
    },
    // 进度条变化 修改
    onProgressChange(progress) {
      const percentage = progress / 100;
      // console.log(progress);
      const location =
        percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
      this.rendition.display(location);
    },
    // 设置主题
    setTheme(index) {
      this.themes.select(this.themeList[index].name);
      this.defaultTheme = index;
    },
    registerTheme() {
      this.themeList.forEach(theme => {
        this.themes.register(theme.name, theme.style);
      });
    },
    // 设置字体
    setFontSize(fontSize) {
      this.defaultFontSize = fontSize;
      if (this.themes) {
        // fontSize 为数值，需要加上px显示
        this.themes.fontSize(fontSize + "px");
      }
    },
    // 显示隐藏上下菜单栏
    toggleTitleMenu() {
      this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow;
      if (!this.ifTitleAndMenuShow) {
        this.$refs.menuBar.hideSetting();
        // console.log(this.$refs.menuBar.ifSettinShow)
      }
    },
    prevPage() {
      // rendition.prev
      if (this.rendition) {
        this.rendition.prev();
      }
    },
    nextPage() {
      // rendition.next
      if (this.rendition) {
        this.rendition.next();
      }
    },
    // 电子书的解析和渲染
    showEpub() {
      // 生成book
      this.book = new Epub(DOWNLOAD_URL);
      // 生成Redition
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth,
        height: window.innerHeight
      });
      // 生成Redition.display 渲染电子书
      this.rendition.display();
      // 获取到theme对象
      this.themes = this.rendition.themes;
      // 设置默认字体
      this.setFontSize(this.defaultFontSize);
      // this.themes.register(name, styles)
      // this.themes.select(name)
      this.registerTheme(); // 注册主题
      // 设置默认样式
      this.setTheme(this.defaultTheme);
      // 获取location 对象
      // epub钩子函数
      this.book.ready
        .then(() => {
          this.navigation = this.book.navigation; 
          return this.book.locations.generate();
        })
        .then(() => {
          this.locations = this.book.locations;
          this.bookAvailable = true;
          // this.onProgressChange(50);
        });
    }
  },
  mounted() {
    this.showEpub();
  }
};
</script>

<style scoped lang='scss'>
@import "assets/styles/global";
.ebook {
  position: relative;

  .read-wrapper {
    .mask {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 100;
      display: flex;
      width: 100%;
      height: 100%;
      .left {
        flex: 0 0 px2rem(100);
      }
      .center {
        flex: 1;
      }
      .right {
        flex: 0 0 px2rem(100);
      }
    }
  }
}
</style>
