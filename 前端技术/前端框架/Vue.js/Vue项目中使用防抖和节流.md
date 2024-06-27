# Vue项目中使用防抖和节流

## 一、防抖（debounce）

> 触发高频事件后n秒内函数只会执行一次，如果在n秒内高频事件再次被触发，则重新计算时间。
>
> 简单的说也就是**一定时间段的无论点击多少次，只会执行最后一次的调用。**

在methods中直接实现

```js
debounce (func, delay = 1000, immediate = false) {
  //闭包
  let timer = null
  //不能用箭头函数
  return function () {
    if (timer) {
      clearTimeout(timer)
    }
    if (immediate && !timer) {
      func.apply(this,arguments)
    }
    timer = setTimeout(() => {
      func.apply(this,arguments)
    }, delay)
  }
},
```

### Demo

```java
<template>
  <el-input v-model="text" @input="debounceInput" />
</template>

<script>
  export default {
    mounted () {
      this.debounceInput = this.debounce(this.handle, 1000,false)
    },
    data () {
      return {
        text: '',
        debounceInput: () => {},
      }
    },

    methods: {
      handle (value) {
        console.log(value)
      },

      debounce (func, delay = 1000, immediate = false) {
        let timer = null
        //不能用箭头函数
        return function () {
          //在时间内重复调用的时候需要清空之前的定时任务
          if (timer) {
            clearTimeout(timer)
          }
          //适用于首次需要立刻执行的
          if (immediate && !timer) {
            func.apply(this,arguments)
          }
          timer = setTimeout(() => {
            func.apply(this,arguments)
          }, delay)
        }
      },
    }
  }
</script>
```

## 二、节流（throttle）

> 触发高频事件后，但在n秒内函数只会执行一次.
>
> 简单的说也就是**一定时间内的无论点击多少次，都只会执行第一次，其余的直接return**

### 基于时间戳实现

```js
export const throttle = (fn, delay = 2000) => {
  let lastTime = 0, timer = null

  return function () {
    let _this = this
    let _arguments = arguments
    let now = new Date().getTime()
    clearTimeout(timer)
    // 判断上次触发的时间和本次触发的时间差是否小于delay,创建一个timer
    if (now - lastTime < delay) {
      timer = setTimeout(function () {
        lastTime = now
        console.log("执行器触发")
        fn.apply(_this, _arguments)
      }, delay)
    } else {
      // 否则可以直接执行
      lastTime = now
      console.log("直接触发")
      fn.apply(_this, _arguments)
    }
  }
}
```

```js
<template>
  <el-input v-model="text" @input="throttleInput" />
</template>

<script>
  import {throttle} from '../utils'
  export default {
    name: 'throttle',
    data () {
      return {
        text: '',
      }
    },
    methods: {
      handle (value) {
        console.log(value)
      },

      throttleInput: throttle(function (...args) {
        this.handle(...args)
      })

    }

  }
</script>
```

## 参考

- [Vue项目中使用防抖和节流 - 掘金](https://juejin.cn/post/7006257717820162056)