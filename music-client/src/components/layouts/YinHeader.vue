<template>
  <div class='yin-header'>
    <!--图标-->
    <div class='header-logo' @click='goPage()'>
      <yin-icon :icon="iconList.ERJI"></yin-icon>
      <span>{{ musicName }}</span>
    </div>
    <ul class='navbar'>
      <li :class='{active: item.name === activeNavName}' v-for='item in headerNavList' :key='item.path' @click='goPage(item.path, item.name)'>
        {{ item.name }}
      </li>
      <!--搜索框-->
      <li class='header-search'> 
        <input type='text' placeholder='搜索音乐' @keyup.enter='goSearch()' v-model='keywords'>
      </li>
    </ul>
    <!--设置-->
    <ul class='header-right sign' v-if='!token'>
      <li :class='{active: item.name === activeNavName}' v-for='item in signList' :key='item.type' @click='goPage(item.path, item.name)'>
        {{ item.name }}
      </li>
    </ul>
    <el-dropdown class='header-right' v-if='token' trigger="click">
      <div class='user'>
        <img :src='attachImageUrl(userPic)' alt=''>
      </div>
      <template #dropdown>
        <el-dropdown-menu>
          <el-dropdown-item v-for='(item, index) in menuList' :key='index' @click.stop='goMenuList(item.path)'>{{ item.name }}</el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import YinIcon from './YinIcon'
import mixin from '@/mixins'
import {
  HEADERNAVLIST,
  SIGNLIST,
  MENULIST,
  ICON,
  MUSICNAME,
  HOME,
  SEARCH,
  SIGN_OUT,
  NAV_NAME,
} from '@/enums'

export default {
  name: 'YinHeader',
  mixins: [mixin],
  components: {
    YinIcon
  },
  data () {
    return {
      musicName: MUSICNAME,
      headerNavList: HEADERNAVLIST, // 左侧导航栏
      signList: SIGNLIST, // 右侧导航栏
      menuList: MENULIST, // 用户下拉菜单项
      keywords: '',
      iconList: {
        ERJI: ICON.ERJI
      }
    }
  },
  computed: {
    ...mapGetters([
      'activeNavName',
      'userPic'
    ])
  },
  methods: {
    goPage (path, name) {
      if (!path && !name) {
        this.changeIndex(NAV_NAME.HOME)
        this.routerManager(HOME, { path: HOME })
      } else {
        this.changeIndex(name)
        this.routerManager(path, { path })
      }
    },
    goMenuList (path) {
      if (path === 0) this.$store.commit('setIsCollection', false)

      if (path) {
        this.routerManager(path, { path })
      } else {
        this.$store.commit('setToken', false)
        this.routerManager(SIGN_OUT, { path: SIGN_OUT })
      }
    },
    goSearch () {
      if (this.keywords !== '') {
        this.$store.commit('setSearchWord', this.keywords)
        this.routerManager(SEARCH, { path: SEARCH, query: { keywords: this.keywords } })
      } else {
        this.$notify({
          title: '搜索内容不能为空',
          type: 'error'
        })
      }
    }
  }
}

</script>

<style lang='scss' scoped>
@import "@/assets/css/var.scss";
@import "@/assets/css/global.scss";

/*头部*/
.yin-header {
  position: fixed;
  @include layout;
  width: 100%;
  height: $header-height;
  line-height: $header-height;
  padding: $header-padding;
  margin: $header-margin;
  background-color: $theme-header-color;
  @include box-shadow($box-shadow);
  box-sizing: border-box;
  z-index: 100;
  display: flex;
  white-space: nowrap;
  flex-wrap: nowrap;
}

/*左侧LOGO*/
.header-logo {
  font-size: $font-size-logo;
  font-weight: bold;
  margin: 0 15px;
  cursor: pointer;
  .icon {
    @include icon(($header-height / 3) * 2, $color-black);
    vertical-align: middle;
    margin-right: 1rem;
  }
}

/*navbar*/
.navbar {
  display: flex;
  white-space: nowrap;
  flex: 1;
  /*搜索输入框*/
  .header-search {
    width: 100%;
    input {
      height: $header-search-height;
      min-width: $header-search-width;
      border-radius: $header-search-radius;
      border: 0;
      font-size: $font-size-default;
      text-indent: 10px;
      background-color: $color-light-grey;
      &:focus {
        outline: none;
      }
    }
  }
}

.navbar,
.header-right.sign {
  li {
    margin: $header-nav-margin;
    padding: $header-nav-padding;
    font-size: $font-size-header;
    color: $color-grey;
    cursor: pointer;
  }
}
/*用户*/
.header-right {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  justify-content: flex-start;

  .user {
    overflow: hidden;
    width: $header-user-width;
    height: $header-user-width;
    border-radius: $header-user-radius;
    margin-right: $header-user-margin;
    cursor: pointer;

    img {
      width: 100%;
    }
  }
  .setting {
    font-size: 30px;
    cursor: pointer;
  }
}

.show {
  display: block;
}

.active {
  color: $theme-color !important;
  border-bottom: 5px solid $theme-color !important;
}

</style>
