## 函数柯里化

```js
/**
 * 将函数柯里化
 * @param fn    待柯里化的原函数
 * @param len   所需的参数个数，默认为原函数的形参个数
 */
function curry(fn, len = fn.length) {
  return _curry.call(this, fn, len)
}

/**
 * 中转函数
 * @param fn    待柯里化的原函数
 * @param len   所需的参数个数
 * @param args  已接收的参数列表
 */
function _curry(fn, len, ...args) {
  return function(...params) {
    let _args = [...args, ...params]
    if (_args.length >= len) {
      return fn.apply(this, _args)
    } else {
      return _curry.call(this, fn, len, ..._args)
    }
  }
}
```

使用占位符

```js
/**
 * @param  fn           待柯里化的函数
 * @param  length       需要的参数个数，默认为函数的形参个数
 * @param  holder       占位符，默认当前柯里化函数
 * @return {Function}   柯里化后的函数
 */
function curry(fn, length = fn.length, holder = curry) {
  return _curry.call(this, fn, length, holder, [], [])
}
/**
 * 中转函数
 * @param fn            柯里化的原函数
 * @param length        原函数需要的参数个数
 * @param holder        接收的占位符
 * @param args          已接收的参数列表
 * @param holders       已接收的占位符位置列表
 * @return {Function}   继续柯里化的函数 或 最终结果
 */
function _curry(fn, length, holder, args, holders) {
  return function(..._args) {
    //将参数复制一份，避免多次操作同一函数导致参数混乱
    let params = args.slice()
    //将占位符位置列表复制一份，新增加的占位符增加至此
    let _holders = holders.slice()
    //循环入参，追加参数 或 替换占位符
    _args.forEach((arg, i) => {
      //真实参数 之前存在占位符 将占位符替换为真实参数
      if (arg !== holder && holders.length) {
        let index = holders.shift()
        _holders.splice(_holders.indexOf(index), 1)
        params[index] = arg
      }
      //真实参数 之前不存在占位符 将参数追加到参数列表中
      else if (arg !== holder && !holders.length) {
        params.push(arg)
      }
      //传入的是占位符,之前不存在占位符 记录占位符的位置
      else if (arg === holder && !holders.length) {
        params.push(arg)
        _holders.push(params.length - 1)
      }
      //传入的是占位符,之前存在占位符 删除原占位符位置
      else if (arg === holder && holders.length) {
        holders.shift()
      }
    })
    // params 中前 length 条记录中不包含占位符，执行函数
    if (params.length >= length && params.slice(0, length).every(i => i !== holder)) {
      return fn.apply(this, params)
    } else {
      return _curry.call(this, fn, length, holder, params, _holders)
    }
  }
}
```

1.判断对象的数据类型

```js
const isType = type => target => `[object ${type}]` === Object.prototype.toString.call(target)
const isArray = isType('Array')
console.log(isArray([])) > true
```

2.循环实现数组 map 方法

```js
const selfMap = function(fn, context) {
  let arr = Array.prototype.slice.call(this)
  let mappedArr = Array(arr.length - 1)
  for (let i = 0; i < arr.length; i++) {
    if (!arr.hasOwnProperty(i)) continue
    mappedArr[i] = fn.call(context, arr[i], i, this)
  }
  return mappedArr
}

Array.prototype.selfMap = selfMap
;[1, 2, 3].selfMap(number => number * 2)
```

函数防抖就是，延迟一段时间再执行函数，如果这段时间内又触发了该函数，则延迟重新计算

```js
/**
 * @desc 函数防抖
 * @param func 函数
 * @param wait 延迟执行毫秒数
 * @param immediate true 表立即执行，false 表非立即执行
 */
function debounce(func, wait, immediate = false) {
  let timeout

  return function() {
    let context = this
    let args = arguments

    if (timeout) clearTimeout(timeout)
    if (immediate) {
      var callNow = !timeout
      timeout = setTimeout(() => {
        timeout = null
      }, wait)
      if (callNow) func.apply(context, args)
    } else {
      timeout = setTimeout(function() {
        func.apply(context, args)
      }, wait)
    }
  }
}

var fn = function() {
  console.log('boom')
}

setInterval(debounce(fn, 500), 1000) // 第一次在1500ms后触发，之后每1000ms触发一次

setInterval(debounce(fn, 2000), 1000) // 不会触发一次（我把函数防抖看出技能读条，如果读条没完成就用技能，便会失败而且重新读条）
```

节流：函数间隔一段时间后才能再触发，避免某些函数触发频率过高，比如滚动条滚动事件触发的函数

```js
// 简单实现
/**
 * @desc 函数节流
 * @param func 函数
 * @param wait 延迟执行毫秒数
 * @param type 1 表时间戳版，2 表定时器版
 */
function throttle(func, wait, type) {
  if (type === 1) {
    let previous = 0
  } else if (type === 2) {
    let timeout
  }
  return function() {
    let context = this
    let args = arguments
    if (type === 1) {
      let now = Date.now()

      if (now - previous > wait) {
        func.apply(context, args)
        previous = now
      }
    } else if (type === 2) {
      if (!timeout) {
        timeout = setTimeout(() => {
          timeout = null
          func.apply(context, args)
        }, wait)
      }
    }
  }
}
```
