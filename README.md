## commonjs循环加载

> 当`main.js`加载`a.js`时，然后`a.js`反过来加载`b.js`。在这一点上，`b.js`试图加载`a.js`。为了防止无限循环，未完成`a.js`的exports对象的的副本被返回给`b.js`模块。`b.js`随后完成加载，其exports对象被提供给`a.js`模块。

1. 当`b.js`中继续加载`a.js`时，发现发生了循环加载，就去`a.js`模块中的exports对象上取值，由于此时`a.js`模块还没加载完毕，所以只有exports.done = false，这个对象被返回给`b.js`模块。`b.js`模块继续执行直到完成，然后`a.js`模块继续执行直到完成。
```javascript
console.log('b starting');
exports.done = false;
const a = require('./a.js');
console.log('in b, a.done = %j', a.done);
exports.done = true;
console.log('b done'); 
```
2. `main.js`中`const b = require('./b.js');`加载b模块时，由于b模块已经在`const a = require('./a.js');`加载a模块的时候加载过了，所以不会小重复加载，会直接取缓存。
```javascript
console.log('main starting');
const a = require('./a.js');
const b = require('./b.js');
console.log('in main, a.done = %j, b.done = %j', a.done, b.done); 
```

**注意** 发生循环加载时，由于模块可能未加载完成，只能返回部分的值，所以引用变量的时候要注意:
```javascript
const a = require('a.js') // 安全的写法
const foo = require('a.js').foo // 不安全的写法，以为模块a的返回对象可能会有变化，或者foo的值可能会变化。例如官方案例中的exports.done
```

## es6循环加载
执行`a.mjs`
```shell
b.mjs
file:///d:/learn/manual-api/module/cycles/b.mjs:3
console.log(foo);
            ^

ReferenceError: Cannot access 'foo' before initialization
    at file:///d:/learn/manual-api/module/cycles/b.mjs:3:13
    at ModuleJob.run (node:internal/modules/esm/module_job:194:25)

Node.js v18.16.0
```

#### es6如何处理循环加载
1. 执行`a.mjs`以后，引擎发现它加载了`b.mjs`，会优先执行`b.mjs`，然后再执行`a.mjs`。接着，执行`b.mjs`的时候，已知它从`a.mjs`输入了foo接口，这时不会去执行`a.mjs`，而是认为这个接口已经存在了，继续往下执行。执行到第三行`console.log(foo)`的时候，才发现这个接口根本没定义，因此报错。
2. 所以通过将`a.mjs`模块中的foo改为声明式函数可以解决报错，因为函数可以提升到引用`b.mjs`之前。