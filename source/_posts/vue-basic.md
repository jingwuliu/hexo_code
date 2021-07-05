---
title: vue basic
date: 2021-04-12 14:41:53
tags: [vue]
---

slot，也就是插槽，是Vue组件的一块HTML模板，该模板是否显示、如何显示由其父组件决定。插槽是一块模板，从模板种类角度来分，可以分为非插槽模板和插槽模板两类。
- 非插槽模板是html模板，像div、span、ul、table这些，它们的显示与否以及显示效果是由组件自身控制。
- 插槽模板是slot，它是一个空壳子，插槽显示的位置由子组件自身决定，slot写在组件template的什么位置，父组件传过来的模板就会显示在什么位置。

```
<template>
    <div class="father">
        <h3>父组件</h3>
        <child>
            <div class="temp1>
                <span>menu1</span>
                <span>menu2</span>
            </div>
        </child>
    </div>
</template>

<template>
    <div class="child">
        <h3>子组件</h3>
        <slot></slot>
    </div>
</template>
```
在这里，子组件的插槽被父组件下的html模板使用了。所以子组件会变成这样。
```
<div class="temp1>
    <span>menu1</span>
    <span>menu2</span>
</div>
```

## 具名插槽
匿名插槽没有name属性，插槽加了name属性就变成了具名插槽。它可以在一个组件中出现N次，出现在不同的位置。
```

<template>
  <div class="father">
    <h3>这里是父组件</h3>
    <child>
      <div class="tmpl" slot="up">
        <span>菜单1</span>
        <span>菜单2</span>
        <span>菜单3</span>
        <span>菜单4</span>
        <span>菜单5</span>
        <span>菜单6</span>
      </div>
      <div class="tmpl" slot="down">
        <span>菜单-1</span>
        <span>菜单-2</span>
        <span>菜单-3</span>
        <span>菜单-4</span>
        <span>菜单-5</span>
        <span>菜单-6</span>
      </div>
      <div class="tmpl">
        <span>菜单->1</span>
        <span>菜单->2</span>
        <span>菜单->3</span>
        <span>菜单->4</span>
        <span>菜单->5</span>
        <span>菜单->6</span>
      </div>
    </child>
  </div>
</template>

<template>
  <div class="child">
    // 具名插槽
    <slot name="up"></slot>
    <h3>子组件</h3>
    // 具名插槽
    <slot name="down"></slot>
    // 匿名插槽
    <slot></slot>
  </div>
</template>
```

## 作用域插槽/数据插槽
这样的插槽往往带有动态的数据，需要在slot上绑定数据，
父组件提供了样式，但没有提供数据，数据使用的子组件插槽自己绑定的数组。
```
<template>
    <div class="father">
        <h3>父组件</h3>
        <child>
            <template slot-scope="user">
                <div class="temp1">
                    <span v-for="item in user.data">{{ item }}</span>
                </div>
            </template>
        </child>

        <child>
            <template slot-scope="user">
                <ul>
                    <li v-for="item in user.data">{{ item }}</li>
                </ul>
            </template>
        </child>

        <child>
            <template slot-scope="user">
                {{ user.data }}
            </template>
        </child>
    </div>
</template>

<template>
    <div class="child">
        <h3>子组件</h3>
        <slot :data="data"></slot>
    </div>
</template>

export default {
    data: function() {
        return {
            data: ['zhangsan','lisi','wanwu','zhaoliu','tianqi','xiaoba']
        }
    }
}

```

## table中的插槽
在使用[ant design vue](https://www.antdv.com/components/table-cn)的table的时候，我们经常可以看见<slot>,<slot-scope>等字段，这里其实就用到了插槽。
![table-slot](table-slot.png)
slot="name"表示是在name这一列中插入"查看"和"删除操作，
slot-scope="text record"的话，record表示当前行的数据，可以通过传递record来获取当前行的所有数据。