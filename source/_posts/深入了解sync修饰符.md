---
title: 深入了解sync修饰符
tags: vue
categories: 随笔
keywords: sync修饰符
description: vue中用sync修饰符实现prop数据的双向绑定
top_img: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-f3d09f2930406c4c186d8fcf4e4d612d_1440w.jpg
cover: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-f3d09f2930406c4c186d8fcf4e4d612d_1440w.jpg
abbrlink: a64c000c
date: 2022-03-17 18:41:03
---

# 深入理解 sync 修饰符

🤗vue 修饰符 aync 功能：{% note success no-icon %} 当一个子组件改变了一个 prop 的值时，这个变化也会同步到父组件中。 {% endnote %}

一个组件上只能定义一个 v-model，如果其他 prop 也要实现双向绑定怎么办？简单的方法就是子组件发送一个事件，父组件监听该事件然后更新 prop 数据。

子组件：

```
// info.vue组件定义了一个value 属性， 和一个valueChanged事件
<template>
    <div>
        <input @input="onInput" :value="value"/>
    </div>
</template>

<script>
export default {
    props: {
        value: {
            type: String
        }
    },
    methods: {
        onInput(e) {
            this.$emit("valueChanged", e.target.value)
        }
    }
}
</script>
```

父组件：

```
<template>
    <info :value="myValue" @valueChanged="e => myValue = e"></info>
</template>

<script>
    inport info from './info.vue';
    export default {
        components: {
            info,
        },
        data() {
            return {
                myValue: 1234,
            }
        },
    }
</script>
```

用 sync 修饰符简化上述代码：

用法 1：v-bind:prop.sync="propvalue"

父组件：

```
// info.vue组件
...
methods: {
    onInput(e) {
        this.$emit("update:value", e.target.value)
    }
}
```

子组件：

```
// index.vue组件
<info :value.sync="myValue"></info>
```

用法 2：v-bind.sync="obj"

如果一个组件的多个 prop 都要实现双向绑定，根据上面学到的知识，只需要每个 prop 加 sync 修饰符

```
<info :a.sync="value1" :b.sync="value2" :c.sync="value2" :d.sync="value2"></info>
```

这样写太麻烦，vue 提供了一种更简便的方法， v-bind.sync = "对象"

```
<info v-bind.sync="obj"></info>
...
<script>
..
data() {
    obj: {a: '', b: '', c: '', d: ''}
}
..
</script>
```

注意：

{% note danger no-icon %} 带有.sync 修饰符的 v-bind 不能喝表达式一起使用（例如 v-bind:title.sync = "doc.title + '!'"是无效的）。取而代之的是，你只能你想要绑定的属性名。{% endnote %}
