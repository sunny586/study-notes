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

## 一个合格的中级前端工程师必须要掌握的 28 个 JavaScript 技巧

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
  return maooedArr
}

Array.prototype.selfMap = selfMap
[1, 2, 3].selfMap(number => number * 2)
```
