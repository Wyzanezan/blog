---
title: Vue中插槽的使用
date: 2020-05-16 18:06:52
tags:
categories: Frontend
---

今天介绍下 Vue 中插槽的使用。

<!--more-->

## 基本使用

使用 slot 标签表示插槽，在vue中插槽是内容分发的出口。

下面看一个简单的例子：

CSlot1.vue：

```html
<template>
  <div>
    <h2>插槽测试1</h2>
    <slot></slot>
  </div>
</template>

<script>
export default {
  data () {
    return {
      name1: 'hello slot1'
    }
  }
}
</script>
```

CSlot2.vue：

```html
<template>
  <div>
    <CSlot1>
      <h3>插槽测试2</h3>
      <p>hello slot</p>
      <p>{{ name1 }}</p>
    </CSlot1>
  </div>
</template>

<script>
import CSlot1 from './CSlot1'

export default {
  data () {
    return {
      name2: 'hello slot2'
    }
  },
  name: 'SClot2',
  components: {
    CSlot1
  }
}
</script>
```

其中，CSlot2是CSlot1的父组件。当不在子组件中使用<slot></slot>时，父组件中<CSlot1>中的内容就不会显示出来，只有使用了<slot></slot>才会显示出来。

所以，插槽是使用在子组件中，为了将父组件中子组件模板的数据正常显示。



在父组件的子组件模板中，是不能使用子组件中的数据的，即CSlot2中不能访问CSlot1中的name1，可以使用CSlot2中的name2。即父组件中的所有内容（不管是插槽还是什么），作用域只能是父组件。



可以为插槽设置默认内容。

CSlot1.vue：

```html
<template>
  <div>
    <h2>插槽测试1</h2>
    <slot>默认内容</slot>
  </div>
</template>

<script>
export default {
  data () {
    return {
      name1: 'hello slot1'
    }
  }
}
</script>
```

CSlot2.vue：

```html
<template>
  <div>
    <CSlot1>
    </CSlot1>
  </div>
</template>

<script>
import CSlot1 from './CSlot1'

export default {
  data () {
    return {
      name2: 'hello slot2'
    }
  },
  name: 'SClot2',
  components: {
    CSlot1
  }
}
</script>
```

当父组件中的子组件模板中没有内容时，插槽中的默认内容会被显示，反之则不会被显示。



## 具名插槽

具名插槽中含有一个 name 属性，例子如下：

CSlot1.vue：子组件

```html
<template>
  <div>
    <h2>插槽测试1</h2>
    <slot name="s1"></slot>
    <p>====================</p>
    <slot name="s2"></slot>
  </div>
</template>

<script>
export default {
  data () {
    return {
      name1: 'hello slot1'
    }
  }
}
</script>
```

CSlot2.vue：父组件

```html
<template>
  <div>
    <h2>插槽测试2</h2>
    <CSlot1>
      <div slot="s1">
        <p>武汉加油，湖北加油，全国加油！</p>
      </div>
      <div slot="s2">
        <p>wuhan courage</p>
      </div>
    </CSlot1>
  </div>
</template>

<script>
import CSlot1 from './CSlot1'

export default {
  data () {
    return {
      name2: 'hello slot2'
    }
  },
  name: 'SClot2',
  components: {
    CSlot1
  }
}
</script>
```

父组件中，通过使用slot="xxx"，给插槽指定名称，在子组件中使用<slot name="xxx"></slot>来接收对应插槽中的数据。

父组件还可以这样写：使用v-slot:xxx 指令

```html
<template>
  <div>
    <h2>插槽测试2</h2>
    <CSlot1 slot="cslot1">
      <template v-slot:s1>
        <p>武汉加油，湖北加油，全国加油！</p>
      </template>
      <template v-slot:s2>
        <p>wuhan courage</p>
      </template>
    </CSlot1>
  </div>
</template>

<script>
import CSlot1 from './CSlot1'

export default {
  data () {
    return {
      name2: 'hello slot2'
    }
  },
  name: 'SClot2',
  components: {
    CSlot1
  }
}
</script>
```

## 作用域插槽

前面说过，在父组件中不能使用子组件中的数据。在这里，作用域插槽中，父组件可以使用子组件中的数据。

CSlot1.vue：子组件，子组件的插槽中使用 :data 绑定数据，以便在父组件中接收数据

```html
<template>
  <div>
    <h2>插槽测试1</h2>
    <slot :data="name1"></slot>
  </div>
</template>

<script>
export default {
  data () {
    return {
      name1: 'hello slot1'
    }
  }
}
</script>
```

CSlot2.vue：在父组件中使用 slot-scope 接收子组件中的数据，然后就可以使用了

```html
<template>
  <div>
    <h2>插槽测试2</h2>
    <CSlot1>
      <div  ="info">
        {{ info.data }}
      </div>
    </CSlot1>
  </div>
</template>

<script>
import CSlot1 from './CSlot1'

export default {
  data () {
    return {
      name2: 'hello slot2'
    }
  },
  name: 'SClot2',
  components: {
    CSlot1
  }
}
</script>
```

