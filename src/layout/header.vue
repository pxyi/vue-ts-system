<template>
  <div class="header-container">

    <div class="nav-fold" @click="navFolded">
      <i :class="[ isCollapse ? 'el-icon-s-unfold' : 'el-icon-s-fold' ]"></i>
    </div>

    <div class="router-tabs">
      <el-tabs @tab-remove="closeTab" :value="$route.path" @tab-click="tabClick">
        <el-tab-pane v-for="i in routeTabs" :key="i.path" :label="i.meta.keepAlive" :name="i.path" :closable="routeTabs.length > 1" />
      </el-tabs>
    </div>

    <div class="default-size">
      <el-tooltip content="布局大小" placement="bottom">
        <el-dropdown @command="defautlSizeChange" trigger="click">
            <i class="cus-icon">&#xe608;</i>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item v-for="s in sizeList" :key="s.v" :command="s.v"  :disabled="size === s.v">{{ s.t }}</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </el-tooltip>
    </div>

    <div class="header-user">
      <el-dropdown @command="commandList.get($event)()">
        <div class="el-dropdown-link">
          <el-avatar :size="28" :src="avatar" />
          <span class="user-name">用户名</span>
          <i class="el-icon-arrow-down"></i>
        </div>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item icon="el-icon-user" disabled>个人中心</el-dropdown-item>
          <el-dropdown-item icon="el-icon-setting" disabled>主题设置</el-dropdown-item>
          <el-dropdown-item divided command="logout">退出登录</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
  </div>
</template>
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import RouteStore from '@/store/modules/route';

@Component({
  name: 'lay-header'
})
export default class extends Vue {

  private isCollapse = false;
  private navFolded() {
    this.isCollapse = !this.isCollapse;
    this.$bus.$emit('nav-is-collapse', this.isCollapse)
  }

  private size = this.$storage.get('size') || 'default';
  private sizeList = [{v: 'default', t: '默认'}, {v: 'medium', t: '中等'}, {v: 'small', t: '小型'}, {v: 'mini', t: '超小型'}]
  private defautlSizeChange(size: string) {
    this.$storage.set('size', size);
    location.reload();
  }

  get routeTabs() { return RouteStore.routeTabs }

  private closeTab(path: string) {
    /* ------------ 找到 keep-alive 暴力删除其 Cache */
    try {
      let vueComponent: any = this.$parent.$parent.$children[1].$vnode.componentInstance?.$children[0].$vnode.parent?.componentInstance;
      let cache = vueComponent.cache;
      Object.keys(cache).map((key: string) => {
        key.includes(path) && delete cache[key];
      })
    } catch (error) {
      console.warn('清除 keep-alive 缓存失败');
    }
    RouteStore.REMOVE_ROUTE_TAB(path);
    let toPath = this.routeTabs[this.routeTabs.length - 1].path;
    toPath !== this.$route.path && this.$router.push(toPath);
  }

  private tabClick(el: any) {
    this.$route.path !== el.name && this.$router.push(el.name);
  }

  private commandList = new Map([
    ['logout', () => this.$router.push('/login')]
  ]);
  
  private avatar = require('@/assets/default-boy.png');
}
</script>
<style lang="less" scoped>
.header-container {
  height: 60px;
  display: flex;
  .nav-fold {
    margin-right: 20px;
    font-size: 20px;
    cursor: pointer;
    &:hover {
      color: #409EFF;
    }
    &:active {
      transform:scale(.96);
    }
  }
}
.router-tabs {
  float: left;
  flex: 1
}
.header-user {
  flex: 0 0 auto;
  cursor: pointer;
  margin-left: 10px;
}
::v-deep .el-tabs__header {
  margin-top: -2px;
}
::v-deep .el-tabs__header {
  margin-bottom: 0;
}
::v-deep .el-tabs__nav-wrap::after {
  display: none;
}
::v-deep .el-tabs__nav-next, .el-tabs__nav-prev {
  line-height: 45px;
  font-size: 15px;
  cursor: pointer;
}
::v-deep .el-avatar {
  vertical-align: middle;
  margin-right: 8px;
}
.user-name {
  margin-right: 3px;
}
.default-size {
  padding: 0 12px;
  &:hover {
    background: #f9f9f9;
  }
  i {
    cursor: pointer;
    font-size: 22px;
    font-family: "cusfont" !important;
    font-style: normal;
  }
}
</style>