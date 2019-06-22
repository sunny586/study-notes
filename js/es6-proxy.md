```js
var handler = {
    get: function(target, name){
      return name in target ? target[name] : 'No prop!';
    },
    set: function(target, key, value) {
      target[key] = value;
   }
};

var p = new Proxy({}, handler);
p.a = 1;
p.b = 2;

console.log(p.a);    //1
console.log(p.b);    //2
console.log(p.c);    //No prop!
```

```js
let onWatch = (obj, setBind, getLogger) => {
  let handler = {
    get(target, property, receiver) {
      getLogger(target, property)
      return Reflect.get(target, property, receiver)
    },
    set(target, property, value, receiver) {
      setBind(value, property)
      return Reflect.set(target, property, value)
    }}
  return new Proxy(obj, handler)
}

let obj = { a: 1 }
let p = onWatch(
  obj,
  (v, property) => {
    console.log(`监听到属性${property}改变为${v}`)00
  },
  (target, property) => {
    console.log(`'${property}' = ${target[property]}`)
  }
)
p.a = 2 // 监听到属性a改变
p.a // 'a' = 2
```
