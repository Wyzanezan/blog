---
title: Vue中子组件与父组件
date: 2020-05-24 12:21:17
tags:
categories: Frontend
---

今天介绍下 Vue 中子组件与父组件之间如何传递数据。

<!--more-->

# 父组件向子组件传递数据

## 传递数据

在子组件中，使用props用来接收父组件中传递的数据。使用如下：

Parent.vue：通过:userInfo="userInfo"绑定需要传递的数据

```html
<template>
  <div>
    <h2>父组件</h2>
    <Child :userInfo="userInfo">
    </Child>
  </div>
</template>

<script>
import Child from './Child'

export default {
  data () {
    return {
      userInfo: {
        name: '小明',
        age: 16
      }
    }
  },
  components: {
    Child
  }
}
</script>
```

Child.vue：在子组件中，通过props接收传递进来的数据

```html
<template>
  <div>
    <h2>子组件</h2>
    {{ userInfo }}
  </div>
</template>

<script>
export default {
  props: ['userInfo']
}
</script>
```



## 传递方法

在子组件中使用父组件中的方法有三种方式实现。

### 使用 this.$parent

在子组件中使用 this.$parent 调用父组件中的方法。

Child.vue：

```html
<template>
  <div>
    <h2>子组件</h2>
    <button @click="getData">调用父组件中的方法</button>
  </div>
</template>

<script>
export default {
  methods: {
    getData () {
      var data = this.$parent.createData()
      alert(data.name)
    }
  }
}
</script>
```



### 使用 this.$emit

Child.vue：

```html
<template>
  <div>
    <h2>子组件</h2>
    <button @click="getData">调用父组件中的方法</button>
  </div>
</template>

<script>
export default {
  data () {
    return {
      info: 'child com'
    }
  },
  methods: {
    getData () {
      // 调用子组件中的方法
      var data = this.$emit('createData', this.info)
      console.log(data)
    }
  }
}
</script>
```

Parent.vue：

```html
<template>
  <div>
    <h2>父组件</h2>
    <!-- 把父组件中的方法传递给子组件 -->
    <Child @createData="createData">
    </Child>
  </div>
</template>

<script>
import Child from './Child'

export default {
  data () {
    return {
      userInfo: {
        name: '小明',
        age: 16
      }
    }
  },
  components: {
    Child
  },
  methods: {
    createData (info) {
      console.log(this.userInfo.name)
      alert(info)
      return this.userInfo.name
    }
  }
}
</script>
```



### 将方法传入子组件中

Child.vue：

```html
<template>
  <div>
    <h2>子组件</h2>
    <button @click="getData">调用父组件中的方法</button>
  </div>
</template>

<script>
export default {
  props: {
    // 定义待接收的方法
    createData: {
      type: Function,
      default: null
    }
  },
  methods: {
    getData () {
      var data = this.createData()
      alert(data.name)
    }
  }
}
</script>
```

Parent.vue：

```html
<template>
  <div>
    <h2>父组件</h2>
    <!-- 将方法传递给子组件 -->
    <Child :createData="createData">
    </Child>
  </div>
</template>

<script>
import Child from './Child'

export default {
  data () {
    return {
      userInfo: {
        name: '小明',
        age: 16
      }
    }
  },
  components: {
    Child
  },
  methods: {
    createData () {
      return this.userInfo
    }
  }
}
</script>
```

# 子组件向父组件传递数据

子组件中通过使用 this.$emit 向父组件传递数据，传递时指定一个 key，父组件中使用 @key 接收数据。

Parent.vue：

```html
<template>
  <div>
    <h2>父组件</h2>
    <!-- 接收子组件中传递进来的数据，
	@sendMsg要与子组件中this.$emit('sendMsg', this.info)对应 -->
    <Child @sendMsg="getData">
    </Child>
  </div>
</template>

<script>
import Child from './Child'

export default {
  data () {
    return {
      userInfo: {
        name: '小明',
        age: 16
      }
    }
  },
  components: {
    Child
  },
  methods: {
    getData (info) {
      // info是子组件中传递进来的数据
      alert(info)
    }
  }
}
</script>
```

Child.vue：

```html
<template>
  <div>
    <h2>子组件</h2>
    <button @click="sendData">发送数据</button>
  </div>
</template>

<script>
export default {
  data () {
    return {
      info: 'i am child'
    }
  },
  methods: {
    sendData () {
      // 向父组件中传递数据,sendMsg名称要与父组件中对应的名称一致
      this.$emit('sendMsg', this.info)
    }
  }
}
</script>
```

