---
title: 基于Vue和And-Design-Vue的动态导航树组件
date: 2019-09-27 20:55:03
tags: Tree  动态导航树 And-Design-Vue
categories: 前端
---

前端路由可以固定写死，但是在有用户权限的项目中，可能会动态生成，对于一般的 Vue 路由通常是含有一级导航和二级导航，其实也就是好几颗棵树。此时前端需要动态生成路由与导航。这里可以看 vue-router 的[动态添加路由](https://router.vuejs.org/zh/api/#router-addroutes)相关技术，最后要动态生成导航。此文章动态树导航基于 [vue](https://cn.vuejs.org/) 和 [and-design-vue](https://vue.ant.design/docs/vue/introduce/)。

###### 先看效果，再看代码。

![treemenu](https://s2.ax1x.com/2019/09/29/u87kJP.gif)

###### 导航数据

```js
let arrayTree = [
  {
    id: 1,
    pid: 0,
    title: "菜单A"
  },
  {
    id: 5,
    pid: 1,
    title: "菜单A-1"
  },
  {
    id: 6,
    pid: 1,
    title: "菜单A-2"
  },
  {
    id: 2,
    pid: 0,
    title: "菜单B"
  },
  {
    id: 7,
    pid: 2,
    title: "菜单B-1"
  },
  {
    id: 8,
    pid: 2,
    title: "菜单B-2"
  },
  {
    id: 3,
    pid: 0,
    title: "菜单C"
  },
  {
    id: 9,
    pid: 3,
    title: "菜单C-1"
  },
  {
    id: 10,
    pid: 9,
    title: "菜单C-1-1"
  },
  {
    id: 4,
    pid: 0,
    title: "菜单D"
  }
];
```

一般获取的导航信息如上所示，这其实是一棵树。

###### 生成嵌套结构的树

写一个方法将带有 id-pid 结构的数组转化成一个树。

```js
function array2Tree(array) {
  //map 用id和当前项建立映射
  let map = {};
  // reduce 方法里的参数顺序：初始值 当前项 索引 当前数组
  array.reduce((initVal, item) => {
    initVal[item.id] = item;
    return initVal;
  }, map);
  return array.reduce((arr, item) => {
    //先获取当前项的pid
    let pid = item.pid;
    //获取当前项的父节点
    let parent = map[pid];
    //父节点存在就往父节点的children上添加当前项
    if (parent) {
      parent.children ? parent.children.push(item) : (parent.children = [item]);
    } else {
      arr.push(item);
    }
    return arr;
  }, []);
}
```

###### 新建 TreeMenu.vue 组件

```vue
<template>
  <div class="menu-tree">
    <a-menu mode="inline" theme="dark">
      <template v-for="item in treeData">
        <a-menu-item v-if="!item.children" :key="item.id">
          <a-icon type="user" />
          <span>{{ item.title }}</span>
        </a-menu-item>
        <sub-menu v-else :menu-info="item" :key="item.id" />
      </template>
    </a-menu>
  </div>
</template>
<script>
import SubMenu from "./SubMenu";
import Utils from "@/utils";
export default {
  name: "MenuTree",
  data() {
    return {
      arrayTree: [
        {
          id: 1,
          pid: 0,
          title: "菜单A"
        },
        {
          id: 5,
          pid: 1,
          title: "菜单A-1"
        },
        {
          id: 6,
          pid: 1,
          title: "菜单A-2"
        },
        {
          id: 2,
          pid: 0,
          title: "菜单B"
        },
        {
          id: 7,
          pid: 2,
          title: "菜单B-1"
        },
        {
          id: 8,
          pid: 2,
          title: "菜单B-2"
        },
        {
          id: 3,
          pid: 0,
          title: "菜单C"
        },
        {
          id: 9,
          pid: 3,
          title: "菜单C-1"
        },
        {
          id: 10,
          pid: 9,
          title: "菜单C-1-1"
        },
        {
          id: 4,
          pid: 0,
          title: "菜单D"
        }
      ],
      treeData: []
    };
  },
  created() {
    this.treeData = Utils.array2Tree(this.arrayTree);
  },
  components: {
    SubMenu
  }
};
</script>
<style scoped></style>
```

###### 新建 SubMenu.vue 组件

```vue
<template functional>
  <a-sub-menu :key="props.menuInfo.key">
    <span slot="title">
      <a-icon type="home" />
      <span>{{ props.menuInfo.title }}</span>
    </span>
    <template v-for="item in props.menuInfo.children">
      <a-menu-item v-if="!item.children" :key="item.key">
        <a-icon type="user" />
        <span>{{ item.title }}</span>
      </a-menu-item>
      <sub-menu v-else :key="item.key" :menu-info="item" />
    </template>
  </a-sub-menu>
</template>
<script>
export default {
  name: "SubMenu",
  props: ["menuInfo"]
};
</script>
<style scoped></style>
```

在这个组件中利用了递归组件实现的 tree 结构，递归组件就是自己调用自己，必须还要有 name 属性和结束递归的条件，巧妙的利用这两点就可以自己定义 tree 组件。
